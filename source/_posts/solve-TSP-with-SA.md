layout: post
title: 利用模拟退火算法求解TSP问题
date: 2015-12-4 20:46:22
tags: 
	- 模拟退火算法
	- 算法
comments: true
---
## 1. 什么是 TSP 问题 ##
旅行商问题，即TSP问题（Travelling Salesman Problem）又译为旅行推销员问题、货郎担问题，是数学领域中著名问题之一。假设有一个旅行商人要拜访n个城市，他必须选择所要走的路径，路径的限制是每个城市只能拜访一次，而且最后要回到原来出发的城市。路径的选择目标是要求得的路径路程为所有路径之中的最小值。


## 2. 什么是模拟退火算法 ##
关于模拟退火算法请参考之前发的一篇文章，请[点击这里](../../../../2015/12/03/introduction-of-SA/)。
</br>

<!--more-->


## 3. 求解思想 ##
关于解的形式，肯定是一组点的序列，表示了访问顺序。第一个点和最后一个点相同，均为起点。
假设所有的点分别为 $\lbrace v_1,v_2,...,v_n \rbrace$,起点为 $v_1$。
(1)解空间
解空间 $S$ 可表为的所有固定起点和终点的循环排列集合，即        
$ S = \lbrace ({\pi}_1,{\pi_2}...,{\pi}_n, {\pi}_1)|{\pi}_1=v_1,({\pi}_2,...,{\pi}_n)为\lbrace v_2,v_3,...,v_n \rbrace的一个排列\rbrace $。   
$S$ 中的每一个排列表示一个回路，也就是一个解。 ${\pi}_i=j$ 表示在第 i-1 次访问目标 j。

(2)目标函数
此时的目标函数为访问所有目标的路径长度或称代价函数。我们要求   
$$ min f({\pi}_1,{\pi_2}...,{\pi}_n, {\pi}_1)=\sum_1^n d $$

(2)新解的产生
任选序号 u,v（u < v）交换u与v之间的顺序产生新的路径

## 4. 编码(C语言) ##

```C
/* 利用模拟退火算法求解tsp问题 */
#include"iostream"
#include"ctime"
#include"cstdio"
#include"cstdlib"
#include"cmath"
#define MAX 10000
#define INF 10000000 
#define E 0.000000001  // 迭代误差 
#define L 20000    // 迭代次数 
#define AT 0.999   // 降温系数 
#define T 1       // 初始温度 
using namespace std; 

struct element{     //用来排序的数据结构 
		int data;  // 数据 
		int index;  // 序号 
};
int tsp(int d[][MAX], int n, double e,int l,double at,double t,int s0[]);  //利用模拟退火算法求解最短路径 
int cmp(const void *a,const void *b); //升序排列 
void rand_of_n(int a[],int n);  //产生 1-n 的随机排列并存到 a[] 中
int random(int m,int n);
int dis[MAX][MAX];   // d[i][j] 表示客户i到客户j的距离，0 表示起始点 
int main(){
	int i,j;
	int n=100;  // 点的个数 
	for(i=0;i<=n;i++)     // 随机产生距离 1-100
		for(j=i;j<=n;j++)
			if(i==j)
				dis[i][j]=0;
			else
				dis[i][j]=dis[j][i]=random(1,100); 
	FILE*fp = fopen("data.txt","w");  // 将距离存入文件中 
	for(i=0;i<=n;i++){  
		for(j=0;j<=n;j++)
			fprintf(fp,"%d ",dis[i][j]);
		fprintf(fp,"\n");
	}
	fclose(fp); 
	int sum,sum0,s0[MAX]; 
	sum0=0;   //顺序遍历时的路程 
	for(i=0;i<n;i++)
		sum0=sum0+dis[i][i+1];
	sum0=sum0+dis[n][0];
	sum=tsp(dis,n, E,L,AT,T,s0);
	fp = fopen("answer.txt","w");  //将结果存入文件中
	for(i=1;i<=n;i++)
		fprintf(fp,"%d ",s0[i]);
	fclose(fp); 
	cout<<"case"<<"#: "<<endl;
	cout<<"距离数组保存在文件data.txt中，优化后的访问顺序保存在文件answer.txt中"<<endl; 
	cout<<"顺序遍历总路程为："<<sum0<<" 优化后为："<<sum<<endl; 
	cout<<endl;
	
	return 0; 
}

int cmp(const void *a,const void *b){   // 升序排序
	return((struct element*)a)->data - ((struct element*)b)->data;
}

void rand_of_n(int a[],int n){
	int i;
	struct element ele[MAX];
	srand((int)time(0));  // 初始化随机数种子 
	for(i=0;i<n;i++){
		ele[i].data=rand();  // 随机生成一个数 
		ele[i].index=i+1;
	}
	qsort(ele,n,sizeof(ele[0]),cmp);  //排序 
	for(i=0;i<n;i++){
		a[i]=ele[i].index;
	}
}

int random(int m,int n){   //产生m-n的随机数
	int a;
	double x=(double)rand()/RAND_MAX;
	a=(int)(x*(n-m)+0.5)+m;
	return a;
}	

int tsp(int d[][MAX], int n, double e,int l,double at,double t,int s0[]){
	int i,j,s[MAX],sum,temp;
	sum=INF; 
	for(i=0;i<1000;i++){  //利用蒙特卡洛方法产生初始解
		rand_of_n(&s[1],n);
		s[0]=0; s[n+1]=0;  //第一个和最后一个点为起始点 
		temp=0;
		for(j=0;j<=n;j++)
			temp=temp+d[s[j]][s[j+1]];
		if(temp<sum){
			for(j=0;j<=n+1;j++)
				s0[j]=s[j];
			sum=temp;
		} 
	}
	
	for(i=0;i<l;i++){    //退火过程 
		int c1,c2;
		c1=random(1,n);
		c2=random(1,n);
		if(c1>c2){
			int temp=c2; c2=c1; c1=temp;
		} 
		if(c1==c2)
			continue;
		int df=d[s0[c1-1]][s0[c2]] + d[s0[c1]][s0[c2+1]] - d[s0[c1-1]][s0[c1]] - d[s0[c2]][s0[c2+1]]; //计算代价函数
		if(df<0){  //接受准则
			while(c1<c2){ 
				int temp=s0[c2]; s0[c2]=s0[c1]; s0[c1]=temp;
				c1++;
				c2--;
			}
			
			sum=sum+df;
		} 
		else if(exp(-df/t)>((double)rand()/RAND_MAX)){
			while(c1<c2){
				int temp=s0[c2]; s0[c2]=s0[c1]; s0[c1]=temp;
				c1++;
				c2--;
			}

			sum=sum+df;
		}
		t=t*at;
		if(t<e)
			break;
	}
	return sum;
}
 
```

