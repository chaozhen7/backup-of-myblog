layout: post
title: Java Web 中 Session 的钝化和活化
date: 2015-12-05 17:20:22
tags: 
	- Java Web
comments: true
---
对于 Session 中的对象，主要有两组状态：(1)绑定和解除绑定，(2)钝化和活化。

Session的钝化机制: 本质就在于把服务器中不经常使用的 Session 对象暂时序列化到系统文件系统或是数据库系统中，当被使用时反序列化到内存中，整个过程由服务器自动完成。

<!--more-->

Tomcat中两种Session钝化管理器(由SessionManager管理)：
(1)org.apache.catalina.session.StandardManager管理器
- Tomcat 服务器被关闭或者重启时，tomcat 服务器会将当前内存中的 Session 对象钝化到服务器文件系统中；
- 另一种情况是 Web 应用程序被重新加载时，内存中的 Session 对象也会被钝化到服务器的文件系统中；
- 钝化后的文件被保存到: Tomcat 安装路径 /work/Catalina/hostname/applicationname/SESSION.ser 中

(2)org.apache.catalina.session.Persistentmanager管理器
- 前面的两种情况与(1)中的前两种情况相似
- 扩充了第三种情况，可以配置驻留内存的 Session 对象数目，将不常用的 Session 对象保存到文件系统或者数据库，当用时再重新加载。
- 默认情况下，tomcat 提供两个钝化驱动类，*org.apache.Catalina.FileStore* 和 *org.apache.Catalina.JDBCStore* 。


