---
layout: post
title:  "Meta XR Hand Demo"  
---

# Meta XR Hand Demo

## 前置准备

- Unity 2022.3 or later

## 一：创建项目

在Unity Hub中新建项目：

1. 点击**项目->新项目**
2. 可选择 **Universal 3D 模版**
    
    ![001.JPEG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/001.jpeg)
    
## 二：导入插件

步骤

1. 在**Unity**上方菜单栏，选择**WIndow->Package Manager**，然后如下图所示，点击**Add package by name**：
    
    ![002.PNG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/002.png)
    
2. **XR hands（必选）**
    1. 输入 `com.unity.xr.hands`
    2. 版本号指定 `1.4.0`
    3. 在sample中导入 `Gestures` 和 `HandVisualizer`
        
        ![003.PNG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/003.png)
        
3. **XR Interaction Toolkit （必选）**
    1. 输入 `com.unity.xr.interaction.toolkit`
    2. 版本号指定 `2.5.4`
        
        ![004.PNG](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/004.png)
        
4. **OpenXR Plugin（必选）**
    
    [https://www.notion.so](https://www.notion.so)
    
5. **Oculus XR Plugin（如果需要透视模式，建议加上）**
    1. 输入 `com.unity.xr.oculus`
    2. 版本号指定 `4.2.0`
        
        ![006.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/006.png)
        
6. **Unity OpenXR Meta（需要加上）**
    1. `com.unity.xr.meta-openxr`
    2. `1.0.1`
        
        ![007.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/007.png)
        

## 四、环境配置

1. `Build Settings` → `Android` → `switch platform`
2. `Build Settings → Player Settings → XR Plug-in Management` 
    1. 勾选 `OpenXR` → 勾选 `Meta Quest feature group`
        
        ![008.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/008.png)
        
    2. OpenXR
        - 增加 `Meta Quest Touch Pro Controller Profile`
        - 增加`Oculus Touch Controller Profile`
        - 勾选设置如图所示
            
            ![009.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/009.png)
            
        - 勾选 Quest 对应 Target
            
            ![010.jpeg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/010.jpeg)
            
- Oculus （如果需要透视模式）
    - 勾选 Quest 2
    - Quest 3 / Quest Pro
        
        ![011.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/011.png)
        
- 如果存在错误：在此处 fix
    
    ![012.jpeg](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/012.jpeg)
    

## 五、测试

### 模拟器测试

1. `Build Settings → XR Plug-in Management → XR Interaction Toolkit` ，勾选 `Use XR Device Simulator in scenes`
    
    ![013.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/013.png)
    
2. 打开DemoScene：  `Sample → XR Hands → 1.4.0 → Gestures → HandGestures`
    
    ![014.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/014.png)
    
3. 手势触发
    1. 使用键盘 `H` 切换到手势
    2. 使用 `T、Y` 切换到左右手
    3. 使用 `O` 打开手掌
    4. 使用鼠标旋转后，发现`Palm Up` 触发成功（高亮显示）
        
        ![015.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/015.png)
        

### 真机测试

1. 打包
    
    ![016.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/016.png)
    
2. 使用 `adb` 或 `SideQuest` 安装 到 `Meta Quest`
    
    ![017.png](https://raw.githubusercontent.com/DikeyKing/dikeyking.github.io/master/_posts/Meta%20XR%20Hand%20Demo/017.png)
    
3. 参考视频

- todo
1. 参考 Demo & APK for Meta

- todo

**下一章节：Meta XR 接入文档二：自定义手势及射线交互**

- todo
