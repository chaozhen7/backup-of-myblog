layout: post
title: matlab 画图（七）： 扇形图
date: 2016-4-24 21:20:22
tags: 
   - Matlab
   - 画图
comments: true
copyright: true
toc: true
---

![](/img/articles/matlab/matlab-pic-007-pie-01.jpg)

<!--more-->

```matlab
X = 1:3;
labels = {'Taxes','Expenses','Profit'};
pie(X,labels)
```



在 matlab 中画 2D 的扇形图用的是 `pie()` 这个函数。用 `pie(X)` 画图，如果 X 向量中值的和大于1，那么会把 X 向量归一化后来画图。如果 X 向量中值的和小于1，那么直接根据这个值来画图，这个时候画出来的图可能不是一个完整的圆，后面会给出示例。

pie()的调用形式可以有如下几种：
- pie(X)
- pie(X,explode)
- pie(X,labels)
- pie(X,explode,labels)
- pie(ax,___)
- p = pie(___)

下面就这几种形式给出示例。

## 1. pie(X,explode)##

这种扇形图主要是突出几个部分。

```matlab
X = [1 3 0.5 2.5 2];
explode = [0 1 0 1 0];%突出第2和第4个部分
pie(X,explode)
```
![](/img/articles/matlab/matlab-pic-007-pie-02.jpg)

## 2. pie(X,labels) ##

这种扇形图主要是可以自定义设置 labels

```matlab
X = 1:3;
labels = {'Taxes','Expenses','Profit'};
pie(X,labels)
```
![](/img/articles/matlab/matlab-pic-007-pie-03.jpg)

## 3. pie(X,explode,labels) ##

前面两种的一个结合。

## 4. pie(ax,) ##

可以利用这种方式在一个图上画多个扇形图。

```matlab
X = [0.2 0.4 0.4];
labels = {'Taxes','Expenses','Profit'};
ax1 = subplot(1,2,1);
pie(ax1,X,labels)
title(ax1,'2012');

Y = [0.24 0.46 0.3];
ax2 = subplot(1,2,2);
pie(ax2,Y,labels)
title(ax2,'2013');
```
![](/img/articles/matlab/matlab-pic-007-pie-04.jpg)

## 5. h=pie() ##

这种方式主要是可以获得扇形图的句柄，然后利用句柄改变图形属性。

具体可以参见 matlab 帮助文档。`help pie`

## 6. partial pie char ##
```matlab
% X的和小于1,所以是不完整的
X = [0.19 0.22 0.41];
pie(X)
```
![](/img/articles/matlab/matlab-pic-007-pie-05.jpg)








