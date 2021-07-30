#### skynet Makefile分析（以linux为例）

##### Makefile第一行"include platform.mk"表示引入platform.mk文件

##### platform.mk文件

> ```makefile
> PLAT ?= none
> PLATS = linux freebsd macosx
> 
> CC ?= gcc
> 
> .PHONY : none $(PLATS) clean all cleanall
> 
> #ifneq ($(PLAT), none)
> 
> .PHONY : default
> 
> default :
> 	$(MAKE) $(PLAT)
> 
> #endif
> 
> none :
> 	@echo "Please do 'make PLATFORM' where PLATFORM is one of these:"
> 	@echo "   $(PLATS)"
> 	
> SKYNET_LIBS := -lpthread -lm
> SHARED := -fPIC --shared
> EXPORT := -Wl,-E
> 
> linux : PLAT = linux					#make linux执行1
> macosx : PLAT = macosx
> freebsd : PLAT = freebsd
> 
> macosx : SHARED := -fPIC -dynamiclib -Wl,-undefined,dynamic_lookup
> macosx : EXPORT :=
> macosx linux : SKYNET_LIBS += -ldl		#make linux执行2
> linux freebsd : SKYNET_LIBS += -lrt		#make linux执行3
> 
> # Turn off jemalloc and malloc hook on macosx
> 
> macosx : MALLOC_STATICLIB :=
> macosx : SKYNET_DEFINES :=-DNOUSE_JEMALLOC
> 
> linux macosx freebsd :					#make linux执行4
> 	$(MAKE) all PLAT=$@ SKYNET_LIBS="$(SKYNET_LIBS)" SHARED="$(SHARED)" EXPORT="$(EXPORT)" MALLOC_STATICLIB="$(MALLOC_STATICLIB)" SKYNET_DEFINES="$(SKYNET_DEFINES)"
> 
> ```
>
> ?=：是如果没有被赋值过就赋予等号后面的值
>
> #ifneq($(PLAT), none)：两者的值不等进入进入条件判断内部
>
> .PHONY：伪目标都是用于定义一些只有命令操作没有目标文件生成的命令，即只有规则没有依赖的目标。
>
> 执行流程(（命令行执行make linux）：按照上述执行流程（代码中备注”make linux执行x），执行4实际上对应"make all PLAT=linux SKYNET_LIBS=-lpthread -lm -ldl -lrt SHARED=-fPIC --shared EXPORT=-Wl,-E MALLOC_STATICLIB=3rd/jemalloc/lib/libjemalloc_pic.a SKYNET_DEFINES="（MALLOC_STATICLIB的赋值在makefile文件中）

##### Makefile文件

> ```makefile
> include platform.mk
> 
> LUA_CLIB_PATH ?= luaclib
> CSERVICE_PATH ?= cservice
> 
> SKYNET_BUILD_PATH ?= .
> 
> CFLAGS = -g -O2 -Wall -I$(LUA_INC) $(MYCFLAGS)
> # CFLAGS += -DUSE_PTHREAD_LOCK
> 
> # lua
> 
> LUA_STATICLIB := 3rd/lua/liblua.a
> LUA_LIB ?= $(LUA_STATICLIB)
> LUA_INC ?= 3rd/lua
> 
> $(LUA_STATICLIB) :
> 	cd 3rd/lua && $(MAKE) CC='$(CC) -std=gnu99' $(PLAT)
> 
> # https : turn on TLS_MODULE to add https support
> 
> # TLS_MODULE=ltls
> TLS_LIB=
> TLS_INC=
> 
> # jemalloc
> 
> JEMALLOC_STATICLIB := 3rd/jemalloc/lib/libjemalloc_pic.a
> JEMALLOC_INC := 3rd/jemalloc/include/jemalloc
> 
> all : jemalloc
> 	
> .PHONY : jemalloc update3rd
> 
> MALLOC_STATICLIB := $(JEMALLOC_STATICLIB)
> 
> $(JEMALLOC_STATICLIB) : 3rd/jemalloc/Makefile		#make linux执行6
> 	cd 3rd/jemalloc && $(MAKE) CC=$(CC) 
> 
> 3rd/jemalloc/autogen.sh :							#make linux执行8
> 	git submodule update --init
> 
> 3rd/jemalloc/Makefile : | 3rd/jemalloc/autogen.sh	#make linux执行7
> 	cd 3rd/jemalloc && ./autogen.sh --with-jemalloc-prefix=je_ --enable-prof
> 
> jemalloc : $(MALLOC_STATICLIB)						#make linux执行5
> 
> update3rd :
> 	rm -rf 3rd/jemalloc && git submodule update --init
> 
> # skynet
> 
> CSERVICE = snlua logger gate harbor
> LUA_CLIB = skynet \
>   client \
>   bson md5 sproto lpeg $(TLS_MODULE)
> 
> LUA_CLIB_SKYNET = \
>   lua-skynet.c lua-seri.c \
>   lua-socket.c \
>   lua-mongo.c \
>   lua-netpack.c \
>   lua-memory.c \
>   lua-multicast.c \
>   lua-cluster.c \
>   lua-crypt.c lsha1.c \
>   lua-sharedata.c \
>   lua-stm.c \
>   lua-debugchannel.c \
>   lua-datasheet.c \
>   lua-sharetable.c \
>   \
> 
> SKYNET_SRC = skynet_main.c skynet_handle.c skynet_module.c skynet_mq.c \
>   skynet_server.c skynet_start.c skynet_timer.c skynet_error.c \
>   skynet_harbor.c skynet_env.c skynet_monitor.c skynet_socket.c socket_server.c \
>   malloc_hook.c skynet_daemon.c skynet_log.c
> 
> all : \													#make linux执行9
>   $(SKYNET_BUILD_PATH)/skynet \
>   $(foreach v, $(CSERVICE), $(CSERVICE_PATH)/$(v).so) \
>   $(foreach v, $(LUA_CLIB), $(LUA_CLIB_PATH)/$(v).so) 
> 
> $(SKYNET_BUILD_PATH)/skynet : $(foreach v, $(SKYNET_SRC), skynet-src/$(v)) $(LUA_LIB) $(MALLOC_STATICLIB)							#make linux执行10
> 	$(CC) $(CFLAGS) -o $@ $^ -Iskynet-src -I$(JEMALLOC_INC) $(LDFLAGS) $(EXPORT) $(SKYNET_LIBS) $(SKYNET_DEFINES)
> 
> $(LUA_CLIB_PATH) :
> 	mkdir $(LUA_CLIB_PATH)
> 
> $(CSERVICE_PATH) :
> 	mkdir $(CSERVICE_PATH)
> 
> define CSERVICE_TEMP									#指令包
>   $$(CSERVICE_PATH)/$(1).so : service-src/service_$(1).c | $$(CSERVICE_PATH)
> 	$$(CC) $$(CFLAGS) $$(SHARED) $$< -o $$@ -Iskynet-src
> endef
> 
> $(foreach v, $(CSERVICE), $(eval $(call CSERVICE_TEMP,$(v))))	#shell指令
> 
> $(LUA_CLIB_PATH)/skynet.so : $(addprefix lualib-src/,$(LUA_CLIB_SKYNET)) | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) $^ -o $@ -Iskynet-src -Iservice-src -Ilualib-src
> 
> $(LUA_CLIB_PATH)/bson.so : lualib-src/lua-bson.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) -Iskynet-src $^ -o $@
> 
> $(LUA_CLIB_PATH)/md5.so : 3rd/lua-md5/md5.c 3rd/lua-md5/md5lib.c 3rd/lua-md5/compat-5.2.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) -I3rd/lua-md5 $^ -o $@ 
> 
> $(LUA_CLIB_PATH)/client.so : lualib-src/lua-clientsocket.c lualib-src/lua-crypt.c lualib-src/lsha1.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) $^ -o $@ -lpthread
> 
> $(LUA_CLIB_PATH)/sproto.so : lualib-src/sproto/sproto.c lualib-src/sproto/lsproto.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) -Ilualib-src/sproto $^ -o $@ 
> 
> $(LUA_CLIB_PATH)/ltls.so : lualib-src/ltls.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) -Iskynet-src -L$(TLS_LIB) -I$(TLS_INC) $^ -o $@ -lssl
> 
> $(LUA_CLIB_PATH)/lpeg.so : 3rd/lpeg/lpcap.c 3rd/lpeg/lpcode.c 3rd/lpeg/lpprint.c 3rd/lpeg/lptree.c 3rd/lpeg/lpvm.c | $(LUA_CLIB_PATH)
> 	$(CC) $(CFLAGS) $(SHARED) -I3rd/lpeg $^ -o $@ 
> 
> clean :
> 	rm -f $(SKYNET_BUILD_PATH)/skynet $(CSERVICE_PATH)/*.so $(LUA_CLIB_PATH)/*.so && \
>   rm -rf $(SKYNET_BUILD_PATH)/*.dSYM $(CSERVICE_PATH)/*.dSYM $(LUA_CLIB_PATH)/*.dSYM
> 
> cleanall: clean
> ifneq (,$(wildcard 3rd/jemalloc/Makefile))
> 	cd 3rd/jemalloc && $(MAKE) clean && rm Makefile
> endif
> 	cd 3rd/lua && $(MAKE) clean
> 	rm -f $(LUA_STATICLIB)
> ```

#####  jemalloc编译相关

> 执行5处依赖变量"MALLOC_STATICLIB"（执行6），执行6依赖"3rd/jemalloc/Makefile"（执行7），执行7依赖3rd/jemalloc/autogen.sh（执行8）
>
> 执行8：用git把jemalloc代码更新一下
>
> 执行7：进入3rd/jemalloc目录并执行autogen.sh脚本，生成3rd/jemalloc/Makefile，然后跳回执行6
>
> 执行6：实际上对应cd 3rd/jemalloc && make CC=cc，调用jemalloc/Makefile

##### skynet编译相关

> ```makefile
> #foreach语法：
> names := a b c
> files := $(foreach v, $(names), $(v).so)
> #files的值为a.so b.so c.so
> ```
>
> shell指令行：循环执行指令包（指令包：相当于把指令封装成变量，用call调用？），实际相当于一下代码
>
> ```makefile
> cservice/snlua.so : service-src/service_snlua.c | cservice
> 	cc -g -O2 -Wall -I3rd/lua -fPIC --shared service-src/service_snlua.c -o cservice/snlua.so -Iskynet-src
> 	
> cservice/logger.so : service-src/service_logger.c | cservice
> 	cc -g -O2 -Wall -I3rd/lua -fPIC --shared service-src/service_logger.c -o cservice/logger.so -Iskynet-src
> 	
> cservice/gate.so : service-src/service_gate.c | cservice
> 	cc -g -O2 -Wall -I3rd/lua -fPIC --shared service-src/service_gate.c -o cservice/gate.so -Iskynet-src
> 	
> cservice/harbor.so : service-src/service_harbor.c | cservice
> 	cc -g -O2 -Wall -I3rd/lua -fPIC --shared service-src/service_harbor.c -o cservice/harbor.so -Iskynet-src
> ```
>
> 执行9：all伪目标依赖于./skynet(执行10) snlua.so logger.so gate.so harbor.so skynet.so client.so bson.so md5.so sproto.so lpeg.so
>
> 执行10：依赖一堆.c源文件和3rd/jemalloc/lib/libjemalloc_pic.a，3rd/lua/liblua.a两个静态库文件，执行命令相当于以下代码（实际上是生成可执行文件skynet）
>
> ```shell
> cc -g -O2 -Wall -I3rd/lua  -o skynet skynet-src/skynet_main.c skynet-src/skynet_handle.c skynet-src/skynet_module.c skynet-src/skynet_mq.c skynet-src/skynet_server.c skynet-src/skynet_start.c skynet-src/skynet_timer.c skynet-src/skynet_error.c skynet-src/skynet_harbor.c skynet-src/skynet_env.c skynet-src/skynet_monitor.c skynet-src/skynet_socket.c skynet-src/socket_server.c skynet-src/malloc_hook.c skynet-src/skynet_daemon.c skynet-src/skynet_log.c 3rd/lua/liblua.a 3rd/jemalloc/lib/libjemalloc_pic.a -Iskynet-src -I3rd/jemalloc/include/jemalloc  -Wl,-E -lpthread -lm -ldl -lrt 
> ```
>
> 接着就是根据执行9的依赖编译各个.so文件

