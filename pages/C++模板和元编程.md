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
		- 后两种模板不需要定义，因为起步产生运行时实体
		-