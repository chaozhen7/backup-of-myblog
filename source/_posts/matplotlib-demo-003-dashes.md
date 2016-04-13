layout: post
title: matplotlib 画图（三）：线的连续与间断
date: 2016-3-19 10:20:22
tags: 
   - Matplotlib
   - 画图
comments: true
---

![](/img/articles/matplotlib/dashes.jpg)

<!--more-->

### **代码** ###

```python
"""
Demo of a simple plot with a custom dashed line.

A Line object's ``set_dashes`` method allows you to specify dashes with
a series of on/off lengths (in points).
"""
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 10)
line, = plt.plot(x, np.sin(x), '--', linewidth=2)
#line, = plt.plot(x, np.sin(x), '-', linewidth=2)

dashes = [10,5,100,5]  # 10 points on, 5 off, 100 on, 5 off
line.set_dashes(dashes)

plt.savefig("dashes.jpg",format="jpg")

```

`set_dashes()` 函数主要是设置线上点的显示与不显示。上面 dashes=[10,5,100,5]，表示的是线上的点`10 个显示5个不显示再100个显示5个不显示`，依次循环。如果 dashes=[10,5] 那么表示的就是`10个显示5个不显示`依次循环。

