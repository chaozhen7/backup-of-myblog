layout: post
title: matlab 画图（六）： 横向柱状图
date: 2016-4-20 16:20:22
tags: 
   - Matlab
   - 画图
comments: true
copyright: true
---

![](/img/articles/matlab/matlab-pic-006-hori-bar-001.jpg)

<!--more-->

```matlab
% Create the data for the temperatures and months
temperatures = [40.5 48.3 56.2 65.3 73.9 69.9 71.1 59.5 48.7 35.3 31.7 30.0];
months = {'Dec', 'Nov', 'Oct', 'Sep', 'Aug', 'Jul', 'Jun', 'May', 'Apr', 'Mar', 'Feb', 'Jan'};

% Plot the temperatures on a horizontal bar chart
figure
barh(temperatures)

% Set the axis limits
axis([0 80 0 13])

% Add a title
title('Boston Monthly Average Temperature - 2001')

% Change the Y axis tick labels to use the months
set(gca, 'YTick', 1:12)
set(gca, 'YTickLabel', months)
```

画垂直柱状图图用的是 bar() 函数，画水平柱状图时用的是 barh() 函数。

barh() 跟 bar() 在形式上和调用方式上基本相似，也有几种不同调用方式：`barh(Y)`, `barh(x,	Y)`,  `barh(_,width)`, `barh(_,style)`, `barh(_,bar_color)`, `barh(_,name,value)`, `barh(axes_handle,_)`, `h=barh(_)` 

垂直柱状图可以实现的样式，水平柱状图基本都可以实现，实现方法类似，参考：[matlab 画图（五）： 垂直柱状图](../../../../2016/04/17/draw-picture-with-matlab-005-vertical-bar/)