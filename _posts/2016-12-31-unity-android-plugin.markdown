---
layout: post
title: Unity Android 插件开发与SDK接入
date: 2016-12-31 11:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林

## 概述

虽然使用 Unity 可以完成制作一款游戏的绝大部分工作，但研发过程中难免会遇到以下几个问题，需要我们建立 Unity 与 Android 的交互。

1. 接入代理商的登录和支付功能，但代理商只提供 Android 平台 SDK。

2. 一些优秀的插件或工具只提供 Android 平台 SDK。

3. 接入微博、微信等平台的分享功能。

//TODO 补充文本

## 工具与环境

* Unity

* Java

* Android SDK

* [Eclipse](http://www.eclipse.org/) with [ADT](https://developer.android.com/studio/tools/sdk/eclipse-adt.html) or [Android Studio](https://developer.android.com/studio/index.html)

## 正文

### Unity 与 Android 交互的方式

Unity 调用 Android 主要分为两种形式。

1. 调用 Android NDK 生成的 .so 库中的 C/C++ 函数

2. 调用 JDK 生成的 .jar 库中的 Java 方法 

#### 调用 C/C++

Unity 为了避免 Name mangling (名字修饰)，只接受 C 形式的声明，所以 C++ 函数需要包装成如下形式，提示编译器使用 C 的方式来处理函数。

```
extern "C" {
  float FooPluginFunction ();
}
```

在 Unity C# 中，每个 C 函数需要定义为如下形式。

```
[DllImport ("PluginName")]
private static extern float FooPluginFunction ();
```

注：此形式和针对 iOS 平台的 Unity 插件开发有共通之处。

#### 调用 Java

> 从原理上讲，Unity 通过 C 和 JNI 连接到 Java。本文略过详细过程。

Unity 封装了两个类，方便我们直接使用 Java。

* UnityEngine.AndroidJavaObject 针对 Java 对象

* UnityEngine.AndroidJavaClass 针对 Java 类

注意：请注意使用`using`包装 AndroidJavaObject 和 AndroidJavaClass，以便调用`IDisposable.Dispose()`。

使用 AndroidJavaObject 创建对象，然后使用 Call 调用方法，接受模版指定返回类型。

```
AndroidJavaObject jo = new AndroidJavaObject("java.lang.String", "some_string");
int hash = jo.Call<int>("hashCode"); 
```

使用 AndroidJavaClass 获取类，然后使用 CallStatic 调用类方法，接受模版指定返回类型。

```
AndroidJavaClass cls = new AndroidJavaClass("java.util.Locale"))
AndroidJavaObject locale = cls.CallStatic<AndroidJavaObject>("getDefault")
```

使用 Set/SetStatic/Get/GetStatic 设置或获取字段。

```
AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
```

#### Java 向 Unity 发送通知

在 Java 中直接使用 UnitySendMessage 方法。

```
UnitySendMessage ("GO", "M", "P");
```

表示向 Unity 名字为`G`的`GameObject`挂载的脚本中，声明为`M(string parameter)`的方法发送带一个参数`"P"`的消息。注意避免多个脚本同时存在方法`M(string parameter)`。

### Unity 接入 Android 平台插件

// TODO

## 参考资料

[Unity - Manual: Building Plugins for Android](https://docs.unity3d.com/Manual/PluginsForAndroid.html)

[開發 Unity Android Plugin – 基本認識 – Basis of Unity Android Plugin](https://douduck08.wordpress.com/2016/06/05/basis-of-unity-android-plugin/)

[開發 Unity Android Plugin – 從零開始](https://douduck08.wordpress.com/2016/06/08/birth-of-unity-android-plugin/)
