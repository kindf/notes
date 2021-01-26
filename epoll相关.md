---
title: "io复用--epoll"
date: 2020-01-10T21:15:11+08:00
draft: true
---

> #### epoll原理
> * ##### 用户态调用epoll_create时,内核会创建一个eventepoll对象(即返回文件描述符所代表的对象)  
> 1.eventpoll较重要的字段:wq(用于保存有哪些进程在等待这个epoll返回)  
> 2.eventpoll较重要的字段:rdllist(用于保存可读写的文件)  
> 3.eventpoll较重要的字段:rbr(用于建立一个快速查找文件句柄的红黑树)  

> * ##### 调用epoll_ctl添加文件描述符  
> 1.内核会把被监听的文件描述符添加到eventepoll结构所维护的红黑树中(描述符就绪时能快速查找)   
> 2.内核会把epitem结构(即该eventepoll对象)添加到被监听文件描述符的等待队列中  
> 3.资源就绪时,中断程序会唤醒相应的eventepoll对象  
> 4.当等待队列被唤醒的时候,将调用ep_poll_callback()函数(把代表该文件描述符的epitem结构放置到eventpoll结构的rdllist队列中并唤醒进程)  

> * ##### 调用epoll_wait()函数开始监听文件  
> 1.内核把当前进程设置为可中断睡眠状态  
> 2.然后把进程添加到eventpoll结构的等待队列中，最后调用schedule_timeout()让出CPU  
> 3.当进程被唤醒时,会判断eventpoll结构的rdllist队列是否为空,不空时返回

> #### 水平触发(LT)与边缘触发(ET)  
> * ##### 边缘触发(ET)  
> 1.句柄在发生读写事件时只会通知用户一次  
> 2.ET模式主要关注fd从不可用到可用或者可用到不可用的情况  
> 3.ET只支持非阻塞模式  
> 缺点：应用层业务逻辑复杂，容易遗漏事件，很难用好  
> 优点：相对LT模式效率比较高  

> * ##### 水平触发(LT)  
> 1.只要句柄一直处于可用状态，就会一直通知用户  
> 2.LT模式下，句柄读缓冲区被读空后，句柄会从可用转变未不可以用，这个时候不会通知用户,写缓冲区只要还没写满，就会一直通知用户  
> 3.
LT模式支持阻塞和非阻塞两种方式,epoll默认的模式是LT  
> 优点：编程更符合用户直觉，业务层逻辑更简单  
> 缺点：效率比ET低  

> #### epoll相关API  
```
struct epoll_event
{
	__uint32_t events;
	epoll_data_t data:
}

typedef union epoll_data
{
	void *ptr;
	inf fd;
	__uint32_t u32;
	__unit64_t u64;
} epoll_data_t;

#include <sys/epoll.h>
int epoll_create(int size);
//成功时返回epoll文件描述符,失败时返回-1
//size:仅用于参考

#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
//成功时返回0,失败时返回-1
//epfd:用于注册监视对象epoll例程的文件描述符
//op:用于指定监视对象的添加、删除或更改等操作
//fd:需要注册的监视对象文件描述符
//event:监视对象的事件类型

第二个参数:
EPOLL_CTL_ADD:将文件描述符注册到epoll例程
EPOLL_CTL_DEL:从epoll例程中删除文件描述符
EPOLL_CTL_MOD:更改注册的文件描述符所关注事件发生情况

第三个参数中的events;变量:
EPOLLIN:需要读取数据的情况
EPOLLOUT:输出缓冲区为空,可以立即发送数据的情况
EPOLLPRI:收到OOB数据的情况
EPOLLRDHUP:断开连接或者半关闭的情况,这在边缘触发方式下非常有用
EPOLLERR:发生错误的情况
EPOLLET:以边缘触发的方式得到事件通知
EPOLLON:发生一次事件后,相应文件描述符不再收到时间通知.因此需要向epoll_ctl函数的第二个参数传递EPOLL_CTL_MOD再次设置事件

#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
//成功时返回发生时间的文件描述符数,失败时返回-1
//epfd:表示事件发生监视范围的epoll例程的文件描述符
//events 保存发生事件的文件描述符集合的结构体地址值
//maxevnets 第二个参数中可以保存的最大时间数
//timeout 以1/1000秒问单位的等待时间,传递-1表示一直等待直到事件发生
```