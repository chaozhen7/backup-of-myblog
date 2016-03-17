layout: post
title: matplotlib 画图（二）：图形填充
date: 2016-3-17 19:20:22
tags: 
   - Matplotlib
   - 画图
comments: ture
---

![](/img/articles/matplotlib/fill.jpg)

<!--more-->

### **代码** ###

```python
"""
Simple demo of the fill function.
"""
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 1)
y = np.sin(4 * np.pi * x) * np.exp(-5 * x)

plt.fill(x, y, 'r')
plt.grid(True)
plt.savefig("fill.jpg",format="jpg")
```

注： grid 表示是否显示图轴网格。函数原型 `matplotlib.pyplot.grid(b=None, which='major', axis='both', **kwargs)`

调用形式： `grid(self, b=None, which='major', axis='both', **kwargs)`
主要参数：
- b： [True|False]或者是布尔数组、或[‘on’,‘off’] 表示网格是否开启
- which： [major(默认)|minor|both] 选择主、次网格开启方式
- axis： [both(默认)|x|y] 选择使用网格的数轴


---

![](/img/articles/matplotlib/fill_features.jpg)


### **代码** ###

```python
"""
Demo of the fill function with a few features.

In addition to the basic fill plot, this demo shows a few optional features:

    * Multiple curves with a single command.
    * Setting the fill color.
    * Setting the opacity (alpha value).
"""
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 100)
y1 = np.sin(x)
y2 = np.sin(3 * x)
plt.fill(x, y1, 'b', x, y2, 'r', alpha=0.3)
plt.savefig("fill_features.jpg",format="jpg")

```

注：alpha 表示透明度，0表示全透明，1表示完全不透明。