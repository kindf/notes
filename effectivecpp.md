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

> #### 条款23:宁以non-member、non-friend替换member函数  
> 这样做可增加分装性、包裹弹性和机能扩充性  

> #### 条款24:若所有参数皆需类型转换,请为此采用non-member函数  

> #### 条款25:考虑写出一个不抛异常的swap函数  
> 没看懂。。

> #### 条款26:尽可能延后变量定义式的出现时间  
> 有助于增加程序清晰度和改善程序效率  

> #### 条款27:尽量少做转型动作  
> const_case<T>(expression):通常被用来将对象的常量性移除  
> dynamic_case<T>(expression):主要用于执行"安全向下转型"  
> reinterpret_case<T>(expression):意图执行低级转型,实际动作及结果可能取决于编译器  
> static_case<T>(expression):用于强迫隐式转换  

> #### 条款28:避免返回handles指向对象内部成分  
> hanles包括references、指针以及迭代器  

> #### 条款29:为"异常安全"而努力是值得的  
> 异常安全的两个条件:不泄露任何资源和不允许数据败坏  

> #### 条款30:透彻了解inlining的里里外外  
> 过度使用inlining函数会导致代码膨胀  
> inlining只是对编译器的申请而不是强制命令  
> 所有对virtual函数(除非是最平淡无奇的)的调用会使inlining落空(inline在编译期而virtual调用函数在运行期才确定)  
> 构造函数和析构函数往往不好inline(编译器会为其生成较复杂的代码)  

> #### 条款31:将文件间的编译依存关系将至最低  
> 

> #### 条款32:确定你的public继承塑模出is-a关系  
> 即devired class对象适用于任何base class适用的场合  

> #### 条款33:避免遮掩继承而来的名称  
> derived class内的名称会遮掩base class内的名称,可使用using声明式或转交函数避免  

> #### 条款34:区分接口继承和实现继承  
> 纯虚函数表示其derived class必须重写该函数,而base class中通常不给出具体实现  
> 要实现"必须重写"和"父类提供实现"两重语义,可在base class中声明pure virtual并提供一个对应的non-virtual版本的实现  
> 或者在base class中直接定义pure virtual函数的实现  

> #### 条款35:考虑virtual函数以外的其他选择  

> ####  条款36:绝不重新定义继承而来的non-virtual函数  
> 注意使用指针调用重新定义virtual函数和重新定义non-virtual函数的区别  
> 注意是"绝不" 

> #### 条款37:绝不重新定义继承而来的缺省参数值  
> 对于non-virtual函数,参考条款36  
> 对于virtual函数:由于virtual函数是动态绑定的而缺省参数值是静态绑定的,即"调用一个定义于derived class内的virtual函数,使用的却是base class为其所指定的缺省参数值  

> #### 条款38:通过复合塑模出has-a或"根据某物是实现出"  
> 复合意味着has-a或者"根据某物实现出"  
> public继承意味着is-a  

> #### 条款39:明智而审慎地使用private继承  
> private继承意味着"根据某物实现出",与复合类似(尽可能使用复合)  

> #### 条款40:明智而审慎地使用多重继承  
> 

> #### 条款41:了解隐式接口和编译期多态  
> 了解"编译期多态"与"运行期多态"的区别  

> #### 条款42:了解typename的双重定义  
> 在嵌套从属类型名称前面加上关键字typename  
> 不得在base class lists(基类列)或member initialization list(成员初值列)内以typename作为base class修饰符  

> #### 条款43:学习处理模板化基类内的名称  
> 通过继承模板类的模板类中,调用父类的成员函数可能导致编译报错  
> 解决1:显示调用this->  
> 解决2:使用using声明式(条款33)  
> 解决3:指明调用函数位于base class中  

> #### 条款44:将与参数无关的代码抽离templates  
> 