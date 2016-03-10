layout: post
title: C语言中的文件操作
date: 2015-8-20 16:40:22
tags: 
	- C语言
comments: true
toc: true
---
## 1. 基本概念

__文件__：记录在外部介质上的数据的集合。

__分类__：
- 文本文件（存储的是ASCII码）
- 二进制文件（存储的是二进制）

__文件的指针__：指向一个结构体的指针。（结构体主要包括：缓冲区地址、缓冲区当前存储的字符位置、对当前文件读还是写、是否出错等）
<!--more-->

## 2. 打开和关闭文件

### **打开文件的函数fopen** ###
   
格式：<code>fopen(文件名，文件打开方式)</code>
例如：
```c
FILE *fp = fopen(“file_a”,”r”);
```

打开文件错误时返回NULL.

打开方式: r, rb, w, wb, a,ab, r+, rb+, w+, wb+, a+, ab+

r : 表示读取，

a : 表示添加，

b : 表示二进制文件，

r+ : 表示读写（在文件末尾添加数据）

w+ : 文件不存在时创建文件，文件存在时，删除原有文件内容重新写文件

C语言自动打开的三个文件：

- stdin: 连接键盘控制的文件
- stdout、stderr: 连接屏幕的

### **关闭文件的函数fclose** ###
   
关闭文件使用fclose函数，使文件指针和文件脱离关系
格式：<code>fclose(文件指针)</code>
成功关闭返回0，否则返回非0


## 3. 文件操作的相关函数
### **getc 和putc 函数**  ### 
putc()格式：<code>putc(字符，文件指针)</code>
输出成功返回所输出的字符；失败返回EOF（End Of the File, stdio.h中定义的符号常量，-1）
getc()格式：<code>字符变量 = getc(文件指针)</code>

### **判断文件结束函数feof** ###
  
只适用于文本文件，因为ASCII码不可能出现-1，二进制形式存储文件就有可能出现-1
格式：<code>feof(文件指针)</code>
返回值为1表示文件结束，返回值为0表示文件还没结束

### **fscanf 和fprintf 函数** ###   

fscanf：只能从文本文件中按格式输入
格式：<code>fscanf(文件指针，格式控制字符串，输入项表)</code>

例如：
```C
fscanf(stdin,”%d”,&a);    //等价于 scanf(“%d”,&a);
```

fprintf：把内存中的数据转换为字符，以ASCII码形式输出到文本文件中
格式：<code>fprintf(文件指针，格式控制字符串，输入项表)</code>
例如：
```C
fprintf(stdout,”%d”,a);   //等价于printf(“%d”,a)
```

### **fgets 和 fputs 函数 ** ###

fgets：用于从文件中读入字符串
格式：<code>fgets(字符串起始地址，字符串长度，文件指针)</code>
如果字符串长度N，那么最多读入N-1个字符

fputs：用于把字符串输出到文件
格式：<code>fputs(字符串起始地址，文件指针);</code>

### **fread 和fwirte函数 ** ###

主要用于读取二进制文件
格式：
<code>fread(buffer,size,count,fp)</code>
<code>fwrite(buffer,size,count,fp)</code>
buffer：数据块指针，size：数据块字节数，count：数据块个数，fp：文件指针

### **文件定位函数** ###
   
文件位置指针：当前读写的数据在文件中的位置
fseek：移动文件位置指针到指定位置。
格式： <code>fseek(文件指针，移动的字节数的长整型，起始点)</code>
SEEK_SET：0，文件开始；
SEEK_END：2，文件结束；
SEEK_CUR：1，文件当前位置

文件为二进制时，移动字节数为正负表示方向，文件为文本文件时，位移量只能是0。
ftell：获取当前文件位置指针的位置。
rewind：反绕函数，位置指针回到文件开头。