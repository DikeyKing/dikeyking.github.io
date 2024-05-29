# Apple Vision Pro

![img_v2_f6407185-21f6-4f9c-9811-24e834b0aa9g.jpg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/img_v2_f6407185-21f6-4f9c-9811-24e834b0aa9g.jpg)

# 基础信息

## 硬件

- 实时三维建图
- 芯片：M2（功耗最高20w）+R1（功耗未知）
    
    ![1280X1280.JPEG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/1280X1280.jpeg)
    
- 传感器：12摄像头 + 5传感器 + 6麦克风
- 响应时间：12毫秒
- 分辨率：单眼像素 > 4K+(像素)，*整机最昂贵的硬件（业界估计单片700💲），90HZ刷新。*
- 续航：2h
- 电池：10000mAh
    - 电流 = 10000mAh / 2h = 5000mA
    - 电压：假定5V
    - 平均功率：电压 × 电流 = 5V × 5000mA = 25W
    - 实际
        - 电池参数：3166 mAh
        - 35.9 Wh，输出支持 13V、6A ；
        - 续航三小时。 35.9 Wh / 3 h = 11.97 W
- 主摄像头：***每秒传输十亿+像素***
    
    ![1280X1280 (3).PNG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/1280X1280_(3).png)
    
- 辅助摄像头
    
    ![1280X1280 (1).JPEG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/1280X1280_(1).jpeg)
    
    - ***Downward cameras（向下摄像头）：提供地面环境信息***
    - ***IR illuminators（红外照明器）：增强低光环境图像质量***
    - ***Side cameras（侧面摄像头）：提供周边环境信息***
- 深度头：  ***实时三维建图***
    
    ![1280X1280 (2).JPEG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/1280X1280_(2).jpeg)
    

## 软件

- 系统：VisionOS**（兼容iOS和iPadOS的应用）**
    
    ![Untitled.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/Untitled.png)
    
- 开发工具
    - Xcode
    - Apple frameworks：`RealityKit` +`ARKit` ..
    - 语言：SwiftUI/Swift/Objective-C/C/C++/C#
    - Reality Composer Pro
    - Unity
- AppStore for `visionOS`
    
    ![Appstore.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/Appstore.png)
    
    ***AppStore for visionOS***
    
    ![已有app兼容.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E5%25B7%25B2%25E6%259C%2589app%25E5%2585%25BC%25E5%25AE%25B9.png)
    
    ***可将已有的iOS app展现在xrOS商店***
    

# visionOS

## 架构

![架构.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E6%259E%25B6%25E6%259E%2584.png)

***visionOS架构***

### 上层

- `ARKit`：跟踪等多种技术
- `RealityKit`：高阶3D API，用于渲染、模型展示等
- `SwiftUI` and `UIKit`：`visionOS` app 的基础界面和系统组件等基础交互

### 底层

- Algorithms：图像、LiDAR、传感器处理等多种算法
- `CoreOS` and `Services`：类似iOS、macOS的底层服务
- `Metal`：底层渲染框架，上层框架的图形重要基础

## 空间概念

![空间概念.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E7%25A9%25BA%25E9%2597%25B4%25E6%25A6%2582%25E5%25BF%25B5.png)

### Window

- 可以打开多个 `Window`
- SwiftUI 编写，包含系统默认的 `views` and `controls` ：默认交互由这些控件承载

### Volume

- 立方体
- 可以在空间中移动
- 用使用 `RealityKit` or Unity 显示3D 内容

### Full Space

- 完全空间不同于默认的共享空间（`Shared Space`），共享空间元素默认并排，完全空间任由开发者使用
- 对 `visionOS`的app来说，这是 app的独立空间（和`Shared Space`平行、不共享）

## Apple frameworks

- SwiftUI、UIKit：负责基础界面、可重用控件以及基础交互逻辑
- RealityKit：3D 内容、动画和视觉效果，兼容MaterialX
- ARKit：平面估计、场景重建、图像定位、世界跟踪、手部骨骼跟踪
- Accessibility：支持手指、手腕或头部、眼部、语音等多种交互方式
- Metal：底层渲染核心

