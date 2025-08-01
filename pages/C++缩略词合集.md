- 摘自[这篇博客](https://quuxplusone.github.io/blog/2019/08/02/the-tough-guide-to-cpp-acronyms/)
- C++程序员的一大特点就是不说人话，满嘴缩略词
- ## AAA
	- **Almost Always Auto**
	- 一种由Herb Sutter引入的编码风格
	- 简单来讲就是在声明且初始化变量的时候使用auto，由编译器自行推导型别
- ## ABC
	- **Abstract Base Class**
	- 也即**虚基类**，指包含至少一个**纯虚函数(Pure Virtual Function)**的类
	- 在经典的面向对象编程范式中，ABC常作为类层级(class hierarchy)中的根类
		- 即ABC不会继承任何其他类
- ## ADL
	- **Argument-Dependent Lookup**
	- 主条目：[[C++中的ADL]]
- ## ADT
	- **Abstract Data Type**
	- 本质上来说，任何只能通过其自身提供的抽象接口来互动的类型都属于ADT
	- 但是再C++中，ADT几乎被用于特指类似于STL中那些可通过参数模板自定义的**类模板**，例如``std::vector``
- ## BMI, CMI
	- 此概念随着C++20引入**模组(Module)**而被某些编译器提及
	- [[C++模组]]
	-