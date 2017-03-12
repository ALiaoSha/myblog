---
title: numpy array的存储和读读取方式
date: 2017-03-08 15:35:32
tags: tech
categories: 技术
toc: true
mathjax: true
---

array 是 numpy 中的一种重要的数据类型。它可以被赋予任意维度，这也是它与numpy matrix不同的地方
(np.matrix只能是二维的矩阵)
# numpy array 的存储方式
np.array的存储遵循的是维度由后到前的方式.
对一个$ m \times n \times k $ 的矩阵, numpy 把它存储为m个 $n\times k$ 维的矩阵，每个 $n\times k$维的矩阵被存为n个k维的向量
```

In [3]: np.arange(60).reshape(3,4,5)
Out[3]:
array([[[ 0,  1,  2,  3,  4],
        [ 5,  6,  7,  8,  9],
        [10, 11, 12, 13, 14],
        [15, 16, 17, 18, 19]],

       [[20, 21, 22, 23, 24],
        [25, 26, 27, 28, 29],
        [30, 31, 32, 33, 34],
        [35, 36, 37, 38, 39]],

       [[40, 41, 42, 43, 44],
        [45, 46, 47, 48, 49],
        [50, 51, 52, 53, 54],
        [55, 56, 57, 58, 59]]])
```
# numpy array 的读取方式
