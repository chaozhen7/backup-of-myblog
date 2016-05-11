layout: post
title: 遗传算法的 matlab 实现
date: 2015-11-03 17:20:22
tags: 
	- Matlab
	- 遗传算法
	- 算法
comments: true
---


<!--more-->

```matlab
function [xv,fv]=myGA(fitness,a,b,NP,NG,Pc,Pm,eps)
% fitness:待优化的目标函数
% a:自变量的下界
% b:自变量的上界
% NP:种群个体数20-100
% NG:最大进化代数100-500
% Pc:杂交概率0.4-0.99
% Pm:变异概率0.0001-0.1
% eps:自变量离散精度
% xm:目标函数取最大值时的自变量值
% fv:目标函数的最大值
L=ceil(log2((b-a)/eps+1));      %根据离散精度确定二进制编码需要的码长
x=zeros(NP,L);
for i=1:NP
    x(i,:)=Initial(L);                %种群初始化
    fx(i)=fitness(Dec(a,b,x(i,:),L)); %个体适应值
end
for i=1:NG
    sumfx=sum(fx);          %所有个体适应值之和
    Px=fx/sumfx;            %所有个体适应值的平均值
    PPx(1)=Px(1);
    for i=2:NP              %用于轮盘赌策略的概率累加
        PPx(i)=PPx(i-1)+Px(i);
    end
    for i=1:NP
        sita=rand();
        for n=1:NP
            if sita<=PPx(n)
                SelFather=n;  %根据轮盘赌策略确定的父亲
                break;
            end
        end
        Selmother=floor(rand()*(NP-1))+1;  %随机选择母亲   
        posCut=floor(rand()*(L-2))+1;     %随机确定交叉点
        r1=rand();
        if r1<=Pc                         %交叉
            nx(i,1:posCut)=x(SelFather,1:posCut);
            nx(i,(posCut+1):L)=x(Selmother,(posCut+1):L);
            r2=rand();
            if r2<=Pm                     %变异
                posMut=round(rand()*(L-1)+1);
                nx(i,posMut)=1-nx(i,posMut);
            end
        else
            nx(i,:)=x(SelFather,:);
        end
    end
    x=nx;
    for i=1:NP
        fx(i)=fitness(Dec(a,b,x(i,:),L));  %子代适应值
    end
end
fv=-inf;
for i=1:NP
    fitx=fitness(Dec(a,b,x(i,:),L)); 
    if fitx>fv
        fv=fitx;                       %取个体中的最好的值作为最终结果
        xv=Dec(a,b,x(i,:),L);
    end
end

% 初始化函数
function result=Initial(length)        
for i=1:length
    r=rand();
    result(i)=round(r);         %round四舍五入取整
end

%二进制编码转换为十进制编码
function y=Dec(a,b,x,L)                
base=2.^((L-1):-1:0);
y=dot(base,x);%dot向量的点乘
y=a+y*(b-a)/(2^L-1);
```

### **示例** ###

```
fy = @(x) x.^2 -2*x +1;
[xv,fv]=myGA(fy,-5,5,100,300,0.5,0.001,0.00001)

```

输出结果为：
> xv = -4.9733
fv =  35.6798


### **写在后面** ###

这里遗传算法求的是函数的最大值，如果要求最小值只要在原来的函数的基础上乘以 -1 作为新的函数来求就可以了。

这里的所优化的函数必须是单变量的函数，如果要求多变量函数的极值需要做一些修改。