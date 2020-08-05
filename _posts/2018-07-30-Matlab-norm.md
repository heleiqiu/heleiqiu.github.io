---
layout:     post
title:      "向量与矩阵的范数及其在Matlab中的用法(norm)"
subtitle:   ""
author:     "heleiqiu"
header-style: text
tags: [Matlab, 范数, 矩阵, 向量]
mathjax: true
---


## 一、常数向量范数
* $L_0$ 范数

$\Vert x \Vert _0\overset{def}=向量中非零元素的个数$
  
其在matlab中的用法：
```matlab
sum( x(:) ~= 0 )
```

* $L_1$ 范数
$\Vert x \Vert_1\overset{def} = \sum\limits_{i=1}^{m} \vert x_{i}\vert = \vert x_{1}\vert + \cdots +\vert x_{m}\vert$，即向量元素绝对值之和

其在matlab中的用法：
```matlab
norm(x, 1)
```

* $L_2$ 范数
$\Vert x \Vert_2=(\vert x_1\vert^2+\cdots+\vert x_m\vert^2)^{1/2}$，即向量元素绝对值的平方和后开方

其在matlab中的用法：
```matlab
norm(x, 2)
```

* $L_{\infty}$ 范数
* 极大无穷范数
$\Vert x \Vert_{\infty}= max \{\vert x_1\vert, \cdots,\vert x_m\vert\}$，即所有向量元素绝对值中的最大值

其在matlab中的用法：
```matlab
norm(x, inf)
```

* 极小无穷范数
$\Vert x \Vert_{\infty}= min \{\vert x_1 \vert, \cdots, \vert x_m\vert\}$，即所有向量元素绝对值中的最小值

其在matlab中的用法：
```matlab
norm(x, -inf)
```

## 二、矩阵范数
诱导范数和元素形式范数是矩阵范数的两种主要类型。
### 1. 诱导范数

* $L_1$ 范数（列和范数）
$\Vert A \Vert_1= \underset{1\leqslant j\leqslant n}{\mathop{\max }}\sum\limits_{i=1}^{m}\{ \vert a_{ij}\vert \}$，即所有矩阵列向量绝对值之和的最大值

其在matlab中的用法：
```matlab
norm(A,1)
```

* $L_2$ 范数
$\Vert A \Vert_2=\sqrt{\lambda _{i}}$，其中 $\lambda_i$ 为 $A^{T}A$ 的最大特征值。

其在matlab中的用法：
```matlab
norm(A,2)
```

* $L_{\infty}$ 范数（行和范数）
$\Vert A \Vert_{\infty}= \underset{1\leqslant i\leqslant m}{\mathop{\max }}\sum\limits_{j=1}^{n}\{\vert a_{ij}\vert\}$，即所有矩阵行向量绝对值之和的最大值

其在matlab中的用法：
```matlab
norm(A,inf)
```

### 2. "元素形式"范数

* $L_{0}$ 范数
$\Vert A \Vert_0\overset{def}=矩阵的非零元素的个数$

其在matlab中的用法：
```matlab
sum(sum(A ~= 0))
```

* $L_{1}$ 范数
$\Vert A \Vert_1\overset{def}=\sum\limits_{i=1}^{m}\sum\limits_{j=1}^{n}\vert a_{ij}\vert$，即矩阵中的每个元素绝对值之和

其在matlab中的用法：
```matlab
sum(sum(abs(A)))
``` 

* $L_{F}$ 范数
$\Vert A \Vert_F\overset{def}=(\sum\limits_{i=1}^{m}\sum\limits_{j=1}^{n}\vert a_{ij}\vert^2)^{1/2}$，即矩阵的各个元素平方之和后开方

其在matlab中的用法：
```matlab
   norm(A,fro)
``` 

* $L_{\infty}$ 范数
$\Vert A \Vert_{\infty}= \underset{i=1,\cdots,m;\ j=1,\cdots,n}{\mathop{\max }}\{\vert a_{ij}\vert \}$，即矩阵的各个元素绝对值的最大值

其在matlab中的用法：
```matlab
max(max(abs(A)))
``` 

* 核范数
$\Vert A \Vert_{*}= \sum\limits_{i=1}^{n}\lambda_i$，$\lambda_i$ 为 $A$ 的奇异值，即所有矩阵奇异值之和

其在matlab中的用法：
```matlab
sum(svd(A))
``` 
