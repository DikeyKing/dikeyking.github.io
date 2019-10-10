---
title: Objective-C-Swift混编Framework
date: 2019-04-23 
categories: 随笔
tags: [X]

---


#### 动机：

所在项目封装的framework基本都是OC写的，希望在旧的项目中逐步用新的技术迭代。但是Swift和OC混编有两个比较重要的问题。

1. Swift ABI不稳定
2. 会带来APP体积的明星增加，iOS 8 的时候是夸张的 **8M** 

而这次的xcode 10.2.1 的更新带来了一些变换：

因为Swift 5.0 之前的ABI (*application binary interface*)并不稳定，所以导致Swift的runtime没有包含在iOS系统之中，而每次APP打包都会带上Swift的runtime，这样就会造成带Swift的APP体积增长非常明显。这个问题随着xcode 10.2.1 和 iOS 10.2的发布解决了：

看Xcode  10.2.2 版本是怎么说的：

```
Swift apps no longer include dynamically linked libraries for the Swift standard library and Swift SDK overlays in build variants for devices running iOS 12.2, watchOS 5.2, and tvOS 12.2. As a result, Swift apps can be smaller when they’re shipped in the App Store, deployed for testing using TestFlight, or thinned in an app archive for local development distribution.
```

重点：苹果表示Swift 不再动态链接Swift标准库，这些标准库内置在 iOS 12.2 系统的各种设备中，这样就可以减小APP Store 中可以下载的APP 体积。

所以想尝试通过OC和Swift混编，看看体积的影响。



#### 过程：

1. [将OC导入Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift>)：framework中不能创建 Objective-C Generated Interface Header，如果framework是静态库，可以直接调用OC的类，如果是动态库，目前会报找不到文件错误，正在研究如何解决
2. [在Objective-C中使用Swift](  https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_swift_into_objective-c) ： 非常简单，在使用OC的代码中导入文件 `#import <ProductName/ProductModuleName-Swift.h>`；在

3. 写完的framework提供给纯OC的项目调用，记得把Swift标准库带上

```
dyld: Library not loaded: @rpath/libSwiftCore.dylib
  Referenced from: /private/var/containers/Bundle/Application/6A98320D-9E8F-469C-91BD-EB2AB8D4AEA0/DynamicDemo.app/Frameworks/InsightSDK_dylib.framework/InsightSDK_dylib
  Reason: image not found
```

需要在build Setting中设置 

```
Build Settings > Always Embed Swift Standard Libraries
```

4. 这里有一个问题，根据 <https://developer.apple.com/library/archive/qa/qa1881/_index.html> 的说法：

```
Swift standard libraries are copied into a bundle if and only if you are building an application and this application contains Swift source files by itself.

If you are building an app that does not use Swift but embeds content such as a framework that does, Xcode will not include these libraries in your app. As a result, your app will crash upon launching with an error message looking as follows:

dyld: Library not loaded: @rpath/libswiftCoreGraphics.dylib
  Referenced from: /private/var/mobile/Containers/Bundle/Application/696F0EAD-E2A6-4C83-876F-07E3D015D167/<Your_App>.app/Frameworks/<Framework_Name>.framework/<Framework_Name>
  Reason: image not found
  
Setting Embedded Content Contains Swift Code to YES
```

按照我的理解，实际上还是把Swift一些标准的动态库库给包含进去了……



#### 体积影响：

framework with Swift 对体积的影响，

关闭bitcode，加入Swift版本，关闭bitcode：

| Device Type         | Download Size | Install Size |
| :------------------ | :------------ | :----------- |
| Universal           | 17.8 MB       | 31 MB        |
| iPad Pro (9.7-inch) | 13.5 MB       | 21.9 MB      |
| iPhone XS           | 13.5 MB       | 21.9 MB      |

关闭bitcode，关闭Swift，关闭bitcode：

| Device Type         | Download Size | Install Size |
| :------------------ | :------------ | :----------- |
| Universal           | 12.8 MB       | 17.5 MB      |
| iPad Pro (9.7-inch) | 11.2 MB       | 14.8 MB      |
| iPhone XS           | 11.2 MB       | 14.8 MB      |

可以看到：如果第三方APP是纯OC的项目，引入Swift会导致下载体积增加 **2.3M**，安装体积增加**7.1M**。



#### 结论：

如果是封装framework给自己用，我觉得增加的2.3M体积已经完全可以忽略不计，可以迁徙到新的技术上去，用Swift提高效率，编写更好的代码。

但是如果是对framework的体积大小非常敏感，从目前结果来看，混编Framework还是不能达到预期。

<!-- more -->
