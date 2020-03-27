#### 第一章 理解网络编程和套接字
> 接受连接请求的套接字创建过程  
> > 第一步：调用socket函数创建套接字  
> > 第二步：调用bind函数分配IP地址和端口号  
> > 第三步：调用listen函数转换为可接受请求状态  
> > 第四步：调用accept函数受理连接请求  

> 请求连接的套接字创建过程
> > 第一步：调用socket函数创建套接字  
> > 第二步：调用connect函数向服务器端发送请求连接  

#### 第二章 套接字类型与协议设置  
> 关于socket函数  
> > 签名式：``` int socket(int domain, int type, int protocol)```  
> > 返回值：成功时返回文件描述符，失败时返回-1  
> > 参数domain：套接字中使用的协议族信息,主要为PF\_INET(ipv4)和PF\_INET6(ipv6)  
> > 参数type：套接字数据传输类型信息，主要为SOCK\_STREAM(面向连接)和SOCK\_DGRAM(面向信息)  
> > 参数protocol：决定最终采用的协议，大部分情况下为0  

#### 第三章 地址族与数据序列  
> 地址信息的表示  
> > ```
> > struct sockaddr_in
> > {
> >     sa_family_t sin_family; //地址族，AF_INET(ipv4),AF_INET6(ipv6)  
> >     unint16_t sin_port;//16位TCP/UDP端口号(需要转换为网络字节序)  
> >     struct in_addr sin_addr;//32位IP地址(需要转换为网络字节序)  
> >     char sin_zero[8]; //不使用  
> > }
> > ```
> >
> > ```
> > struct in_addr  
> > {
> >     in_addr_t s_addr;//32位Ipv4地址  
> > }
> > ```

> 字节序转换相关(h代表host主机，n代表network网络)
> > ```
> >unsigned short htons(unsigned short);  
> >unsigned short ntohs(unsigned short);  
> >unsigned long htonl(unsigned long);  
> >unsigned long ntohl(unsigned long);  
> > ```

> 字符串信息与网络字节序转换(a代表ascii字符串，n代表网络字节序)  
> > ```
> > int addr_t inet_addr(const char* string);  
> > //成功时返回32位大端序(网络序)整数型值，失败时返回INADDR_NONE  
> > ```
> >
> > ```
> > int inet_aton(const char *string, struct in_addr *addr);  
> > //成功时返回1(true)，失败时返回0(false)  
> > ```
> >
> > ```
> > char* inet_ntoa(struct in_addr adr);  
> > //成功时返回转换的字符串地址值，失败时返回-1(该函数不能重入)  
> > ```

#### 第四章 基于TCP的服务器端/客户端(1)  

#### 第五章 基于TCP的服务器端/客户端(2)  
> 有关TCP套接字的I/O缓冲  
> > I/O缓冲在每个TCP套接字中单独存在  
> > I/O缓冲在创建套接字时自动生成  
> > 即使关闭套接字也会继续传递输出缓冲中遗留的数据(可以通过setsockopt设置)  
> > 关闭套接字将丢失输入缓冲中的数据  

#### 第六章 基于UDP的服务器端/客户端  

#### 第七章 优雅地断开套接字连接  
> 基于TCP的半关闭(shutdown函数)  
> > 签名式：``` int shutdown(int sock, int howto); ```  
> > howto参数值SHUT_RD：断开输入流，套接字无法接受数据，输入缓冲收到数据也会抹去，且无法调用输入相关的函数  
> > howto参数值SHUT_WR：断开输出流，套接字无法传输数据，但残留输出缓冲区的数据会继续传送  
> > howto参数值SHUT_REWR：同时断开I/O流  

#### 第八章 域名及网络地址  

#### 第九章 套接字的多种可选项    
> 套接字可选项的读取和设置函数  
> > ``` int getsockopt(int sock, int level, int optname, void *optval, socklen_t* optlen); ```  
> > ``` int setsockopt(int sock, int level, int optname, const void* optval, socklen_t optlen); ```  
> > 成功时返回0，失败时返回-1  
> > 参数sock：选项套接字的文件描述符  
> > 参数level：可选项的协议层  
> > 参数optname：可选项名  
> > 参数optval：可选项信息的缓冲地址  
> > 参数oplent：optval参数的长度  
> > 

