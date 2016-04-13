layout: post
title: matlab 画图（一）： 线条样式设计
date: 2016-4-11 19:20:22
tags: 
   - Matlab
   - 画图
comments: true
copyright: true
---

![](/img/articles/matlab/matlab-pic-001.jpg)

<!--more-->

matlab 中可以用利用线条的句柄来设置线条的各种属性。上图的代码如下：
```matlab
x = [-2*pi:0.01:2*pi];
y1 = sin(x);
y2 = cos(x);

figure;  % 打开一个画板

% 画两条线，返回的是这两条线的句柄,h是一个包含两个句柄的数组
h = plot(x,y1,x,y2);  

% 根据句柄设置线条属性
set(h(1),'LineWidth',2)
% set(h(2),'Marker','*')
set(h(1),'Color',[0.6 0 1])
% set(h(1),'LineStyle',':')
% set(h(2),'MarkerFaceColor',[0 0 1],'MarkerEdgeColor',[0.6 0.6 0.6])
% set(h(2),'MarkerSize',2)

```

在 matlab 中线条的属性主要有：
- **Color:** 颜色
- **LineStyle:** 线型
- **LineWidth:** 线宽
- **Marker:** 标记点的形状
- **MarkerEdgeColor:** 标记点填充颜色
- **MarkerFaceColor:** 标记点边缘颜色
- **MarkerSize:** 标记点大小

以上的这些属性都可以通过 set() 函数来设置，方法如上面代码所示那样。

这些属性的取值如下：

### **Color** ###

常用的线条颜色有八种，除了这八种之外还可以用 RGB 来自己配色，示例见上面的代码。常见的八种如下：

|RGB Vector|Short Name|Long Name|
|:---:|:---:|:---:|
| [1 1 0]  | 'y' | 'yellow' |
| [1 0 1]  | 'm' | 'magenta' |
| [0 1 1]  | 'c' | 'cyan' |
| [1 0 0]  | 'r' | 'red' |
| [0 1 0]  | 'g' | 'green' |
| [0 0 1]  | 'b' | 'blue'|
| [1 1 1]  | 'w' | 'white' |
| [0 0 0]  | 'k' | 'black' |

### **LineStyle** ###

线型的取值如下：

| Specifier | Line Style|
|:----:|:---:|
| '-'  | 实线(默认的线型) |
| '--' | 虚线  |
| ':'  | 点线 |
| '-.' | 点横线 |
| 'none'| 没有线 |

### **LineWidth** ###

设置线宽。


### **Marker** ###

标记点的形状,取值如下：

| Specifier | Marker Symbol|
|:----:|:---:|
| '+'  |       加号   |
| 'o'   |        圆圈   |
| '*'    |       星号   |
| '.'   |       实心点   |
| 'x'    |      叉号   |
| 'square' or 's'   |      正方形   |
| 'diamond' or 'd'   |       钻石形   |
| '^'   |       上三角形  v
| 'v'    |      下三角形   |
| '>'    |     右三角形   |
| '<'    |     左三角形   |
| 'pentagram' or 'p'     |    五角星形   |
| 'hexagram' or 'h'     |    六角星形  |
| 'none'   |  没有标记 |


### **MarkerEdgeColor** ###
### **MarkerFaceColor** ###
### **MarkerSize** ###

**与上面的相同，不一一介绍**