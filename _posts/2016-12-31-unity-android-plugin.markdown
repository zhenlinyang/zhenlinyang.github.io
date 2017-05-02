---
layout: post
title: Unity Android 插件开发与SDK接入
date: 2016-12-31 11:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林 yangzhenlin.com

本篇是[Unity iOS 插件开发与SDK接入](//www.yangzhenlin.com/unity-ios-plugin/)的姊妹篇。

## 概述

虽然使用 Unity 可以完成制作一款游戏的绝大部分工作，但研发过程中难免会遇到以下几个问题，需要建立 Unity 与 Android 的交互。

1. 接入代理商的登录和支付功能，但代理商只提供 Android 平台 SDK。

2. 一些优秀的插件或工具只提供 Android 平台 SDK。

3. 接入微博、微信等平台的分享功能。

本文通过“Unity 与 Android 交互的方式”和“为 Unity 开发 Android 插件”两个部分，讲述如何进行Unity Android 插件开发与SDK接入。

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

Unity 为了避免 C++ 编译有 Name mangling 问题，只接受 C 形式的声明，所以 C++ 函数需要包装成如下形式，提示编译器使用 C 的方式来处理函数。

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

> 此形式和针对 iOS 平台的 Unity 插件开发有共通之处。

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

在 Java 中直接使用 UnitySendMessage 静态方法。

```
UnityPlayer.UnitySendMessage ("G", "M", "P");
```

表示向 Unity 名字为`G`的`GameObject`挂载的脚本中，声明为`public void M(string parameter)`的方法，发送一个参数为字符串`"P"`的消息。注意避免多个脚本同时存在方法`M`。

### 为 Unity 开发 Android 插件

首先创建 `Assets/Plugins/Android/`目录，Android 插件都放置在这个目录中。

#### 是否需要继承 UnityPlayerActivity

`UnityPlayerActivity`是 Unity Android 程序的 Main Activity，游戏启动时会首先进入。

如果我们需要改写或者扩展`UnityPlayerActivity`，需要创建一个子类`CustomUnityPlayerActivity`继承自`UnityPlayerActivity`，同时创建`AndroidManifest.xml`包含如下信息。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.company.product">
  <application android:icon="@drawable/app_icon" android:label="@string/app_name">
    <activity android:name=".CustomUnityPlayerActivity"
             android:label="@string/app_name"
             android:configChanges="fontScale|keyboard|keyboardHidden|locale|mnc|mcc|navigation|orientation|screenLayout|screenSize|smallestScreenSize|uiMode|touchscreen">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
  </application>
</manifest>
```

**兼容性问题**

1. Android 应用只能存在一个 Main Activity，因而一个 Unity 工程最多只能存在一个继承自`UnityPlayerActivity`的插件。

2. 继承和替换 Unity 原生的 UnityPlayerActivity，可能导致其他插件无法正常运行，因为大多数插件会默认`UnityPlayerActivity`是 Main Activity。

从上述两点可以看出，在非必要情况下，不要继承`UnityPlayerActivity`。

#### 插件放置位置

Unity 提供两种方式放置插件，推荐使用第二种方式。

**单一插件**

如果只有一个插件，或者插件继承了`UnityPlayerActivity`，插件放置形式如下。

```
Assets/Plugins/Android/libs/*.jar
Assets/Plugins/Android/libs/x86/*.so
Assets/Plugins/Android/libs/armeabi-v7a/*.so
Assets/Plugins/Android/AndroidManifest.xml
```

Unity 会将`Assets/Plugins/Android/AndroidManifest.xml`做为 Android 工程的 AndroidManifest。

**多个插件**

如果存在多个插件，应为每一个插件创建一个子目录，插件放置形式如下。

```
Assets/Plugins/Android/MyPlugin/libs/*.jar
Assets/Plugins/Android/MyPlugin/libs/x86/*.so
Assets/Plugins/Android/MyPlugin/libs/armeabi-v7a/*.so
Assets/Plugins/Android/MyPlugin/AndroidManifest.xml
Assets/Plugins/Android/MyPlugin/project.properties
```

`project.properties`内容如下，Android Target 版本根据实际情况填写。

```
target=android-23
android.library=true
```

Unity 会将所有插件目录的`AndroidManifest.xml`文件与 Unity 自身的`AndroidManifest.xml`文件合并，做为 Android 工程的 AndroidManifest。

#### 插件制作流程

打开 Eclipse，创建一个 Android Application，注意 Mininum Required SDK 需要小于等于 Unity Player Settings 中设定的值，最好设置成一样的。

![](//source.yangzhenlin.com/unity-android-plugin/new-android-application.png)

在 Configure Project 中设置项目是一个类库。

![](//source.yangzhenlin.com/unity-android-plugin/configure-project.png)

打开 Unity 的安装路径，找到`classes.jar`，复制到`libs`目录中。

`classes.jar`中包含类`com.unity3d.player.UnityPlayer`，`UnityPlayer` 负责连接 Unity 与 Android 插件。

创建一个测试类`MyPluginBridge`，添加一个方法`SendToUnity`向 Unity 发信。

```
package myplugin;

import com.unity3d.player.UnityPlayer;

public class MyPluginBridge {
  public void SendToUnity() {
    UnityPlayer.UnitySendMessage("G", "M", "P");
  }
}
```

打开项目目录，进入`bin/classes`，删除无关项（包括`R.class`，`BuildConfig.class`等），将`MyPluginBridge.java`生成的`MyPluginBridge.class`打成 Jar 包。

```
zqlt:~ lbs$ cd /Users/lbs/Documents/workspace/myplugin/bin/classes 
zqlt:classes lbs$ jar -cvf myplugin.jar *
已添加清单
正在添加: myplugin/(输入 = 0) (输出 = 0)(存储了 0%)
正在添加: myplugin/MyPluginBridge.class(输入 = 515) (输出 = 323)(压缩了 37%)
```

修改`AndroidManifest.xml`，删除多余的`application`部分。

将`myplugin.jar`、`project.properties`、`AndroidManifest.xml`放入 Unity。

在 Unity 中封装对 Android 的调用。

```
public void Call_SendToUnity()
{
  using (AndroidJavaObject jo = new AndroidJavaObject ("myplugin.MyPluginBridge"))
  {
    jo.Call ("SendToUnity");
  }
}
```

#### 关于 UnityPlayer

`com.unity3d.player.UnityPlayer`是 Unity Android 中一个非常重要的连接类。

* `public static void UnityPlayer.UnitySendMessage(String arg0, String arg1, String arg2)`通过此静态方法可以向 Unity 中的脚本发信。

* `public static android.app.Activity currentActivity;`通过此静态字段可以获取当前的 Activity。

## 参考资料

[Unity - Manual: Building Plugins for Android](https://docs.unity3d.com/Manual/PluginsForAndroid.html)

[開發 Unity Android Plugin – 基本認識 – Basis of Unity Android Plugin](https://douduck08.wordpress.com/2016/06/05/basis-of-unity-android-plugin/)

[開發 Unity Android Plugin – 從零開始](https://douduck08.wordpress.com/2016/06/08/birth-of-unity-android-plugin/)