*（框架每个部分都很重要且有大量知识内容，此处不赘述）*

### ARKit

- 平面估计（Plane estimation）：可以检测相机视角下的平面，并将其用作虚拟对象的支撑面，使虚拟对象看起来像与现实世界物体交互一样自然
- 场景重建（Scene reconstruction）：ARKit可以通过多个相机视角对现实场景进行捕捉和分析，从而创建出三维模型，以供虚拟对象精确地与现实场景进行交互
- 场景理解（Scene understanding）：平面和物体检测以及标记
    
    [Scene understand.mp4](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/Scene_understand.mp4)
    

***Plane estimation + 场景理解***

- 图像定位（Image anchoring）：ARKit可以将虚拟对象锚定到现实世界中的特定图像，使得当用户将相机对准该图像时，虚拟对象会出现在该图像上
- 世界跟踪（World tracking）：ARKit可以跟踪设备在现实世界中的位置和方向，以便在设备移动时，虚拟对象能够随之移动并保持稳定
- [NEW] 手跟踪（Hand tracking）：可以检测手势变化并做出响
    
    ![b1c31573-ce55-468c-83cd-821bcab0186e.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/b1c31573-ce55-468c-83cd-821bcab0186e.png)
    
    ***Hand tracking 实现虚拟的手***
    
    ![交互.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E4%25BA%25A4%25E4%25BA%2592.png)
    
    ***来源（建议所有设计师观看Design for visionOS章节）：wwdc2023-10073_Design for spatial input***
    
    Todo：keyboard
    
    ***虚拟输入法***
    

### Metal

- `CompositorLayer`：用于输入数据的合成，并最终显示到屏幕上
- Runloop：类似 `do while` 的循环，用于对渲染生命周期的理解
- Update流程：处理渲染数据、自定义行为的关键时机
    
    ![CompositorLayer.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/CompositorLayer.png)
    
    ***CompositorLayer 渲染***
    
    ![帧渲染流程.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E5%25B8%25A7%25E6%25B8%25B2%25E6%259F%2593%25E6%25B5%2581%25E7%25A8%258B.png)
    
    ***帧渲染流程***
    
    ![CompositorLayer.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/帧渲染流程.png)
    
    ***Update流程***
    
    ![整体渲染流程.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E6%2595%25B4%25E4%25BD%2593%25E6%25B8%25B2%25E6%259F%2593%25E6%25B5%2581%25E7%25A8%258B.png)
    
    ***整体渲染流程***
    
    ![Runloop.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/Runloop.png)
    
    ***Runloop：while 循环***
    

### 渲染概念

- 渲染链路
    
    ```idris
                    
                    +-----------------------+
                    |                       |
                +------------+        +------------+
                |   Scene    |        |     UI     |
                | Renderer   |        |  System    |
                +------------+        +------------+
                    |                       |
                    |                       |
                    v                       v
            +--------------+         +-------------+
            |              |         |             |
            |              |         |             |
            |   Compositor |         |  Display    |
            |              |         |  Manager    |
            |              |         |             |
            +--------------+         +-------------+
                    |                       |
                    |                       |
                    v                       v
            +--------------+         +-------------+
            |              |         |             |
            |              |         |             |
            |   Renderer   |         |   Monitor   |
            |              |         |             |
            |              |         |             |
            +--------------+         +-------------+
    ```
    
- compositor
    - 负责将各个图层（包括场景、UI、视频等）合成为最终图像的组件
    - 负责将每个图层的像素按照特定的顺序、混合模式、透明度等属性进行混合，最终生成一张完整的图像，并将其显示在屏幕上
    - 上游：合成的各个图层的组件，比如场景渲染器、UI 组件
    - 下游：将合成后的图像显示在屏幕上的组件，比如图像显示器、窗口管理器
- 窗口管理器：
    - 上游：UI系统
    - 下游：显示器
- 分成两路以提高重用性和灵活性

# 工具链

## Xcode

