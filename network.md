### 一个游戏的网络模块

#### 大体的结构

> 1、主要分为三部分：
>
> > 1）主线程：循环调用update()函数，对工作线程传入的各种类job进行处理（实际上是调用外部传入callback的OnAccept()、OnRecv()、OnDiscount()等函数），以及对connect线程传入的connect_result进行处理（实际上是调用外部传入callback的OnConnect()函数）。
> >
> > 2）三个connect线程：循环地从connect_req_queue队列中取出connect请求并进行连接，随后将连接的结果push进connect_result队列当中。
> >
> > 3）一个工作线程（发送信息和接受信息的线程）：a.先调用select()/epoll_wait()，随后调用触发事件对应TcpHandle的OnCanRead() 或OnCanWrite()（实际上是调用read()/recv()接受/发送数据，并将接受的数据封装成JobRecv对象push进m_job_to_push中。b.工作线程继续将m_job_to_push中的Job push进主线程的m_job_queue中供主线程使用。

> 2、主要数据成员：
>
> > 1）BasicNetworkHandler类：处理网络连接的基类，TcpHandler类，ListenHandler类均继承该类
> >
> > 2）ListenHandler类：监听套接字所对应的handle处理（在工作线程中工作）
> >
> > > a、 ListenHandler::Listen(...)：创建socket、绑定socket、Listen套接字、将listen套接字设置为非阻塞  
> > >
> > > b、ListenHandler::OnCanRead()：调用accept()建立socket连接，new一个TcpHandler并将其添加至工作线程（BasicNetwork类）中并将数据封装成JobAccept对象交付给BasicNetwork类（由BasicNetwork类交付给主线程执行callbacck的OnAccept()） 
> >
> > 3）JobAccept类：
> >
> > > a、JobAccept::Invoke()：实际上是由BasicNetwork类交付给主线程执行用户传入callback的OnAccept()  
> >
> > 4）TcpHandler类：连接套接字的处理类（在工作线程中工作）  
> >
> > > a、TcpHandler::OnCanRead()：一次性读出socket缓冲区的所有可读数据（由于采用的是ET模式），将读取的完整数据封装为JobRecv类并交付至BasicNetwork类当中（由BasicNetwork类交付至主线程）  
> > >
> > > b、TcpHandler::OnCanWrite()：一次性写入所有数据，发送缓冲区不足时注册可写事件（ET模式）  
> >
> > 5）JobRecv类：
> >
> > > a、JobRecv::Invoke()：实际上是由BasicNetwork类交付给主线程执行用户传入callback的OnRecv()  
> >
> > 6）BasicNetwork类（工作线程主循环类）：
> >
> > > a、BasicNetwork::WorkFunc()：工作线程主循环函数，首先将有ListenHandler/TcpHandler交付的JobAccept/JobRecv交付给主线程（Network类），然后执行select()/epoll_wait()，调用对应事件handler（TcpHandler或ListenHandler）的OnCanRead()/OnCanWrite()  
> >
> > 7）Network类：（外部调用网络模块的接口类，主线程中工作）
> >
> > > a、Network::Start()：开启一个工作线程进行网络io的处理，开启三个connect线程进行网络连接处理
> > >
> > > b、Network::Connect()：异步connect，首先创建非阻塞socket并进行connect（非阻塞socket调用connect会马上返回），然后调用select()关注该socket的可写事件（即关注socket是否已连接），连接完成后new一个TcpHandler对象交付给BasicNetwork
> > >
> > > c、Network::ConnectAsyn()：同步connect，向connect_req_queue中push连接请求并唤醒connect线程  
> > >
> > > d、Network::Listen()：调用socket的listen，并new一个ListenHandler对象交付给BasicNetwork
> > >
> > > d、Network::ConnectThreadHelper()：connect线程循环函数，实际上是不断从connect_req_queue队列中取出连接调用Network::Connect()，并将连接结果push进connect_result_queue队列中（供主线程使用）
> > >
> > > e、Network::Update()：应该是主线程循环调用的函数，先使用用户传入的callback处理由BasicNetwork类交付的的各种Job，然后不断从connect_result_queue队列中取出连接结果并调用用户提供的OnConnect()
> > >
> > > f、Network::Send()：外部调用的发送信息的接口，先把数据复制进对应handler的数据缓冲区中，然后注册该handler的可写事件，可写事件触发调用该handler（TcpHandler）的OnCanWrite()发送数据  
>
> 3、部分实现的细节
>
> > 1）在调用BasicNetwork::RegisterSocketWrite(BasicNetworkHandler* handler)或BasicNetwork::AddSocket(BasicNetworkHandler* handler)时，是吧handler指针填到结构体epoll_event.data.ptr中而不是使用epoll_event.data.fd，这样在事件触发时就能直接调用到该handler的成员函数（epoll）
> >
> > ```c++
> > void BasicNetwork::AddSocket(BasicNetworkHandler* handler)
> > {
> > 	SOCKET sock_fd = handler->GetSocket();
> > 	struct epoll_event ev;
> > 	ev.events = EPOLLIN | EPOLLET;
> > 	ev.data.ptr = (void *)handler;
> > 	if (epoll_ctl(m_epfd, EPOLL_CTL_ADD, sock_fd, &ev) == -1)
> > 	{
> > 		// 添加失败
> > 	}
> > }
> > 
> > void BasicNetwork::PollSocket(HandlerList *readhandler, HandlerList *writehandler)
> > {
> > 	int eret = epoll_wait(m_epfd, m_tmp_event, MAX_EPOLL_SIZE, 10);
> > 	if (eret > 0)
> > 	{
> > 		m_register_table_mutex.Lock();
> > 		for (int i = 0; i < eret; ++i)
> > 		{
> > 			if (m_tmp_event[i].events & EPOLLIN)
> > 			{
> > 				readhandler->push_back((BasicNetworkHandler*)m_tmp_event[i].data.ptr);
> > 			}
> > 			if (m_tmp_event[i].events & EPOLLOUT)
> > 			{
> > 				writehandler->push_back((BasicNetworkHandler*)m_tmp_event[i].data.ptr);
> > 			}
> > 		}
> > 		m_register_table_mutex.Unlock();
> > 	}
> > }
> > ```
> >
> > 
> >
> > 2）异步connect的写法，作用：主要是可以设置超时时间，超时表示此次连接不成功，马上关闭该socket。
> >
> > ```c++
> > bool Network::Connect(IP ip, Port port, NetID *netid_out, unsigned long time_out)
> > {
> > 	SOCKET connect_sock = PISocket::Socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
> > 	if (connect_sock == SOCKET_ERROR)
> > 	{
> > 		PISocket::Close(connect_sock);
> > 		return false;
> > 	}
> > 
> > 	//设置为非阻塞
> > 	unsigned long ul = 1;
> > 	if ( SOCKET_ERROR == PISocket::Ioctl(connect_sock, FIONBIO, &ul))
> > 	{
> > 		PISocket::Close(connect_sock);
> > 		return false;
> > 	}
> > 
> > 	sockaddr_in server_adr;
> > 	memset(&server_adr, 0, sizeof(sockaddr_in));
> > 	server_adr.sin_family = AF_INET;
> > 	server_adr.sin_addr.s_addr = htonl(ip);
> > 	server_adr.sin_port = htons(port);
> > 
> > 	PISocket::Connect(connect_sock, (struct sockaddr *)&server_adr );
> > 
> > 	//检测可写，超时则返回
> > 	struct timeval tv ;   
> > 	tv.tv_sec = time_out / 1000;   
> > 	tv.tv_usec = (time_out % 1000) * 1000;
> > 
> > 	fd_set fdwrite;
> > 	FD_ZERO(&fdwrite);
> > 	FD_SET(connect_sock, &fdwrite);
> > 
> > 	int ret = select((int)connect_sock+1, 0, &fdwrite, 0, &tv);
> > 
> > 	if (ret > 0 && FD_ISSET(connect_sock, &fdwrite))
> > 	{
> > 		int error = 0;
> > 		int size = sizeof(int);
> > 		if ( 0 == PISocket::GetSockopt(connect_sock, SOL_SOCKET, SO_ERROR, (char *)&error, &size) && error == 0)
> > 		{
> > 			TcpHandler *tcphandler = new TcpHandler(connect_sock, m_config.max_package_size);
> > 			NetID netid = m_basicnetwork.Add(tcphandler);
> > 			if (netid_out != 0)
> > 			{
> > 				*netid_out = netid;
> > 			}
> > 			return true;
> > 		}
> > 	}
> > 
> > 	PISocket::Close(connect_sock);
> > 	return  false;
> > 	
> > }
> > ```
> >
> > 3）该网络模块的特点是即可以充当服务器使用（Listen()）也可以充当客户端（connect()）来使用，并且把连接都抽象为一个个TcpHandler来处理，只需要提前把各种事件的回调函数封装成CallBack对象传给该模块就可以了，缺点：该模块使用一个线程来处理所有连接的网络io，当连接量过大数据频繁交换频繁时，会造成响应缓慢。
> >
> > 4）有一点疑问：感觉网络模块的性能瓶颈是网络io部分，但是上述的处理是开一个线程来处理网络io但是却开了三个线程处理网络连接（connect()），是由于需要处理大量的连接请求咩。。。

