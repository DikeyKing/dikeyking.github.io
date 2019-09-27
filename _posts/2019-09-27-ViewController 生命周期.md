---
title: ViewController Life Cycle
date: 2019-09-27 
categories: 随笔
tags: [X]
---

![图片](/assets/images/ViewcontrollerLifeCycle.jpg)

**View Did Load ：** 

在通过storyboard或者代码初始化一个 view controller之后， view controller将会将会加载视图到内存中。这个过程创建了controller会管理的视图。在 view controller完成特定视图加载之后，viewDidLoad() 方法就会被调用。controller 就可以利用这个机会完成一些依赖视图已经被加载和准备完成的操作。

举个栗子：我们可以乘着视图初始化完成添加一个 updateUIColor() 方法，来设置各种控件的颜色。另外一些典型的操作比如**增加额外视图**、**网络请求**、或者**数据访问**。

**viewWillAppear(_:) ：**

这个方法在view出现在屏幕之前调用，**并且每次调用**。所以非常合适做一些需要在视图出现之前完成的操作，并且需要每次调用的操作。举个栗子：比如用户当前所处的**地理位置**，每次进入界面都可以更新它。另外一些包括：**开始网络请求**、**刷新界面**（如status bar，navigation bar， table views）、**旋转屏幕方位**。

**viewDidAppear(_:)：** 

viewDidAppear(_: )  将会在界面出现在屏幕的时候被调用。适合将一些**需要每次调用，并且比较费时的操作**放到这个方法中，来保证界面展示流畅。可供参考的操作：在这个方法中进行网络数据下载等耗时操作，HUD或者动画提醒用户。

**viewWillDisappear(_:)**：

这个方法在界面将要消失在屏幕的时候调用。这个方法会在用户点击**返回**、**tab**、**presenting 或 dismissing Model View**的时候调用。这个方法适合**保存编辑**、**隐藏键盘**、**取消网络**请求等操作。

**viewDidDisappear(_:)**：

当视图从屏幕上被移出的时候调用，典型的情况，当用户通过navigation去了另外一个界面的时候。这个方法执行的时候能确保视图已经消失。所以适合去做一些和视图相关的停止逻辑。比如，**停止正在播放的音乐**或者**移除消息通知**。



**参考**: Apple Education. “App Development with Swift.” Apple Inc. - Education, 2019. Apple Books. 
