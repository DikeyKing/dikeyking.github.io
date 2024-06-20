# visionOS 2

# 前言

visionOS 2.0 比起 visionOS 1.0，完善了更多的基础控件。同时，针对图像的隐私问题，苹果提供了企业级 API，这些 API 可以利用 Apple Vision Pro 的传感器获取更多源数据，做到之前做不到的很多事情；主要分为两类

1. 传感器相关：提供了相机原始 `CVPixelBuffer` 、二维码扫描、透视录屏
2. 平台控制：Apple Neural Engine 访问权限、3D 物体跟踪参数调整、性能控制

# **企业级API**

官申请方文档：[https://developer.apple.com/documentation/visionOS/building-spatial-experiences-for-business-apps-with-enterprise-apis](https://developer.apple.com/documentation/visionOS/building-spatial-experiences-for-business-apps-with-enterprise-apis)  

> If you’re interested in using the Enterprise APIs for visionOS entitlements in your app, the Account Holder of your Apple Developer Program or Apple Developer Enterprise Program can submit a Development Only request. When submitting a new request, you need to answer some questions about your app, and agree to the entitlement’s terms and conditions.
> 

**流程**

1. **账号要求**：苹果官方文档提示 **Apple Developer Program** 或 **Apple Developer Enterprise Program** 都可以参与申请（之前媒体报道需要企业计划，后笔者找苹果确认，**Apple Developer Program** 也可以）
2. 提交 **Development Only** 申请：[https://developer.apple.com/go/?id=69613ca716fe11ef8ec848df370857f4](https://developer.apple.com/go/?id=69613ca716fe11ef8ec848df370857f4)（目前打不开，提示未授权，不确定是否已准备好）
3. 使用企业级 API 完成开发
4. 分发到 **Apple Business Manager** 或 **Apple School Manager**

# 获取更多传感器数据

## [重要]主相机数据

- 首先，我们需要申请相机权限 `com.apple.developer.arkit.main-camera-access.allow` （后文省略）
- 利用企业级API，可以获取主相机 RGB 数据了，可以像 iOS 上一样利用这些数据
    
    ```swift
    
    let formats = CameraVideoFormat.supportedVideoFormats(for: .main, cameraPositions:[.left])
    let cameraFrameProvider = CameraFrameProvider()
    var arKitSession = ARKitSession()
    var pixelBuffer: CVPixelBuffer?
    
    // 1 权限检查
    await arKitSession.queryAuthorization(for: [.cameraAccess])
    
    // 2 使用 cameraFrameProvider 获取数据
    do {
        try await arKitSession.run([cameraFrameProvider])
    } catch {
        return
    }
    
    // 3 获取数据 
    guard let cameraFrameUpdates = 
        cameraFrameProvider.cameraFrameUpdates(for: formats[0]) else {
        return
    }
    
    for await cameraFrame in cameraFrameUpdates {
        guard let mainCameraSample = cameraFrame.sample(for: .left) else {
            continue
        }
        // 4 获取到图像源数据 CVPixelBuffer
        self.pixelBuffer = mainCameraSample.pixelBuffer
    }
    ```
    

## 二维码扫描

- `com.apple.developer.arkit.barcode-detection.allow`
- 可以使用ARKit来检测、定位和解码`条形码`和`二维码` ，可能可以解锁一些工业或商业场景

## 透视模式录屏

- `com.apple.developer.screen-capture.include-passthrough`
- 利用企业级API ，可以使用屏幕录制同时录制虚拟和现实了；之前无法获取真实世界画面；以下是demo中，讲录制的画面实时共享

![img_99860d8aad315a94f2a2425cbfa75204.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_99860d8aad315a94f2a2425cbfa75204.jpg)

# 平台控制

### Apple Neural Engine

- `com.apple.developer.coreml.neural-engine-access`
- 现在可以访问 Apple Neural Engine ，然后用在自定义ML模型中，以下是调度的示例
    
    ```swift
    // Apple Neural Engine access example
    
    let availableComputeDevices = MLModel.availableComputeDevices
    
    for computeDevice in availableComputeDevices {
        switch computeDevice {
            case .cpu: setCpuEnabledForML(true) // Example method name
            case .gpu: setGpuEnabledForML(true) // Example method name
            case .neuralEngine: runMyMLModelWithNeuralEngineAvailable() // Example method name
            default: continue
        }
    }
    ```
    
- 原先：不能直接访问 Apple Neural Engine

![img_d9c8cb8ca02de69decd83e049f5eecb9.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_d9c8cb8ca02de69decd83e049f5eecb9.jpg)

- 现在：可以利用 Apple Neural Engine 完成加速
    
    https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/
    
    ![img_9948c113972de8c8ad4ff360e3695595.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_9948c113972de8c8ad4ff360e3695595.jpg)
    

### 物体跟踪参数设置

- `com.apple.developer.arkit.object-tracking-parameter-adjustment.allow`
- 现在可以通过参数设置来优化已知物体的检测和跟踪
- 增强的对象跟踪能力可以加快物体检测，以及调整应用性能
    
    ```swift
    // Object tracking enhancements example
    
    var trackingParameters = ObjectTrackingProvider.TrackingConfiguration()
    
    // 将可跟踪的最大物体数量从10增加到15
    trackingParameters.maximumTrackableInstances = 15
    
    // 其他参数保留为默认值
    trackingParameters.maximumInstancesPerReferenceObject = 1
    
    // 检测速率
    trackingParameters.detectionRate = 2.0
    
    // 静止物体跟踪速率
    trackingParameters.stationaryObjectTrackingRate = 5.0
    
    // 移动物体跟踪速率
    trackingParameters.movingObjectTrackingRate = 5.0
    
    let objectTracking = ObjectTrackingProvider(
            referenceObjects: Array(referenceObjectDictionary.values),
            trackingConfiguration: trackingParameters)
    
    var arkitSession = ARKitSession()
    arkitSession.run([objectTracking])
    ```
    

### Apple Vision 性能控制

- `com.apple.developer.app-compute-category`
- 可以使用 CPU 和 GPU 的官方超频模式（拉高功率）来提高Vision Pro 性能，来满足高计算需求
- 显然，缺点是会增加发热、加快电池消耗
- 比如：我们需要展示一辆赛车内部工作原理的时候，因为对算力要求高，这时候，我们就可以 Turbo 模式走起
    
    ![img_a3888a7594c2204ec8c6b5ffb1589343.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_a3888a7594c2204ec8c6b5ffb1589343.jpg)
    

# **3D 物体跟踪**

## 3D 跟踪能力

在 VisionOS 2.0中，ARKit 带来更强大的物体跟踪功能。

- 首先：如图所示，ARKit 现在可以获取物体的`位置`、`朝向` 等信息，这些数据用坐标系和边界来表示
    
    ![Untitled](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/Untitled.png)
    
- 其次：ARKit 这次还给3D物体提供了标签功能
    
    ![img_a205e93e0d49b250117e49a8b0b71329.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_a205e93e0d49b250117e49a8b0b71329.jpg)
    
- 物体跟踪 + AR 效果，当然还有物体遮挡之类，ARKit 也帮我们做掉了。
    
    ![img_b3a6d28b3d87008aa6620f55a7611722.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_b3a6d28b3d87008aa6620f55a7611722.jpg)
    
- 要完成整个流程，过程分三步
    
    ![Untitled](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/Untitled%201.png)
    

## 实现步骤一：制作 3D 模型

- USDZ 格式
- 画质需要尽量高
- 可以用 LiDAR iPhone 等等拍摄
- 对物体的要求：静止物体、具有可识别的纹理和刚体、非对称

具体这里不赘述，可以参考 [https://developer.apple.com/augmented-reality/object-capture/](https://developer.apple.com/augmented-reality/object-capture/)

## 实现步骤二：使用 CoreML 完成训练

使用 `CoreML` 创建 `ReferenceObject` 

- 新增Spatial栏目中找到：`Object Tracking`
    
    ![Untitled](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/Untitled%202.png)
    
- CoreML 过程
    
    ![img_6fbd7621567c36b3559e73469b3002b5.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_6fbd7621567c36b3559e73469b3002b5.jpg)
    
- 训练配置
    
    ![img_adc0d7fee64bb130f00923f292268a22.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_adc0d7fee64bb130f00923f292268a22.jpg)
    
    - 配置中，我们可以看到 `All Angels`、`Upright`、`Front` 三个选项，设置这些参数，可以让 `CoreML` 更好地训练物体；显然，地球仪 `All Angels` 最合适，但是前文图中的显微镜，我们不太会翻转过来看，所以选 `Upright` 是合适的，然后图中的示波器，我们只看正面，那么  `Front` 是最合适和训练选项
    - 多个物体：我们可以同时添加多个物体同时训练，比如，地球周围挂一个卫星
    - `CoreML` 中还能显示物体在真实世界的尺寸，我们需要在训练前，调整好尺寸
    - 整个训练过程，可能需要几个小时，训练必须使用 `Apple Silicon` 机器，Intel Mac Pro 用户流泪？

## 实现步骤三：锚定虚拟物体

我们可以使用以下工具来锚定虚拟物体

- `Reality Composer Pro`或`RealityKit` 和 `ARKit` ：前者类似 Unity 中的可视化编辑器，后者则使用代码实现
    
    ![img_122892402350622b7fa9bd0c6faf9bba.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/img_122892402350622b7fa9bd0c6faf9bba.jpg)
    
- 使用 `Reality Composer Pro`
    - 可放置 USDZ 模型用于定位
        
        ![Untitled](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/Untitled%203.png)
        
    - 可以放置轨道，ARKit 和 RealityKit 会自动帮我们计算好遮挡之类的关系
        
        ![Untitled](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/visionOS%202%204cb646adef954d5699b160ba52425583/Untitled%204.png)
        
- 实现辅助页面
    - 显示 `USDZ`
        
        ```swift
        // 显示 USDZ 物体
        struct ImmersiveView: View {
           @State var globeAnchor: Entity? = nil
            var body: some View {
                RealityView { content in
                    // 使用 ARKit 加载 .referenceobject
                    let refObjURL = 
                    Bundle.main.url(forResource: "globe", withExtension: ".referenceobject")
                    let refObject = try? await ReferenceObject(from: refObjURL!)
                    
                    // 从 .referenceobject 中提取 USDZ 路径，加载 Entity 模型
                    let globePreviewEntity = 
                    try? await Entity.init(contentsOf: (refObject?.usdzFile)!)
        
                    // 设置透明度
                    globePreviewEntity!.components.set(OpacityComponent(opacity: 0.5))
                    // 添加到场景
                    content.add(globePreviewEntity!)
                }
            }
        }
        ```
        
    - 检查锚点状态
        
        ```swift
        // 检查锚点状态
        struct ImmersiveView: View {
           @State var globeAnchor: Entity? = nil
            var body: some View {
                RealityView { content in
                    if let scene = try? await Entity(named: "Immersive", in: realityKitContentBundle) {
                        globeAnchor = scene.findEntity(named: "GlobeAnchor")
                        content.add(scene)
                    }
                    let updateSub = content.subscribe(to: SceneEvents.Update.self) {event in
                        if let anchor = globeAnchor, anchor.isAnchored {
                            // 找到对象锚点，触发过渡动画
                        } else {
                            // 未找到对象锚点，显示引导页面
                        }
                    }
                }
            }
        }
        ```
        
    - 获取锚点 `transform`
        
        ```swift
        // 转换空间
        struct ImmersiveView: View {
           @State var globeAnchor: Entity? = nil
            var body: some View {
                RealityView { content in
                    // 为对象和世界锚点设置锚点转换空间
                    let trackingSession = SpatialTrackingSession()
                    // 空间跟踪配置，跟踪对象和世界
                    let config = SpatialTrackingSession.Configuration(tracking: [.object, .world])
                    // 如果成功运行会话，则检查结果中是否包含对象锚点
                    if let result = await trackingSession.run(config) {
                        if result.anchor.contains(.object) {
                            // 如果不包含对象锚点，说明未授权跟踪，提示未授权…… 
                        }
                    }
                    // 获取跟踪物体的世界坐标
                    let objectTransform = globeAnchor?.transformMatrix(relativeTo: nil)
                    
                    // 动画实现等等
                }
            }
        }
        
        ```

# **小结**

在 visionOS 1.0 版本中，图像数据、3D物体跟踪之类都做不到，现在 visionOS 2 都已经可以尝试。

伴随诸多新特性，visionOS 上应该有更多有价值的场景可以挖掘。

# **参考**

- [Introducing Enterprise APIs for visionOS](https://developer.apple.com/videos/play/wwdc2024/10139/)
- [Explore object tracking for visionOS](https://developer.apple.com/videos/play/wwdc2024/10101/)
- [Main Camera Access](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_arkit_main-camera-access_allow)
- [https://developer.apple.com/documentation/visionOS/building-spatial-experiences-for-business-apps-with-enterprise-apis](https://developer.apple.com/documentation/visionOS/building-spatial-experiences-for-business-apps-with-enterprise-apis)
- [https://developer.apple.com/documentation/arkit/cameraframe/sample/4443454-pixelbuffer](https://developer.apple.com/documentation/arkit/cameraframe/sample/4443454-pixelbuffer)
- [https://news.nweon.com/121701](https://news.nweon.com/121701)