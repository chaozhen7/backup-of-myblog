layout: post
title: Hexo 主题优化
date: 2016-2-14 14:20:22
tags: 
	- Hexo
comments: true
toc: true
copyright: true
---

**写在前面**

之前写过一些关于 Hexo 主题优化的文章，今天在这里把之前写的和新的东西做一个汇总，这样更加直观和便于查找。

<!--more-->

## **1. 添加访问人数统计功能** ##

内容较长，请[点击这里](../../../../2015/11/30/add-counter-to-hexo/)

## **2. 给文章添加目录** ##

内容较长，请[点击这里](../../../../2015/12/13/add-toc-to-hexo/)


## **3. 添加 RSS** ##

在博客文件夹根目录下运行：
`npm install hexo-generator-feed --save`
注意，后面的参数--save绝对不能省，否则该插件信息不会写入package.json。
运行 `hexo clean`，`hexo g` 后查看 public 文件夹，可以看到atom.xml文件。

## **4. 添加 sitemap** ##

在博客文件夹根目录下运行：
`npm install hexo-generator-sitemap --save`
运行 `hexo clean`，`hexo g` 后查看public文件夹，可以看到sitemap.xml文件。
sitemap的初衷是给搜索引擎看的，为了提高搜索引擎对自己站点的收录效果，我们最好手动到 google 和百度等搜索引擎提交sitemap.xml。

然后在 Hexo 根目录下的 _config.yml 里配置一下
```
sitemap:
   path: sitemap.xml
```
`path` 表示 Sitemap 的路径. 默认为 `sitemap.xml`.
对于国内用户还需要安装插件 `hexo-generator-baidu-sitemap`, 顾名思义是为百度量身打造的.
```
$ npm install hexo-generator-baidu-sitemap --save
```
然后在 Hexo 根目录下的 _config.yml 里配置一下
```
baidusitemap:
    path: baidusitemap.xml
```
同上.

## 5. 百度链接收录自动提交 ##

百度站长平台还提供了一个自动推送链接的功能，自动推送是百度站长平台为提高站点新增网页发现速度推出的工具，安装自动推送JS代码的网页，在页面被访问时，页面URL将立即被推送给百度。详情[请点击这里](http://zhanzhang.baidu.com/linksubmit/index?site=http://www.cighao.com/)。

以 yilia 主题为例：

首先我们可以在 `yilia\layout\_partial\post` 文件夹下新建一个 `baidu_js_push.ejs` 文件，在其中写入百度自动推送代码，代码可以在官网上复制。

然后我们在 `yilia\layout\_partial\left-col.ejs` 文件的最前面加入如下代码：
```html
<div class="baidu_js_push">
	<%- partial('post/baidu_js_push') %>
</div>
```
完成！

## **6. 添加返回顶部按钮** ##

以yilia主题为例：

**step1: 添加 html 代码**

打开文件夹 `/themes/yilia/layout/_partial` 在此文件夹下，新建文件 `totop.ejs`，并向其中加入如下代码：
```html
<div>
   <a href="javascript:;" id="totop" title="回到顶部"></a> 
</div>
```
**step2: 添加 js 代码**

打开文件夹 `/themes/yilia/source/js`，新建文件 `totop.js`，向其中写入回到顶部的 js 代码。

**step3: 添加样式**

打开文件夹 `/themes/yilia/source/css/_partial`，新建文件 `totop.styl`，向其中写入回到顶部按钮的css样式代码。

添加引用： 打开 /themes/yilia/source/css 文件加下的 style.styl 文件，在最后加上： `@import '_partial/totop' `

**step4: 添加文件引用**

打开文件 `/themes/yilia/layout/_partial/after_footer.ejs`，在文件的末尾添加以下两行代码：
```
<%- partial('totop') %>
<script src="<%- config.root %>js/totop.js"></script>
```

**step5: 添加按钮图片**

将按钮图片复制到 `/themes/Yilia/source/img` 目录下，文件名为`totop.png`，页面足够长时，就看到按钮出现了。


具体实现的代码和按钮图片可以参考[这篇文章](../../../../2016/01/09/toTop-by-javascript/)


## **7. 添加打赏功能** ##

内容较长，请[点击这里](../../../../2016/02/23/add-donate-to-hexo/)



## 8. 添加版权信息 ##

[戳这里](../../../../2016/03/01/add-copyright-to-hexo/)

</br>
注：此文章会不定期更新!