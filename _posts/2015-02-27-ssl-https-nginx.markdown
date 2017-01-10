---
layout: post
title: NGINX服务器配置SSL证书部署HTTPS网站
date: 2015-02-27 12:00:00.000000000 +08:00
tags: Network
---

作者：杨振林 yangzhenlin.com

本文讲解服务器配置SSL证书部署HTTPS网站。

环境是阿里云服务器ECS ，系统是CentOS6 64bit，Web服务器是Nginx。需要SSL模块的支持。

签发SSL证书的CA机构是 沃通电子认证服务有限公司 WoSign CA Limited。

部署HTTPS网站一般需要有服务器的控制权，（云）主机、VPS均可，虚拟主机基本上做不到。具体步骤如下：

## 向CA机构申请SSL证书

SSL证书的价格一般较贵，申请之前可以去网上搜索一下，最好选择信任度高、浏览器和移动终端支持较好的机构。本文以沃通免费SSL证书为例。

由于沃通已经暂停了免费SSL证书的申请服务，大家可以去 [StartSSL](http://www.startssl.com/) 申请。

![](//source.yangzhenlin.com/ssl-https-nginx/001.png)

申请之后等待CA签发，签发后取回SSL证书保存至本地。解压缩后选择“for Nginx”版本。Apache、Tomcat、IIS用户请自行选择适合版本。

## 上传服务器

将**1-[域名]-bundle.crt**，**2-[域名].key**上传到服务器中的适当位置，给予适当的权限，并记住存放路径。可以使用securtCRT中的rz命令进行上传，或者使用FTP上传并移动至安全的位置。

## 开辟空间，绑定域名

开辟一定的Web空间，将域名（本文中的secret.yangzhenlin.com）做A记录或者CNAME记录，指向主机空间IP地址。此Web空间中的页面在配置完成后可以使用HTTPS进行访问。在下一步中修改Nginx配置文件实现主机绑定域名。

## 修改Nginx的配置文件

核心代码如下，请根据自己的情况替换[域名]部分。

```
server {
server_name [域名];
listen 443;
ssl on;
ssl_certificate /[中间路径]/1_[域名]_bundle.crt;
ssl_certificate_key /[中间路径]/2_[域名].key;
}
```

HTTPS的默认端口是443，HTTP的默认端口是80。可以做301重定向，将HTTP 80端口重定向到HTTPS 443端口。代码如下：

```
server {
listen 80;
server_name [域名];
rewrite ^(.*) https://$server_name$1 permanent;}
```

## 重启Nginx

```
/[中间路径]/nginx/sbin/nginx -s reload
```

访问HTTPS链接即可，例如 **https://secret.yangzhenlin.com/** 效果如下：

![](//source.yangzhenlin.com/ssl-https-nginx/002.png?imageView/2/h/500)

查看证书信息如下：

![](//source.yangzhenlin.com/ssl-https-nginx/003.png?imageView/2/h/500)

![](//source.yangzhenlin.com/ssl-https-nginx/004.png?imageView/2/h/500)

## 可能出现的问题

若使用的域名和向CA机构申请证书填写的域名不符，则会出错。在Chrome和Firefox浏览器中显示效果如下：

![](//source.yangzhenlin.com/ssl-https-nginx/005.png?imageView/2/h/500)

![](//source.yangzhenlin.com/ssl-https-nginx/006.png?imageView/2/h/500)

出现此问题最可能的两个原因是：1、使用的域名和向CA机构申请证书填写的域名不符，2、CA机构不被系统信任（例如12306），让系统信任此CA机构后也能显示出安全的标识，所以12306提示安装根证书。
