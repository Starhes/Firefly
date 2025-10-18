---
title: 机场秒变代理池-glider
published: 2024-08-25
updated: 2024-08-25T18:20:50.962+08:00
description: 介绍如何将机场（V2Ray / Shadowsocks）秒变代理池，利用 glider 提升速度和稳定性。
tags: [代理池, glider, 网络, V2Ray, Shadowsocks]
category: 网络工具
slug: glider-proxy-pool
draft: false
---

项目地址：

[https://github.com/nadoo/glider](https://github.com/nadoo/glider)

![nadoo/glider](https://cdn-ak.f.st-hatena.com/images/fotolife/m/maohais/20240825/20240825192221.png)

# 安装

①下载系统对应的版本

②解压下载到的安装包

③根据配置文件模板创建配置文件

此处由linux_amd64举例

```raw
wget https://github.com/nadoo/glider/releases/download/v0.16.3/glider_0.16.3_linux_amd64.tar.gz
tar xvf glider_0.16.3_linux_amd64.tar.gz 
cd glider_0.16.3_linux_amd64/
cp config/examples/4.multiple_forwarders/glider.conf ./
cat glider.conf
```

# 配置节点

我们通过glider提供的模板配置文件进行修改。

## 获取符合glider格式的节点内容

### trojan的机场节点

要求格式为  forward=trojan://password@domain

我们可以使用

```raw
curl -s http://你的机场订阅链接 | base64 -d | sed 's/^/forward=&/g'
```

得到

### vmess和ss格式的节点

我在github找到了这个库https://github.com/Rain-kl/glider_guid41asd4asd

[glider_guid41asd4asd-master.zip](https://1drv.ms/u/c/d2c4359881890683/ER2YtoDLjDFOkbrli1CeiTcBAUuFfRtCUj1-ywp-QH0Jqg?e=M5hHGX)

①安装依赖

②将clash内配置文件全部复制到config.yml内

③运行订阅转换.py

这样我们就得到glider所需要的 forward=xxx格式的节点啦   这里注意由这个py转换出来的列表vmess和ss格式之间会有一道分界线，记得删去哦

### 修改配置文件

然后我们就可以将这些节点填到配置文件里了

```raw
# Verbose mode, print logs
verbose=True

listen=:8443

# 机场节点放在这里
forward=trojan://****
forward=vmess://****
forward=ss://****

# Round Robin mode: rr
# High Availability mode: ha
strategy=rr

# forwarder health check
check=http://www.msftconnecttest.com/connecttest.txt#expect=200

# check interval(seconds)
checkinterval=3000
```

同时listen后面是我们监听的端口，修改好后，我们保存

## 运行

### 直接启动

```raw
./glider -config ./glider.conf
```

运行它，成功启动了

![](https://cdn-ak.f.st-hatena.com/images/fotolife/m/maohais/20240825/20240825192426.png)

![](https://cdn-ak.f.st-hatena.com/images/fotolife/m/maohais/20240825/20240825192500.png)

### 后台运行

我选择使用screen，当然也还可以写成service，用systemctl调用

首先确保你已经安装了screen

```raw
#centos系统
yum install screen -y

#debian/ubuntu系统
apt install screen -y
```

然后创建一个新的窗口

```raw
screen -R glider
```

接下来在新窗口中启动项目

```raw
./glider -config ./glider.conf
```

最后CTRL+A+D退出窗口

# 完结撒花*★,°*:.☆(￣▽￣)/$:*.°★* 。