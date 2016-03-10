layout: post
title: python 中安装 requests 模块
date: 2015-12-05 17:20:22
tags: 
	- Python
	- 爬虫
comments: true
---

在 windows 系统下，只需要输入命令 `pip install requests` ，即可安装。

在 linux 系统下，只需要输入命令 `sudo  pip install requests` ，即可安装。

注：关于 python 第三方库的安装最好少使用 easy_install，因为 easy_install 只能安装不能卸载，如果要卸载需要进入到 python 的安装目录下面的 lib 的文件夹下手动删除对应的模块内容。所以建议多用 pip 的方式安装，安装时，用 `pip install + 模块名称` 命令来安装，卸载时，用 `pip uninstall +模块名称` 命令来删除。

<!--more-->

由于在国内使用 pip 或者 easy_install 安装时经常会撞墙，下面着重介绍另外一种安装方法。


**step1: 下载requests**

打开这个网址， [http://www.lfd.uci.edu/~gohlke/pythonlibs](http://www.lfd.uci.edu/~gohlke/pythonlibs) 在这个网站上面有很多 python 的第三方库文件，我们按 ctrl+f 搜索很容易找到 requests 。点击那个 `.whl` 文件然后下载下来。

**step2: 添加到 lib 中**

将 .whl 文件下载下来后，将文件重命名，将后缀名从 .whl 改为 .zip ，然后解压文件，我们可以得到两个文件夹，我们将第一个文件夹，也就是 `requests` 文件夹复制到 python 的安装目录下的 `lib` 目录下。

**step3: 测试**

到这里，requests 已经安装完毕，我们可以输入 `import requests` 命令来试试是否安装成功，没有报错，说明 requests 已经成功安装了。


最后，这种方法不仅适合 requests 模块的安装，其他模块同样适合，可以下载解压后将相应的内容直接复制到 `lib` 目录下。