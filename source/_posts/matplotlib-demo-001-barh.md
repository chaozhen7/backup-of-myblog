layout: post
title: matplotlib 画图（一）：横向柱状图
date: 2016-3-16 11:20:22
tags: 
   - Matplotlib
   - 画图
comments: ture
---

![](/img/articles/matplotlib/barh.jpg)

<!--more-->

## 代码 ##
```python
"""
Simple demo of a horizontal bar chart.
"""
import matplotlib.pyplot as plt
plt.rcdefaults()
import numpy as np
import matplotlib.pyplot as plt

# Example data
people = ('Tom', 'Dick', 'Harry', 'Slim', 'Jim')
y_pos = np.arange(len(people))
performance = 3 + 10 * np.random.rand(len(people))
error = np.random.rand(len(people))

plt.barh(y_pos, performance, xerr=error, align='center', alpha=0.4)
plt.yticks(y_pos, people)
plt.xlabel('Performance')
plt.title('How fast do you want to go today?')
plt.savefig("barh.eps",format="eps")

```


## 解析 ##

**plt.rcdefaults()**

恢复 rc 的默认设置

**barh()**

**主要功能：**做一个横向条形图，横向条的矩形大小为: left, left + width, bottom, bottom + height

**参数：**barh ( bottom , width , height =0.8, left =0, ** kwargs )

**返回类型：**一个 class 类别，`matplotlib.patches.Rectangle`实例

**参数说明：**
- bottom: Bars 的垂直位置的底部边缘
- width: Bars 的长度


**可选参数：**
- height: bars 的高度
- left: bars 左边缘 x 轴坐标值
- color: bars 颜色
- edgecolor: bars 边缘颜色
- linewidth: bar 边缘宽度;None 表示默认宽度;0 表示不 i 绘制边缘
- xerr: 若不为 None,将在 bar 图上生成 errobars
- yerr: 若不为 None,将在 bar 图上生成 errobars
- ecolor: 指定 errorbar 颜色
- capsize: 指定 errorbar 的顶部(cap)长度
- align: ‘edge’ (默认) | ‘center’:‘edge’以底部为准对齐;‘center’以 y 轴作为中心?
- log: [False|True] False (默认),若为 True,使用 log 坐标






