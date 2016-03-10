layout: post
title: 图论相关算法(Kruscal、Prim 和 Dijkstra)
date: 2015-9-15 19:36:22
tags: 
	- 算法
	- 图论
comments: true
toc: true
---
## 1. 最小生成树(Kruscal算法) ##
基本思想：kruskal 算法总共选择 n-1 条边（共n个点），所使用的贪婪准则是：从剩下的边中选择一条不会产生环路的具有最小耗费的边加入已选择的边的集合中。注意到所选取的边若产生环路则不可能形成一棵生成树。kruskal算法分 e 步，其中 e 是网络中边的数目。按耗费递增的顺序来考虑这 e 条边，每次考虑一条边。当考虑某条边时，若将其加入到已选边的集合中会出现环路，则将其抛弃，否则，将它选入。

<!--more-->

```C
#include "iostream"
#include "cstdlib"
#include "cstring"
#include "cstdio"
#define N 10000
#define M 100000
using namespace std;

struct edge //边集 
{
    int start;  //起点 
    int end;  //终点 
    int weight;  //权值 
}e[M];  
int n,m; //结点数，边数 
int cmp(const void *a,const void *b);//按升序排列
void kruscal(int *sum);

int main(){
	//freopen("a.txt","r",stdin);
    int i,sum;
  	cin>>n>>m;
    for(i=0;i<m;i++)
    	cin>>e[i].start>>e[i].end>>e[i].weight;//输入每条边的权值
    kruscal(&sum);
    cout<<sum<<endl;
	return 0;
}

int cmp(const void *a,const void *b)//按升序排列
{
    return((struct edge*)a)->weight - ((struct edge*)b)->weight ;
}
void kruscal(int *sum){
	int i,k,g,x[N],num;
	for(i=1;i<=n;i++)
        x[i]=i;
    qsort(e,m,sizeof(e[0]),cmp);  //给边排序 
    *sum=num=0;
    for(i=0;i<m && num < n-1;i++){
        for(k=e[i].start;x[k]!=k;k=x[k])//判断线段的起始点所在的集合
            x[k]=x[x[k]];
        for(g=e[i].end;x[g]!=g;g=x[g])//判断线段的终点所在的集合
            x[g]=x[x[g]];
        if(k!=g){     //如果线段的两个端点所在的集合不一样
            x[g]=k;
            *sum+=e[i].weight ;
            num++;
            //printf("最小生成树中加入边：%d %d\n",e[i].start,e[i].end);
        }
    }

}
```


## 2. 最小生成树(Prim算法) ##
基本思想：假设 $G＝(V，E)$ 是连通的，TE是G上最小生成树中边的集合。算法从 $U＝{u_0}（u_0∈V)、TE＝{}$开始。重复执行下列操作：在所有 $u∈U，v∈V－U$ 的边$ (u，v)∈E$ 中找一条权值最小的边 $(u_0,v_0)$ 并入集合TE中，同时 $v_0$ 并入 U，直到 V＝U 为止。此时，TE 中必有n-1条边，T=(V，TE) 为 G 的最小生成树。Prim 算法的核心:始终保持TE中的边集构成一棵生成树。

```C
#include "iostream"
#include "cstdio"
#define M 100000  //最大边数 
#define N 10000  //最大结点数 
#define INF 100000 
using namespace std;

struct edge{ //边集 
    int start;  //起点 
    int end;    //终点 
    int weight;  //权值 
}e[M];  
int n,m;   //定点数和边数 
int getWeight(int v,int w); //求w到v距离
void prim(int fa[],int *sum);

int main(void)
{
	//freopen("a.txt","r",stdin);
	int sum;    //最小生成树总长 
	int i; 
	cin>>n>>m;
	for(i=0; i<m; i++)
		cin>>e[i].start>>e[i].end>>e[i].weight; 
   	int fa[N];
   	prim(fa,&sum);
   	cout<<sum<<endl;
    return 0;
}

int getWeight(int v,int w){
	int i;
	for(i=0; i<m; i++){
		if((e[i].start==v&&e[i].end==w)||(e[i].start==w&&e[i].end==v))  //无向图 
			return e[i].weight;
	} 
	return INF;
} 

void prim(int fa[],int *sum){
    int i, j, m, k;
    int d[N];     /*d[j]可以理解成结点j到生成树(集合)的距离，它的最终值是w[j][fa[j]]*/
    *sum=0;
    for(j=1; j<=n; j++) {
        d[j] = (j == 1 ? 0 :getWeight(1,j));  /*将第一个结点加入集合，并初始化集合与其他结点的距离*/
        fa[j] = 1;   /*当前集合中有且只有一个结点1，其他结点暂时未加入集合，所以没有父结点，就先姑且初始化成1*/
    }
    for(i=2; i <=n; i++){
        m = INF;
        for(j=1; j<= n; j++)
            if(d[j] <= m && d[j] != 0) 
				m = d[k = j];  /*选取与集合距离最小的边*/
        *sum=*sum+d[k];   
		d[k] = 0;   /*0在这里表示与集合没有距离，也就是说赋值0就是将结点k添加到集合中*/          
        for(j=1; j <=n; j++) /*对刚加入的结点k进行扫描，更新d[j]的值*/
            if(d[j]>getWeight(k,j) && d[j] != 0){
                d[j] = getWeight(k,j); 
                fa[j] = k;
            }
    }
}
```


