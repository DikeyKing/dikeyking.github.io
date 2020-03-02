## 前言

有时候，我们需要在iOS工程中使用一些C的库，会需要 CMake 编译；但是对于iOS开发者来说，用 xcodeproj 管理代码的方式更加友好，也方便后续统一管理和开发。

直观觉得好用的可能会有：

- 动态库、静态库编译支持
- 架构（arm/x86）区分和支持
- bitcode 编译设置
- 其它编译设置等：debug symbols、编译优化…

## 配置

我们需要提前准备好以下工具，不在这里追溯：

- CMake
- Xcode
- ios-cmake：https://github.com/leetal/ios-cmake

```
# CMake :
export CMAKE_ROOT=/Applications/CMake.app/Contents/bin/
export PATH=$CMAKE_ROOT:$PATH

# Xcode :
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
```

# 编译



```
# 进入到 CMakeLists.txt 文件的目录
mkdir ios-build
cd ios-build

# /Users/king/Downloads/ios-cmake-master/ios.toolchain.cmake 是交叉编译工具位置
# https://github.com/leetal/ios-cmake
cmake .. -GXcode -DCMAKE_TOOLCHAIN_FILE=/Users/king/Downloads/ios-cmake-master/ios.toolchain.cmake -DPLATFORM=OS64

# .xcodeproj 编译完成，根据情况添加bitcode（编译静态库需要修改Mach-O 选项，以及后缀dylib -> a，设置证书）
```


