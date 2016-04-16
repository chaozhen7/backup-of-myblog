layout: post
title: matlab 画图（二）： 基本作图技巧
date: 2016-4-13 19:20:22
tags: 
   - Matlab
   - 画图
comments: true
copyright: true
toc: true
---
## **1. 创建图** ##

matlab 中创建图是使用 plot() 函数。

根据输入的不同，plot 函数有不同的窗体。如果y是向量的形式，plot(y)则在y对应的轴上作出一个分段线状图。如果指定要求含两个向量时，则plot(x,y)作出一个y相对于x的图表。

```matlab
y=[1 2 10 4 5 3 2];
plot(y)
```

<!--more-->

![](/img/articles/matlab/matlab-pic-002-1.jpg)

```matlab
x = 0:pi/100:2*pi;
y = sin(x);
plot(x,y)
%现在给轴加上标签和标题，用\pi作符号。
xlabel('x = 0:2\pi')
ylabel('Sine of x')
title('Plot of the Sine Function','FontSize',12)
```

![](/img/articles/matlab/matlab-pic-002-2.jpg)

## **2. 一个图表中的多重数据集** ##

一个函数作图命令 plot 使不同的 (x-y) 变元函数生成不同的函数图象。MATLAB 自动地通过预设地颜色库来区别不同的函数（也可用户自设）。例如，以下是三个x的相关函数的图象，每条曲线都由各自不同的颜色加以区分。

```matlab
y2 = sin(x-.25);
y3 = sin(x-.5);
plot(x,y,x,y2,x,y3)
% legend命令提供一种简易方式来辨别不同的函数作图
legend('sin(x)','sin(x-.25)','sin(x-.5)')
```

![](/img/articles/matlab/matlab-pic-002-3.jpg)

## **3. 指定线型和颜色以及点的标记** ##

在使用plot命令来为数据作图时，你可以自定义颜色，线型和标记（比如加号和圆圈）。
```matlab
plot(x,y,'color_style_marker')
```
color_style_marker是一个由一到四个字符构成的字符串（用单引号括起来），用以定义颜色，线型和标记形式。 

详细内容请参考之前的一篇博文[matlab 画图（一）： 线条样式设计](../../../../2016/04/11/draw-picture-with-matlab-001-line-style/)


## **4. 虚数与复数数据** ##
当语句是作(plot)复变量时,虚部是忽略的，除非作图给出一个单复变元。对于这种特殊情况，有一捷径的命令来画出实部以及与之对应的虚部，
```matlab
plot(Z)
```
其中Z是复向量或矩阵，其等价于
```matlab
plot(real(Z),imag(Z))
```
例如,
```matlab
t = 0:pi/10:2*pi;
plot(exp(i*t),'-o')
axis equal
```
作出一个二十边形并用小圆标记顶点，命令axis equal使x轴与y轴的单位长度等长，这使作出的图更象一个圆。

![](/img/articles/matlab/matlab-pic-002-4.jpg)

## **5. 在现有图中添加图** ##
hold 命令让你能在现有图中添加图，输入
```matlab
hold on
```
当你输入另一个作图命令时，MATLAB不会替换现有图；而是把新数据图添加到现有图中，必要时会改变坐标轴的标尺。  

例如，以下语句先作一个peaks 函数的轮廓图，并在同一个函数图象中添加伪彩色图。

```matlab
[x,y,z] = peaks;
% 画出来的图是没有色彩的，黑白的
contour(x,y,z,20,'k')
% 添加上颜色
hold on
pcolor(x,y,z)
shading interp
hold off
```
hold on 使命令伪彩色图与轮廓图在同一图象中相结合。

`contour(x,y,z)`: 等高线绘制函数，z是xy平面的高度

`pcolor(x,y,z)`: 在网格 xy 中填充颜色 z

`shading interp`: 使色彩平滑过渡

![](/img/articles/matlab/matlab-pic-002-5.jpg)

