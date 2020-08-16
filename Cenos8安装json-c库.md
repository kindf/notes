#### Cenos8安装json-c库

##### 1、现把文件从https://github.com/json-c/json-c克隆到本地目录（本人存放在~/Documents/json中

##### 2、进入json-c目录分别执行cmake ./，sudo make install（cmake要先下好）

##### 3、安装完成后查看一下/usr/local/include目录下是否有json-c目录（没有就是安装的时候没给权限导致复制失败）

##### 4、查看/usr/local/lib目录或者/usr/local/lib64目录下是否有json-c的动态库文件（没有的话应该时安装有问题）

##### 5、添加动态路的路径：sudo vim /etc/ld.so.conf添加/usr/local/lib或者/usr/local/lib64（根据步骤4的目录）

##### 6、用 sudo ldconfig更新文件/etc/ld.so.conf的修改生效

##### 7、使用时要包含json.h头文件，编译时需要加上-ljson-c（我看网上的都是-json的，这个应该是要看动态库的名称确定吧）

##### 8、安装时遇到的问题

> 1）没有安装cmake工具导致执行cmake命令时出现问题（cmake的版本也要确定一下）
>
> 2）使用-json-c编译导致编译失败
>
> 3）make install时没给权限导致install失败（安装时弹了一个error，没仔细看出来）
>
> 4）没有添加动态库的加载路径，导致执行文件时会报错找不到动态库文件，用ldd查看一下是libjson-c.so.5文件没找到，加了路径记得要sudo ldconfig更新一下不然还是不生效的

