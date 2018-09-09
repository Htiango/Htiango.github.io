---
title: 并查集算法简介
date: 2018-09-09 15:58:10
tags: [union-find,algorithm]
categories: [算法]
---


本文将简要介绍并查集算法。

<!-- more -->

## 并查集算法

对于动态连通性的问题，一般都可以用并查集算法解决。

假设给定一个有`n`个节点的`graph`，我们首先要构建两个大小为`n`的orders和rank的list。其中：orders表示的是与该节点联通的父节点（未联通时为-1），rank表示的是联通时的等级，每次联通时都并入等级高的part（如果等级相同时随机并入并更新并入后的节点的rank）。

算法步骤如下：

+ 构建rank和orders两个list，rank初始都为0，orders初始都为-1.
+ 构建find方法，用递归的方法更新每个节点的父节点
+ 构建union方法，找出两个节点对应的父节点，比较父节点的rank，并入等级高的父节点。若两个父节点的rank相等，则并入第一个父节点并对第一个父节点的rank+1

以下为python代码部分

```python
class UnionFind:
    
    def __init__(self, n):
        self.orders = [-1 for i in range(n)]
        self.rank = [0 for i in range(n)]
        # count stands for number of connected part
        self.count = 0
        
    def setOrders(self, i):
        self.orders[i] = i
        self.count += 1
    
    def isValid(self, i):
        return self.orders[i] >= 0
    
    def find(self, i):
        # here use recursive way to update the orders.
        if self.orders[i] != i:
            self.orders[i] = self.find(self.orders[i])
        return self.orders[i]
    
    def union(self, i, j):
        x = self.find(i)
        y = self.find(j)
        
        if x != y:
            if self.rank[x] > self.rank[y]:
                self.orders[y] = self.orders[x]
            elif self.rank[x] < self.rank[y]:
                self.orders[x] = self.orders[y]
            else:
                self.orders[y] = self.orders[x]
                self.rank[x] += 1
            self.count -= 1
```

## 应用

### Minimum Spanning Tree 最小生成树
最小生成树的定义： 

1. 首先要有一个带权重值的无向图，也就是Edge不再仅仅是连接两个vertex，还具有一个权重值weight 
2. 最小生成树就是这个Graph的一棵子树，包含了Graph中的所有节点，并且没有任何的循环和周期 
3. 最小生成树中的所有连接线的权重加起来为最小 


最小生成树的解法是基于贪心法的kruskal算法，用union-find来判断图是否会构成一个环

具体算法步骤如下所示:

1. 对edge基于weight进行sort
2. 选取weight最小的edge，判断是否和已生成的树构成环，若无则选取该edge，反之则放弃该edge
3. 重复步骤2知道最小生成树中包含了v-1个edge


