layout: post
title: matlab 画图（四）： 句柄作图示例
date: 2016-4-16 21:20:22
tags: 
   - Matlab
   - 画图
comments: true
---

[前面](../../../../2016/04/14/draw-picture-with-matlab-003-introduction-of-handle-graphics/)介绍了句柄作图，现在举个例子详细说明下。

先看一个简单的例子。

```MATLAB
x = 0:0.1:pi;
plot(x, sin(x), 'o-', x, cos(x), ':+');
```

<!--more-->

运行完原始代码后，在图形窗口点击File -> Generate M-File…，就可以查看MATLAB 自动生成的代码。
```matlab
function createfigure(X1, YMatrix1)
%CREATEFIGURE(X1, YMATRIX1)
%  X1:  x 数据的矢量
%  YMATRIX1:  y 数据的矩阵

%  由 MATLAB 于 16-Apr-2016 20:35:33 自动生成

% 创建 figure
figure1 = figure;

% 创建 axes
axes1 = axes('Parent',figure1);
box(axes1,'on');
hold(axes1,'all');

% 使用 plot 的矩阵输入创建多行
plot1 = plot(X1,YMatrix1);
set(plot1(1),'Marker','o');
set(plot1(2),'Marker','+','LineStyle',':');
```
执行完一个绘图命令后， MATLAB 背后做了更多具体的工作，可以看出MATLAB 完整
的绘制一个图形是分为三步的：
- 1）建立图形窗口figure1 = figure('PaperSize',[20.98 29.68]);
- 2）在图形窗口上建立坐标系axes('Parent',figure1);
- 3）在坐标系上绘制图形plot1 = plot(X1,YMatrix1);

进一步，如果把图形窗口，坐标系都看成容器的话，则坐标系为图形窗口（作为'Parent'）的子容器，同时又为线条（作为'Children'）的父容器。图形窗口、坐标系以及线条对象都有对应的入口地址，称之为函数句柄（图形窗口为整数，坐标轴及线条对象为双精度浮点数double）。

要控制绘制出图形的特征，就要获得这些句柄，常见的函数句柄如下：

| 句柄 |  描述 |
|:-----:|:----|
|  gco |  当前对象的句柄（get current object）|
|  gcf |  当前图形的句柄（get current figure）|
|  gca |  当前坐标轴的句柄（get current axes）|
|  h = plot(…)  |   a column vector of handles to lineseries objects, one handle per line|
|  new_handle = copyobj(handle,parent) |  拷贝由句柄handle 指定的对象到父容器parent 中，并返回这个新生成对象的句柄new_handle|


每个句柄中都包含很多参数-值对，要了解这些参数-值对的话，可以先生成一个句柄，然后使用get 函数查看，通过find 函数查找特定的参数，并通过set 函数修改这些参数的值。看例子：


```matlab
clc,clear,close all;
x = 0:0.4:2*pi;
figure1 = figure(1);
h = plot(x, exp(x));
% 查看参数-值对
get(gca);
% 更改Y 轴颜色
set(gca, 'YColor', 'red');
% YLabel 为gca 的子句柄, 查看其信息
get(get(gca, 'YLabel'));
% 更改YLabel
set(get(gca, 'YLabel'), 'String', '$y = e^x$', ...
'Interpreter', 'latex', 'FontSize', 16);
% 函数xlabel 能达到同样的效果
xlabel('$x$', 'Interpreter', 'latex', 'FontSize', 16);
% 获取线条对象的参数信息
get(h);
% 对一些常见的特征进行修改
% set(findobj('Type','line'),'LineWidth', 2)
set(h, 'Marker', 'o', 'MarkerFaceColor', 'green', ...
'MarkerEdgeColor', 'blue', 'Markersize', 6, ...
'LineStyle', '-', 'LineWidth', 2);
```

![](/img/articles/matlab/matlab-pic-004.jpg)


由上面的代码 get(gca) 可以看出坐标轴既有 Parent 又有 Children，其值分别与 figure1 和 h 相等。

此外如果绘制少量图形，嫌使用句柄麻烦，可以手工编辑或者使用 inspect 函数，直接跳出编辑的GUI。

不知道怎样用的可以先help inspect 一下。编辑后再用 MATLAB 自动生成一下代码，看官方是怎么设置的。