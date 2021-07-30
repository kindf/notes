```sequence
Title: 服务器初始化
main->watchdog: newservice()
watchdog->gate: newservice()
watchdog->agent: newservice()
watchdog->watchdog: mng.init()
watchdog->agent: skynet.call(AGENT, "init")
agent->agent: mng.init()
main->watchdog: skynet.call(WATCHDOG, "start")
watchdog->gate: skynet.call(GATE, "open")
```

时序流

a：创建watchdog服务

b：创建gate服务

c：创建agent服务

d：初始化watchdog管理器mng

e：初始化agent管理器mng

f：调用watchdog的start方法

g：调用gate的open方法

```sequence
Title: 简单登录请求
client->gate: c2s_login
gate->gate: handler.message()
gate->watchdog: skynet.call(watchdog, "data")
watchdog->watchdog: mng.c2s_login()
watchdog->agent: skynet.call(AGENT, "login")
agent->gate: skynet.call(GATE, "forward") 
```

时序流

a：客戶端发送登录请求到gate服务

b：gate判断消息分发的服务（登录请求发送到watchdog，登录后消息发送到agent）

c：watchdog根据消息的pid（实际上就是mng的函数名）调用mng的方法（登录请求的pid是“c2s_login”，参数为登录名，密码等）

d：登录验证成功后调用agent服务的login方法表示用户登录成功，在agent服务导入用户数据库信息并记录上线的用户

e：agent服务调用gate服务的forward方法，告知gate只有该用户的消息发送至agent服务

