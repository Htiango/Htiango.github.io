---
title: proxy-lab
date: 2016-12-09 22:29:47

tags: [CSAPP,Lab]

categories: 
- CS Knowledge

---

Proxy Lab 作为 cmu 18600 以及 15213 这两门课的最后一个lab，其综合性非常强。既需要掌握好 web programming 以及 concurrent programming 的相关知识，还需要结合之前涉及的 shell 和 cache 的相关操作。<br>
本文将详细介绍 proxy lab 的解题思路。

<!-- more -->

## 什么是proxy

首先，我们需要知道什么是**proxy**？<br>
当我们平时打开浏览器的时候，输入一个URL，浏览器会向服务器发送相应的请求，服务器在接收到请求后会将相应的response发回给浏览器，如此循环往复从而加载完网页中的全部内容。<br>
此时所有的请求和响应之间的交流全是发生在 client (浏览器)和 server (服务器)之间的。而有时我们会在client和server之间添加代理，来进行相关的处理，这个代理就是实验要求我们完成的proxy。<br>

*proxy的大致示意图如下所示*

![流程图](http://oh1ulkf4j.qnssl.com/Screen Shot 2016-12-09 at 10.35.56 PM.png)<br>


## 实验准备

在本次实验中，我们采用的浏览器是Firefox，设置代理的过程如下所示：<br>
*打开设置中的高级，选择网络，点设置并按照如下设置（若proxy在本地则选择localhost或是127.0.0.1）*
*需要注意的是，端口一定要和之后运行proxy时的端口一致*
![代理设置1](http://oh1ulkf4j.qnssl.com/Screen Shot 2016-12-09 at 9.15.20 PM.png)<br>
![代理设置2](http://oh1ulkf4j.qnssl.com/Screen Shot 2016-12-09 at 9.15.37 PM.png)<br>


## proxy如何处理request

打开Firefox网页，Mac下alt + cmd + q，Win下按F12进行观察，点击每条可以显示出请求和响应的内容
![网页1](http://oh1ulkf4j.qnssl.com/Screen Shot 2016-12-09 at 9.10.05 PM.png)<br>

![网页2](http://oh1ulkf4j.qnssl.com/Screen Shot 2016-12-09 at 9.31.06 PM.png)<br>

通过这种方法，我们可以很轻松地看到请求和响应头。

而作为一个proxy，所需要做的事情主要有这么几件：<br>
1. 从请求中获取请求的方法，请求网址的hostname，path以及port（没有的话为80）*(有多种方法解析，我采用的是正则表达式)* <br>  需要注意的是，本实验中不支持非get的方法，同时也不支持任何以HTTPS开头的网页请求。本实验中以501错误返回这类请求。<br> 
2. 改变请求头中的一些内容（比如User-Agent，connection改为close）<br>
3. 添加 Proxy-Connection: close，来确定请求响应的交换是否结束
4. 改变原先请求中的version。*（从 HTTP/1.1 到HTTP/1.0）* <br>

对网页端发送请求进行修改之后，发送给服务器，再将response返回给网页端，如此循环往复直到网页内容加载完毕

## 处理多线程操作

在完成上述内容之后，我们就实现了一个逐条处理网页端请求的proxy。但是在现实中这样的效率极其低下。所以对于我们的proxy还需要使其支持多线程操作。<br>
其所涉及的函数如下所示<br>

```c
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);


    int pthread_detach(pthread_t thread);
	
```

其中pthread_create函数用于打开一个新的线程，通过调用start_routine这个函数，而其中的arg是start_routine函数的参数。<br>
特别要注意的是，arg一定要事先进行malloc，为每个线程ID分配一个独立的块，并将指向这个块的指针传给start_routine。不然线程会出现竞争问题导致错误。同时在结束线程时一定要释放这些块来避免memory leak*（我就是在这里跪了很久的。。。）*

而第二个函数是用于在线程中防止线程被其他线程回收或杀死。

在第一步中加入上述函数，基本上就能够实现proxy的多线程操作的部分。

## 存储网页内容

以上的proxy已经基本完成了代理的要求。不过当我们重复请求某一个网址的时候，它还是要重新加载一遍，这就有点低效了。如果我们能够把之前网页端获取的响应存下来呢？这样当我们重复加载的时候就无需连接到服务器了。<br>
所以，我们还要让我们的proxy能够存储网页的内容。<br>在本次实验中，我用一个类似于队列的双向链表来表示存储的cache。proxy在每次处理完网页端发来的请求后，先遍历整个链表，看是否有相同的request存在cache中，如果有就直接获取对应的response。没有的话就现将请求发动到server，将server返回的response写入到cache中。<br>
在具体的操作中，我采用的是FIFO，每次都将新的request/response加在链表的头。一旦cache存储已满，就从尾部pop。<br>
需要注意的是，一旦找到匹配的request之后，我们还需要将对应的node移到链表的头指针处。这样才符合FIFO。<br>

上述就是我对与proxy lab的总结，希望大家都能做出一个完美的proxy！！


