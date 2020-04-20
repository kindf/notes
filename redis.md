#### redis网络模块分析

> #### struct redisServer结构体  
>
> > ```
> > struct redisServer 
> > {
> >  	int port; 							//tcp监听端口
> >  	int tcp_backlog; 					//TCP listen() backlog
> >  	char *bindaddr[REDIS_BINDADDR_MAX]; //地址 (16)
> >  	int bindaddr_count;					// 地址数量
> >  	int ipfd[REDIS_BINDADDR_MAX]; 		//TCP socket file descriptors(16)
> >  	int ipfd_count;						//文件描述符数量  
> >  	aeEventLoop *el;					//事件处理器
> >     void *clientData;	//多路复用库的私有数据(对于epoll来说就是fd和epoll_event的结构体)
> > }
> > ```

> #### initServerConfig()函数：初始化服务器的配置(实际上就是初始化全局变量struct redisServer server的一些成员变量)
>
> >  erver.port = REDIS_SERVERPORT;  (指定监听端口号6379)
> >
> >  server.tcp_backlog = REDIS_TCP_BACKLOG;  (监听队列大小511)  
> >
> >  erver.ipfd_count = 0;  
> >
> >  erver.bindaddr_count = 0;
> >
> > ```
> > void initServerConfig() 
> > {
> > 	...		//忽略无关代码
> > 	erver.port = REDIS_SERVERPORT;  (指定监听端口号6379)
> >  	server.tcp_backlog = REDIS_TCP_BACKLOG;  (监听队列大小511)
> >  	erver.ipfd_count = 0; 
> >  	erver.bindaddr_count = 0;
> > 	...		//忽略无关代码
> > }
> > ```
>
> #### initServer()函数：初始化srever(为其分配各种空间)
>
> > listenToPort(server.port,server.ipfd,&server.ipfd_count

#### socket创建的过程  

> #### 在main函数中调用initServerConfig()初始化server的配置  
>
> > ```
> > server.port = REDIS_SERVERPORT;  (指定监听端口号6379)
> > server.tcp_backlog = REDIS_TCP_BACKLOG;  (监听队列大小511)  
> > erver.ipfd_count = 0;  
> > erver.bindaddr_count = 0;
> > server.maxclients = REDIS_MAX_CLIENTS;	(10000)
> > ```
>
> #### 在main函数中的initserver()初始化server  
>
> > ##### initserver()中调用listenToPort()
>
> > ```
> > if (server.port != 0 &&
> >         listenToPort(server.port,server.ipfd,&server.ipfd_count) == REDIS_ERR)
> >         exit(1);
> >         
> > int listenToPort(int port, int *fds, int *count)
> > ```
> >
> > ##### listenToPort中调用anetTcp6Server()
> >
> > ```
> > fds[*count] = anetTcp6Server(server.neterr,port,NULL,
> >                 server.tcp_backlog);
> >                 
> > int anetTcp6Server(char *err, int port, char *bindaddr, int backlog)
> > {
> >     return _anetTcpServer(err, port, bindaddr, AF_INET6, backlog);
> > }
> > ```
> >
> > ##### 实际上是调用_anetTcpServer()创听套接字
> >
> > ```
> > if ((rv = getaddrinfo(bindaddr,_port,&hints,&servinfo)) != 0) {
> >         anetSetError(err, "%s", gai_strerror(rv));
> >         return ANET_ERR;
> >     }
> >     for (p = servinfo; p != NULL; p = p->ai_next) {
> >         if ((s = socket(p->ai_family,p->ai_socktype,p->ai_protocol)) == -1)
> >             continue;
> > 
> >         if (af == AF_INET6 && anetV6Only(err,s) == ANET_ERR) goto error;
> >         if (anetSetReuseAddr(err,s) == ANET_ERR) goto error;
> >         if (anetListen(err,s,p->ai_addr,p->ai_addrlen,backlog) == ANET_ERR) goto error;
> >         goto end;
> >     }
> > ```
> >
> > ##### 调用anetListen()监听套接字  
> >
> > ```
> > static int anetListen(char *err, int s, struct sockaddr *sa, socklen_t len, int backlog) 
> > {
> > 	if (bind(s,sa,len) == -1) 
> > 	{
> >         anetSetError(err, "bind: %s", strerror(errno));
> >        	 close(s);
> >         return ANET_ERR;
> >     }
> > 
> >     if (listen(s, backlog) == -1) 
> >     {
> >         anetSetError(err, "listen: %s", strerror(errno));
> >         close(s);
> >         return ANET_ERR;
> >     }
> >     return ANET_OK;
> > }
> > ```

#### 事件处理器对象struct aeEventLoop

> ```
> typedef struct aeEventLoop 
> {
>    	// 目前已注册的最大描述符
>     int maxfd;   /* highest file descriptor currently registered */
>     // 目前已追踪的最大描述符
>     int setsize; /* max number of file descriptors tracked */
>     // 用于生成时间事件 id
>     long long timeEventNextId;
>     // 最后一次执行时间事件的时间
>     time_t lastTime;     /* Used to detect system clock skew */
>     // 已注册的文件事件
>     aeFileEvent *events; /* Registered events */
>     // 已就绪的文件事件
>     aeFiredEvent *fired; /* Fired events */
>     // 时间事件
>     aeTimeEvent *timeEventHead;
>     // 事件处理器的开关
>     int stop;
>     // 多路复用库的私有数据
>     void *apidata; /* This is used for polling API specific data */
>     // 在处理事件前要执行的函数
>     aeBeforeSleepProc *beforesleep;
> } aeEventLoop;  
> ```

#### 事件处理器关于socket的设置  

> #### 在initserver()中调用aeCreateEventLoop()创建aeEventLoop对象  
>
> > ```
> > void initServer() 
> > {
> > 	...		//忽略无关代码
> > 	server.el = aeCreateEventLoop(server.maxclients+REDIS_EVENTLOOP_FDSET_INCR);
> > 	//server.maxclients = REDIS_MAX_CLIENTS;(maxclients在initserverconfig中初始化为10000)
> > 	//宏值REDIS_EVENTLOOP_FDSET_INCR为32+96
> >      ...	//忽略无关代码
> > }
> > ```
>
> #### aeCreateEventloop()中创建并初始化aeEventLoop对象，返回对象指针
>
> > ```
> > aeEventLoop *aeCreateEventLoop(int setsize) 
> > {
> >     aeEventLoop *eventLoop;
> >     int i;
> >     // 创建事件状态结构
> >     if ((eventLoop = zmalloc(sizeof(*eventLoop))) == NULL) goto err;
> >     // 初始化文件事件结构和已就绪文件事件结构数组
> >     eventLoop->events = zmalloc(sizeof(aeFileEvent)*setsize);
> >     eventLoop->fired = zmalloc(sizeof(aeFiredEvent)*setsize);
> >     if (eventLoop->events == NULL || eventLoop->fired == NULL) goto err;
> >     // 设置数组大小
> >     eventLoop->setsize = setsize;
> >     // 初始化执行最近一次执行时间
> >     eventLoop->lastTime = time(NULL);
> >     // 初始化时间事件结构
> >     eventLoop->timeEventHead = NULL;
> >     eventLoop->timeEventNextId = 0;
> >     eventLoop->stop = 0;
> >     eventLoop->maxfd = -1;
> >     eventLoop->beforesleep = NULL;
> >     if (aeApiCreate(eventLoop) == -1) goto err;
> >     /* Events with mask == AE_NONE are not set. So let's initialize the
> >      * vector with it. */
> >     // 初始化监听事件
> >     for (i = 0; i < setsize; i++)
> >         eventLoop->events[i].mask = AE_NONE;
> >     // 返回事件循环
> >     return eventLoop;
> > err:	//申请内存失败后都需要释放所有申请的空间防止内存泄露
> >     if (eventLoop) {
> >         zfree(eventLoop->events);
> >         zfree(eventLoop->fired);
> >         zfree(eventLoop);
> >     }
> >     return NULL;
> > }
> > ```
> >
> > 
>
> #### 调用aeApiCreate()函数初始化aeEventLoop中的apidata成员  
>
> > ```
> > static int aeApiCreate(aeEventLoop *eventLoop)
> > {
> >     aeApiState *state = zmalloc(sizeof(aeApiState));
> >     if (!state) return -1;
> >     // 初始化事件槽空间
> >     state->events = zmalloc(sizeof(struct epoll_event)*eventLoop->setsize);
> >     if (!state->events) {
> >         zfree(state);
> >         return -1;
> >     }
> >     // 创建 epoll 实例
> >     state->epfd = epoll_create(1024); /* 1024 is just a hint for the kernel */
> >     if (state->epfd == -1) {
> >         zfree(state->events);
> >         zfree(state);
> >         return -1;
> >     }
> >     // 赋值给 eventLoop
> >     eventLoop->apidata = state;
> >     return 0;
> > }
> > 
> > typedef struct aeApiState {
> >     int epfd;		    // epoll_event 实例描述符
> >     struct epoll_event *events;		    // 事件槽
> > } aeApiState;
> > ```
> >
> > 
>
> #### 在initserver()中调用aeCreateFileEvent()设置server.el(事件处理器的文件事件)：
>
> > 这里的sever.ipfd_count == 1(只有一个监听套接字)，所以实际上是创建监听套接字的文件事件  
> >
> >     void initServer() 
> >     {
> >     	...		//忽略无关代码
> >     	for (j = 0; j < server.ipfd_count; j++) 
> >     	{
> >         	if (aeCreateFileEvent(server.el, server.ipfd[j], AE_READABLE,
> >             	acceptTcpHandler,NULL) == AE_ERR)
> >             	{
> >                 	redisPanic(
> >                     	"Unrecoverable error creating server.ipfd file event.");
> >             	}
> >          }
> >          ...	//忽略无关代码
> >     }
>
> #### aeCreateFileEvent()中注册监听socket的读事件以及读事件的回调
>
> > ```
> > int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask,
> >         aeFileProc *proc, void *clientData)
> > {
> > 	if (fd >= eventLoop->setsize) {...}
> >     if (fd >= eventLoop->setsize) return AE_ERR;
> >     // 取出文件事件结构
> >     aeFileEvent *fe = &eventLoop->events[fd];
> >     // 监听指定 fd 的指定事件
> >     if (aeApiAddEvent(eventLoop, fd, mask) == -1)
> >         return AE_ERR;
> >     // 设置文件事件类型，以及事件的处理器
> >     fe->mask |= mask;
> >     if (mask & AE_READABLE) fe->rfileProc = proc;
> >     if (mask & AE_WRITABLE) fe->wfileProc = proc;
> >     // 私有数据
> >     fe->clientData = clientData;
> >     // 如果有需要，更新事件处理器的最大 fd
> >     if (fd > eventLoop->maxfd)
> >         eventLoop->maxfd = fd;
> >     return AE_OK;
> > }
> > ```
>
> #### 监听socket读事件的回调函数acceptTcpHandler()
>
> ```
> void acceptTcpHandler(aeEventLoop *el, int fd, void *privdata, int mask)
> {
>     int cport, cfd, max = MAX_ACCEPTS_PER_CALL;
>     char cip[REDIS_IP_STR_LEN];
>     REDIS_NOTUSED(el);
>     REDIS_NOTUSED(mask);
>     REDIS_NOTUSED(privdata);
> 
>     while(max--) {
>         // accept 客户端连接
>         cfd = anetTcpAccept(server.neterr, fd, cip, sizeof(cip), &cport);
>         if (cfd == ANET_ERR) {
>             if (errno != EWOULDBLOCK)
>                 redisLog(REDIS_WARNING,
>                     "Accepting client connection: %s", server.neterr);
>             return;
>         }
>         redisLog(REDIS_VERBOSE,"Accepted %s:%d", cip, cport);
>         // 为客户端创建客户端状态（redisClient）
>         acceptCommonHandler(cfd,0);
>     }
> }
> ```
>
> 
>
> #### aeApiAddEvent()中往epoll中注册监听socket的读事件()
>
> > ```
> > static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask) {
> >     aeApiState *state = eventLoop->apidata;
> >     struct epoll_event ee;
> >     {
> >     	int op = eventLoop->events[fd].mask == AE_NONE ?
> >             EPOLL_CTL_ADD : EPOLL_CTL_MOD;
> > 
> >     	// 注册事件到 epoll
> >     	ee.events = 0;
> >     	mask |= eventLoop->events[fd].mask; /* Merge old events */
> >     	if (mask & AE_READABLE) ee.events |= EPOLLIN;
> >     	if (mask & AE_WRITABLE) ee.events |= EPOLLOUT;
> >     	ee.data.u64 = 0; /* avoid valgrind warning */
> >     	ee.data.fd = fd;
> > 
> >     	if (epoll_ctl(state->epfd,op,fd,&ee) == -1) return -1;
> > 
> >     	return 0;
> >     }
> > ```
> 
> #### acceptTcpHandler()函数
> > ```
> > 
> > dfkldjls
> > ```
>
> #### main函数调用aeMain()执行主循环
> > ```
> > void aeMain(aeEventLoop *eventLoop) {
> >     eventLoop->stop = 0;
> >     while (!eventLoop->stop) {
> >         if (eventLoop->beforesleep != NULL)
> >             eventLoop->beforesleep(eventLoop);
> >         aeProcessEvents(eventLoop, AE_ALL_EVENTS|AE_CALL_AFTER_SLEEP);
> >     }
> > }
> > ```
>
> #### aeProcessEvents()调用  
> >
> > ```
> > int aeProcessEvents(aeEventLoop *eventLoop, int flags)
> > {
> >     ... //忽略无关代码
> >     for (j = 0; j < numevents; j++) 
> >     {
> >         aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
> >         int mask = eventLoop->fired[j].mask;
> >         int fd = eventLoop->fired[j].fd;
> >         int fired = 0; 
> >
> >         int invert = fe->mask & AE_BARRIER;
> > 
> >         if (!invert && fe->mask & mask & AE_READABLE) {
> >             fe->rfileProc(eventLoop,fd,fe->clientData,mask);
> >             fired++;
> >         }
> >         if (fe->mask & mask & AE_WRITABLE) {
> >             if (!fired || fe->wfileProc != fe->rfileProc) {
> >                 fe->wfileProc(eventLoop,fd,fe->clientData,mask);
> >                 fired++;
> >             }
> >         }
> > 
> >         if (invert && fe->mask & mask & AE_READABLE) {
> >             if (!fired || fe->wfileProc != fe->rfileProc) {
> >                 fe->rfileProc(eventLoop,fd,fe->clientData,mask);
> >                 fired++;
> >             }
> >         }
> >         processed++;
> >     }
> > }
> >     if (flags & AE_TIME_EVENTS)
> >         processed += processTimeEvents(eventLoop);
> > 
> >     return processed;
> > } 
> > ```
