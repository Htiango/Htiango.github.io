---
title: 从EM算法到GMM模型
date: 2017-03-08 21:09:41
tags: [EM,GMM,algorithm]
categories: 
- Machine Learning
- Algorithms
---

最近温习了下机器学习经典算法之一的EM算法，顺便对GMM模型有了更加深入的理解，本文将对这两个概念进行简要介绍。
<!-- more -->

## 最大似然估计（MLE）
Maximum Likelihood Estimation

在介绍EM算法之前，不得不提最大似然估计。

最大似然估计可以解释为：

1. 假设 $ p (x | \omega) $ 是一个有参数向量 $\theta $ 唯一确定的概率分布
2. 参数 $\theta $ 是固定但是未知的。
3. 假设我们有一个按照概率分布 $ p (x | \omega) $ 的数据集 D，D 中的样本彼此独立。
4. MLE 就是 $\theta $ 的一个最能解释描述该数据集的值。

我们也可以通俗地解释为：

最大似然估计一般用于求分布参数 $\theta $ : 给定一个概率分布，我们从概率分布中抽n个值的采样，通过这些采样数据来估计概率分布的参数 $\theta $，定义似然函数如下所示：

$$
\begin{eqnarray}
lik(\theta ) & = & f_{D}(x_{1},x_{2},...,x_{n} | \theta ) \\
& = & \prod_{k = 1}^{N}p(x_k| \theta ))
\end{eqnarray}
$$

$$
\theta' = \arg \max_{\theta }\{p(Data|\theta )\}
$$

在 $\theta $ 的所以取值上令一阶导数等于0，使得这个函数取到最大值，这个使可能性最大的 $ \theta ' $ 值即为 $\theta $ 的最大似然估计。

最大似然估计是建立在这样的思想上：已知某个参数能使这个样本出现的概率最大，我们就把这个参数作为估计的真实值。

那么我们要如何来解决ML estimate的问题呢？

1. 我们先设 $\mathbf{\theta }$ 为一个p元素的向量 $\mathbf{\theta } = \[\theta_1,\theta_2,...,\theta_p\]^T$
2. 让其成为梯度算子 $ \bigtriangledown_\theta =\[\frac{\partial }{\partial \theta_1}, \frac{\partial }{\partial \theta_2},..., \frac{\partial }{\partial \theta_p} \] $
3. 已有：$ p (x | \mathbf{\theta}) = \prod_{k=1}^{n}p(x_k | \theta)) $
4. 我们定义 $ l(\mathbf{\theta})$ 为最大似然函数,以及对应的梯度算子：
$$ 
l(\mathbf{\theta}) = log(p(D |\mathbf{\theta})) = \sum_{k = 1}^{n}log(p(x_k | \mathbf{\theta}))
$$
$$
\bigtriangledown_\theta l(\mathbf{\theta}) =\bigtriangledown_\theta log(p(D |\mathbf{\theta})) = \sum_{k = 1}^{n}\bigtriangledown_\theta log(p(x_k | \mathbf{\theta}))
$$
5. 我们通过算子为0得到最大似然估计：
$$
\bigtriangledown_\theta l(\mathbf{\theta}) = 0
$$ 

ML主要应用于单高斯和多高斯中。

## EM算法
Expectation-Maximum Algorithm

EM算法一般用于在概率模型中找最大似然估计，主要针对含有潜在变量的，将不完全的数据补成完全的。

对于含有潜在变量的，如果我们还是按照求最大似然估计的方法来解决的话（分别求偏导），会出现“和的对数”这种难以解决的情况。对于这种情况，我们要用辅助函数来帮助解决。

![](https://oh1ulkf4j.qnssl.com/Screen%20Shot%202017-03-09%20at%207.21.41%20PM.png)

我们定义一个辅助函数 $ A(x,x^t)$ 与 f(x) 在 $x^t$ 处相等，且满足 $f(x) \geq A(x,x^t) $

我们只要将辅助函数的最大值设为新的 $\theta$ ，通过多次迭代逐渐逼近 f(x) 的最大值。

我们可以利用Jensen不等式来推导：

根据Jensen不等式，我们可以得到，如果f是凸函数，x是随机变量，则：

$$
E\[f(x)\] \geq f(E\[x\])
$$

如果是凹函数则反之

因此我们可以推导得出一个辅助函数：(将潜在变量引入其中，log函数是凹函数)

$$	
\begin{eqnarray}
\log{p(\mathbf{X} | \Theta)} & = & \log{\sum_y{p(\mathbf{x},y | \Theta)}} \\
& = & \log{q(y)\frac{\sum_y{p(\mathbf{x},y | \Theta)}}{q(y)}} \\
& \geq & \sum_y q(y) \log(\frac{p(\mathbf{x},y | \Theta)}{q(y)})
\end{eqnarray}
$$

因此，EM算法具体可以分为以下四个步骤：

1. 对参数进行初始化设置，为 $\Theta^{old}$
2. E-step: 估计出 $p(y | x, \Theta^{old})$
$$
Q(\Theta, \Theta^{old}) = \sum_y{p(y | x, \Theta^{old}) \log(p(\mathbf{x},y|\Theta))}
$$
3. M-step: 用最大似然估计来估计$ \Theta^{new}$
$$
\Theta^{new} = \arg \max_\Theta Q(\Theta, \Theta^{old})
$$
4. 检查是否收敛，如果不是$\Theta^{new} \rightarrow \Theta^{old}$ 并返回步骤2

EM 模型主要用于高斯混合模型 GMM 中。

## GMM模型
在一个多高斯混合模型中，数据是从多个单高斯中生成出来的，即：
$$
P(x) = \sum_{k=1}^{K} \pi_k N(x;u_k,\Sigma_k)
$$
其中，$\pi_k$ 是权重值，总的和为1.

对于一个GMM模型，如果K的值足够大的话，混合模型就可以逼近任意的连续概率分布。

GMM模型本质上是一种聚类算法，每个高斯分布就是一个聚类中心。和K-means不同的在于，GMM是采用概率模型来表达聚类原型，因此可以给出一个样本属于某类的概率大小。

在每个样本的分类确定的情况下，GMM的参数可以直接用MLE的方法来确定。但是在只知道样本点而不知道其分类情况下，我们会得到一个这样的log-likehood的函数：
$$
\sum_{i = 1}^N \log(\sum_{k = 1}^K \pi_k N(x;u_k,\Sigma_k))
$$
在这个似然函数中，由于每个样本$x_i$所属的类别$z_k$是属于隐含变量的未知值，所以我们需要用EM算法来计算出模型的参数（使得上式的期望最大），然后可以用这些算好的参数来对样本进行分类。

E-step: 假设模型参数已知，求隐含变量分类Z的期望，即个个高斯分布的概率

M-step: 根据E-step中求得的分类，用MLE的方法求得参数。




