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

最近在研究`Android`上播放视频，自然是绕不过 `ijkplayer`。 其中在编译 `ijkplayer` 的过程中学习到很多概念，也踩了很多坑。 在此记录一下。

主要问题来自几个方面：
  1. Gradle 版本 2.1.4 不支持最新版本的AS编译， 需要更新Gradle版本。
  2. Window上MingW不支持Homwbrew，需要手动安装，所以放弃在Win上编译，转战到Ubuntu上编译so，在Window下引用so来编译应用。
  3. 需要使用**已不受支持的NDK版本(NDK14)**才能编译。
  4. 对于适配多种CPU架构，编译多个ABI版本 lib。

以下编译过程按照官方默认的方式编译，即不支持`https`以及`ffmpeg`为3.4版本。 如果后续需要适配`https`编译，也会同步更新到此博文。

现在开始。 

# 安装环境

 - Oracle Java 11
 - Android NDK 14
 - Android SDK 33
 - Android Studio 4.2.2
 - Gradle Plugin 4.2.2
 - Gradle 6.7.1

# 构建过程
    编译过程中Ijkplayer的编译说明有些已经无法使用了, 已做了相应更新，编译过程已经能够成功编译。

## 安装之前

在bash下执行以下命令:

```bash 
# install homebrew
# 这个已经不支持了 
# ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# install git, yasm
brew install git
brew install yasm

# 设置 SDK 以及 NDK (超过14则需要改代码,所以目前最好用14)
# add these lines to your ~/.bash_profile or ~/.profile
export ANDROID_SDK=<your sdk path>
export ANDROID_NDK=<your ndk path>      

# For Ubuntu/Debian users.
# choose [No] to use bash
sudo dpkg-reconfigure dash
```


## 开始编译 ijkPlayer

### 1. 克隆 ijkplayer 并切换到 `k0.8.8`

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android
git checkout -B latest k0.8.8
```


### 2. 初始化

```
./init-android.sh
```

### 3. 编译ffmpeg

在这里卡了很久，一直提示 `ERROR: Failed to create toolchain.`, 解决方法参考文章最后。

```bash
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

编译完成

![](/images/posts/android/success.png)

### 4. 编译 ijkplayer

```bash
./compile-ijk.sh all 
```
![build-ijkplayer](/images/posts/android/build-ijkplayer.png)

编译完成后，lib会生成到对应 ijkplayer-XXXX/src/main/libs 目录下


## 运行Example

Tisp: 操作过程中，**修改`build.gradle`都需要sync**。 中间遇到问题可以参考文章底下的**疑难解决**。

### 1. 更新 Gradle
默认 `ijkplayer` 的 Gradle版本如下

 - Gradle 2.1.4
 - Gradle Plugin 2.1.3

但是在新版的`Android-Studio`上不支持, 所以需要更新Gradle版本

将 Gradle 改为如下

 - Gradle 6.7.1
 - Gradle Plugin 4.2.2

### 2. 更新 Repository

打开 `project` 下的 build.gradle, 添加 repo
```groovy

buildscript {
    repositories {
        jcenter()
        google()        # add
    }

    // ...
}

allprojects {
    repositories {
        jcenter()
        mavenCentral()  # add
        google()        # add
    }

    // ...
}

```

### 3. 更新 compileSdkVersion

```groovy
ext {
    compileSdkVersion = 30          
    buildToolsVersion = "30.0.3"

    targetSdkVersion = 30
}
```

### 4. 更新所有模块引用方式

每个模块中使用的方式 `compile` 改为 `implementation`

![](/images/posts/android/update-reference-method.png)

### 5. 添加FlavorDimensions

新版的Gradle要求每个Flavor必须要添加 `FlavorDimension` 

![](/images/posts/android/add-flavor-dimension.png)

### 6. 引用lib并运行 example

1. 拷贝 ijkplayer-XXXX/src/main/libs 下的文件目录，到Win上的`Android-Studio`上, 如下。
![copy-libs](/images/posts/android/copy-libs.png)

2. 模块 ijkplayer-arm64, armv5, armv7, x86, x86_64下的 `build.Gradle`下修改
`jniLibs.srcDirs` 改为 `libs`

![](/images/posts/android/modify-module-build-gradle.png)

3. 根据你的模拟器， 选择对用的Variant
   
![](/images/posts/android/select-variant.png)


4. 运行 example

![Run examples](/images/posts/android/run-example.png)


# 疑难解决

## 创建 toolchain 失败

 > Error: Failed to create toolchain

有很多错误会导致出现这个问题，如没有安装`python`, 或者`NDK > 14`等。。

目前我遇到的情况时**NDK下载成了Window版本的NDK**。 按照下面的提示，在NDK里边没有找到linux-x86_64, 打开目录定睛一看，**目录下是window-x86_64**
![](/images/posts/android/failed-to-create-toolchain.png)

可以再这里 [Unsupported NDK Download](https://github.com/android/ndk/wiki/Unsupported-DownIoads) 找到 NDK14 版本的NDK。

## 程序包android.test不存在

打开 项目 `build.gradle` 修改 `targetSdkVersion` 以及 `compileSdkVersion` 为 `27`

## 找不到构建工具版本 33.0.0

 > Failed to find Build Tools revision 33.0.0

打开 项目 `build.gradle` 修改 `buildToolsVersion` 为 `"30.0.3"`

## NDK 版本问题

 > You need the NDKr10e or later

NDK版本太高了，配置环境变量 `ANDROID_NDK` 指向 `NDK14b 版本` 目录

## 找不到 `libijkffmpeg.so` 文件

 > library "libijkffmpeg.so" not found

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

## 编译 ffmpeg 4.0 出错 

### Unknown option "--disable-ffserver".

出错原因：ffmpeg 4.0 删除了 ffserver

解决方案：注释掉 ffserver 配置，参考链接，修改 config/module.sh 文件，注释掉以下两行：

```bash
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-ffserver"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-vda"
```

### error: undefined reference to 'ff_ac3_parse_header'

出错原因：ffmpeg 4.0 不再支持 eac3
解决方案：禁掉 eac3，参考链接，修改 config/module.sh 文件，增加如下一行：

```bash
# 在 export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-bsfs" 下方添加：
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-bsf=eac3_core"
```

 - 作者：星海流萤
 - 链接：https://juejin.cn/post/7213307533279182908
 - 来源：稀土掘金
 - 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


# 参考

Android ijkplayer 编译踩坑与记录(ijk0.8.8--ffmpeg4.0)
[https://juejin.cn/post/7213307533279182908]

android NDK - make standalone toolchain fail
[https://stackoverflow.com/questions/19142886/android-ndk-make-standalone-toolchain-fails]