---
title: "五种io模型"
date: 2019-12-09T22:47:37+08:00
draft: true
---

# 一、阻塞IO（blocking I/O）
1、io两个阶段：

> 1）等待数据准备

> 2）将数据从内核拷贝到进程中  

![QDbLeH.png](https://s2.ax1x.com/2019/12/10/QDbLeH.png)
   
2、调用recvfrom后调用进程/线程阻塞直到内核把数据准备好并把数据拷贝到用户内存后，内核返回结果，调用进程/线程的阻塞状态解除，继续运行（阻塞io）

3、实际上，除非特别指定，几乎所有的IO接口 ( 包括socket接口 ) 都是阻塞型的


# 二、非阻塞IO（noblocking I/O）
1、调用recvfrom后直接返回结果，若内核数据并没有准备好则返回error。

2、在非阻塞式IO中，用户进程其实是需要不断的主动询问kernel数据准备好了没有（这种方法会导致cpu占用率大幅上升）

3、与阻塞io的主要差异是调用接口后立即返回调用结果而不是阻塞调用方直到内核返回结果。

4、Linux下，可以通过设置socket使其变为non-blocking

![QDjPCn.png](https://s2.ax1x.com/2019/12/10/QDjPCn.png)

# 三、多路复用IO（IO multiplexing）
1、select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程
    
2、 当用户进程调用了select，process会阻塞在select上而不是单个io操作上，而同时，kernel会“监视”所有select负责的socket，当任何一个socket的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

![QDjkvV.png](https://s2.ax1x.com/2019/12/10/QDjkvV.png)

# 四、异步IO（Asynchronous I/O）
1、用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。
   
2、真正的异步io（全过程并没有被阻塞，其他四种可能阻塞在数据准备阶段或者数据复制阶段）。
    
![QDji3q.png](https://s2.ax1x.com/2019/12/10/QDji3q.png)

# 信号驱动IO（signal driven I/O， SIGIO）
1、首先我们允许套接口进行信号驱动I/O,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。当数据报准备好读取时，内核就为该进程产生一个SIGIO信号。我们随后既可以在信号处理函数中调用recvfrom读取数据报，并通知主循环数据已准备好待处理，也可以立即通知主循环，让它来读取数据报。无论如何处理SIGIO信号，这种模型的优势在于等待数据报到达(第一阶段)期间，进程可以继续执行，不被阻塞。免去了select的阻塞与轮询，当有活跃套接字时，由注册的handler处理。
   
2、阻塞在数据复制阶段。

![QDjFg0.png](https://s2.ax1x.com/2019/12/10/QDjFg0.png)

