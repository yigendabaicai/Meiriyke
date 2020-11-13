# Meiriyke
代码展现 | 如何实现macOS桌面端的录屏采集？

第一步，使用Capture Session管理数据流：
AVCaptureSession *screenCaptureSession = [[AVCaptureSession alloc] init];

第二步，使用preset配置分辨率：
 if ([self.screenCaptureSession canSetSessionPreset:AVCaptureSessionPresetHigh])
        [self.screenCaptureSession setSessionPreset:AVCaptureSessionPresetHigh];

第三步是配置Capture Inputs，添加到Session中。
AVCaptureScreenInput *captureScreenInput = [[AVCaptureScreenInput alloc] initWithDisplayID:CGMainDisplayID()];
    if ([self.screenCaptureSession canAddInput:captureScreenInput])
        [self.screenCaptureSession addInput:captureScreenInput];

第四步，使用Capture Outputs从Session中获取输出流：
AVCaptureVideoDataOutput *captureVideoOutput = [[AVCaptureVideoDataOutput alloc] init];
[captureVideoOutput setSampleBufferDelegate:self queue:dispatch_get_main_queue()];

第五步，使用CaptureConnections获取处理完成的流数据。
- (void)captureOutput:(AVCaptureOutput *)output didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection{}


首先我们需要获取所有的窗口列表
 CFArrayRef array =(CGWindowListCopyWindowInfo(kCGWindowListOptionAll | kCGWindowListExcludeDesktopElements | kCGWindowListOptionOnScreenOnly, kCGNullWindowID));
获取窗口后，我们可以根据获取到的窗口列表再获取到每个窗口的图像
 CGImageRef windowImage = CGWindowListCreateImage(CGRectNull, kCGWindowListOptionIncludingWindow, windowID, kCGWindowImageBoundsIgnoreFraming);
 
# 绘制鼠标：

 // 创建绘制图像的上下文
     CGContextRef context = CGBitmapContextCreate(imgData,
                                     width,
                                     height,
                                     8, // 8 bits per component
                                     bytesPerRow,
                               CGImageGetColorSpace(pSourceImage),
    // 将桌面窗口图像绘入上下文中
    CGContextDrawImage(context, bgBoundingBox, pSourceImage);
    // 将鼠标图像绘入上下文中
    CGContextDrawImage(context, CGRectMake(mouse_x , mouse_y ,mouse_w, mouse_h), [mouse CGImageForProposedRect:NULL context:NULL hints:NULL]);
    // 最后生成合入鼠标图像的桌面或者窗口图像
    CGImageRef pFinalImage = CGBitmapContextCreateImage(context);
    free(imgData);
    CGContextRelease(context);

# 过滤窗口


//列举出桌面所有窗口
CFArrayRef onScreenWindows = CGWindowListCreate(kCGWindowListOptionOnScreenOnly, kCGNullWindowID);
CFMutableArrayRef finalList = CFArrayCreateMutableCopy(NULL, 0, onScreenWindows);
//过滤移除指定的窗口
for (long i = CFArrayGetCount(finalList) - 1; i >= 0; i--) {
     CGWindowID window = (CGWindowID)(uintptr_t)CFArrayGetValueAtIndex(finalList, i);
     if ([self.excludeWindows containsObject:@(window)]) {
         CFArrayRemoveValueAtIndex(finalList, i);
      }
}
CGImageRef windowImage = CGWindowListCreateImageFromArray(item.deskBounds, finalList, kCGWindowImageDefault);
CFRelease(finalList);
CFRelease(onScreenWindows);

