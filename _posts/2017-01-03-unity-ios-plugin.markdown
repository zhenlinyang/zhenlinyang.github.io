---
layout: post
title: Unity iOS 插件开发与SDK接入
date: 2017-01-03 10:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林 yangzhenlin.com

本篇是[Unity Android 插件开发与SDK接入](http://yangzhenlin.com/unity-android-plugin/)的姊妹篇。

## 概述

虽然使用 Unity 可以完成制作一款游戏的绝大部分工作，但研发过程中难免会遇到以下几个问题，需要建立 Unity 与 iOS 的交互。

1. 接入代理商的登录和支付功能，但代理商只提供 iOS 平台 SDK。

2. 一些优秀的插件或工具只提供 iOS 平台 SDK。

3. 接入微博、微信等平台的分享功能。

4. 需要通过 Objective-C 调用 iOS 原生类库代码。

本文通过演示 Unity 通过 C 与 Objective-C 连接，讲述 Unity iOS 插件开发与SDK接入。

## 正文

### Unity 与 iOS 交互的方式

Unity 通过 C 与 iOS 进行交互。因为 Objective-C 可以与 C/C++ 进行混编，所以使用 C 代码封装对应的 Objective-C 代码即可。

> 注意：建议 C 函数名加有特定意义的前缀，避免函数名冲突。

#### Unity 调用 iOS

示例代码展示了通过 Objective-C 进行加法计算的程序。

iOS 部分

MyPluginBridge.h

```
#import <Foundation/Foundation.h>

@interface MyPluginBridge : NSObject

@end

#if defined (__cplusplus)
extern "C"
{
#endif
    
    int AddC(int a, int b);
    
#if defined (__cplusplus)
}
#endif
```

MyPluginBridge.m

```
#import "MyPluginBridge.h"

@implementation MyPluginBridge

+(int) Add:(int)a and:(int)b{
    return a + b;
}

@end

#if defined (__cplusplus)
extern "C"
{
#endif
    
    int AddC(int a, int b){
        return [MyPluginBridge Add:a and:b];
    }
    
#if defined (__cplusplus)
}
#endif
```

Unity 部分

```
using System.Runtime.InteropServices;

public static class MyPluginBridge
{
    [DllImport("__Internal")]
    private static extern int AddC(int a, int b);

    public static int Add(int a, int b)
    {
        return AddC(a, b);
    }
}
```

#### iOS 向 Unity 发送通知

```
UnitySendMessage("G", "A", "B");
```

### 插件添加到工程

如果插件只包含 .h/.m/.mm/.c/.cpp/.a 文件，直接放入 Unity `Assets/Plugins/iOS` 目录即可。在 Unity 导出 Xcode 工程时，这些文件会放置在`Unity-iPhone/Libraries/Plugins/iOS`目录。

如果插件包含 .framework/.bundle 文件，需要在 Unity 导出 XCode 工程之后，手动拖入 Xcode 工程中。勾选`Copy items if needed`，选择`Create groups`，勾选`Add to targets: Unity-iPhone`。

![](http://source.yangzhenlin.com/unity-ios-plugin/choose-options.png)

### 在 Unity 中修改 PBXProject 和 Plist

如果每次导出 Xcode 之后手动修改 PBXProject 和 Plist，将会非常麻烦。UnityEditor 提供一个`[PostProcessBuild]`标签，可以帮助我们在导出 Xcode 之后执行一些自动化操作。

#### 标记 PostProcessBuild 方法

```
[PostProcessBuild]
public static void OnPostprocessBuild(BuildTarget buildTarget, string path)
{
    if (BuildTarget.iOS != buildTarget)
    {
        return;
    }

    ModifyPBXProject(path);

    ModifyPlist(path);

    Debug.Log("Xcode修改完毕");
}
```

#### 修改 PBXProject

```
private static void ModifyPBXProject(string path)
{
    string projPath = PBXProject.GetPBXProjectPath(path);
    PBXProject proj = new PBXProject();

    proj.ReadFromString(File.ReadAllText(projPath));
    string target = proj.TargetGuidByName("Unity-iPhone");

    //执行修改操作

    File.WriteAllText(projPath, proj.WriteToString());
}
```

修改 SEARCH_PATHS

```
proj.SetBuildProperty(target, "LIBRARY_SEARCH_PATHS", "$(inherited)");
proj.AddBuildProperty(target, "LIBRARY_SEARCH_PATHS", "$(SRCROOT)");
proj.AddBuildProperty(target, "LIBRARY_SEARCH_PATHS", "$(PROJECT_DIR)/Libraries");
```

添加 .framework/.tbd

bool 参数 true 表示框架是 optional，false 表示框架是 required。

```
//苹果内购
proj.AddFrameworkToProject(target, "StoreKit.framework", false);
```

添加 OTHER_LDFLAGS

```
proj.AddBuildProperty(target, "OTHER_LDFLAGS", "-ObjC");
```

#### 修改 Plist

```
private static void ModifyPlist(string path)
{
    //Info.plist
    string plistPath = path + "/Info.plist";
    PlistDocument plist = new PlistDocument();
    plist.ReadFromString(File.ReadAllText(plistPath));

    //ROOT
    PlistElementDict rootDict = plist.root;

    //执行修改操作

    //写入
    File.WriteAllText(plistPath, plist.WriteToString());
}
```

设置语言

```
//设置使用简体中文
rootDict.SetString("CFBundleDevelopmentRegion", "zh_CN");
```

iOS 10 设置使用权限说明

```
//摄像机权限
rootDict.SetString("NSCameraUsageDescription", "AR");

//定位权限
rootDict.SetString("NSLocationWhenInUseUsageDescription", "LBS");

//录音权限
rootDict.SetString("NSMicrophoneUsageDescription", "VoiceChat");
```

添加第三方应用的 URL Scheme 到白名单

```
PlistElementArray LSApplicationQueriesSchemes = rootDict.CreateArray("LSApplicationQueriesSchemes");
//微信
LSApplicationQueriesSchemes.AddString("weixin");
```

添加自己应用的 URL Scheme

```
PlistElementArray urlTypes = rootDict.CreateArray("CFBundleURLTypes");

//网页唤起
PlistElementDict webUrl = urlTypes.AddDict();
webUrl.SetString("CFBundleTypeRole", "Editor");
webUrl.SetString("CFBundleURLName", "web");
PlistElementArray webUrlScheme = webUrl.CreateArray("CFBundleURLSchemes");
webUrlScheme.AddString("productname"); //换成自己的产品名
```

## 参考资料

[Unity - Manual: Building Plugins for iOS](https://docs.unity3d.com/Manual/PluginsForIOS.html)

[Unity - Scripting API: PostProcessBuildAttribute](https://docs.unity3d.com/ScriptReference/Callbacks.PostProcessBuildAttribute.html)

[Unity - Scripting API: PBXProject](https://docs.unity3d.com/ScriptReference/iOS.Xcode.PBXProject.html)

[Unity - Scripting API: PlistDocument](https://docs.unity3d.com/ScriptReference/iOS.Xcode.PlistDocument.html)
