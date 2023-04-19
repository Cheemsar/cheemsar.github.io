---
layout: post
title: IjkPlayer 构建记录
categories: [Android, Ijkplayer]
description: IjkPlayer构建中踩过得一些坑
keywords: Ijkplayer, Android
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

[TOC]

记录一下，怕是把坑都踩了一遍。

# 安装环境

 - Oracle Java 11
 - Android NDK 14
 - Android SDK 33
 - Android Studio 4.2.2
 - Gradle Plugin 4.2.2
 - Gradle 6.7.1

# 构建过程
    编译过程中Ijkplayer的编译说明有些已经无法使用了，需要更新

## 安装依赖


```bash 
# Install HomeBrew

# Install Git, Yasm

```

## 构建IjkPlayer

### 检查环境

```

```

### 编译

在这里卡了很久，从试图在Window上编译再到Ubuntu上编译, 一直提示 `ERROR: Failed to create toolchain.`。 

```bash
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```


### 编译FFmpeg

```
# ERROR: Failed to create toolchain.

```

编译完成

![](/images/posts/android/success.png)

### 编译 ijkplayer

![build-ijkplayer](/images/posts/android/build-ijkplayer.png)


### 放在Android Studio 

![](/images/posts/android/fail-to-find-the-so-file.png)
# 问题排查

## Error: Failed to create toolchain

有很多错误会导致出现这个问题，如没有安装`python`, 或者`NDK > 14`等。。

目前我遇到的情况时**NDK下载成了Window版本的NDK**。 按照下面的提示，在NDK里边没有找到linux-x86_64, 打开目录定睛一看，**目录下是window-x86_64**
![](/images/posts/android/failed-to-create-toolchain.png)

可以再这里 [Unsupported NDK Download](https://github.com/android/ndk/wiki/Unsupported-DownIoads) 找到 NDK14 版本的NDK。

## 错误： 程序包android.test不存在

打开 项目 `build.gradle` 修改 `targetSdkVersion` 以及 `compileSdkVersion` 为 `27`

## Failed to find Build Tools revision 33.0.0

打开 项目 `build.gradle` 修改 `buildToolsVersion` 为 `"30.0.3"`

## You need the NDKr10e or later

NDK版本太高了，配置环境变量 `ANDROID_NDK` 指向 `NDK14b 版本` 目录

## java.lang.UnsatisfiedLinkError: dlopen failed: library "libijkffmpeg.so" not found

### 1. 没有CPU架构对应的ABI

这里卡了很久。默认编译ijkplayer只编译了armv7a， 然后我用的模拟器是CPU架构是x86_64，所以一直提示没有发现 `libijkffmpeg.so`。

![Can't find the library "libijkffmpeg.so"](/images/posts/android/cant-load-lib.png)

### 2. Ubuntu上编译，然后在Window上引用so 

这边出现的问题是当 将编译好的 `so` 文件拷贝到Window项目如下
```gradle

# copy these so files into `ijkplayer-x86_64/src/main/libs`
# ijkplayer-x86_64/src/main/libs/libijkffmpeg.so
# ijkplayer-x86_64/src/main/libs/libijkplayer.so
# ijkplayer-x86_64/src/main/libs/libijksdl.so

# Module build.Gradle
sourceSets.main {
        jniLibs.srcDirs 'src/main/libs'
        // ...
    }
```

以上的路径都是按照Ubuntu上编译生成出来的so目录下拷贝的，但结果是在 Android Pane 上没有显示 `jnilibs`

![Failed to reference libs](/images/posts/android/failed-to-ref-libs.png)

解决如下：

拷贝相应的so文件放到模块的根目录的libs下，并修改 `build.gradle` 执行 `libs`

```gradle
# copy these so files into `ijkplayer-x86_64/src/main/libs`
# ijkplayer-x86_64/libs/libijkffmpeg.so
# ijkplayer-x86_64/libs/libijkplayer.so
# ijkplayer-x86_64/libs/libijksdl.so

# Module build.Gradle
sourceSets.main {
        jniLibs.srcDirs 'libs'
        // ...
    }
```

### 3. 没有选择正确的Variant

如果你运行基于x86_64架构上运行，而你使用的32位的编译的，也会存在找不到 `so` 文件， 就像我遇到的一样！

![](/images/posts/android/select-variant-2023-04-19-16-45-14.png)


# 参考

android NDK - make standalone toolchain fail [https://stackoverflow.com/questions/19142886/android-ndk-make-standalone-toolchain-fails]