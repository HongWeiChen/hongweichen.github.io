---
layout:     post
title:      React-Native中常用的命令
subtitle:   笔记
date:       2021/12/15
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - React-Native
---

# 创建项目

react-native init

# 运行iOS

react-native run-ios

# 运行iOS指定模拟器

react-native run-ios —simulator "iPhone 12"

# 运行Android

react-native run-android(真机)

# 检查环境是否正确

react-doctor

# 不存在iOS/Android文件时创建Native Code

react-native eject

# 编译Index Android+iOS

Android：react-native bundle --entry-file='index.js' --bundle-output='./android/app/src/main/assets/index.android.bundle' --dev=false --platform='android'

iOS：react-native bundle --entry-file index.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false

# 打包Android APK文件

cd rootProject -> cd android -> ./gradlew assembleRelease

# 打包iOS IPA文件

在Xocde中使用Archive打包

# Android Release签名

jarsigner -verbose -keystore release.keystore -signedjar release.apk app-release_unsign.apk my-key-alias

# 查看依赖库版本

npm view <packagename> versions --json

# Android或者iOS有时出现映射index.js文件失效，提示Ensure that the packager server is running

adb reverse tcp:8081 tcp:8081

# Android 需要Hot reload 或者 调整Debug模式

adb shell input keyevent 82

**此命令不一定适用于所有手机**

# React-Native初始化Android SDK环境

export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
