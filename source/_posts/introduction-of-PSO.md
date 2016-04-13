layout: post
title:  粒子群优化算法简介
date: 2015-11-30 22:19:22
tags: 
	- 粒子群算法
	- 算法
comments: true  
toc: true
copyright: true
---
## 1. 背景
*Boyd* 和 *Richerson* 在研究人类的决策过程中，提出了个体学习和文化传递的概念。他们认为，人们在决策过程中使用两类重要的信息：一类是自身的经验，一类是他人的经验。也就是说，人们通常是根据自身的经验和别人的经验来进行自己的决策。这也是粒子群算法产生的启发之一。

1995年，美国社会心理学家 *James Kennedy* 和电气工程师 *Russell Eberhart* 受早期对鸟类、鱼类群体行为以及人工生命的研究的启发，并利用生物学家 *Frank Heppner* 的生物群体模型，模拟自然界的生物群体的行为来构造了一种随机的优化算法，由此首次正式提出了粒子群算法的概念。粒子群算法有以下特点：
- 分散式搜寻；
- 具有记忆性；
- 需要调整的参数较少，容易实现；
- 适合在连续性的范围内搜寻。
<!--more-->
粒子群优化算法是基于群体智能理论的优化算法，它是通过群体中粒子间的合作与竞争产生的群体智能指导优化搜索。与进化算法比较，粒子群算法保留了基于种群的全局搜索策略，但是它又采用了一种相对简单的速度－位移模型，避免了复杂的遗传算子操作。同时它特有的记忆功能使其可以动态跟踪当前的搜索情况而调整其搜索策略。因此，粒子群优化算法是一种更高效的并行搜索算法，收敛速度快，设置参数少。
## 2. 基本粒子群优化算法
- 在基本粒子群算法中，我们可以把每个优化问题的潜在解看作是n维搜索空间上的一个点，称之为“粒子”或“微粒”(Particle)，并假定它是没有体积和重量的。
- 所有的粒子都有一个被目标函数所决定的适应度值(Fitness Value)
- 和一个决定它们位置和飞行方向的速度，然后粒子们就以该速度追随当前的最优粒子在解空间中进行搜索，
- 其中，粒子的飞行速度根据个体的飞行经验和群体的飞行经验进行动态的调整。

假设 $X_i=(x_i,_1,x_i,_2,…,x_i,_n)$ 是粒子$i$的当前位置，$V_i,_j$是粒子$i$的当前飞行速度，那么基本粒子群算法的进化方程如下：
$v_i,_j(t+1)=v_i,_j(t)+c_1\*rand()\*(p_i,_j(t)-x_i,_j(t))+c2\*rand()\*(P_g(t)-x_i,_j(t))$
$x_i,_j(t+1)=x_i,j(t)+v_i,_j(t+1)$
其中，
- $t$表示迭代次数；
- $p_i,_j(t)$表示粒子$i$迄今为止经历过的历史最好位置；
- $P_g(t)$是当前粒子群搜索到的最好位置，也称全局最好位置；
- $c_1,c_2$为学习因子，通常在$0-2$间取值；
- $c_1$主要是为了调节粒子向自身的最好位置飞行的步长；
- $c_2$主要是为了调节微粒向全局最好位置飞行的步长；
- $rand()$是产生在$[0,1]$上的随机数

### 局部和全局最优
上述的进化方程中，第一部分$v_i,_j(t)$ 为粒子的先前速度，第二部分$c_1\*rand()\*(p_i,_j(t)-x_i,_j(t))$为“认知部分”，表示粒子本身的思考，第三部分$c2\*rand()(P_g(t)-x_i,_j(t))$为“社会”部分，表示粒子间的信息共享。
我们设$pbest$和$gbest$分别表示粒子的局部和全局最优位置，那么，
当$c_1=0时$，粒子没有认知能力，变为只有社会的模型，
$V_i=V_i+c_2\*rand()\*(gbest_i-x_i)$
这被称为全局PSO算法.粒子有扩展搜索空间的能力，具有较快的收敛速度，但由于缺少局部搜索，对于复杂问题比标准PSO 更易陷入局部最优。
当$C_2＝0$时，则粒子之间没有社会信息，模型变为只有认知的模型,
$V_i=V_i+c_1\*rand()\*(pbest_i-x_i)$
这被称为局部PSO算法。由于个体之间没有信息的交流，整个群体相当于多个粒子进行盲目的随机搜索，收敛速度慢，因而得到最优解的可能性小。

