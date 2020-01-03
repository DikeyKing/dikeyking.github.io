#### 1. 前言

​        在和 iOS 相机打交道的时候，我们经常会遇到 CVPixelBuffer 这个类型。它代表了 iOS 摄像头捕获的数据流中的一帧。CV 前缀表示它属于 CoreMedia 框架。在有一些时候，我们可能需要直接对数据流进行处理，包括裁剪、旋转、镜像、缩放、以及色彩空间转换。

​        一个典型的情况，需要每秒30帧处理的场景。比如需要实现人脸的跟踪和贴纸的SDK，需要计算人脸特征点，这时候可能需要将原始输入的 CVPixelBuffer 从 1280 * 720的图像做顺时针旋转，拿到 720 * 1280 的CVPixelBuffer。可能同时还需要裁剪人脸图像，还有缩放人脸图像用于深度学习等操作。在最后，将处理完成的 CVPixelBuffer 通过GPU 渲染到屏幕。



    CVPixelBuffer-->| 裁剪/旋转/缩放等操作 |OutCVPixelBuffer-->|GPU Render|View

#### 2. 框架

​        对 CVPixelBuffer 做旋转、裁剪、镜像、缩放、色彩空间转换等，我们主要有两个工具。 一个是底层级Accelerate 中的 **vImage** 。另外一个是 **Core Image**。 

​         **vImage**  是一个高性能的图像处理框架，它会尽量利用 CPU 上的向量处理器做运算。它包括卷积、几何变换格式长肉等等功能。vImage 特别适合大图像和实时视频流的处理。

​        Core Image 则是基于 Core Graphics、Image I/O 、Core Video 之上更高级的封装。最底层是 OpenGL/OpenGL ES （选择GPU渲染时），GCD（选择CPU渲染时）。

#### 3. vImage 实现

##### 3.1 vImage 裁剪

第一步，获取 CVPixelBufferRef 的色彩空间，默认 kCVPixelFormatType_32BGRA

```
OSType inputPixelFormat = CVPixelBufferGetPixelFormatType(sourcePixelBuffer);
```

第二步，锁住输入 CVPixelBufferRef

```
if (CVPixelBufferLockBaseAddress(sourcePixelBuffer, kCVPixelBufferLock_ReadOnly) != kCVReturnSuccess) {
    NSLog(@"Could not lock base address");
    return nil;
}
```

第三步，获取CVPixelBufferRef 数据（首地址）以及每行排列字节数和裁剪的偏移位置

```
void *sourceData = CVPixelBufferGetBaseAddress(sourcePixelBuffer);
size_t sourceBytesPerRow = CVPixelBufferGetBytesPerRow(sourcePixelBuffer);
size_t offset = CGRectGetMinY(croppingRect) * sourceBytesPerRow + CGRectGetMinX(croppingRect) * 4;

```

第四步，裁剪  CVPixelBufferRef

```
vImage_Buffer croppedvImageBuffer = {
    .data = ((char *)sourceData) + offset,
    .height = (vImagePixelCount)CGRectGetHeight(croppingRect),
    .width = (vImagePixelCount)CGRectGetWidth(croppingRect),
    .rowBytes = sourceBytesPerRow
};
```

第五步，获取裁剪后的 CVPixelBuffer

```
OSType pixelFormat = CVPixelBufferGetPixelFormatType(sourcePixelBuffer);
CVPixelBufferRef outputPixelBuffer = NULL;
CVPixelBufferCreateWithBytes(nil, croppingRect.size.width, croppingRect.size.height, pixelFormat, croppedvImageBuffer.data , croppedvImageBuffer.rowBytes, pixelBufferReleaseCallBack, nil, nil, &outputPixelBuffer);
```

如果再加上一些判空等，可以大致得到下面的代码：

```objective-c
CVPixelBufferRef vImageCropPixelBuffer(CVPixelBufferRef sourcePixelBuffer,
                                          CGRect croppingRect)
{
    OSType inputPixelFormat = CVPixelBufferGetPixelFormatType(sourcePixelBuffer);

    if (CVPixelBufferLockBaseAddress(sourcePixelBuffer, kCVPixelBufferLock_ReadOnly) != kCVReturnSuccess) {
        NSLog(@"Could not lock base address");
        return nil;
    }

    void *sourceData = CVPixelBufferGetBaseAddress(sourcePixelBuffer);
    if (sourceData == NULL) {
        NSLog(@"Error: could not get pixel buffer base address");
        CVPixelBufferUnlockBaseAddress(sourcePixelBuffer, kCVPixelBufferLock_ReadOnly);
        return nil;
    }
    
    size_t sourceBytesPerRow = CVPixelBufferGetBytesPerRow(sourcePixelBuffer);
    size_t offset = CGRectGetMinY(croppingRect) * sourceBytesPerRow + CGRectGetMinX(croppingRect) * 4;

    // Crop
    vImage_Buffer croppedvImageBuffer = {
        .data = ((char *)sourceData) + offset,
        .height = (vImagePixelCount)CGRectGetHeight(croppingRect),
        .width = (vImagePixelCount)CGRectGetWidth(croppingRect),
        .rowBytes = sourceBytesPerRow
    };

    /* The ARGB8888, ARGB16U, ARGB16S and ARGBFFFF functions work equally well on
     * other channel orderings of 4-channel images, such as RGBA or BGRA.*/
    CVPixelBufferUnlockBaseAddress(sourcePixelBuffer, kCVPixelBufferLock_ReadOnly);
    
    OSType pixelFormat = CVPixelBufferGetPixelFormatType(sourcePixelBuffer);
    CVPixelBufferRef outputPixelBuffer = NULL;
    CVReturn status = CVPixelBufferCreateWithBytes(nil, croppingRect.size.width, croppingRect.size.height, pixelFormat, croppedvImageBuffer.data , croppedvImageBuffer.rowBytes, pixelBufferReleaseCallBack, nil, nil, &outputPixelBuffer);

    if (status != kCVReturnSuccess) {
        NSLog(@"Error: could not create new pixel buffer");
        free(sourceData);
        return nil;
    }

    return outputPixelBuffer;
}

```

