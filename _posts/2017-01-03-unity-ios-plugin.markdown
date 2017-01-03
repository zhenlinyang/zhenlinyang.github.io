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

//TODO 补充

## 正文

### Unity 与 iOS 交互的方式

Unity 通过 C 与 iOS 进行交互。因为 Objective-C 可以与 C/C++ 进行混编，所以使用 C 代码封装对应的 Objective-C 代码即可。

> 注意：建议 C 函数名加有特定意义的前缀，避免函数名冲突。

#### Unity 调用 iOS

示例代码展示了通过 Objective-C 进行加法计算的程序。

iOS 部分

MyPluginBridge.h

```
#ifndef MyPluginBridge_h
#define MyPluginBridge_h


#endif /* MyPluginBridge_h */

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

如果插件只包含 .h/.m/.mm/.c/.cpp/.a 文件，直接放入 Unity `Assets/Plugins/iOS` 目录即可。在 Unity 导出 XCode 工程时，这些文件会放置在`Unity-iPhone/Libraries/Plugins/iOS`目录。

如果插件包含 .framework/.bundle 文件，需要在 Unity 导出 XCode 工程之后，手动拖入 Xcode 工程中。勾选`Copy items if needed`，选择`Create groups`，勾选`Add to targets: Unity-iPhone`。

![](http://source.yangzhenlin.com/unity-ios-plugin/choose-options.png)

### 在 Unity 中修改 PBXProject 和 Plist

//TODO

