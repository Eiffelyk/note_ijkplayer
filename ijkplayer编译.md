# 【一撸到底--ijkplayer】0.ijkplayer编译（包括android和iOS）
***
## 描述
jkplayer是b站开源的视频播放器，是一个基于FFmpeg的轻量级Android/iOS视频播放器。支持播放本地网络视频，也支持流媒体播放

FFmpeg的是全球领先的多媒体框架，能够解码，编码， 转码，复用，解复用，流，过滤器和播放大部分的视频格式。
它提供了录制、转换以及流化音视频的完整解决方案。
它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多code都是从头开发的。

- ijkplayer : <https://github.com/bilibili/ijkplayer> 
- 引用的openssl : <https://github.com/Bilibili/openssl.git>
  - tag：OpenSSL_1_0_2u
- 引用的ffmpeg : <https://github.com/Bilibili/FFmpeg.git> 
  - tag：ff4.0--ijk0.8.8--20210426--001
- 引用的libyuv : <https://github.com/Bilibili/libyuv.git>
  - tag：ijk-r0.2.1-dev
- 引用的soundtouch : <https://github.com/Bilibili/soundtouch.git>
  - tag：ijk-r0.1.2-dev



##### 写在前边
- 编译过程中遇到问题不要害怕，你遇到的问题，99%的问题其他人也都遇到过
- 去Issues中找答案
- 去Google中找答案
- 确认编译环境是否正常（需要的软件，环境变量[这个修改需要重启终端或者执行source命令]，SDK，NDK等）
- 删除源码，从头开始
- 稳住心态
### 编译环境
- 硬件  
   - Mac OS X 11.5.2  
