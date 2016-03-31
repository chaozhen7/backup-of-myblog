layout: post
title: 利用粒子群算法求解非线性二层规划问题
date: 2015-11-07 19:00:22
tags: 
	- 粒子群算法
	- 二层规划
	- 算法
comments: true
toc: true
copyright: true
---
## 1. 问题示例 
$ \min \limits_{x\ge0}\{F(x,y)=-x_1^2-3\*x_2-4\*y_1+y_2^2} $  
$ s.t. \quad x_1^2+2\*x_2\le4 $


$ \min \limits_{y\ge0}\{f(x,y)=2\*x_1^2+y_1^2-5\*y_2} $
$s.t. \quad x_1^2-2\*x_1+x_2^2-2\*y_1+y_2\ge-3 $
$\quad \quad x_2^2+3\*y_1-4\*y_2\ge4 $

<!--more-->

## 2. 约束处理

对约束条件的处理主要是用了罚函数的方法。

## 3. 符号说明

fbest: 上层函数的最优值（最小值）；
xbest: 最优的x；
ybest: 最优的y；

## 4. 算法步骤

- step1. 根据上层规划问题的约束条件产生上层决策变量 x 的初始解 x0，初始化 fbest = INF；
- step2. 置 k=1，给定迭代次数M；
- step3. 将 x0 代入到二层规划的下层，利用粒子群算法求解下层问题得到最优解 y0；
- step4. 将得到的 y0 值返回上层，利用粒子群算法对上层问题进行求解得到最优解 x 及相应的最优值 f；
- step5. 如果 f<fbest，那么更新 fbest = f，xbest = x，ybest=y0，否则直接进入到step6；
- step6. 如果满足结束条件（误差足够好或者达到最大迭代次数）退出，否则令 x0 = xbest，k=k+1，然后进入到step3；
- step7. 输出最优的x,y的值 xbest, ybest；  

## 5. 源码(matlab)
  
上层问题 f1.m
``` matlab
function f = f1(x,y)  
f = -1*x(1).^2 - 3*x(2) - 4*y(1) + y(2).^2 ;
```
上层问题的约束 h.m
``` matlab
function f = h(x)
f = x(1).^2 + 2*x(2) - 4;   
```
下层问题 f2.m
``` matlab
function f = f2(x,y)  
f = 2*x(1).^2 + y(1).^2 - 5*y(2);  
```
下层问题的约束 g1.m
``` matlab
function f = g1(x,y)  
f = x(1).^2 - 2*x(1) + x(2).^2 - 2*y(1) + y(2) + 3;  
```
下层问题的约束 g2.m
``` matlab
function f = g2(x,y)  
f = x(2) + 3*y(1) - 4*y(2) - 4;  
```
罚函数法处理上层问题 minf1.m
``` matlab  minf1.m
function f = minf1(x,y,M)  
f = f1(x,y) + M*(max(0,h(x))).^2;  
```
罚函数法处理下层问题 minf2.m
``` matlab
function f = minf2(x,y,M)  
f = f2(x,y) + M*( (max(0,-1*g1(x,y))).^2  + (max(0,-1*g2(x,y))).^2  );  
```
粒子群算法 PSO.m
``` matlab
function xm=PSO(choose,value,N,c1,c2,w,M,D,s)  
% fitness:待优化的目标函数  
% N:粒子数目  
% c1,c2:学习因子1，学习因子2  
% w:惯性权重  
% M:最大迭代次数  
% D:问题的维数  
% xm:目标函数取最小值时的自变量值  
% fv:目标函数最小值  
% choose:为0时表示已知y=value求上层x,为1时表示已知x=value求下层y,  
% s: 罚函数法的系数  
format long;  
if(choose==0)  
    fitness = @(x) minf1(x,value,s);  
else  
    fitness = @(x) minf2(value,x,s);  
end  
%---------初始化种群的个体-------------  
for i=1:N  
    for j=1:D  
        x(i,j)=randn;  
        v(i,j)=randn;  
    end  
end  
%---------先计算各个粒子的适应度，并初始化Pi和Pg----------  
for i=1:N  
    p(i)=fitness(x(i,:));  
    y(i,:)=x(i,:);  
end  
pg=x(N,:);          %pg为全局最优  
for i=1:(N-1);  
    if fitness(x(i,:))<fitness(pg)  
        pg=x(i,:);  
    end  
end  
%---------进入主循环，按照公式依次迭代----------  
for t=1:M  
    for i=1:N  
        v(i,:)=w*v(i,:)+c1*rand*(y(i,:)-x(i,:))+c2*rand*(pg-x(i,:));  
        x(i,:)=x(i,:)+v(i,:);  
        if fitness(x(i,:))<p(i)  
            p(i)=fitness(x(i,:));  
            y(i,:)=x(i,:);  
        end  
        if p(i)<fitness(pg)  
            pg=y(i,:);  
        end  
    end  
    pbest(t)=fitness(pg);  
end  
xm=pg;  
```
主函数求解 main.m
``` matlab
clc;clear;  
maxf=10000000;  
D=2;%维数  
while(1)        %初始解  
    x0=rand(1,D);  
    if(h(x0)<=0)  
        break;  
    end  
end  
for i=1:100  
    y0 = PSO(1,x0,40,1.5,1.5,0.6,200,2,100000);  
    x = PSO(0,y0,40,1.5,1.5,0.6,200,2,100000);  
    if maxf>f1(x,y0)  
        maxx = x;  
        maxy = y0;  
        maxf = f1(x,y0);  
    end  
    x0=maxx;  
end  
maxx - x  
f1(maxx,maxy)  
f2(maxx,maxy)
```