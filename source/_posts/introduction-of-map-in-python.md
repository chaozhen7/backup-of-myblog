layout: post
title: python 并行化之 map 函数的介绍
date: 2016-3-1 12:20:22
tags: 
	- Python
comments: true
copyright: true
---

关于 python 并行化编程有一个特别简洁的做法就是使用 map 函数。

map 函数一手包办了序列操作，参数传递和结果保存等一系列的操作。

python 在 Pool 中封装了 map 方法，所以使用起来也特别方便。

<!--more-->


举个简单的示例：
```python
from multiprocessing.dummy import Pool as ThreadPool

def printStr(str):
    print(str)
    return str

if __name__ == '__main__':
    strList = ['hello1','hello2','hello3','hello4']
    #两行代码即可搞定
    pool = ThreadPool(4)  #这里的4可以是cpu的核数
    results = pool.map(printStr,strList)

    print("结果")
    for each in results:
        print(each)
```

结果：

> hello1
hello2
hello3
hello4
结果
hello1
hello2
hello3
hello4