- 软件
  - Android
    - [NDK r14b](https://github.com/android/ndk/wiki/Unsupported-Downloads) , [NDK其他版本](https://developer.android.com/ndk/downloads)
    - Android Studio 2020.3.1 patch 2
    - Gradle 7.0.2
  - iOS
     - Xcode 12.5.1 (12E507)
  - brew 
  - git
  - yasm
### 编译准备
检查编译环境  
1.brew、2.git、3.yasm、4.SDK、5.NDK（4、5是android编译特有）是否安装完成
### android
```shell
    在~/.bash_profile 或者   ~/.zcshrc添加如下两行
    export ANDROID_SDK=<your sdk path>
    export ANDROID_NDK=<your ndk path>
    
    执行如下命令
    source ~/.zshrc
    source ~/.bash_profile
```
### iOS  
无特殊配置
### 编译（增加了openssl支持） [官方步骤点击此处](https://github.com/bilibili/ijkplayer)
#### Android
0. 老天爷眷顾的男人可以直接copy这段代码，一键交叉编译完成
```shell
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-source
cd ijkplayer-source
git checkout -B latest k0.8.8
./init-android.sh
./init-android-openssl.sh 
cd android/contrib
./compile-openssl.sh clean
./compile-openssl.sh all
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
cd ..
./compile-ijk.sh all
```


1. 获取源码
```shell
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android
git checkout -B latest k0.8.8 #可以不执行
```

2. 获取编译所需的其他资源
   1. 修改init-android.h文件中的内容，可以修改引用的ffmpeg版本，此处使用的版本在https://github.com/bilibili/FFmpeg/tags下
   2. 修改init-config.h文件中的内容，可以修改ffmpeg编译参数
   3. init-android-openssl.sh，可以修改引用的openssl版本，此处使用的版本在https://github.com/bilibili/openssl/tags下
```shell
#获取编译资源FFmpeg并设置ffmpeg编译参数，libyuv，soundtouch
./init-android.sh
#获取编译资源openssl
./init-android-openssl.sh 
```

3. 编译第三方资源
   1. clean是清除编译缓存
   2. all是编译所有支持的架构
   3. ffmpeg和openssl都可以带单独参数进行编译armv5 armv7a arm64 x86 x86_64，例如./compile-xxx.sh armv7a
   4. 由于此编译使用NDK，需提前配置好NDK的环境变量。目前只支持ndk版本11，12，13，14
   5. 编译输出结果在android/contrib/build/对应架构文件/output中
```shell
cd android/contrib
./compile-openssl.sh clean
./compile-openssl.sh all
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```
4. 编译ijkPlayer  
   将编译结果关联到到android/ijkplayer中对应架构下
```shell
cd ..
./compile-ijk.sh all
```
5. 打开demo
```shell
# Android Studio:
#     Open an existing Android Studio project
#     Select android/ijkplayer/ and import
#
#     define ext block in your root build.gradle
#     ext {
#       compileSdkVersion = 23       // depending on your sdk version
#       buildToolsVersion = "23.0.0" // depending on your build tools version
#
#       targetSdkVersion = 23        // depending on your sdk version
#     }
#
# If you want to enable debugging ijkplayer(native modules) on Android Studio 2.2+: (experimental)
#     sh android/patch-debugging-with-lldb.sh armv7a
#     Install Android Studio 2.2(+)
#     Preference -> Android SDK -> SDK Tools
#     Select (LLDB, NDK, Android SDK Build-tools,Cmake) and install
#     Open an existing Android Studio project
#     Select android/ijkplayer
#     Sync Project with Gradle Files
#     Run -> Edit Configurations -> Debugger -> Symbol Directories
#     Add "ijkplayer-armv7a/.externalNativeBuild/ndkBuild/release/obj/local/armeabi-v7a" to Symbol Directories
#     Run -> Debug 'ijkplayer-example'
#     if you want to reverse patches:
#     sh patch-debugging-with-lldb.sh reverse armv7a
#
```


### Build iOS （对应命令解释说明请看android对应命令说明）
0. 乔布斯眷顾所有iOS开发者，一键交叉编译完成
```shell
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
git checkout -B latest k0.8.8

./init-ios.sh
./init-ios-openssl.sh

cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all

# Demo
#     open ios/IJKMediaDemo/IJKMediaDemo.xcodeproj with Xcode
# 
# Import into Your own Application
#     Select your project in Xcode.
#     File -> Add Files to ... -> Select ios/IJKMediaPlayer/IJKMediaPlayer.xcodeproj
#     Select your Application's target.
#     Build Phases -> Target Dependencies -> Select IJKMediaFramework
#     Build Phases -> Link Binary with Libraries -> Add:
#         IJKMediaFramework.framework
#
#         AudioToolbox.framework
#         AVFoundation.framework
#         CoreGraphics.framework
#         CoreMedia.framework
#         CoreVideo.framework
#         libbz2.tbd
#         libz.tbd
#         MediaPlayer.framework
#         MobileCoreServices.framework
#         OpenGLES.framework
#         QuartzCore.framework
#         UIKit.framework
#         VideoToolbox.framework
#
#         ... (Maybe something else, if you get any link error)
# 
```
### 编译完成（demo按照上述设置无法运行）
#### android的demo修改（在android studio中修改）
1. project的build.gradle中的
   1. ext的各个参数修改为你需要的版本
   2. repositories中的jcenter()已经不再支持，修改为google()和mavenCenter()，国内的小伙伴可以使用镜像地址
   3. classpath 'com.android.tools.build:gradle:2.1.3'修改为你设备支持的版本（注意和gradle-wrapper.properties中保持一致）
2. gradle-wrapper.properties修改gradle版本，
   1. 一般android studio导入新项目的时候提示升级gradle，按照提示操作就可以
3. module中的build.gradle中修改
   1. dependencies中的compile修改为api,all32Compile修改为all32Api，all64Compile修改为all64Api
   2. dependencies中的引用的其他lib进行升级
   3. android中添加ndkVersion（gradle7.0之后）
   4. 更新引用的第三方类库的版本（旧版本可能未从jcenter中迁移出来，导致获取不到）
   5. ijkplayer-example的build.gradle在gradle7.0后需要添加flavorDimensions [Configure product flavors](https://developer.android.com/studio/build/build-variants#product-flavors)
4. 由于com.android.support在api28之后不再更新，升级为androidx。
   1. 快捷方式，android studio菜单栏中Refactor中Migrate to AndroidX
   2. 后续按照提示更改，涉及到xml布局文件中的引用，java代码中的引用
5.其他编译错误，按照提示进行修改
#### iOS的demo修改(在Xcode中修改)
1. 修改IJKMediaDemo和IJKMediaPlayer的buildSettings中的iOS Deployment Target为9以上
2. 其他编译错误，按照提示进行修改
## 我遇到的问题
### Android环境变量导致编译失败
### 编译脚本说明
### android NDK版本说明
### android demo修改说明
### 代码结构说明