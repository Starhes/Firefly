---
title: 在 serv00 上部署 filen-s3
published: 2025-03-03
updated: 2025-03-03T20:54:47.944492+08:00
description: 本文介绍如何在 serv00 上部署 filen-s3，实现免费对象存储服务的自建方案。
tags: [serv00, filen-s3, 对象存储, 部署, 自建云]
category: 服务器部署
slug: serv00-filen-s3-deploy
draft: false
---

# 在serv00上部署[filen-s3]

（部署了但是没有实际应用，等大佬测试

今天想要找一个S3用一下，但是不想被流量限制（也不想被cf请求数限制）（虽然后面还是用了R2），但是再后来看到了filen-cil支持webdav和s3，很容易找到了filen-webdav但是没看到filen-s3，所以自己试着搞了一下然后记录一下。

## 在panel面板新建网站

首先登录你的serv00面板，点开WWW websites──Add new website，添加一个node类型的网站，域名使用你想要使用的域名，这里我用s3.serv00.net举例  
​![Image](https://img0.mystar.eu.org/2025/03/7ed0c3b3-2716-44ea-b2f8-49616cb3a253)  
然后在将你的域名解析至serv00提供的两个ip其中一个  
进入ssl-WWW websites，可以看到两个ip，二选其一即可

​![Image](https://img0.mystar.eu.org/2025/03/55900e01-9c7c-4c8e-8c5f-9f471200f881)  
下一步点进你选择的ip-manage  
​![Image](https://img0.mystar.eu.org/2025/03/55224d18-4964-4f08-8750-2c308c0c5b60)​

上传/创建一个证书，这里如果你选择了**cf的十年证书/此ip已被墙/担心ip被墙**可以选择，上传**cf的十年证书，并且在cf中开启小黄云**。

下一步**开放一个端口**并记下备用

## 部署项目

### 文件准备

进入**domains/你的域名/public_nodejs**，首先删除pubilc文件夹，然后在public\_nodejs目录下新建以下文件结构  
/home/LOGIN/domains/DOMAIN/public\_nodejs/  
├──app.js  
├──package.json  
└──server.js

server.js内容  
[https://gist.github.com/maohais/1a41f00f33ab3a08b1e971420ea7998a](https://gist.github.com/maohais/1a41f00f33ab3a08b1e971420ea7998a)  
这里注意修改15，16，17行处的filen登录信息，21行实际开放端口，30，31，38，39行的s3认证信息

app.js内容  
[https://gist.github.com/maohais/e4205417639cba0231d5696e3a53d7f8](https://gist.github.com/maohais/e4205417639cba0231d5696e3a53d7f8)  
需要修改13行的端口信息

package.json内容  
[https://gist.github.com/maohais/18a788f22f8ae32417c1d6a86b0c3f21](https://gist.github.com/maohais/18a788f22f8ae32417c1d6a86b0c3f21)

### 安装依赖

连接ssh进入本目录，运行`npm22 install`​

## 使用项目

不出意外的话，到这里项目就安装完成了，下面你只需要访问你的域名，即可唤醒你的项目进行使用。  
不需要额外进行保活，这个方案使用了saika大佬的保活方案，访问即唤醒，如果有需要就监控以下自己的网址就行。

## S3的使用

> **Important:**  When connecting to the S3 server, you need to enable `s3ForcePathStyle`​ and set the region to `filen`​.

来源：[https://docs.filen.io/docs/cli/webdav-and-s3-server/](https://docs.filen.io/docs/cli/webdav-and-s3-server/)

关于S3兼容性，参见[https://github.com/FilenCloudDienste/filen-s3](https://github.com/FilenCloudDienste/filen-s3)

‍