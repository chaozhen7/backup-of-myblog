layout: post
title: 分解质因数
date: 2015-3-29 21:35:22
tags: 
	- 算法
	- 数论
comments: true
---
请尊重原创作者，[原文链接](http://www.2cto.com/kf/201311/258904.html)

## 1. 原理和方法
把一个合数分解为若干个质因数的乘积的形式，即求质因数的过程叫做分解质因数，分解质因数只针对合数。
求一个数分解质因数，要从最小的质数除起，一直除到结果为质数为止。分解质因数的算式的叫短除法，和除法的性质差不多，还可以用来求多个个数的公因式：
以24为例：
2 -- 24
2 -- 12
2 -- 6
3 （3是质数，结束）
得出 $24=2\*2\*2\*3 = 2^3\*3$
可先用素数筛选法，筛选出符合条件的质因数，然后for循环遍历即可，通过一道题目来show一下这部分。
<!--more-->
### 题目描述：  
求正整数N(N>1)的质因数的个数。  
相同的质因数需要重复计算。如 $120=2\*2\*2\*3\*5$，共有5个质因数。  
### 输入：  
可能有多组测试数据，每组测试数据的输入是一个正整数N，($1 \le N \le 10^9$)。  
### 输出：  
对于每组数据，输出N的质因数的个数。  
### 样例输入：  
120  
### 样例输出：  
5  
### 提示：  
注意：1不是N的质因数；若N为质数，N是N的质因数。  
``` C
#include <stdio.h>  
int main()  
{  
    int n, count, i;  
    while (scanf("%d", &n) != EOF) {  
        count = 0;  
        for (i = 2; i * i <= n; i ++) {  
            if(n % i == 0) {  
                while (n % i == 0) {  
                    count ++;  
                    n /= i;  
                }  
            }  
        }  
        if (n  > 1) {  
            count ++;  
        }  
        printf("%d\n", count);  
    }  
    return 0;  
}  
```


## 2. 深入理解
我所谓的深入理解，就是通过4星的题目来灵活运用分解质因数的方法，题目如下
### 题目描述：  
给定n，a求最大的k，使n！可以被$a^k$整除但不能被$a^{k+1}$整除。  
### 输入：  
两个整数$n(2 \le n \le1000)，a(2 \le a \le1000)$  
### 输出：  
一个整数.  
### 样例输入：  
6 10  
### 样例输出：  
1  
### 思路
$a^k$ 和 $n!$ 都可能非常大，甚至超过long long int的表示范围，所以也就不能直接用取余操作判断它们之间是否存在整除关系，因此我们需要换一种思路，从分解质因数入手，假设两个数a和b：
$ a=p_1^{e_1} \* p_2^{e_2} \* ... \* p_n^{e_n} $   
$b = p_1^{d_1} \* p_2^{d_2} \* ... \* p_n^{d_n}$
则b除以a可以表示为：
$b/a=p_1^{d_1} \* p_2^{d_2} \* ... \* p_n^{d_n}/p_1^{e_1} \* p_2^{e_2} \* ... \* p_n^{e_n}$
若b能被a整除，则 b / a必为整数，且两个素数必互质，则我们可以得出如下规律：
若a存在质因数$p_x$，则b必也存在该质因数，且该素因数在b中对应的幂指数必不小于在a中的幂指数。
另$b=n!$, $a^k=p_1^{ke_1} \* p_2^{ke_2} \* ... \* p_n^{ke_n}$，因此我们需要确定最大的非负整数k即可。要求得该k，我们只需要依次测试a中每一个素因数，确定b中该素因数是a中该素因数的幂指数的多少倍即可，所有倍数中最小的那个即为我们要求得的k
分析到这里，剩下的工作似乎只是对a和n!分解质因数，但是将n!计算出来再分解质因数，这样n!数值太大。考虑n!中含有素因数p的个数，即确定素因数p对应的幂指数。我们知道n!包含了从1到n区间所有整数的乘积， 这些乘积中每一个p的倍数（包括其本身）都对n!贡献至少一个p因子，且我们知道在1到n中p的倍数共有n/p个。同理，计算 $p^2，p^3,...$ 即可
``` C
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
   
#define N 1001  
   
int prime[N], size;  
   
/**  
 * 素数筛选法进行预处理 
 */  
void initProcess()  
{  
    int i, j;  
       
    for (prime[0] = prime[1] = 0, i = 2; i < N; i ++) {  
        prime[i] = 1;  
    }  
    size = 0;  
    for (i = 2; i < N; i ++) {  
        if (prime[i]) {  
            size ++;  
            for (j = 2 * i; j < N; j += i) {  
                prime[j] = 0;  
            }  
        }  
    }  
}  
int main(void)  
{  
    int i, n, a, k, num, count, base, tmp, *ansbase, *ansnum;  
       
    // 预处理  
    initProcess();  
   
    while (scanf("%d %d", &n, &a) != EOF) {  
        ansbase = (int *)calloc(size, sizeof(int));  
        ansnum = (int *)calloc(size, sizeof(int));  
   
        // 将a分解质因数  
        for (i = 2, num = 0; i < N && a != 1; i ++) {  
            if (prime[i] && a % i == 0) {  
                ansbase[num] = i;  
                ansnum[num] = 0;  
                   
                while (a != 1 && a % i == 0) {  
                    ansnum[num] += 1;  
                    a = a / i;  
                }  
   
                num ++;  
            }  
        }  
   
        // 求最小的k  
        for (i = 0, k = 0x7fffffff; i < num; i ++) {  
            base = ansbase[i];  
            count = 0;  
            while (base <= n) {  
                count += n / base;  
                base *= ansbase[i];  
            }  
            tmp = count / ansnum[i];  
            if (tmp < k) k = tmp;  
        }  
   
        printf("%d\n", k);    
    }  
  
    return 0;  
}  
```
## 3. 约数个数定理
对于一个大于1的正整数n可以分解质因数：
$n=p_1^{a_1} \* p_2^{a_2} \* ... \* p_n^{a_n}$,
则n的正约数的个数为： $(a_1 + 1) \* (a_2 + 1) \* ... \*(a_n + 1)$.其中$p_1,p_2,..p_n$都是n的质因数，$a_1, a_2...a_n$ 是 $p_1,p_2,..p_n$的指数。
证明：
n可以分解质因数：$n=p_1^{a_1} \* p_2^{a_2} \* ... \* p_n^{a_k}$,
由约数定义可知 $p_1^{a_1}$ 的约数有: $p_1^0, p_1^1, p_1^2......p_1^{a_1}$ ，共 $(a_1+1)$ 个
同理 $p_2^{a_2}$的约数有 $(a_2+1)$ 个，......，$p_k^{a_k}$ 的约数有$(a_k+1)$个。
故根据乘法原理：n的约数的个数就是$(a_1+1)\*(a_2+1)\*(a_3+1)\*…\* (a_k+1)$个。
### 题目描述：  
输入n个整数,依次输出每个数的约数的个数  
### 输入：  
输入的第一行为N，即数组的个数(N<=1000)  
接下来的1行包括N个整数，其中每个数的范围为(1<=Num<=1000000000)  
当N=0时输入结束。  
### 输出：  
可能有多组输入数据，对于每组输入数据，  
输出N行，其中每一行对应上面的一个数的约数的个数。  
### 样例输入：  
5  
1 3 4 6 12  
### 样例输出：  
1  
2  
3  
4  
6
``` C  
#include <stdio.h>  
#include <stdlib.h>  
   
#define N 40000  
   
typedef long long int lint;  
   
int prime[N], size;  

void init()  
{  
    int i, j;  
   
    for (prime[0] = prime[1] = 0, i = 2; i < N; i ++) {  
        prime[i] = 1;  
    }  
       
    size = 0;  
   
    for (i = 2; i < N; i ++) {  
        if (prime[i]) {  
            size ++;  
            for (j = 2 * i; j < N; j += i)  
                prime[j] = 0;  
        }  
    }  
}  
lint numPrime(int n)  
{  
    int i, num, *ansnum, *ansprime;  
    lint count;  
   
    ansnum = (int *)malloc(sizeof(int) * (size + 1));  
    ansprime = (int *)malloc(sizeof(int) * (size + 1));  
   
    for (i = 2, num = 0; i < N && n != 1; i ++) {  
        if (prime[i] && n % i == 0) {  
            ansprime[num] = i;  
            ansnum[num] = 0;  
            while (n != 1 && n % i == 0) {  
                ansnum[num] += 1;  
                n /= i;  
            }  
            num ++;  
        }  
    }  
   
    if (n != 1) {  
        ansprime[num] = n;  
        ansnum[num] = 1;  
        num ++;  
    }  
   
    for (i = 0, count = 1; i < num; i ++) {  
        count *= (ansnum[i] + 1);  
    }  
   
    free(ansnum);  
    free(ansprime);  
   
    return count;  
}  
int main(void)  
{  
    int i, n, *arr;  
    lint count;  
   
    init();  
   
    while (scanf("%d", &n) != EOF && n != 0) {  
        arr = (int *)malloc(sizeof(int) * n);  
        for (i = 0; i < n; i ++) {  
            scanf("%d", arr + i);  
        }  
   
        for (i = 0; i < n; i ++) {  
            count = numPrime(arr[i]);  
            printf("%lld\n", count);  
        }  
   
        free(arr);  
    }  
   
    return 0;  
}  
```