### 算法流程
- Step1: 初始化一群粒子(群体规模为M)，包括随机位置和速度；
- Step2: 评价每个粒子的适应度；
- Step3: 对每个粒子，将其适应值与其经过的最好位置pbest作比较，如果较好，则将其作为当前的最好位置pbest;
- Step4: 对每个粒子，将其适应值与其所有粒子经历过的最好位置gbest作比较，如果较好，则将其作为当前的最好位置gbest;
- Step5: 根据公式调整粒子速度和位置；
- Step6: 未达到结束条件则转Step2。

### 缺点与改进
飞行速度的大小直接影响着算法的全局收敛性。当微粒的飞行速度过大时，各微粒初始将会以较快的速度飞向全局最优解邻近的区域，但是当逼近最优解时，由于微粒的飞行速度缺乏有效的控制与约束，则将很容易飞越最优解，转而去探索其它区域，从而使算法很难收敛于全局最优解；当微粒的飞行速度过小时，微粒在初期向全局最优解邻近区域靠近的搜索时间就需要很长。这一现象说明了该算法在速度缺乏有效的控制策略时，不具备较强的局部搜索能力或精细搜索能力。正是基于上述分析，Y.Shi和R.C.Eberhart在1998年的IEEE国际进化计算学术会议上发表了题为*“A Modified Particle Swarm Optimizer”*的论文，首次提出了一种带有惯性权重（inertia weight）的粒子群算法。

## 3. 带惯性权重的粒子群优化算法
$v_i,_j(t+1)=w\*v_i,_j(t)+c_1\*rand()\*(p_i,_j(t)-x_i,_j(t))+c2\*rand()\*(P_g(t)-x_i,_j(t))$
- w 为惯性权重，它本身具有维护全局和局部搜索能力的平衡作用，可以使微粒保持运动惯性，使其有扩展空间搜索的趋势，有能力搜索到新的区域；其它符号表示意义同上。
- $V_{max}$粒子飞行速度的上限，间接地影响算法的全局搜索能力。
	- 当$V_{max}\le2$时，$w$的取值最好选择接近1；
	- 当$V_{max}\ge3$时，$w=0.8$是最佳的选择;
	- 当$0.9 \le V_{max}\le1.2$时，一般可达到比较理想的效果。
- 认知学习系数$c_1$和社会学习系数$c_2$之和最好接近4.0，通常取$c_1=c_2=2.05$

## 4. 测试
### *Rosenbrock*函数
$F_1=f(x)=100\*(x_1^2-x_2^2)^2+(1-x_1)^2,\quad \quad -2.048 \le x_1,x_2 \le 2.048$
该测试函数是由 *K.A.DeJong* 提出的，被称为 *Rosenbrock* 函数，非凸、病态函数，在$x_1=x_2=1$时达到极小值$F(1,1)=0$ 。
### *Schaffer*函数
$F_2=f_i(x)=1+\frac{sin^2\sqrt{x_1^2+x_2^2}-0.5}{[1+0.001\*(x_1^2+x_2^2)]^2}, \quad \quad -100 \le x_i\le100$
该函数是著名的 *Schaffer* 函数，由 *Schaffer* 提出，全局极大点是 $(0,0)$，在距离全局极大点大约3.14的范围内存在无限多的次全局极大点，函数强烈震荡的性态使其难于全局最优化，它的全局最小值为$F_2(0,0,)=0.5$。

