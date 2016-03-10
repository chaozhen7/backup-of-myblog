layout: post
title: mysql windows 下zip安装
date: 2015-3-29 13:12:22
tags: 
	- Mysql
comments: true  
---
## step1
将下载好的mysql解压，解压在任意路径下均可，这里是解压到D盘根目录下 D:\mysql-5.6.23-winx64
<!--more-->
## step2
打开目录 D:\mysql-5.6.23-winx64，修改其中的 my-default.ini文件（可将其重命名为my.ini）修改如下:
```
# These are commonly set, remove the # and set as required.
basedir = "D:\mysql-5.6.23-winx64"
datadir = "D:\mysql-5.6.23-winx64\data"
# port = .....
# server_id = .....
```
其中D:\mysql-5.6.23-winx64  为mysql解压文件的目录，根据个人情况而定
## step3
添加环境变量。计算机->属性->高级系统设置->环境变量->系统变量选择path 后选择编辑，将mysql解压后的bin子目录路径添加到path环境变量中，在这里添加的是D:\mysql-5.6.23-winx64\bin
## step4
进入到mysql安装目录下的bin子目录（在这里是D:\mysql-5.6.23-winx64\bin），按住shift ，点击鼠标右键后选择在此处打开命令窗口(确保是在管理员权限下运行命令窗口)
## step5
在命令窗口下输入 mysqld --install 命令后回车如果不出意外这个时候会看到安装成功的消息。如果这个时候显示的是：Install/Remove of the Service Denied! 表示在第4步中打开dos窗口时并没有获得管理员权限，解决办法如下：开始->附件,找到 命令提示符 右键以管理员的身份运行，这个时候在命令提示行下输入下面两条指令:
``` bash
	cd d:\mysql-5.6.23-winx64\bin
	d:
```
即可到达step4相同的目的。然后输入mysqld --install  命令就可以成功安装
## step6
输入 net start mysql 启动mysql服务     
## step7
输入mysql -u root -p  即可登录mysql，第一次登录密码直接按回车即可