##### 3.2 vImage 缩放

如果需要在裁剪之后加上缩放操作，非常容易：

```
// scaledSize 是 CGSize 类型
size_t scaledBytesPerRow = scaledSize.width * 4;
void *scaledData = malloc(scaledSize.height * scaledBytesPerRow);
if (scaledData == NULL) {
    NSLog(@"Error: out of memory");
    return nil;
}

vImage_Buffer scaledvImageBuffer = {
    .data = scaledData,
    .height = (vImagePixelCount)scaledSize.height,
    .width = (vImagePixelCount)scaledSize.width,
    .rowBytes = scaledBytesPerRow
};
```

##### 3.3 vImage 旋转

旋转的核心也是类似的逻辑，先将 CVPixelBuffer 转换成 vImage_Buffer inbuff，然后声明 outbuff，用**vImageRotate90_ARGB8888** （主要色彩空间）进行旋转，核心代码如下

```
vImage_Buffer inbuff                = {srcBuff, height, width, bytesPerRow};
vImage_Buffer outbuff               = {dstBuff, outHeight, outWidth, bytesPerRowOut};
vImageRotate90_ARGB8888(&inbuff, &outbuff, rotationConstant, bgColor, 0);
```

##### 3.4 vImage 色彩空间变换

色彩空间变换需要了解一下色彩排列的概念，本文不赘述。步骤是用先用 vImage_Buffer 获取到通道，然后通过vImage 中 **Conversion.h** 中相应的 vImageConvert 方法去生成对应的 vImage_Buffer，然后转成对应色彩空间的 CVPixelBufferRef ，以 **YUV** 转 **BGRA** 举例：

```objective-c
- (CVPixelBufferRef) YUV2BGRA:(CVPixelBufferRef)imageBuffer
{
    CVPixelBufferLockBaseAddress(imageBuffer, 0);
    
    // 声明 kCVPixelFormatType_32BGRA CVPixelBufferRef
    int width = (int)CVPixelBufferGetWidthOfPlane(imageBuffer, 0);
    int height = (int)CVPixelBufferGetHeightOfPlane(imageBuffer, 0);
    if (destBuffer == nil) {
        CVPixelBufferCreate(kCFAllocatorDefault, width, height, kCVPixelFormatType_32BGRA, nil, &destBuffer);
    }
    CVPixelBufferLockBaseAddress(destBuffer, 0);
    
    // 通过vImage_Buffer 获取原始 CVPixelBufferRef 中的 Y 通道
    vImage_Buffer srcYp;
    srcYp.height = height;
    srcYp.width = width;
    srcYp.rowBytes =  CVPixelBufferGetBytesPerRowOfPlane(imageBuffer, 0);
    srcYp.data = CVPixelBufferGetBaseAddressOfPlane(imageBuffer, 0);
    
    // 通过vImage_Buffer 获取原始 CVPixelBufferRef 中的 UV 通道
    vImage_Buffer srcCbCr;
    srcCbCr.height = (int)CVPixelBufferGetHeightOfPlane(imageBuffer, 1);
    srcCbCr.width = (int)CVPixelBufferGetWidthOfPlane(imageBuffer, 1);
    srcCbCr.rowBytes =CVPixelBufferGetBytesPerRowOfPlane(imageBuffer, 1);
    srcCbCr.data = CVPixelBufferGetBaseAddressOfPlane(imageBuffer, 1);
    
    // 32BGRA 通过vImage_Buffer
    vImage_Buffer dest;
    dest.height = height;
    dest.width = width;
    dest.rowBytes = dest.width*4;
    dest.data = CVPixelBufferGetBaseAddress(destBuffer);
    
    vImage_Error error;
    
    // 核心转换方法 vImageConvert_420Yp8_CbCr8ToARGB8888
    //  BGRA - iOS 只支持 BGRA
    uint8_t permuteMap[4] = {3, 2, 1, 0};
    error = vImageConvert_420Yp8_CbCr8ToARGB8888(&srcYp, &srcCbCr, &dest, _conversionInfo, permuteMap, 255, 0);
    CVPixelBufferUnlockBaseAddress(imageBuffer, 0);
    
    if (error != kvImageNoError) {
        CVPixelBufferUnlockBaseAddress(destBuffer,0);
        return nil;
    }
    return destBuffer;
}
```

