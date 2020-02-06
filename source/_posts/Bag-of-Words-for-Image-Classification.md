---
title: Bag of Words for Image Classification
date: 2017-03-22 20:07:38
tags: [image processing,BoW,algorithm,classification,NLP]
categories: 
- Machine Learning
- Computer Vision
---

最近了解到了一种很有意思的关于图像分类的算法，用bag of words的思想来对图像进行处理，很巧妙地将原本是图像处理的问题变换成了自然语言处理的问题。本文将简要对这种方法进行介绍。
<!-- more -->

## Bag of Words (BoW) 词袋模型原理
Bag of words model (BoW model) 最早出现在NLP和IR领域。该模型将一段文字视为一系列无序的单词，忽略掉文本的语法和语序。近年来，BoW模型被广泛应用于计算机视觉中。与应用于文本的BoW类比，图像的特征被当作单词。


在BoW中，我们统计的是一个词在一份文档中出现的次数，比如：

```
(1) John likes to watch movies. Mary likes movies too.
(2) John also likes to watch football games.
```

中有10个不同的词，分别可以被构建为一个如下的向量：

```
(1) [1, 2, 1, 1, 2, 0, 0, 0, 1, 1] 
(2) [1, 1, 1, 1, 0, 1, 1, 1, 0, 0] 
```

在邮件过滤中，这种方法有着很好地应用。

在[浅谈主题模型](https://tianyuh.com/2017/03/28/%E6%B5%85%E8%B0%88%E4%B8%BB%E9%A2%98%E6%A8%A1%E5%9E%8B%EF%BC%88%E4%B8%80%EF%BC%89/)中，我所介绍的TF-IDF在本质上也是一种BoW模型，不同在于加上了Term Weighting。


## 提取图像特征
机器学习的第一步也是最重要的一步就是提取特征，这关乎到随后一系列工作中数据的质量问题。在BoW处理图像的过程中，我们可以用Filter Bank或是SIFT等方法来提取

### Filter Bank
用不同的滤波器对图像进行滤波，得到一系列滤波处理后的图片，随机选取一些pixel，组成一列，然后将所有滤波后的图片一起组成一个矩阵。矩阵中每个元素的值是rgb三值。以此构成的矩阵中的每一行是一个特征向量。

本文下述的相关操作都是基于filter bank的方法来做的。

### SIFT
SIFT特征虽然也能描述一幅图像，但是每个SIFT矢量都是128维的，而且一幅图像通常都包含成百上千个SIFT矢量，在进行相似度计算时，这个计算量是非常大的，通行的做法是用聚类算法对这些矢量数据进行聚类，然后用聚类中的一个簇代表BoW中的一个视觉词，将同一幅图像的SIFT矢量映射到视觉词序列生成码本，这样每一幅图像只用一个码本矢量来描述，这样计算相似度时效率就大大提高了。

在BoW中我们对每一幅图像提取SIFT特征（每一幅图像提取多少个SIFT特征不定）。每一个SIFT特征用一个128维的描述子矢量表示。每一个SIFT特征即为特征向量。

## 构建词库
创建完特征向量后，我们需要将特征向量和词对应起来（map to words）。

我们可以用k-means的方法对特征向量进行聚类，用欧氏距离表示cluster和特征点之间的距离，通过多次迭代得到的每个cluster就是词。因此我们在进行k-mean的时候要仔细确定好词的数量，太多的词会导致过拟合而太少的词会出现欠拟合现象。

完成这一步后，我们就构建完成了一个词库，接下来就要在图像中表现出这些词。

下图为一些出现的高频“词”

![](/images/old-resources/14901949601534.jpg)


## 图像中词的直方图分布（Visual Words）
对于图像中的每个点，用欧式距离法判断属于哪个词，从而得到图像中词的一个直方图分布。下图表示的是visual word的信息。

![](/images/old-resources/14901947887360.jpg)


为了不完全抛弃图中pixel的位置，我们还可以采用spacial pyramid matching的方法来保留一部分spacial information（表现在所取的每个patch中）

![](/images/old-resources/14901945203509.jpg)


此外，我们甚至可以将TF-IDF的思想引入其中，得到具体的值，在test的过程中通过内积来做相似性度量。

## KNN进行图像分类（testing）
将training set中的每个直方图分布视为一个vector，先获取testing set中的一幅图的直方图分布，比较和training set中图的距离，找出最近的k个点中所对应training image的对应分类即是该图的分类结果。

![](/images/old-resources/14901944615694.jpg)


需要注意的一点是，k的取值不能太大，尤其是当某一类中的图片较少时更是如此。



