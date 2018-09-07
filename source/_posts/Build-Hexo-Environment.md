---
title: 用github搭建hexo静态博客
date: 2018-09-06 23:00:34
tags: [blog,hexo]
categories: [工欲善其事]
---

本文将简单介绍下如何用github搭建hexo静态博客。

<!-- more -->

挖坟抢救下我的blog。。。不知不觉已经有一年多没有更新过了，简直是懒癌的最高境界啊。

为了唤醒下我的记忆，本篇先从基本的配置hexo环境并利用github搭建一个属于自己的静态博客开始介绍吧。和其他教程不同的是，本篇教程介绍的是一种可以对hexo source files（而非发布files）进行版本控制，从而使得能在多台pc上进行博客修改的方法。

## 安装 Hexo
根据[官网的教程](https://hexo.io/docs/)中的安装走一遍就行啦。

+ 安装 git
+ 安装 Node.js
+ 安装 Hexo

这里不做赘述

## Start Hexo
安装完成后，我们就可以着手开始hexo的设置了。

首先过一遍官方[设置教程](https://hexo.io/docs/setup)，记在心里就可以了，接下来首先需要设置github从而使得能够对Hexo的source进行版本控制。

### Github设置

在介绍如何设置Hexo之前，首先需要了解下我们deploy的环境——github。

每个用户，都可以在github上搭建属于自己的site，详见[Github.io](https://github.io/)。我们只需创建一个以 *username.github.io* 结尾的repository，即可搭建属于自己的静态网页。

而对于hexo而言，通过设置blog文件夹下的`_config.yml`文件，我们可以将hexo生成的结果deploy到任一一个remote repository上。但是这样的话，一旦你想要用别的电脑修改并发布blog，就必须复制原有的文件，并且非常不利于版本控制。所以我的想法是利用这个*username.github.io*  repository，即实现对hexo source 的版本控制，又能够构建一个属于自己的网站。

如何实现呢？利用branch！

在*username.github.io*下新建一个branch（姑且叫`hexo`），`master` 用于hexo的deploy，`hexo`用于实现source文件的版本控制。

首先在github上创建一个*username.github.io* 的仓库

cd到在你想要创建博客的文件夹下

```bash
git clone https://github.com/[username]/username.github.io.git
mv username.github.io.git <blog-folder-name>  #如果不嫌文件名难看的话，就不用这条指令改名字了
cd <blog-folder-name>
git branch hexo     # 新建branch
git checkout hexo   # checkout new branch
```

接下来我们就可以在hexo branch下进行操作了

## Hexo setup
现在，可以走一遍[设置教程](https://hexo.io/docs/setup)了

```bash
cd ..
hexo init <blog-folder-name>
cd <blog-folder-name>
npm install
```

初始化结束！

站点的设置信息都在`_config.yml`中，通过设置实现deploy到github上

```yaml
deploy:
  type: git
  repository:
      github: git@github.com:username/username
  branch: master
```

## Hello world!
好啦，接下来就可以写文章发布blog了。(第一篇当然是hello world?

cd到`blog`文件夹

```
hexo new "hello-world"
```

此时`./source/_posts`下就会出现一个`hello-world.md`的文件，用markdown编辑好 (hello world)

```bash
hexo generate       # generate static pages
hexo deploy         # deploy (to github)
```

如此，你的第一篇`hello world`就诞生了