#### 4. Core Image 实现

Core Image 处理则非常简单，我们不需要去关心 vImage 中bitmap相关的部分，不过需要注意的一点是 [CIContext:render:toCVPixelBuffer]  需要iOS 9.3 之后的系统。

##### Core Image 中的处理

第一步，声明输入 CVPixelBuffer 的  CIImage

```
CIImage *image = [CIImage imageWithCVImageBuffer:pixelBuffer];
```

第二步，将CGSize 转成 Core Image 中的坐标（和UIKit 不同，左下角为原点）：

```
size_t originHeight = CVPixelBufferGetHeight(pixelBuffer);
CGRect realCropRect =  CGRectMake(cropRect.origin.x, originHeight -  cropRect.size.height - cropRect.origin.y , cropRect.size.width , cropRect.size.height);
```

裁剪：

```
image = [image imageByCroppingToRect:realCropRect];
```

缩放：

```
CGFloat scaleX = scaleSize.width / CGRectGetWidth(image.extent);
CGFloat scaleY = scaleSize.height / CGRectGetHeight(image.extent);
image = [image imageByApplyingTransform:CGAffineTransformMakeScale(scaleX, scaleY)];
```

旋转：

    image = [image imageByApplyingOrientation:orientation];
中间，我们还需要一步操作：

```
image = [image imageByApplyingTransform:CGAffineTransformMakeTranslation(-image.extent.origin.x, -image.extent.origin.y)];

[CIContext:render:toCVPixelBuffer]
```

为了使用  [CIContext:render:toCVPixelBuffer] ，我们需要进行一步

```
image = [image imageByApplyingTransform:CGAffineTransformMakeTranslation(-image.extent.origin.x, -image.extent.origin.y)]; 
```

最后：

```
CVPixelBufferRef output = NULL;
CVPixelBufferCreate(nil,
                    CGRectGetWidth(image.extent),
                    CGRectGetHeight(image.extent),
                    CVPixelBufferGetPixelFormatType(pixelBuffer),
                    nil,
                    &output);
if (output != NULL) {
    [context render:image toCVPixelBuffer:output]; // 注意 iOS 9.3 之后
}
```

####  对比

既然是不同的框架达到同样的目的，顺便来一发性能对比（注意，要用Release 模式才准确）：

* 相机模式： AVCaptureSessionPresetPhoto

* 测试机型： iPhone XR /  iOS 13.3 

需要注意的是，这里增加了Core Image context 中各种设置，这个设置和Core Image 底层调度相关：

* 是否强制使用GPU
* 是否始终在GPU中处理（没有GPU 和 CPU 进行内存交换，效率会更高）

对比结果如下：

| Type                                    | CPU Usage （%） | Memery usage （MB） | Time usage（ms） |
| --------------------------------------- | --------------- | ------------------- | :--------------: |
| Core Image / kEAGLRenderingAPIOpenGLES2 | 19              | 60                  |        10        |
| Core Image contextWithOptions           | 15              | 52.9                |        9         |
| Core Image CPU                          | 11              | 40                  |        7         |
| vImage                                  | 45              | 51                  |        11        |

CIContext 的声明方式分别是：

```
eaglctx = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];   
context = [CIContext contextWithEAGLContext:eaglctx]; // 记得复用 CIContext
```

以及：

```
// context = [CIContext contextWithOptions: nil];  
context = [CIContext contextWithOptions: [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:YES] forKey:kCIContextUseSoftwareRenderer]];
```

可见 Core Image 比起 vImage 还是快了一些，API 也更简单，所以如果版本允许（高版本API），更推荐使用Core Image 进行CVPixelBuffer的各种处理。

不过，Core Image 要考虑另外一些因素，比如GPU和CPU上的图片分辨率限制，内存交换等等，需要读者自行去阅读文档和探索。

例如：CPU 上可处理的最高分辨率是 16384 * 16384，GPU上最大的分辨率是 4096 * 4096

#### 参考

1. [vImage](https://developer.apple.com/documentation/accelerate/vimage)
2. [Core Image 你需要了解的那些事~](https://colin1994.github.io/2016/10/21/Core-Image-OverView/#3-_CPU_/_GPU)
3. [Core Image 介绍](https://objccn.io/issue-21-6/)
4. [参考代码](https://github.com/DikeyKing/DKCamera/blob/master/DKCamera/DKImageConverter/DKImageConverter.mm/)
5. [Core Image Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_performance/ci_performance.html#//apple_ref/doc/uid/TP30001185-CH10-SW1)
