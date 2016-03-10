layout: post
title: Ngrok： 一个提供内网到外网映射的工具
date: 2015-12-18 17:20:22
tags: 
	- Ngrok
comments: true
---

**什么是 Ngrok呢？**

ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放。

**应用场景**

在本机开发好的网站想让客户测试不再需要上传到服务器上面，使用 ngrok 轻松解决，仅需一条命令；微信开发也不需要再上传到服务器，本站服务器帮您解决80端口问题，实现微信服务器直接访问到您本机的web服务。

　<!--more-->

**使用方式**

鉴于 [ngrok的官网](https://ngrok.com/) 已经被墙了，已经被墙了怎么办呢，相信国人的能力总是无穷的，哪里有墙哪里就有梯子。下面一步一步教大家怎么做。

下载工具就不用说了，鉴于官网被墙了，[点击这里](https://yunpan.cn/c3ISXbMPJU9WE)下载吧(访问密码 **f2c7**)。

下载解压后有两个文件 ngrok.exe 和 ngrok.cfg ，ngrok.cfg 是一个配置文件，主要是用来解决国内被墙的问题。

我们在解压后的目录下打开命令行输入下面的命令就可以了：
```
ngrok -config ngrok.cfg -subdomain example 8080
```
这里的 example 是自定义的二级域名，你的必须跟别人的是不一样的，否则是不行的。 输入命令后如果看到了这样的消息：
> http://example.ngrok.cc -> 127.0.0.1:8080
> https://example.ngrok.cc -> 127.0.0.1:8080

那么就说明成功了。

下面只要在浏览器输入：example.ngrok.cc/项目地址  就可以了。

关于 ngrok 更多的强大功能请看 ngrok 官方文档。

最后给出一个国内提供 ngrok 服务的网站，[点击这里](http://www.ngrok.cc/)。 