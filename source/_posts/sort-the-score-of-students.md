layout: post
title: 如何给数百万考生的成绩排序
date: 2015-12-17 17:13:22
tags: 
	- 算法
comments: true
---


这个问题是在联系中科大的导师时被导师问到的，那个时候已经通过了中科大的免试研究生的面试，然后联系了一个导师，得知我人在合肥时让我去他办公室聊聊，其实大概就是了解下专业水平吧。问了很多问题，这是其中一个。虽然距离现在已经过去了好几个月，但是突然想到了这个问题，就想把它写下来。

下面来说说这个问题吧，忽略内存的问题，假设这些考试成绩是可以一次被装载进内存的。由于数据比较多，哪怕是利用快速排序算法，性能也不是很好。其实这个时候我们可以好好分析下问题，被排序的对象是分数，分析下分数的特征，分数都是整数而且范围都在 0-100 之间。其实这个时候就应该要联想到桶排序了，下面来解决下这个问题。

<!--more-->

其实我们这里只需借助一个一维数组就可以解决这个问题。请确定你真的仔细想过再往下看哦。

首先我们需要申请一个大小为 101 的数组 int a[101]。OK 现在你已经有了 101 个变量，编号从 a[0]~a[100]。刚开始的时候，我们将 a[0]~a[100] 都初始化为 0，表示这些分数还都没有人得过。例如 a[0] 等于 0 就表示目前还没有人得过 0 分，同理 a[1]等于 0 就表示目前还没有人得过 1 分……a[100]等于 0 就表示目前还没有人得过 100 分。

下面我们开始遍历所有人的分数，例如第一个人得了80分，那么我们令 a[80]++，如果第二个得了90分，那么我们令 [90]++,...,依次遍历当一个人得了 i 分之后我们就令 a[i]++。

最后你发现没有，a[0]~a[100]中的数值其实就是 0 分到 100 分每个分数出现的次数。接下来，我们只需要将出现过的分数打印出来就可以了，出现几次就打印几次，a[0]为 0，表示“0”没有出现过，不打印。我们可以写出下面的程序:
```C
#include <stdio.h>
int main(){
	int a[101],i,j,t;
	for(i=0;i<=100;i++)
		a[i]=0; //初始化为0
	for(i=1;i<=5;i++){ //循环读入5个数，
		scanf("%d",&t); //把每一个数读到变量t中
		a[t]++; //进行计数
	}
	for(i=0;i<=10;i++) //依次判断a[0]~a[100]
		for(j=1;j<=a[i];j++) //出现了几次就打印几次
			printf("%d ",i);
	return 0;
}
```

这样基本只要一次遍历就可以完成排序了，时间复杂度为 O(n)，比一般的排序算法快很多。我们进一步思考，如果我们要输出的不仅仅是分数还有学生信息呢，比如说从小到大输出所有学生的姓名和分数，那么这个时候也只要做一点小小的变动就好了。

我们用一个结构体去表示一个学生，如下：
```c
typedef struct students{ 
	char name[15];
	unsigned int score;
	students *next;
}student;
```

然后把所有学生信息放在一个链表中去存储。

接下来考虑排序过程中处理的问题，这个时候排序的数组就不能简单的用 int a[101] 了，我们这时候就要用一个 student 的指针数组 student \* score[101] 。    

排序的过程中，我们依次遍历所有的学生分数，假设某个学生的分数为 i ,那么就在 score[i] 的后面插入该学生的节点：    

```C
unsigned int s = q->score;	
q->next = score[s]->next;
score[s]->next = q;
```
遍历完所有的学生分数后，我们就会发现整个学生信息的存储结构就会发生一个很大的变化，之前是所有的学生节点连成一个链表。但是现在排序完后，所有相同分数的学生信息放在一个链表的里，这些链表的头节点又都放在数组里，数组的下标就是他们的分数。

完整的代码如下：
```c
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
typedef struct students{ 
	char name[15];
	unsigned int score;
	students *next;
}student;

int main(){
	//freopen("data.txt","r",stdin);
	int i,n; 
	student *head = (student *)malloc(sizeof(student));
	head->next = NULL;
	student *p,*u;
	p=head;
	scanf("%d",&n);   // 输入学生总人数 
	for(i=0;i<n;i++){          
		u = (student *)malloc(sizeof(student));
		scanf("%s%d",u->name,&(u->score));  // 输入学生信息 
		p->next = u;
		p=u;
	}
	p->next = NULL;
	printf("排序之前:\n");
	p=head->next;
	while(p!=NULL){
		printf("%s:%d\n",p->name,p->score);  // 输出排序前的学生信息 
		p=p->next;
	}
	// sort
	student * score[101]; // score[i] 表示分数位 i 的学生组成的链表的头节点 
	memset(score,NULL,sizeof(score));
	p=head->next;
	student * q;
	while(p!=NULL){
		q=p;
		p=p->next; 
		unsigned int s = q->score;	
		if(score[s]==NULL){
			score[s] = (student *)malloc(sizeof(student));
			score[s]->next = NULL;
		} 	
		q->next = score[s]->next;
		score[s]->next = q;
	}	
	printf("排序之后:\n");
	for(i=0;i<101;i++){
		if(score[i]==NULL)
			continue;
		p=score[i]->next;
		while(p!=NULL){
			printf("%s:%d\n",p->name,p->score); 
			p=p->next;
		}
	}
	
	return 0;
}
```

测试输入:
> 7
a 20
b 30
d 10
c 20
e 5
f 5
g 40

测试输出：
> 排序之前:
a:20
b:30
d:10
c:20
e:5
f:5
g:40
排序之后:
f:5
e:5
d:10
c:20
a:20
b:30
g:40