> 常见的套接字可选项  
> > SO\_RCVBUF：输入缓冲大小的可选项  
> > SO\_SNDBUF：输出缓冲大小的可选项  
> > SO\_REUSEADDR:设置内核可将Time-wait状态的套接字重新分配  
> > TCP\_NODELAY：Nagle算法设置(默认启用)  

#### 第十章 多进程服务器端  

#### 第十一章 进程间通信  

#### 第十二章 I/O复用  
> select设置文件描述符宏  
> > ``` FD_ZERO(fd_set *fdset) ```  
 :将fd_set变量的所有位初始化为0  
> > ``` FD_SET(int fd, fd_set* fdset) ``` ：在参数fdset指向的变量中注册文件描述符反对的信息  
> > ``` FD_CLR(int fd, fd_set *fdset) ``` ：从参数fdset指向的变量中清除文件描述符fd的信息  
> > ``` FD_ISSET(int fd, fd_set* fdset) ``` ：若参数fdset指向的变量中包含文件描述符fd的信息，则返回true  

> select函数  
> > 签名式：``` int select(int maxfd, fd_set * readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout) ```  
> > 返回值：成功时返回发生事件的文件描述符数，失败时返回-1，超时返回0  
> > maxfd：监视对象文件描述符数量  
> > readset：读数据的文件描述符所注册到的fd_set的地址  
> > writeset：写数据的文件描述符所注册的fd_set的地址  
> > exceptset：异常的文件描述符所注册到的fd_set的地址  
> > timeout：超时信息  
> > 注意项：每次select返回都会改变传入的fd_set，故每次调用select前都需要重新设置fd_set  

#### 第十三章  多种I/O函数  

#### 第十四章 多播与广播  

#### 第十五章 套接字和标准I/O  
> 标准I/O函数优缺点
> > 标准I/O函数具有良好的移植性  
> > 标准I/O函数可以利用缓冲提高性能  
> > 不容易进行双向通信  
> > 有时可能频繁调用fflush函数  
> > 需要以FILE结构体指针的形式返回文件描述符  

> 几个常用的标准I/O函数  
> > ``` 
> > FILE *fdopen(int fildes, const char* mode;) 
> > //成功时返回转换的FILE结构体指针，失败时返回NULL  
> > //fildes：需要转换的文件描述符  
> > //mode：将要创建的FILE结构体指针的模式信息  
> > //将文件描述符转换为FILE指针  
> > ```   
> > 
> > ```
> > int fileno(FILE * stream);  
> > //成功时返回转换后的文件描述符，失败时返回-1  
> > //将FILE指针转换为文件描述符     
> > ```

#### 第十六章 关于I/O流分离的其他内容  

#### 第十七章 优于select的epoll
> epoll相关的函数与结构体  
> > ```
> > struct epoll_event
> > {
> >     __unint32_t events;
> >     epoll_data_t data; 
> > }
> > ```
> > 
> > ```
> > typedef union epoll_data
> > {
> >     void* ptr;
> >     int fd;
> >     __unint32_t u32;
> >     __unint64_t u64;
> > } epoll_data_t;
> > ```
> > 
> > ```
> > int epoll_create(int size);
> > //成功时返回epoll文件描述符，失败时返回-1
> > ```
> > 
> > ```
> >  int epoll_ctl(int epfd, int op, int fd, struct epoll_event* event);
> >  //成功时返回0，失败时返回-1
> >  //op：指定操作(EPOLL_CTL_ADD:注册，EPOLL_CTL_DEL：删除，EPOLL_CTL_MOD：改变)
> >  ```
> >  
> >  ```
> >  int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
> >  //成功时返回发生事件的文件描述符数，失败时返回-1
> >  //events：发生事件文件描述符集合的结构体地址值(需要动态分配)
> >  //timeout:以1/1000秒为单位，传-1时，一直等待知道事件发生  
> >  ```

> epoll_event成员中events的事件类型  
> > EPOLLIN：需要读取数据的情况()   
> > EPOLLOUT：输出缓冲为空，可以立即发送数据的情况  
> > EPOLLPRI：收到OOB数据的情况  
> > EPOLLRDHUP：断开连接或者半关闭的情况，在边缘触发方式下非常有用  
> > EPOLLERR：发生错误的情况  
> > EPOLLET：以边缘触发的方式得到事件通知  
> > EPOLLONESHOT：发生一次事件后，对应文件描述符不再收到事件通知  


