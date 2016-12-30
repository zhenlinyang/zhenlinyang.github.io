---
layout: post
title: Unity Android DLL 热更新与加密
date: 2016-12-29 16:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林

## 摘要

本文主要介绍，在安卓环境下，通过更换或解密 Unity-Mono 加载的 Assembly-CSharp.dll 数据文件的方法，达到更新或加密代码的目的。

## 工具与环境

* OS `macOS Sierra 10.12.2`

* Mono `mono unity-5.5` [https://github.com/Unity-Technologies/mono/tree/unity-5.5](https://github.com/Unity-Technologies/mono/tree/unity-5.5)

* Unity `Unity 5.5.0f3 for Mac`

* Java `JavaSE-1.8`

* Android SDK

* Android NDK `Android NDK r10e`

* autoconf

* automake

* libtool

* pkg-config

> autoconf automake libtool pkg-config 四项可以使用 Homebrew 安装。
>
> 获取 Homebrew `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
>
> 安装 `brew install autoconf automake libtool pkg-config`

## 正文

### 编译 libmono.so

**【步骤一】**选择 Unity 版本对应的 Mono 分支，例如本文选择 Unity 5.5.0f3 则对应 Mono 的版本是 unity-5.5。进入 mono 根目录，mono 根目录是编译 Mono 的工作目录。

**【步骤二】**修改`./external/buildscripts/build_runtime_android.sh`和`./external/buildscripts/build_runtime_android_x86.sh`。

`build_runtime_android_x86.sh`主要负责编译 x86 架构下的 libmono.so。

`build_runtime_android.sh`主要负责编译 arm 架构下的 libmono.so，然后调用`build_runtime_android_x86.sh`。

//TODO android 配图 和 android_x86 配图

`build_runtime_android_x86.sh`去掉`-g`，去掉调试符号。

`build_runtime_android.sh`将 CFLAGS 下的`-g`修改为`-O2`，去掉调试符号，并增加优化符号。

> `-g` 调试符号
>
> `-O2` 优化符号

注意：“O”是英文第15个字母的大写，不是零。

`build_runtime_android.sh`删除编译`armv5`、`armv6_vfp`，只剩下`armv7a`。

在 mono 根目录下，执行`./external/buildscripts/build_runtime_android.sh`。大约会执行几分钟的时间。

编译成功后，终端会提示：

```bash
Build SUCCESS!
Build failed? Android STATIC/SHARED library cannot be found... Found        4 libs under builds/embedruntimes/android
total 0
drwxr-xr-x  4 lbs  staff  136 12 30 11:40 armv7a
drwxr-xr-x  4 lbs  staff  136 12 30 11:42 x86
```

在`./builds/embedruntimes/android`目录下，会有`armv7a`和`x86`环境下的`libmono.so`。

### DLL热更新与加密原理

打开`./mono/metadata/image.c`文件，找到`mono_image_open_from_data_with_name (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, const char *name)`函数。

## 技术支持

[Unity-Technologies/mono](https://github.com/Unity-Technologies/mono)

[Android NDK](https://developer.android.com/ndk/downloads/index.html)

## 参考资料

[Unity3D-重新编译Mono加密DLL](http://www.luzexi.com/unity3d/%E6%B8%B8%E6%88%8F%E6%9E%B6%E6%9E%84/%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF/2015/04/11/Unity3D-%E9%87%8D%E6%96%B0%E7%BC%96%E8%AF%91Mono%E5%8A%A0%E5%AF%86DLL.html)

[Unity Android 动态更新 Assembly-CSharp.dll](http://blog.sina.com.cn/s/blog_9e5d42ee0102vvtg.html)

[Unity3D研究院之Android加密DLL与破解DLL .SO](http://www.xuanyusong.com/archives/3553)