---
layout:     post
title:      React-Native常用的一些命令笔记
subtitle:   React-Native常用的一些命令笔记
date:       2021/12/15
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - React-Native
---

## 初始化项目
react-native init

## 运行

iOS运行指定模拟器：react-native run-ios —simulator “iPhone 12”
  or react-native run-ios (default Simulator iPhone X or iPhone 11)

Android（真机）：react-native run-android

## 检查环境是否正确
react-doctor

## 若不存在iOS/android文件使用这个命令创建native code

react-native eject 若不存在iOS/android文件使用这个命令创建native code

## 打包

#### 编译index.js

iOS：react-native bundle --entry-file='index.js' --bundle-output='./android/app/src/main/assets/index.android.bundle' --dev=false --platform='android'

在Xcode中编译IPA包

Android：react-native bundle --entry-file index.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false

编译APK
cd rootProject
cd android
./gradlew assembleRelease

签名
jarsigner -verbose -keystore release.keystore -signedjar release.apk app-release_unsign.apk my-key-alias

## 查看当前项目依赖库版本

npm view <packagename> versions --json

## Ensure that the packageer server is running....

adb reverse tcp:8081 tcp:8081

## Android 弹出指令框

adb shell input keyevent 82

## tips

如果你不是通过Android Studio安装的sdk，则其路径可能不同，请自行确定清楚
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
