---
title: 利用serv00为waline加入异步邮件推送
published: 2024-10-26
updated: 2024-11-23T14:33:13.796+08:00
description: 本文介绍了如何利用 serv00 为 Waline 添加异步邮件推送功能，以提升评论系统的响应速度。
tags: [Waline, serv00, 异步, 邮件推送, 技术]
category: 后端开发
slug: waline-async-mail-push
draft: false
---

# 利用serv00为waline加入异步邮件推送

事情的起因是因为有人在我网站评论了但我没看到，一直过了很久才发现回复，于是就想为我的waline加入一个邮件推送，及时提醒评论区的动态，以及提醒评论者评论的回复。

## waline自带推送的缺点

waline自带的邮件推送会使评论这个过程变得很慢，甚至直接失败。后面在[大佬文章](https://blog.xsot.cn/archives/waline-async-mail.html)里知道Waline 是在评论时直接进行发送邮件的，同步进行发信便会导致评论耗时较长，很影响用户体验，有时评论需要耗时几秒甚至超时。
幸好这个大佬也做出了解决方案，使用异步发信的方式，利用Waline 可以设置一个 Webhook 地址，开发了[这个项目](https://github.com/soxft/waline-async-mail)

## 利用serv00部署项目

秉承着能白嫖绝不付费的理念，我选择了利用serv00来实现这个项目。总体分为三步 ①设置域名邮箱（可选，如果你选择用大厂smtp服务此过程可以省略）②项目本体的运行和保活 ③设置waline的webhook地址并重新部署

### 设置域名邮箱（可选）

这里其实任意一个smtp服务都可以，例如163邮箱。如果想要使用域名邮箱也可以选择其它的提供商，这里选择的是lark，也可以选择使用serv00提供的免费的域名邮箱，这里给一个[大佬的配置教程](https://linux.do/t/topic/97186)
记录下你的SMTP服务器地址，账号和密码备用

### 项目本体的运行和保活

#### 放行端口

根据serv00邮件中给的地址进入面板，放行两个tcp端口，记录下两个端口
![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/iixya/20241026/20241026220849_original.png)
记为端口1和端口2

#### Add a New Website

在侧边栏中选择WWW websites 然后选择Add New Website
![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/iixya/20241026/20241026220606_original.png)


| Key            | Value                                                                             |
| -------------- | --------------------------------------------------------------------------------- |
| Domain         | `你晚点要用的webhook的域名`（也可以把原有的 USERNAME.serv00.net 删掉后重新添加,） |
| Website Type   | Node.js                                                                           |
| Node.js binary | Node.js v22.9.0                                                                   |
| Environment    | Production                                                                        |

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：


| Type                                  | Domain                                                                           |
| ------------------------------------- | -------------------------------------------------------------------------------- |
| Generate Let’s Encrypted certificate | 与刚刚添加的站点域名保持一致（如果是原有的`USERNAME.serv00.net` ，可以省略此步） |

但是注意，由于serv00可能已经处于被墙的状态，这里建议使用托管在cf上的域名，后续我们可以开启cf小黄云拯救被墙

建议此处即开启**cf小黄云**

#### SSH操作

根据邮件给的信息，ssh登入执行

```jsx
devil binexec on && killall -u $(whoami)
```

接着断开 SSH 并重新连接，输入以下命令：

```jsx
#更新go环境
# 创建安装目录
mkdir -p ~/local/soft && cd ~/local/soft
# 下载编译好的 go1.22 的程序包
wget https://dl.google.com/go/go1.22.0.freebsd-amd64.tar.gz
# 解压
tar -xzvf go1.22.0.freebsd-amd64.tar.gz
# 删除压缩文件
rm go1.22.0.freebsd-amd64.tar.gz
# 修改 .profile 文件
echo 'export PATH=~/local/soft/go/bin:$PATH' >> ~/.profile
# 使 .profile 的修改生效
source ~/.profile
# 检查 go 版本
go version
```

下面开始安装waline_mailer，这里选择刚刚新建的网站为安装目录**~/domains/刚刚填入的域名/public_nodejs**
切换至安装目录后开始执行

````jsx
git clone https://github.com/soxft/waline-async-mail && cd waline-async-mail
go get github.com/go-playground/locales
go build -o waline_mailer main.go
cp config.example.yaml config.yaml
chmod +x waline_mailer
````

下面修改config.yaml为你的配置文件，主要修改的有监听端口（端口1）网站名称，redis端口（端口2），主人邮箱，推送方式和smtp配置

修改完成后执行
`nohup redis-server --port 端口2 >/dev/null 2>&1 & cd /home/USERNAME/domains/域名/waline-async-mail && ./waline_mailer &`
如果成功启动了，那么我们就可以关闭程序了，然后进行保活操作

#### 保活

进入 `~/domains/域名/public_nodejs`
删除该目录下的public文件夹，并在public_nodejs文件夹内放置文件app.js
`app.js`的内容如下：

```jsx
const express = require("express");
const app = express();
const port = 3000;
var exec = require("child_process").exec;
const { createProxyMiddleware } = require("http-proxy-middleware");
const path = require('path');
const fs = require('fs');

const currentDir = __dirname;
process.chdir(currentDir);

app.use('/', createProxyMiddleware({
  target: 'http://127.0.0.1:PORT', //改成端口1
  changeOrigin: true,  
  ws: true, 
  onError: (err, req, res) => {
    res.writeHead(500, {
      'Content-Type': 'text/plain',
    });
    res.end('Please wait for a while and try to refresh the page.');
  },
}));

function keep_web_alive() {
    exec("pgrep -laf waline_mailer", function (err, stdout, stderr) { //改成进程名，如：alist
      if (stdout.includes("./waline_mailer")) {  //改成进程实际运行命令，如：alist server
        console.log("web 正在运行");
      } else {
        exec(
          "nohup redis-server --port 端口2 >/dev/null 2>&1 & cd /home/USERNAME/domains/域名/waline-async-mail && nohup ./waline_mailer >/dev/null 2>&1 &", // 这里注意修改为自己的实际信息
          function (err, stdout, stderr) {
            if (err) {
              console.log("保活-调起web-命令行执行错误:" + err);
            } else {
              console.log("保活-调起web-命令行执行成功!");
            }
          }
        );
      }
    });
  }
  setInterval(keep_web_alive, 10 * 1000);

app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```

注意修改文件中所执行的启动命令，
访问域名，刷新几次，看到网站跳转至该项目的github仓库后即为启动成功，我们只需要监控这个域名即可实现对该程序的保活
可以使用以下公共服务对网页进行监控：

1 [cron-job.org](https://console.cron-job.org)

2 [UptimeRobot](https://uptimerobot.com/)

同时，你也可以选择自建 [Uptime-Kuma](https://github.com/louislam/uptime-kuma) 等服务进行监控。

最后加入一个定时对自己ssh连接的命令，防止因为超过三个月未连接而被官方封号
![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/iixya/20241026/20241026220654_original.png)
指令为 `sshpass -p '密码' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt 用户名@地址 "exit"`

### 启用webhook并重新部署

在vercel的waline项目中添加环境变量webhook，值为你设置的webhook的url
然后重新部署项目完成！
![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/iixya/20241026/20241026220702_original.png)

## 至此，部署已经完成

感谢[soxft](https://blog.xsot.cn/)和[saika](https://saika.us.kg/)大佬的异步邮件推送方案和serv00保活方案，接下来体验一个不卡评论的邮件推送！