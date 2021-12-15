---
title: 解决jupyter中matplotlib乱码问题
date: 2021-01-26
categories: 数学建模
tags: 计算机小技巧
top: false
---

原因：由于matplotlib不默认支持中文，所以当不进行默认设置的时候会显示乱码（麻将字）

解决方案：在导入plt时加入如下参数设计
```python
import matplotlib.pyplot as plt
plt.rcParams[font.family] = 'SimHei'        #将字体设置成黑体
plt.rcParams[axes.unicode_minus] = False    #解决坐标轴符号变成方块的问题
```

建议：在导包时这几条就要先行导入，方便作图