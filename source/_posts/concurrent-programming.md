---
title: Concurrent Programming
date: 2016-12-10 14:21:07
tags: [CSAPP]
categories: 
- CS Knowledge

---

并发在应用程序中起着很重要的作用，本文将详细介绍应用级的并发

<!-- more -->

## 进程
进程是构造并发程序最为简单的方法。在CSAPP的第八章中就已经介绍过：进程本质上就是一个执行中的程序的实例，每当我们运行一个程序的时候就会创建一个进程并在其上运行相关文件。<br>
而进程在真正运行的过程中并不是独占处理器的，根据不同的逻辑控制流（每个进程的PC值），不同的进程轮流使用处理器。如果不同的进程在运行过程中有时间的重叠，则两者之间是并发的关系。<br>
采用进程的并发方式可以在父子进程之间共享文件，同时两者不同的地址可以避免彼此信息的覆盖问题。但是这种方式不得不采用IPC（进程间通信）的方式来交换彼此的信息，而这是一种开销很大的方式，大大降低运行的速度。<br>

## I/O多路复用
当我们在浏览一个网页的时候，服务器可以同时处理浏览器发送的请求和用户输入的指令，而这主要采用的就是I/O多路复用的方法。<br>
其核心思想就是采用select函数，要求内核挂起进程，仅当一个或多个I/O事件发生后才将其返回给应用程序。本质上这种方法下我们创建自己的逻辑流，利用I/O多路复用来进行流的调度。<br>
这种方法的一个最大的优点就是信息交换的便捷，共享数据来得更为高效*(无需在流之间切换)*，使我们对程序有着更好的掌控<br>
但是与此同时，与第一种方法相比，编码量的复杂度大大提升。而且一旦某一逻辑流在读某一文本，其他流就不能读了。这也是不是很高效的一点。

## 线程
### 什么是线程
与进程是运行在系统中的逻辑流对应的，线程是运行在进程中的逻辑流，每个线程都有着唯一的整数ID、栈指针、栈、计数器、寄存器等等，运行在一个进程中的线程共享该进程的整个虚拟地址空间。从本质上讲，这种方法更像是上述两种方法的结合。<br>
### 线程是如何执行的
所有的进程在最开始的时候都是单线程的，这个线程就是主线程。随后在某一时间点主线程会创建一个对等线程并与之一起并发运行*（来回切换）*
![线程示意图](/images/old-resources/Screen%20Shot%202016-12-10%20at%203.50.33%20PM.png)
由于线程的context对比进程而言要小得多，所以线程之间的切换也要快得多。主线程和对等线程之间基本上是相同的，都能读写相同的共享信息。<br>
### 线程相关函数
#### 创建线程

{% codeblock lang:c %}
#include <pthread.h>typedef void *(func)(void *);

// Returns 0 if OK, nonzero on errorint pthread_create(pthread_t *tid, pthread_attr_t *attr, func *f, void *arg);  

// Returns thread ID of caller
pthread_t pthread_self(void); 

{% endcodeblock %}



#### 结束线程
当调用下述函数时，主线程会等待所有对等线程终止时在终止自己和整个进程。否则则当线程运行完后隐式终止。<br>

{% codeblock lang:c %}
// terminate threads
#include <pthread.h>void pthread_exit(void *thread_return);

{% endcodeblock %}

#### 分离线程
在线程被创建后，其默认是可结合的，即可以被其他线程回收杀死，而下面的函数则可以将其分离，仅当其终止时才自动释放存储。<br>

{% codeblock lang:c %}
// detach threads
#include <pthread.h>int pthread_detach(pthread_t tid);   // Returns 0 if OK, nonzero on error

{% endcodeblock %}

### 线程中同步变量
各个线程彼此之间可以共享变量和文件，但是如果不加限制有时会造成同步错误。<br>
因此，在文件或是变量同步(读写)的过程中，并发的程序有着种种的限制。在本书讲pipeline的过程中就介绍过read after write的问题。在pipeline中如果先写后读则读的过程至少需要等待三个周期才能保证不出错*（当然在forwarding的方法下我们可以将等待周期减为1个）*。<br>
同样的，我们在并发线程中进行文件或是变量读写操作的时候，也会遇到类似的问题：如果在某一线程读取某一变量值的同时，另一线程正在对改写这一变量**(这里的同时指的并不是完全意义上的同时，而是很短的时间)**，由于读和写都要一定时间，这就可能会造成数据的错误。因此我们需要对线程间的变量同步加以限制。主要采用Posix中的 P 和 V 操作。<br>
+ P(s)：加锁操作。若s非零则将其减1返回，否则挂起线程直至s非零。
+ V(s)：解锁操作。若有线程被P操作挂起则将s加1，重启该线程。

因此，我们可以通过 P 和 V 操作实现线程中的变量同步。<br>
以下代码展示了读者优先的线程，只要有一个读者在读，其他的读者就能忽略锁而毫无障碍的读取变量。

{% codeblock lang:c %}
// global variables
int readcnt;
sem_t mutex, w;
void reader(void){
    while(1){
        P(&mutex);
		readcnt++;
		if (readcnt == 1)
		{
			P(&w);
		}
		V(&mutex);

		// do the reading

		P(&mutex);
        if (readcnt == 0)
        {
        	V(&w);
        }
        V(&mutex);
	}
}
{% endcodeblock %}

### 线程中的竞争问题
如果我们在构建线程时，每次创建一个新的对等线程都是通过传递一个指向唯一整数ID的指针的话，很有可能会导致程序的错误，因为在这种情况下各个线程会产生竞争。<br>
而解决这种问题的方法也很简单，只需要用一个malloc函数为每个线程动态分配一个整数ID的指针，并将这个这个指针传递给构建线程的函数中。同时最后别忘了对指针进行free来避免memory leak。

