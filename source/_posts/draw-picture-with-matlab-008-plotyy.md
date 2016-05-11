layout: post
title: matlab 画图（八）： 双坐标绘图
date: 2016-4-29 21:20:22
tags: 
   - Matlab
   - 画图
comments: true
copyright: true
---

![](/img/articles/matlab/matlab-pic-008-plotyy-01.jpg)

<!--more-->
```matlab
clc,clear,close all;
x = 0:0.4:2*pi;
figure1 = figure;
%AX是两个坐标轴的句柄，Ha,Hb是两条线的句柄
[AX,Ha,Hb]=plotyy(x, sin(x), x, exp(x));
% 设置坐标轴颜色
set(AX(1), 'YColor', 'k', 'Box', 'off');
set(AX(2), 'YColor', 'r', 'Box', 'off');
% 设置线条宽度及Marker
set(Ha,'LineWidth',2,'LineStyle','-','Marker','o');
set(Hb,'LineWidth',2,'LineStyle','-','Marker','+');
% 生成legend
HL = legend([Ha,Hb],'$sin(x)$','$e^x$');
get(HL);
set(HL,'Interpreter','latex','Box','off','FontSize',14);
% 设置横纵坐标说明
set(get(AX(1), 'XLabel'), 'String', '$x$','Interpreter', 'latex', 'FontSize',16);
set(get(AX(1), 'YLabel'), 'String', '$sin(x)$','Interpreter', 'latex', 'FontSize',16);
set(get(AX(2), 'YLabel'), 'String', '$e^x$','Interpreter', 'latex', 'FontSize',16);
```

matlab 中双坐标轴画图用的是 `plotyy()` 这个函数。调用形式主要有：
- plotyy(X1,Y1,X2,Y2)
- plotyy(X1,Y1,X2,Y2,function)
- plotyy(X1,Y1,X2,Y2,'function1','function2')
- plotyy(AX1,___)
- [AX,H1,H2] = plotyy(___)



关于 plotyy(X1,Y1,X2,Y2,"function") 中的参数 function，既可以是一个**函数句柄**，也可以是这些字符串 `plot`, `semilogx`, `semilogy`, `loglog`, `stem`， 还可以是matlab定义的函数，比如 `h=function(x,y)`。


plotyy(x1,y1,x2,y2,function)：表示用指定的函数去画图.

ployyy(x1,y1,x2,y2,function1,function2)：用 function(x1,y1) 画左边的图，用 function2(x2,y2) 画右边的图。


例如：
```matlab
x = 0:0.1:10;
y1 = 200*exp(-0.05*x).*sin(x);
y2 = 0.8*exp(-0.5*x).*sin(10*x);

figure
plotyy(x,y1,x,y2,'plot','stem')
```

![](/img/articles/matlab/matlab-pic-008-plotyy-02.jpg)


用 plotyy() 还可以利用 y 轴画多条线。

```matlab
x = linspace(0,10);
y1 = 200*exp(-0.05*x).*sin(x);
y2 = 0.8*exp(-0.5*x).*sin(10*x);
y3 = 0.2*exp(-0.5*x).*sin(10*x);

figure
[hAx,hLine1,hLine2] = plotyy(x,y1,[x',x'],[y2',y3']);//用左边的y轴画一组数据，用右边的y轴画两组数据
```

![](/img/articles/matlab/matlab-pic-008-plotyy-03.jpg)


画出两个y轴还有一种方法就是利用 `yyaxis` 函数，这里就不再介绍了。


