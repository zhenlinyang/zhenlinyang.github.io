---
layout: post
title: 五子棋双人在线对战 GOBANG ONLINE
date: 2014-12-03 12:00:00.000000000 +08:00
tags: Java
---

作者：杨振林 yangzhenlin.com

/scr 目录下全部代码本人完成，课内作品，公开全部源代码，遵循General Public Licence协议。

托管至 Github 链接：https://github.com/zhenlinyang/gobang

***

## README.md

Version 1.0 基本实现在线五子棋功能。运行Main不需要参数。 master是最新版本。运行Main需要参数。

### 使用方法

先运行 Server.java ，开启服务器。master运行参数为 agrs[0]=SERVER_PORT 然后运行两次 Client.java ，产生两个不同的客户端，分别作为黑棋方与白棋方的客户端。master运行参数为 agrs[0]=IP agrs[1]=PORT 开始运行后，正常游戏即可。

### jar

jar目录下为导出的带有Main方法的jar包。 server.jar 部署到服务器。运行方式 java -jar server.jar args[0]=SERVER_PORT client.jar 下载到玩家机器。运行方式 java -jar client.jar args[0]=IP args[1]=PORT

### 导出jar
本程序使用Eclipse开发，下载后导入Eclipse即可直接使用。 如果想导出jar包并通过命令行运行，请更改涉及图片资源语句。用带有#的被注释语句替换上一行代码。

***

附工程包（更新至2014/12/3）

[gobang-master.zip](http://source.yangzhenlin.com/gobang-master.zip)