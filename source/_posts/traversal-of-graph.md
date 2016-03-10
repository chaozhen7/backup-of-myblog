layout: post
title: 图的深度优先遍历(DFS)和广度优先遍历(BFS)
date: 2015-09-15 16:17:22
tags: 
	- 算法
	- 图论
comments: true
---

## 1.首先介绍下图的存储结构 ##
邻接矩阵的存储结构比较简单就不介绍了，主要讲下邻接表的存储。邻接表的处理方法是这样的：
（1）图中顶点用一个一维数组存储，当然，顶点也可以用单链表来存储，不过，数组可以较容易的读取顶点的信息，更加方便。    
（2）图中每个顶点 $v_i$ 的所有邻接点构成一个线性表，由于邻接点的个数不定，所以，用单链表存储，无向图称为顶点 $v_i$ 的边表，有向图则称为顶点 $v_i$ 作为弧尾的出边表。具体如下图所示。
![图的邻接表表示](/img/articles/graph.png)

<!--more-->

## 2. 宽度优先遍历(BFS) ##
基本思想：首先，从图的某个顶点 $v_0$ 出发，访问了 $v_0$ 之后，依次访问与 $v_0$ 相邻的未被访问的顶点，然后分别从这些顶点出发，广度优先遍历，直至所有的顶点都被访问完。
算法描述：
step1: 将起始节点放入队列尾部
step2: While(队列不为空）
- 取得并删除队列首节点Node
- 处理该节点Node
- 把Node的未处理相邻节点加入队列尾部

## 3. 深度优先遍历 ##
 基本思想：首先从图中某个顶点 $v_0$ 出发，访问此顶点，然后依次从 $v_0$ 相邻的顶点出发深度优先遍历，直至图中所有与 $v_0$ 路径相通的顶点都被访问了；若此时尚有顶点未被访问，则从中选一个顶点作为起始点，重复上述过程，直到所有的顶点都被访问。可以看出深度优先遍历是一个递归的过程。

## 4. 源码 ##
```C
#include"iostream"
#include"cstdlib"
#include"cstdio"
#include"queue"
#include"stack"
#include"cstring"
#define N 100   //顶点最大数 
#define M 10000   //边最大数 
using namespace std;

typedef struct node  //边节点
{  
   int adjvex;  
   node* next;  
}EdgeNode;  
EdgeNode * node[N];  //node[i] 表示顶点i的第一个邻接点地址  
int visited[N];      //访问标志 

int firstadj(int v);   //返回v的第一个邻接点 
int nextadj(int v,int w); //返回v的在w之后的邻接点 
void dfs(int v);     
void dfs1(int v); //dfs非递归算法
void bfs(int v); 
int main()  
{  
	//freopen("a.txt","r",stdin);
	int n,m;  //顶点数和边数 
	int i; 
	EdgeNode *s;
	cin>>n>>m;  
	for(i=1;i<n;i++)    //初始化 
		node[i]=NULL; 
	for(i=0;i<m;i++){  //输入边，邻接矩阵构建图 
		int x,y;
		cin>>x>>y;
		s=(EdgeNode*)malloc(sizeof(EdgeNode)); 
		s->adjvex=y;       //插入表头 
		s->next=node[x];
		node[x]=s;
		s=(EdgeNode*)malloc(sizeof(EdgeNode)); //无向图 
		s->adjvex=x;
		s->next=node[y];
		node[y]=s;
	} 
		
	for(i=1;i<=n;i++){  // 打印每个结点及其邻接点 
		cout<<i<<" : ";
		int w=firstadj(i);
		while(w!=0){
			cout<<w<<" ";
			w=nextadj(i,w); 
		}
		cout<<endl;
	}
	memset(visited,0,sizeof(visited));
	cout<<"BFS:";	bfs(1);  //bfs
	cout<<endl;
	memset(visited,0,sizeof(visited));
	cout<<"DFS:";	dfs(1);  //dfs
	cout<<endl;
	memset(visited,0,sizeof(visited));
	cout<<"DFS:";	dfs1(1);  //dfs
	cout<<endl;
	return 0;
}  

int firstadj(int v){
	if(node[v]!=NULL)
		return node[v]->adjvex;
	return 0;
} 

int nextadj(int v,int w){
	EdgeNode *p=node[v]; 
	while(p!=NULL){
		if(p->adjvex==w)
			break;
		p=p->next;
	}
	p=p->next;
	if(p!=NULL)
		return p->adjvex;
	return 0;
}

void bfs(int v){    // bfs 
	int w,u;
	queue<int> q;
	visited[v]=1;
	cout<<v<<" ";
	q.push(v);
	while(!q.empty()){
		w=q.front();
		q.pop();
		u=firstadj(w);
		while(u!=0){
			if(!visited[u]){
				cout<<u<<" ";
				visited[u]=1;
				q.push(u);
			}
			u=nextadj(w,u);
		}		
	} 
} 

void dfs(int v){	//dfs
	int w;
	visited[v]=1;
	cout<<v<<" "; 
	w=firstadj(v);
	while(w!=0){
		if(!visited[w])
			dfs(w);
		w=nextadj(v,w);
	}
}

void dfs1(int v){  //dfs的非递归写法
	int temp,w;
	stack<int> s;
	visited[v]=1;
	cout<<v<<" "; //输出其起始结点，随后入栈 
	s.push(v);
	while(!s.empty()){
		temp=s.top();  // temp指向栈顶元素 
		w=firstadj(temp);  //栈顶元素第一邻接点 
		if(!visited[w]){
			cout<<w<<" ";  //如果没被访问，则访问并入栈 
			visited[w]=1;
			s.push(w);
		} 
		else{       //否则，访问下一个未被访问的邻接点并入栈 
			w=nextadj(temp,w);  
			while(w!=0){
				if(!visited[w]){
					cout<<w<<" ";
					visited[w]=1;
					s.push(w);
					break;
				} 
				else
					w=nextadj(temp,w);
			}
			if(w==0)  // 所有邻接点都被访问则出栈 
				s.pop();			
		}	 	
	 }
}
```