## 3. 单源最短路径(Dijkstra算法) ##
基本思想：设 G=(V,E) 是一个带权有向图，把图中顶点集合 V 分成两组，第一组为已求出最短路径的顶点集合（用S表示，初始时 S 中只有一个源点，以后每求得一条最短路径 , 就将加入到集合 S 中，直到全部顶点都加入到S中，算法就结束了），第二组为其余未确定最短路径的顶点集合（用 U 表示），按最短路径长度的递增次序依次把第二组的顶点加入S中。在加入的过程中，总保持从源点 v 到 S 中各顶点的最短路径长度不大于从源点 v 到 U 中任何顶点的最短路径长度。此外，每个顶点对应一个距离，S中的顶点的距离就是从 v 到此顶点的最短路径长度，U 中的顶点的距离，是从 v 到此顶点只包括 S 中的顶点为中间顶点的当前最短路径长度。
```C
#include"iostream"
#include"cstdlib"
#include"cstdio"
#include"cstring"
#define N 100   //顶点最大数 
#define M 10000   //边最大数 
#define INF 1000000  //两点之间没有路径 
using namespace std;

typedef struct node  //边节点
{  
   int adjvex;
   int weight;  
   node* next;  
}EdgeNode;  
EdgeNode * node[N+1];  //node[i] 表示顶点i的第一个邻接点地址,下标从 1 开始 
int D[N+1];       // 保存最短路径长度 
int p[N+1][N+1];   // 路径
int final[N+1];  // 若final[i]  =  1则说明 顶点vi已在集合S中
int n,m;      //顶点数和边数 
void Dijkstra(int v);   // DijKstra
int getWeight(int v,int w);  //得到v到w的距离 

int main()  
{  
	//freopen("a.txt","r",stdin);
	int i; 
	EdgeNode *s;
	cin>>n>>m;  
	for(i=1; i<n; i++)    //初始化 
		node[i] = NULL; 
	for(i=0; i<m; i++){  //输入边，邻接矩阵构建图 
		int x,y,z;
		cin>>x>>y>>z;
		s = (EdgeNode*)malloc(sizeof(EdgeNode)); 
		s->adjvex = y;       //插入表头 
		s->weight = z;
		s->next = node[x];
		node[x] = s;
		s = (EdgeNode*)malloc(sizeof(EdgeNode)); //无向图,若为有向图则去掉下面的 
		s->adjvex = x;
		s->weight = z;
		s->next = node[y];
		node[y] = s;
	} 
	int v0=1; 
	Dijkstra(v0);
	for(i=1;i<=n;i++){
		cout<<v0<<"->"<<i<<": "<<D[i]<<endl;
	}

	return 0;
}  

void Dijkstra(int v0){
	int v,w; 
	for (v=1; v<=n; v++){  //循环初始化
		final[v] = 0; 
		D[v] = getWeight(v0,v); 
		for (w=1; w<=n; w++) 
			p[v][w]  =  0;  //设空路径
		if (D[v]<INF){
			p[v][v0] = 1; 
			p[v][v] = 1;
		}
	}
	D[v0]  =  0; final[v0] = 1; //初始化 v0顶点属于集合S
	//开始主循环 每次求得v0到某个顶点v的最短路径 并加v到集合S中
	int i;
	for(i=1; i<=n; i++){
		int min = INF;
		for (w=1; w<=n; w++){    //核心过程--选点
            if (!final[w]){ //如果w顶点在V-S中
            //这个过程最终选出的点 应该是选出当前V-S中与S有关联边
            //且权值最小的顶点 ,也可以说是当前离V0最近的点
            	if (D[w]<min) {
					v = w; 
					min = D[w];
				}
        	}
    	}
		final[v] = 1;    //选出该点后加入到合集S中
        for (w=1; w<=n; w++){   //更新当前最短路径和距离 
            /*在此循环中 v为当前刚选入集合S中的点
            则以点V为中间点 考察 d0v+dvw 是否小于 D[w] 如果小于 则更新
            比如加进点 3 则若要考察 D[5] 是否要更新 就 判断 d(v0-v3) + d(v3-v5) 的和是否小于D[5]
            */
            if (!final[w] && (min+getWeight(v,w)<D[w])){
                D[w] = min+getWeight(v,w);
                // p[w] = p[v];
                p[w][w] = 1; //p[w] = p[v] +　[w]
            }
        }
	}
}
int getWeight(int v,int w){
	EdgeNode *p;
	p = node[v];
	while(p!=NULL){
		if(p->adjvex==w)
			return p->weight;
		p=p->next;
	} 
	return INF;
} 
```