- Swift、SwiftUI
- visionOS SDK
- visionOS Simulator
    
    ![工具链-模拟器.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E5%25B7%25A5%25E5%2585%25B7%25E9%2593%25BE-%25E6%25A8%25A1%25E6%258B%259F%25E5%2599%25A8.png)
    
    Todo：Reality Composer Pro
    
    ***模拟器演示：SwiftUI（基础交互界面）、多窗口（设置+播放器）***
    

## Reality Composer Pro

![Reality Composer Pro.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/Reality_Composer_Pro.png)

This content is only supported in a Feishu Docs

***通过拖拽完成模型摆放***

- 用于预览和准备3D内容
- 强大的编辑工具：支持3D模型、材质和声音
    
    ![编辑工具.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E7%25BC%2596%25E8%25BE%2591%25E5%25B7%25A5%25E5%2585%25B7.png)
    
    ***仅仅通过拖拽，完成模型或场景***
    
- 与Xcode紧密结合
- 可使用`Object Capture`将真实世界的物体转变成`USDZ`并放入`Reality Composer Pro`
- USDZ 文件，可以直接在xrOS打开，多个工业级软件支持：`Maya`、`Blender`、
    
    ![usdz.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/usdz.png)
    
    ***USDZ文件预览***
    

## Unity

支持 Unity 的3D渲染**（‼️ 需Unity 2022或更高版本）**

- 输入支持
    - look & tap：视线注视、点击
    - head pose：头部的倾斜和旋转等操作
    - hands：手部的动作和手势
        
        ![手势.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/%25E6%2589%258B%25E5%258A%25BF.png)
        
        ***多种技术结合的demo展示：Plane estimation、Hand tracking、tapping***
        
    - 增强现实数据（AR Foundation Package）
    - 蓝牙设备：键盘、手柄等
- UI
    - Unity UI 系统：UGUI、UI Toolkit
    - 其它UI：Mesh、renderer 、 RenderTexture
- 其它
    - 支持即时预览（模拟器也可以）
- 已支持前期迁徙准备
    - **[重要]** Unity 2022 或以上版本
    - shaders准备：转换成Shader Graph
    - 采用通用渲染管线

## Web

- Safari
    
    ![safari.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/safari.png)
    

***新的<Model>可以包含3D模型USDZ（Beta）***

![usdz.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/usdz%201.png)

***Combine* QuickLock *with Safari***

- 和`WebGL`保持兼容
    
    ![web3d.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Apple%20Vision%20Pro%2036f831d7d5b942d68bc14bdb27e9811d/web3d.png)
    

***和主流Web 3D引擎保持良好兼容性***

## 官方支持

### Compatibility evaluations

- 兼容性：可以向苹果请求线上app和`visionOS`的兼容性评估并获取报告

### Developer labs

- 线下体验： Apple Vision Pro 上运行的 visionOS、iPadOS 、 iOS app
- 官方优化和支持：在 Apple 的直接支持下测试和优化app，为 Apple Vision Pro 上市做好准备
- Apple 在全球六个地方设置了 Apple Vision Pro Developer Lab
    - 美国库比蒂诺
    - 英国伦敦
    - 德国慕尼黑
    - **中国上海**
    - 日本东京
    - 新加坡

### Developer kit

- 方式一：顶级头部公司有有机会直接拿到 Apple Vision Pro 的 Developer kit 进行开发
- 方式二：开发者可以通过报名的方式申请使用相关设备，开放报名的时间预计最快会在 6 月底放
- 方式三：开发者 Developer Tool Kit 设备（类似Mac Mini M1，开发者付费购买，预计 7 月中旬放出）

# 时间节点

- 【官方】2023年6月：visionOS SDK
- 【官方】2023年7月：Compatibility evaluations/Developer labs/Developer kit
- 【官方】2023年7月25日：可申请Developer kit、可申请Developer labs
- 【官方】Developer labs，2023年8月15日-8月25日：中国上海市浦东新区源深路385号
- 【官方】2024年年初：美国上市
- 【官方】2024年晚些时候：预计中国上市