## 5. 编码
### Matlab实现
``` matlab
function [xm,fv]=PSO(fitness,N,c1,c2,w,M,D)
% fitness:待优化的目标函数
% N:粒子数目
% c1,c2:学习因子1，学习因子2
% w:惯性权重
% M:最大迭代次数
% D:问题的维数
% xm:目标函数取最小值时的自变量值
% fv:目标函数最小值
format long;
%---------初始化种群的个体-------------
for i=1:N
    for j=1:D
        x(i,j)=randn;
        v(i,j)=randn;
    end
end
%---------先计算各个粒子的适应度，并初始化pbest和gbest----------
for i=1:N
    p(i)=fitness(x(i,:)); %每个粒子的适应度
    pbest(i,:)=x(i,:); %单个粒子最优位置
end
gbest=x(N,:); %gbest为全局最优
for i=1:(N-1);
    if fitness(x(i,:))<fitness(gbest)
        gbest=x(i,:);
    end
end
%---------进入主循环，按照公式依次迭代----------
for t=1:M
    for i=1:N
        v(i,:)=w*v(i,:)+c1*rand*(pbest(i,:)-x(i,:))+c2*rand*(gbest-x(i,:));
        x(i,:)=x(i,:)+v(i,:);
        if fitness(x(i,:))<p(i)
            p(i)=fitness(x(i,:));
            pbest(i,:)=x(i,:); %更新pbest
        end
        if p(i)<fitness(gbest)
            gbest=pbest(i,:); %更新gbest
        end
    end
end
xm=gbest';
fv=fitness(gbest);
```
### C语言实现
``` C
#include"stdio.h"
#include "math.h"
#include "stdlib.h"
#include "time.h"
#define N 100  //最大粒子数 
#define D 10  //最大问题维数 

double fitness(double x[],int n);
typedef double (*funType)(double [],int n);
double pso(funType fp,int n,double c1,double c2,double w,int m,int d,double xbest[]);

int main(){
	double x[D];
	double f;
	int n;
	n=2; //问题维数 
	f = pso(fitness,40,2.05,2.05,0.8,2000,n,x);
	printf("最优的x为：");
	for(int i=0;i<n;i++)
		printf("%f ",x[i]); 
	printf("\n");
	printf("函数的最优值为：%f\n",f);
	return 0;

}
double fitness(double x[],int n){  //待优化函数,n表示x维数 
	double f = 0;
	f = 100*(x[0]*x[0]-x[1]*x[1]) *(x[0]*x[0]-x[1]*x[1])+(1-x[0])*(1-x[0]);   // Rosenbrock函数
	                                                                 // f(x)=100(x0^2-x1^2)^2+(1-x0)^2; 
	return f;	
}

/*
	fp: 待优化的目标函数
	n: 粒子数目
	c1,c2: 学习因子1，学习因子2
	w: 惯性权重
	M: 最大迭代次数
	d: 问题的维数
	x: 返回的最优解 
*/
double pso(funType fp,int n,double c1,double c2,double w,int m,int d,double xbest[]){
	int i,j,t;
	double x[N][D],v[N][D];
	double gbest[D];   //全局最优解
	double p[N];    //粒子适应度
	double pbest[N][D]; // 单个粒子最优位置 
	//step1.初始化种群的个体
	srand((int)time(0)); 
	for(i=0;i<n;i++)          	
		for(j=0;j<d;j++){
			x[i][j] = rand()*1.0/RAND_MAX;
			v[i][j] = rand()*1.0/RAND_MAX;
		}
	//step2. 计算粒子适应度，初始化p,pbest和gbest 
	for(i=0;i<d;i++)
		gbest[i] = x[n-1][i]; // gbest为全局最优 
	for(i=0;i<n;i++){
		p[i] = fp(x[i],d);
		for(j=0;j<d;j++)
			pbest[i][j] = x[i][j];
	}
	for(i=0;i<n-1;i++){
		if(p[i]<p[n-1]){
			for(j=0;j<d;j++)
				gbest[j] = x[i][j];
		}		
	}
	//step3. 进入主循环，按照公式依次迭代 	
	for(t=0;t<m;t++){         
		for(i=0;i<n;i++){
			for(j=0;j<d;j++){
				v[i][j] = w*v[i][j] + c1*(rand()/RAND_MAX)*(pbest[i][j]-x[i][j]) + c2*(rand()/RAND_MAX)*(gbest[j]-x[i][j]); //更新 
				x[i][j] = x[i][j] + v[i][j];
			}
			if(fp(x[i],d)<p[i]){
				p[i] = fp(x[i],d);  //更新p
				for(j=0;j<d;j++)
					pbest[i][j] = x[i][j]; //更新pbest
			}
			if(p[i]<fp(gbest,d)){
				for(j=0;j<d;j++)
					gbest[j] = pbest[i][j];//更新gbest
			}
		} 
		
	}
	
	for(i=0;i<d;i++) //最优解 
		xbest[i] = gbest[i];  
	return fp(gbest,d);
}
```