---
title: "makefile学习"
date: 2019-12-27T22:29:05+08:00
draft: true
---

编译时，编译器只检测程序语法和函数、变量是否被声明。如果函数未被声明，编译器会给出一个警告，但能生成object file   

链接时，链接器会在所有object file中寻找函数的实现，如找不到就会报链接错误码（Linker Error）

自动推导：只要make看到一个.o文件就会自动把.c文件加到依赖关系中，并且cc -c ....c也会被推导出来

.PHONY:clean  .PHONY表示clean是个伪目标文件

include &lt;filename&gt; filename可以是当前操作系统shell的文件模式（可以包含路径和通配符），include前面不能以tab键开始

> include查找路径（若没指定路径）:

>> 1、如果make执行时，有“-I”或“--inlcude-dir”参数，make会在该参数所指定的目录下寻找

>> 2、如果目录&lt;prefix&gt;/include（一般是：/usr/local/bin 或/usr/include）存在的话，make 也会去找

>> -include /&lt;filename&gt;表示无论include过程中出现什么错误都不要报错继续执行

##### GUN的make工作时的执行步骤：

> 1、读入所有的 Makefile。

> 2、读入被 include 的其它 Makefile。

> 3、初始化文件中的变量。

> 4、推导隐晦规则，并分析所有规则。

> 5、为所有的目标文件创建依赖关系链。

> 6、根据依赖关系，决定哪些目标要重新生成。

> 7、执行生成命令。

##### 文件搜索

> VPATH特殊变量:如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make 就会在当当前目录找不到的情况下，到所指定的目录中去找寻文件了。  
>> 例：VPATH = src:../headers（先在当前目录搜索，然后是src目录和../headers目录

> vpath关键字（全小写）：  
>> 1、vpath &lt;pattern&gt; &lt;directories&gt;：为符合模式&lt;pattern&gt;的文件指定搜索目录&lt;directories&gt;。  
>> 2、vpath &lt;pattern&gt;：清除符合模式&lt;pattern&gt;的文件的搜索目录。  
>> 3、vpath：清除所有已被设置好了的文件搜索目录。  
>> 4、vapth 使用方法中的&lt;pattern&gt;需要包含“%”字符，“%”的意思是匹配零或若干字符。  
>> 例子：vpath %.h ../headers(表示，要求 make 所有以“.h”结尾的文件在“../headers”目录下搜索)  

##### 静态模式

> 语法：<targets ...>: &lt;target-pattern&gt;: &lt;prereq-patterns ...&gt;   
 &emsp;&emsp;&emsp;   &lt;commands&gt;  
 &emsp;&emsp;&emsp;   ...
 
> &lttargets ...&gt ： 定义了一系列的目标文件，可以有通配符。是目标的一个集合。  
> target-parrtern:是指明了 targets 的模式，也就是的目标集模式。(即在targets中取符合要求的文件)（应含有%通配符）  
> prereq-parrterns:是目标的依赖模式，即对target-parrtern再进行一次依赖目标的定义（应含有%通配符）  
> 例子:  
>> objects = foo.o bar.o  
>> all: $(objects)  
>> $(objects): %.o: %.c (%.o表示从目标集合中获取所有.o文件，%.c表示去.o文件并替换.o为.c)  
>> $(CC) -c $(CFLAGS) $< -o $@ (“$<”表示所有的依赖目标集（也就是“foo.c bar.c”），“$@”表示目标集（也就是“foo.o bar.o”）)