## **6. Figure窗口 ** ##

如果屏幕上还没有 figure 窗口，作图函数会自动打开一个新的 figure 窗口。如果figure窗口已经存在，MATLAB 会用它来输出图象。如果已有多个 figure 窗口，MATLAB 会在指定的“当前”窗口作图。 

要使已有的窗口成为当前窗口，可以用鼠标点击该窗口或者输入: `figure(n)`, 其中 n 是标题栏中的窗口号。结果会显示在该窗口。 

要打开一个新窗口并使它成为当前窗口，则输入: `figure`

## **7. 同一Figure中作多幅图 ** ##

用 subplot 命令可以在同一窗口中作多幅图或把它们打印到同一纸上。输入
```matlab
subplot(m,n,p)
```

把 figure 窗口分成 m*n 个子区域及选择第 p 个区域为当前图。所作图是从 figure 窗口的顶行开始标号，然后第 2 行，依次类推。例如，以下语句在 figure 窗口的 4 个不同子区域分别作图。
```matlab
t = 0:pi/10:2*pi;
%函数返回圆柱体x，y，z轴的坐标值，圆柱体沿其周长有20个等距分布的点
[X,Y,Z] = cylinder(4*cos(t));
subplot(2,2,1); mesh(X)
subplot(2,2,2); mesh(Y)
subplot(2,2,3); mesh(Z)
subplot(2,2,4); mesh(X,Y,Z)
```

`mesh(z)`: mesh(z)的作用是生成一张曲面，x坐标的取值从1取到m,间距为1，y坐标的取值是从1取到n构成的网络节点，曲面的高度就是z矩阵里面元素值。

![](/img/articles/matlab/matlab-pic-002-6.jpg)



## **8. 轴的控制** ##

`axis` 命令可以规定图象的缩放比例，方位，和纵横比，可以交互的使用指令进行操作。

**设置轴的范围**

默认时，MATLAB可以根据数值的最大值和最小值决定合适的范围，用 axis 命令可以自己定义数值的标尺范围： `axis([xmin xmax ymin ymax])`

三维图则用: `axis([xmin xmax ymin ymax zmin zmax])`

用命令 `axis auto` 使MATLAB重新自动选择范围。

**设定纵横比**

用 axis 也可以指定预先确定的数。例如，

`axis square`: 使x轴和y轴等长。

`axis equal`: 使x轴与y轴的单位长度相等。也就是说 `plot(exp(i*[0:pi/10:2*pi]))` 无论后面跟着 `axis square` 还是 `axis equal` 都把椭圆变成正圆。

`axis auto normal`: 返回默认模式中定义的缩放比例。

**设定轴的可见性**

用 axis 命令还可以使轴隐藏或显示。

`axis on`: 使轴显示出来。这是默认情况。

`axis off`: 使轴隐藏。

**设置网格线**

grid命令设置网格线显示或隐藏。语句

`grid on`: 使网格线显示，

`grid off`: 隐藏网格线。


## **9. 轴的标签与标题** ##

用 xlabel, ylabel, 及 zlabel 命令添加 x-,y-,z- 等标签。用 title 命令在图象顶部加标题，用 text 函数在图象中任何部位添加文本。TeX 标记的子集则产生希腊字母。可以交互地设置这些选项。

```matlab
t = -pi:pi/100:pi;
y = sin(t);
plot(t,y);
axis([-pi pi -1 1]);
xlabel('-\pi \leq {\itt} \leq \pi')
ylabel('sin(t)')
title('Graph of the sine function')
text(1,-1/3,'{\itNote the odd symmetry.}')
```

![](/img/articles/matlab/matlab-pic-002-7.jpg)

## **10. Figure的保存** ##

要保存图形，从 File 菜单选择 Save。要用图形格式如 TIFF 保存，以便在其他应用中使用，则从File 菜单选择 Export。还可以从命令行中保存-用 saveas 命令，包括任何以其他格式保存图象的选项。

