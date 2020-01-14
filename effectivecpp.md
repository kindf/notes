> #### 条款01:视C++为一个语言联邦  
> 主要包括四个层面:C;Object-Oriented C++;Template C++;STL  

> #### 条款02:尽量以const,enum,inline替代#define  
> 用const替换#define表示常量(包括class常量成员)
> 
> 用inline函数替换#define定义的形似函数的宏  

> #### 条款03:尽可能使用const  
> const成员函数:两个成员函数可以因为常量性不同而被重载,const对象优先执行const成员函数(不改变成员变量),
> bitwise constness(编译器的做法)与logical constness:主要是探讨类似指针成员变量本身不改变但所指之物有所改变这种情况下是否属于const成员函数的问题
> 编译器会强制实施bitwise constness但编程时应使用logical constness的思想  
> mutable关键字:被修饰的成员变量总是会被改变的,即使在const成员函数内
> 为避免代码重复的问题,可以用const成员函数实现non-const成员函数(示例如下)
```
Class TextBlock{
public:
...
	const char& operator[](std::size_t position) const
	{
	...
		return text[position];
	}
	char& operator[](std::size_t position)
	{
		return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
		//两次转型操作(第一次为*this添加const,第二次是从const operator[]返回值移除const)
	}
}
```

> #### 条款04:确定对象被使用前已先被初始化  
> 对于内置类型对象,应在使用对象之前将其初始化
> 对于非内置类型对象,应在其构造函数尽量使用初始化列表(效率更高)
> 可以通过指定无物作为初始化实参的初始化列表构造成员变量(示例如下)
```
ABEntry:ABEntry():thName(), theAddress(), thePhones(), numTimesConsulted(0) {}
//若成员变量为const或reference,则一定需要初值而不能赋值
//内置类型也能使用
```

> #### 条款05:了解C++默默编写并调用哪些函数
> 若没声明,编译器会自动生成默认构造函数、复制构造函数、copy assignment操作符、析构函数  
> 

> #### 条款06:若不想使用编译器自动生成的函数,应明确拒绝
> 可将相应的成员函数声明为private并不予实现

