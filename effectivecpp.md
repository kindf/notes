> #### 条款01:视C++为一个语言联邦  
> 主要包括四个层面:C;Object-Oriented C++;Template C++;STL  

> #### 条款02:尽量以const,enum,inline替代#define  
> 用const替换#define表示常量(包括class常量成员)
> 用inline函数替换#define定义的形似函数的宏  

> #### 条款03:尽可能使用const  
> const成员函数:两个成员函数可以因为常量性不同而被重载,const对象优先执行const成员函数(不改变成员变量)  
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
> 可将相应的成员函数声明为private并不予实现(默认构造、复制构造、copy assignment、析构函数)  

> #### 条款07:为多态基类声明virtual析构函数  
> 当derived class对象由一个base class指针被删除,而base class带有一个non-virtual析构函数,其结果未有定义  
> 实际上通常发生对象的derived成分没被销毁(局部销毁)  
> 当class不企图被当做base class,不声明virtual析构函数  
> 如果class带有任何virtual函数,其析构函数也应为virtual  

> #### 条款08:别让异常逃离析构函数  
> 应在析构函数中捕捉异常并处理  
> 如果客户需要对某个操作函数抛出的异常做出反应,class应提供一个普通函数执行该操作  

> #### 条款09:绝不在构造和析构过程中调用virtual函数  
> derived class对象内的base class成分会在derived class自身成分被构造之前先构造妥当  
> 因此base class构造函数中的virtual函数调用的还是base class内的版本  
> 析构函数:derived class析构函数执行时,对象内derived class成员变量先被析构,进入base class析构时,调用的virtual还是base class内的版本  
> 可将相应的virtual改为non-virtual并要求derived class构造函数传递必要的信息给base class构造函数  

> #### 条款10:令operator=返回一个reference to *this  
> 为实现“连续赋值”的处理  

> #### 条款11:在operator=中处理“自我赋值”
> 即在operatot=中判断是否为“自我赋值”  

> #### 条款12:复制对象时勿忘其每一个成分  
> copying函数应该确保复制"对象内的所有成员变量"和"所有base class成分"  
> 不要尝试以某个copying函数实现另一个copying函数。应该将共同技能放进第三个函数中,并由两个coping函数共同调用  

> #### 条款13:以对象管理资源  
> 为防止资源泄露,请使用RALL对象,即在构造函数中获取资源在析构函数中释放资源  
> auto_ptr的复制动作会使原来的指针指向NULL(使用share_ptr更佳)  

> #### 条款14:在资源管理类中小心copying行为  
> 禁止复制
> 对底层资源祭出引用计数法(类似share_ptr)  
> 复制底部资源(深复制)  
> 转移底部资源的拥有权  

> #### 条款15:在资源管理类中提供对原始资源的访问  
> 由于APIs往往要求访问原始资源  
> 对原始资源的访问可经由显示转换或隐式转换(看情况)  

> #### 条款16:成对使用new和delete时要采用相同的形式  
> 尽量不对数组形式做typedefs动作(防止使用delete时的误用)  

> #### 条款17:以独立语句将newed对象置入智能指针  
> 防止内存泄露(异常导致内存泄露why)  

> #### 条款18:让接口容易被正确使用,不易被误用  

> #### 条款19:设计class犹如设计type  

> #### 条款20:宁以pass-by-reference-to-const替换pass-by-value  
> pass-by-reference-to-const通常比较高效且避免了切割问题  
> 小型的type并不意味着copy构造函数不昂贵  

> #### 条款21:必须返回对象时,别妄想返回其reference  
> 返回local stack对象,会使pointer或reference指向一个废除的空间  
> 返回heap-allocated对象,客户不能正确释放其空间,导致内存泄露  
> 返回local static对象,想象返回多个对象时,其都指向同一个static对象  

> #### 条款22:将成员变量声明为private  
> proteced并不比public更具封装性  



