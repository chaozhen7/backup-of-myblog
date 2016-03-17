layout: post
title: 一种时间复杂度为 O(N) 整数排序算法
date: 2016-3-15 17:20:22
tags: 
	- 算法
	- 排序
comments: true
---


问题是这样的，对于一组具有如下特性的数据：
- 都是非负整数
- 没有重复的数据
- 数据量很大

<!--more-->

如何利用一种比快速排序还快的算法来给这组数据排序。

对于这组数据的特殊性，我们可以考虑利用位图的方法：首先我们申请一块内存空间并把这块内存的所有位都置0，然后依次读入这组数据，如果读入的数据为 i ，那么就这块内存的第 i 个bit为置为 1，数据读入完成之后，我们就可以从头开始一位一位地依次遍历这块内存，如果第 i 个bit位为 1 那么就输出 i ，最后输出的数据就是原始数据的从小到大的一个排序了。

代码：
```C
#include "stdio.h"
#define BITSPERWORD 32
#define SHIFT 5
#define MASK 0x1F
#define N 10000000

void set(int a[], int i);
void clr(int a[], int i);
int test(int a[], int i);
int a[N/BITSPERWORD];
int main(){
    int i,n,temp;
    scanf("%d",&n);
    for(i=0;i<N;i++)   // initialize
        clr(a,i);
    for(i=0;i<n;i++){   // input
        scanf("%d",&temp);
        set(a,temp);
    }
    for(i=0;i<N;i++)   // output
        if(test(a,i)) 
            printf("%d ",i);
    return 0;
}

void set(int a[], int i){ 
    a[i>>SHIFT] |=  (1<<(i & MASK));	
}
void clr(int a[], int i){
    a[i>>SHIFT]  &=  ~(1<<(i & MASK));
}

int test(int a[], int i){
    return a[i>>SHIFT] & (1<<(i & MASK));
}
```


## FAQ ##

Q: 如果每个数至多出现10次呢？

A: 可以用 4 个 bit 位来保存一个数出现的次数

Q: 如果不知道出现多少次呢？

A: 可以参考之前的一篇文章：[如何给数百万考生的成绩排序](../../../../2015/12/17/sort-the-score-of-students/)


<br/>
读 《编程珠玑》 笔记