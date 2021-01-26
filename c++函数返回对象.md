---
title: "C++函数返回对象"
date: 2021-01-26T22:06:01+08:00
draft: true
---

#### c++函数返回对象

```
class TestObj
{
	int key;
	int val;
}
TestObj TestReturn()
{
	TestObj obj;
	obj.key = 1;
	return obj;
}
int main()
{
	TestObj main_obj;
	main_obj = TestReturn();
	return 0;
}
```



> ##### 1.首先main函数在栈上开辟空间，将其一部分作为传递返回值的临时对象（暂称temp）
>
> ##### 2.将temp的地址作为隐藏参数传递给被调用函数
>
> ##### 3.被调用函数将返回值拷贝给temp对象（此处发生一次拷贝构造），并将temp对象的地址传出
>
> ##### 4.函数返回后，main函数将返回地址的对象（temp）的内容赋值给main_obj（此处发生一次赋值操作）