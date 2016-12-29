---
layout: post
title: Unity Android DLL 热更新与加密
date: 2016-12-29 16:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林

## 摘要

安卓环境下，通过更换或解密 Unity-Mono 加载的 Assembly-CSharp.dll 数据文件的方法，达到更新或加密代码的目的。

## 准备环境

* Java

* Android SDK

* Android NDK

* autoconf

* automake

* libtool

* pkg-config

> autoconf automake libtool pkg-config 四项可以使用 Homebrew 安装。

> (1)获取 Homebrew `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

> (2)安装 `brew install autoconf automake libtool pkg-config`

## 正文

//TODO

## 测试环境

* macOS Sierra 10.12.2

* Unity 5.5.0f3 for Mac

* mono unity-5.5 [https://github.com/Unity-Technologies/mono/tree/unity-5.5](https://github.com/Unity-Technologies/mono/tree/unity-5.5)

* Android NDK r10e

## 技术支持

[Unity-Technologies/mono](https://github.com/Unity-Technologies/mono)

[Android NDK](https://developer.android.com/ndk/downloads/index.html)

## 参考资料

[Unity3D-重新编译Mono加密DLL](http://www.luzexi.com/unity3d/%E6%B8%B8%E6%88%8F%E6%9E%B6%E6%9E%84/%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF/2015/04/11/Unity3D-%E9%87%8D%E6%96%B0%E7%BC%96%E8%AF%91Mono%E5%8A%A0%E5%AF%86DLL.html)

[Unity Android 动态更新 Assembly-CSharp.dll](http://blog.sina.com.cn/s/blog_9e5d42ee0102vvtg.html)

[Unity3D研究院之Android加密DLL与破解DLL .SO](http://www.xuanyusong.com/archives/3553)