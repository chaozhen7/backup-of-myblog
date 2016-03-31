layout: post
title: 程序员得长多高
date: 2016-2-26 23:20:22
tags: 
	- 随笔
	- Python
	- 爬虫
comments: true
copyright: true
---

首先来说说写这篇文章的初衷吧，相信程序员们对[伯乐在线](http://www.jobbole.com/)这个网站一点都不陌生， 今天无意中发现了伯乐在线居然有一个 [面向对象](http://date.jobbole.com/) 栏目，其实说白了就是相亲吧。

关于面向对象栏目的介绍，来看看官方的说法：

<!--more-->

> 面向对象是一个专门为IT单身男女服务的征婚传播平台。
面向对象作为伯乐在线的一个频道，通过这里发布的征友信息，可以传播到数百万的微博微信粉丝和网站读者。

面向对象有个规则就是：只允许女性朋友在这里发帖，把个人信息发给管理员，管理员在这里登出来，所以，哪个女的想找个程序员男朋友就可以来这里了。因为除了程序员基本没人会关注这个网站。

好了，我并不是来做广告的，下面进入正题好了。今天无意中闯入到了这个所谓的相亲栏目，映入眼帘的是如下一幕：

![](/img/articles/height-of-code1.jpg)

出于好奇心，随便点开了几个链接，里面长这样子：

![](/img/articles/height-of-code2.jpg)

随便打开了几个基本都是这种格式，映像比较深刻的就是大多数人在**最低要求**这一栏都有身高方面的要求。


出于好奇和闲得蛋疼，索性用 python 写了一个爬虫程序，把所有的这种征对象的帖子都爬出来，然后再分析大家的最低要求，看看有多少人对身高有要求，要求有多高。于是乎开始工作了。

由于这个栏目下的旧贴子会因为某些原因（已经找到对象或者不想找了等原因）会被删除，新的帖子随着时间的推移也会加进来，所以最终的数据只爬到了截止到目前为止的150+个贴。

首先把贴子中的对另一半的最低要求这一栏提取出来了，截个图分享下：

![](/img/articles/height-of-code3.jpg)

下一步的工作就是提取出其中关于身高的敏感信息了，对于语言表述的差异性，表达身高的方式有很多种，比如身高172以上，173cm以上，170+，身高不低于175，身高>175,等等，所以从中提取身高信息的要求还是蛮高的，通过大量数据的分析，总结出了十几种的表达模式，在150+的数据里面提炼出了70+的身高信息，截图如下：

![](/img/articles/height-of-code4.jpg)

这些都是最低身高要求啊。说多了都是泪。

最后把这些身高数据进行分析。得出如下结论：

**有效的征友贴一共有 152 个。其中在最低要求一栏对身高提出最低要求的一共有 79 人（可能统计不完全）。最低身高要求的分布如下：**

![](/img/articles/height-of-code5.jpg)

**对所有身高进行加权平均，得到平均最低身高要求是：173.06cm**

哎，写到这里写不下去了，我去哭一会了。

ps：纯属娱乐，不喜勿喷。


## 代码： ##
```python
# -*- coding: utf-8 -*- 
import requests
import re
import os
import sys
from bs4 import BeautifulSoup

pageNum = 10;  # 所有帖子总共10页
urlsFile = os.path.join(sys.path[0],'urls.txt');# 保存帖子url的本地路径
infoNum = 0; #有效信息的总条数
num = 0;   #包含敏感信息的总条数

# 获取所有帖子的url
def getUrls():
    if(os.path.exists(urlsFile)):
        return getUrlsFromFile();#如果本地存在数据直接从本地读取
    urlList = list();
    for i in range(1,pageNum):
        url = "http://date.jobbole.com/page/"+str(i)+"/?sort=latest";
        html = requests.get(url);
        pattern = "href=\"(.*?)\">.*?</a><label class=\"hide-on-480 small-domain-url\"></label>";
        result = re.findall(pattern,html.text);
        urlList = urlList + result
    saveUrls(urlList); #保存到本地
    return urlList;

#本地读取所有帖子的url
def getUrlsFromFile():
    urlList = list();
    f = open("urls.txt","r");
    for line in f.readlines():
        urlList.append(line);
    f.close();
    return urlList;

#将帖子信息存入本地文件中
def saveUrls(urlList):
    f = open("urls.txt","w")
    f.write('%s' % '\n'.join(urlList));

#查询该帖子下的内容
def viewUrl(url):
    global infoNum;
    result = "";
    html = requests.get(url);
    soup = BeautifulSoup(html.text,"lxml");
    info = soup.find("div",{"class":"p-entry"});
    info = str(info);
    #pattern = "身高：(.*?)<br/>";  #发帖者身高
    #r = re.findall(pattern,info);
    #if(len(r)):
        #print(r[0]);
    pattern = "对另一半的最低要求是：(.*?)<br/>";
    r = re.findall(pattern,info);
    if(len(r)>0 and (r[0]!="")):
        infoNum  = infoNum+ 1;
        result = r[0];
        isAboutH(result);

#是否有涉及身高的敏感信息
def isAboutH(info):
    global num;
    key = ["身高(\d{3})","身高：(\d{3})","(\d{3})以上","(\d{3})cm以上",
           "(\d{3})\+","(\d{3})cm\+","(\d{3})CM\+","身高不低于(\d{3})cm",
           "身高≥(\d{3})","身高&gt;(\d{3})","身高&gt;=(\d{3})","身高不.*?低于(\d{3})",
           "(\d{3})左右","身高≧(\d{3})","(\d{3})cm左右","(\d{3})CM左右"];
    #print(info)
    for p in key:
        r = re.findall(p,info);
        if(len(r)):
            num = num + 1;
            f = open("infoH.txt","a");
            f.write(str(r[0])+"\n");
            f.close();
            #print(r[0])
            break;
    print("\n")
    
if __name__=='__main__':
    urlList = getUrls();
    num = 0;
    for url in urlList:
        viewUrl(url);
    print(infoNum)
    print(num);
    
```