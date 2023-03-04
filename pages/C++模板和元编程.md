- C++的模板可以看作一种元编程的手段，且是使用宿主语言直接进行元编程的类型
- **模板元编程**(Template Metaprogramming, **TMP**)，借助编译器处理模板的能力，TMP可以在**编译期间**生成代码，这些代码最终和普通的C++代码一起编译执行
- TMP已被证明是**图灵完备**的，即任何可又计算机程序表达的计算都可以通过TMP来完成
- # Introduction
	- ## TMP历史
		- 最初，C++的**编译计算能力**并非是有意设计的
		- 1994年，有人展示了一个可以产生质数的程序
		- 在此之后出现了很多TMP的库
			- boost: [MPL](https://www.boost.org/doc/libs/1_76_0/libs/mpl/doc/index.html)(2003), [hana](https://www.boost.org/doc/libs/1_76_0/libs/hana/doc/html/index.html)(2016), [mp11](https://www.boost.org/doc/libs/1_76_0/libs/mp11/doc/html/mp11.html)(2017)
			- github: [brigand](https://github.com/edouarda/brigand)(2015), [meta](https://github.com/ericniebler/meta)(2015), [metal](https://github.com/brunocodutra/metal)(2015)
		- C++标准在C++11之后对TMP进行了大幅支持
	- ## TMP的优势
		- ### 将一部分计算转移到编译时
			- 可以降低运行时的运行效率
			- 代价是增加编译时间和目标文件的体积
			- 类似于渲染中的**烘焙**技术
			- 考虑到一般项目大多是一次编译多次运行，因此用编译时间来换取运行效率是相当划算的
		- ### TMP可以对类型(type)做计算
			- 在运行时根据变量的类型编写逻辑基本不可能
			- 但是编译时，类型可以作为模板的实参(Argument)参与逻辑运算
		- ### 使代码更加简洁
			- TMP自身的代码晦涩难懂，但是可以让使用它的外部代码变得简洁优雅
			- 类似于在C语言中使用宏
		- ### TMP应用广泛
			- TMP已经成为了广泛使用且不可或缺的技术
			- 许多程序库，尤其是泛型程序库，都是用了TMP，学习使用TMP对阅读这些库源码和深度使用它们相当有帮助
- # 模板基础
	- ## 模板声明(declare)和定义(definition)
		- C++中一共可以声明物种不同的模板，分别是
			- **类模板**(class template)
			- **函数模板**(function ~)
			- **变量模板**(variable ~)
			- **别名模板**(alias ~)
			- **概念**(concept)
		- ```
		  // declarations
		  template <typename T> struct  class_tmpl;
		  template <typename T> void    function_tmpl(T);
		  template <typename T> T       variable_tmpl;          // since c++14
		  template <typename T> using   alias_tmpl = T;         // since c++11
		  template <typename T> concept no_constraint = true;   // since c++20
		  ```
		- 前三种模板可以拥有**定义**(definition)
		- ```
		  // definitions
		  template <typename T> struct  class_tmpl {};
		  template <typename T> void    function_tmpl(T) {}
		  template <typename T> T       variable_tmpl = T(3.14);
		  ```
		- 后两种模板不需要定义，因为不会产生运行时实体
		- 相较于普通的类，函数，变量，他们的模板化声明和定义只是多了一个**template**关键字和使用``<>``括起来的模板参数
		- 模板参数通常是**类型**因为模板的本意是事项**泛型编程**(Generic Programming)
			- 也因为如此，其本意并不是为了进行计算，因此TMP的代码语法往往反人类，难读难写
		- **注意**：
			- **类模板**(class template)和**模板类**(template class)并不相同，前者是一个模板，而后者是由模板生成的一个类，函数类和类函数等也都是类似的关系
			- 优点类似于类和对象的关系
				- 类对一个对象的特征做出**描述**，在**运行时**对象根据这些描述被创建(一块内存区间)
				- 模板对代码做出**描述**，在**编译时**按描述生成对应代码
		- **注意**：
			- TODO 学习C++lambda表达式
			- C++14中新增了泛型lambda(generic lambda)，其定义看起来像模板，但并不是模板
			- 其函数调用运算符是一个函数模板
			- ```
			  // NOTE: Generic lambda (since c++14) is NOT a template,
			  // its operator() is a member function template.
			  auto glambda = []<typename T>(T a, auto&& b) { return a < b; };
			  ```
	- ## 模板的形参和实参
		- 一个模板可以有三种类型的**形参**(parameters)
			- **非类型模板形参**(Nontype Template Parameters)
				- 可以接受一个确定类型的**常量**作为**实参**(arguments)
				- 具体地说，非类型模板形参必须是一个**结构化类型**(structural type)，包括：
					- 整形，如int, char, long
					- 枚举型
					- 指针和引用类型
					- 浮点数类型和[[$red]]==字面量==(literal)类型(C++20之后支持)
						- /TODO 学习C++中的字面量
					- [[$red]]==**强调**==：必须是常量，因为模板是在编译时展开，这个阶段只有常量，没有变量
			- **类型模板形参**(Type Template Patameters)
			- **模板模板形参**(Template Template Parameters)
			- ```
			  // There are 3 kinds of template parameters:
			  template <int n>                               struct NontypeTemplateParameter {};
			  template <typename T>                          struct TypeTemplateParameter {};
			  template <template <typename T> typename Tmpl> struct TemplateTemplateParameter {};
			  ```
			-