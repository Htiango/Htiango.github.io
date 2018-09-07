---
title: 再见多说，你好Disqus
date: 2017-05-29 15:03:41
tags: [blog,hexo,comment]
categories: [工欲善其事]
---

早在三个月前就知晓了多说将在6月1日后关闭的噩耗，然而懒癌晚期患者不到最后一刻是不会做出行动的。花了半小时的时间完成了Disqus的搭建，相信这款墙外产品一定会有着比多说更好的体验（前提是梯子要稳。。。）

<!-- more -->

由于我的hexo选用的主题的next，内置config就支持Disqus的使用，所以我们只需要在Disqus上注册生成一个site(通过首页上点击get started进入)，然后将shortname输入config文件中即可，如下：

```
disqus:
  enable: true
  shortname: your-shortname
  count: true
```

Disqus的相关注册问题可以参考[这篇文章](http://www.jianshu.com/p/c4f65ebe23ad)

配置完成后，只要更新hexo即可启用Disqus评论，enjoy it！

（至于如何将原有的评论从多说导出到Disqus，[github上的这篇文章](https://github.com/JamesPan/duoshuo-migrator)已经说的很详细，按步骤做就行，很方便的）。

最后再说一遍，国内用梯子一定要稳，一定要稳！

