---
title: Deep Learning Chapter 2
date: 2017-03-09 23:39:55
tags: [deep learning,线性代数]
categories: [读数笔记]
---

在Deep Learning一书的第二章，主要介绍了线性代数的基本思想。

<!-- more -->

## 基本概念
标量：单独的数

向量：一列数

矩阵：二维数组

张量：多维数组

在深度学习中，我们有时允许矩阵和向量相加，产生另一个矩阵：
$\mathbf{C} = \mathbf{A} + \mathbf{b} $，其中 $C_{i,j} = A_{i,j} + b_j$， 即向量b和矩阵A的每一行相加。

矩阵中元素对应的乘积表示为： $\mathbf{A \odot B}$

同时需要注意的一点是，只有非奇异的方阵才有逆。所谓非奇异也就是说，方阵的所有列向量都是线性无关的。一个列向量线性相关的方阵被称为奇异的。

## 范数
$L^p$ 范数的定义如下：
$$
\left \| \mathbf{x} \right \|_p = (\sum_i{|x_i|^p})^{\frac{1}{p}}
$$

范数是将向量映射到非负值的函数。向量x的范数是衡量从原点到点x的距离。其需要满足下列三个条件：

1. $f(\mathbf{x}) = 0 \Rightarrow \mathbf{x} = \mathbf{0}$
2. $f(\mathbf{x} + \mathbf{y}) \geq f(\mathbf{x}) + f(\mathbf{y})$ (三角不等式)
3. $ \forall \alpha \in \mathbb{R}, f(\alpha \mathbf{x}) = |\alpha| f(\mathbf{x}) $

p = 2时，$L^2$被称为欧几里得范数，表示的是原点和x之间的欧氏距离。

平方$L^2$范数： $\mathbf{x}^T x$。

平方$L^2$范数较之$L^2$范数更加的方便，因为前者的导数只和对应元素相关，而后者的导数却是和整个向量相关。

但是有时我们仍然会遇到在原点附近增长缓慢的情况，对于一些区分零和非零小值的机器学习应用而言是十分致命的。因此很多时候会采用$L^1$范数，每当x中的某一元素增加了一个小量时，对应的防暑也会增加这个小量。

深度学习中，我们有时为了衡量矩阵的大小，使用类似$L^2$范数的Frobenius范数：

$$
\left \| \mathbf{A} \right \|_F = \sqrt{\sum_i{A_{i,j}^2}}
$$


## 正交矩阵
正交矩阵是指行向量和列向量都是**标准正交**的方阵：
$$
\mathbf{A}^T \mathbf{A} = \mathbf{A} \mathbf{A}^T = \mathbf{I}
$$
即：
$$
\mathbf{A}^{-1} = \mathbf{A}^T
$$

由于正交矩阵的求逆计算代价很小，所以在机器学习中经常得到运用

## 特征分解
所谓特征分解就是讲矩阵分解为一组特征向量和特征值。
方阵**A**的特征向量是指与**A**相乘后等于对该向量进行缩放的非零向量**v**：（我们一般只考虑单位特征向量）
$$
\mathbf{A v} = \lambda \mathbf{v}  
$$
其中的标量λ被称为这个特征向量对应的特征值。

假设矩阵**A**有n个线性无关的特征向量$\{\mathbf{v^1},...,\mathbf{v^n}\}$，对应着特征值$\{\lambda_1,...\lambda_n\}$，我们将特征向量连接为一个矩阵，使得每一列是一个特征向量：$\mathbf{V} = \{\mathbf{v^1},...,\mathbf{v^n}\}$，将特征向量连接成一个向量$\mathbf{\lambda} = \{\lambda_1,...\lambda_n\}^T$
A的特征分解：
$$
\mathbf{A} = \mathbf{V} diag(\mathbf{\lambda})\mathbf{V}^{-1}
$$

但是，不是每一个矩阵都可以分解为特征值和特征向量的。只有实对称矩阵才可以分解为实特征向量和实特征值，但是分解结果可能不唯一。如果两个或多个特征向量拥有相同的特征值，那么这组特征向量生成子空间中，任意一组正交向量都是该特征值对应的特征向量。
$$
\mathbf{A} = \mathbf{Q} \Lambda \mathbf{Q}^{T}
$$
上式中Q是A的特征向量组成的正交矩阵，$\Lambda$是对角矩阵。因为Q是正交矩阵，我们可以将A看作是沿方向$\mathbf{v}^{i}$延展$\lambda_i$倍的空间

![Screen Shot 2017-03-14 at 12.54.28 A](https://oh1ulkf4j.qnssl.com/Screen Shot 2017-03-14 at 12.54.28 AM.png)



## 奇异值分解 SVD
奇异值分解也是一种分解矩阵的方法，将矩阵分解为奇异向量和奇异值，可以得到一些类似特征分解的信息。

SVD的应用更广，对于没有特征分解的非方阵矩阵，只能使用奇异值分解。

$$
\mathbf{A} = \mathbf{U D V}^T
$$

U和V都是正交矩阵，D是对角矩阵。

矩阵U的列向量是左奇异向量，是$\mathbf{A} \mathbf{A}^T$的特征向量
矩阵V的列向量是右奇异向量，是$\mathbf{A}^T \mathbf{A}$的特征向量

SVD主要用于拓展矩阵求逆到非方矩阵上。

## Moore-Penrose 伪逆
非方矩阵的逆矩阵没有定义，对于下列的问题我们希望可以用矩阵A的左逆B来求解线性方程。

$$
\mathbf{A x} = \mathbf{y} 
$$

$$
\mathbf{x} = \mathbf{B y}
$$

如果矩阵A的行数大于列数，那么上述方程可能没有解。如果矩阵A的行数小于列数，那么上述矩阵可能有多个解。

Moore-Penrose 伪逆定义：
$$
\mathbf{A}^+ = \lim_{a \rightarrow 0}(\mathbf{A}^T \mathbf{A} + \alpha \mathbf{I})^{-1} \mathbf{A}^T
$$

$$
\mathbf{A}^+ = \mathbf{V} \mathbf{D}^+ \mathbf{U}^T
$$

其中矩阵U、D、V是由A奇异值分解后得到的矩阵，对角矩阵D的伪逆$\mathbf{D}^+$是其所有非零元素取导数之后再转置得到的。

## PCA 应用

