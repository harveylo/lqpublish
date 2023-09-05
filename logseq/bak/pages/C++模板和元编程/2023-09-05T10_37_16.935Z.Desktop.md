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
			- ```
			  // There are 3 kinds of template parameters:
			  template <int n>                               struct NontypeTemplateParameter {};
			  template <typename T>                          struct TypeTemplateParameter {};
			  template <template <typename T> typename Tmpl> struct TemplateTemplateParameter {};
			  ```
			- **非类型模板形参**(Nontype Template Parameters)
			  collapsed:: true
				- 可以接受一个确定类型的**常量**作为**实参**(arguments)
				- 具体地说，非类型模板形参必须是一个**结构化类型**(structural type)，包括：
					- 整形，如int, char, long
					- 枚举型
					- 指针和引用类型
					- 浮点数类型和[[$red]]==字面量==(literal)类型(C++20之后支持)
						- /TODO 学习C++中的字面量
					- [[$red]]==**强调**==：必须是常量，因为模板是在编译时展开，这个阶段只有常量，没有变量、
				- ```
				  template <float &f>
				  void foo() { std::cout << f << std::endl; }
				  
				  template <int i>
				  void bar() { std::cout << i << std::endl; }
				  
				  int main() {
				    static float f1 = 0.1f;
				    float f2 = 0.2f;
				    foo<f1>();  // output: 0.1
				    foo<f2>();  // error: no matching function for call to 'foo', invalid explicitly-specified argument.
				  
				    int i1 = 1;
				    int const i2 = 2;
				    bar<i1>();  // error: no matching function for call to 'bar',
				                // the value of 'i' is not usable in a constant expression.
				    bar<i2>();  // output: 2
				  }
				  ```
			- **类型模板形参**(Type Template Patameters)
			  collapsed:: true
				- 使用关键字``typename``声明给出的参数是一个类型
				- 关键字``typename``**可以替换为**``class``
					- 建议使用``typename``，更加符合字面语义，便于阅读理解
			- **模板模板形参**(Template Template Parameters)
			  collapsed:: true
				- 和类模板类似，也是在类型的前面加上``template <..>``
				- 接受类模板或类的别名模板作为实参
				- 作为实参的模板的形参列表必须要与形参模板的形参列表**匹配**
				- ```
				  template <template <typename T> typename Tmpl>
				  struct S {};
				  
				  template <typename T>             void foo() {}
				  template <typename T>             struct Bar1 {};
				  template <typename T, typename U> struct Bar2 {};
				  
				  S<foo>();   // error: template argument for template template parameter
				              // must be a class template or type alias template
				  S<Bar1>();  // ok
				  S<Bar2>();  // error: template template argument has different template
				              // parameters than its corresponding template template parameter
				  ```
		- 一个模板可以声明一个**变长形参列表**(Template Parameter Pack)
			- 可接受0个或多个非类型常量，类型或模板作为模板实参
			- 变长形参列表必须出现在所有模板形参的**最后**
			- ```
			  // template with two parameters
			  template <typename T, typename U> struct TemplateWithTwoParameters {};
			  
			  // variadic template, "Args" is called template parameter pack
			  template <int... Args>                            struct VariadicTemplate1 {};
			  template <int, typename... Args>                  struct VariadicTemplate2 {};
			  template <template <typename T> typename... Args> struct VariadicTemplate3 {};
			  
			  VariadicTemplate1<1, 2, 3>();
			  VariadicTemplate2<1, int>();
			  VariadicTemplate3<>();
			  ```
		- 模板可以声明**默认实参**，类似于函数的默认实参
			- 只有**主模板**(Primary TEmplate)参可以声明默认实参，**模板特化**(Template Specialization)不行
			- ```
			  // default template argument
			  template <typename T = int> struct TemplateWithDefaultArguments {};
			  ```
	- ## 模板实例化(Instantiation)
		- 由模板定义生成具体的类型，函数和变量的过程
		- 实例化过程中，模板形参被替换(substitute)为实参
		- 实例化分为两种
			- **按需(隐式)实例化**(On-demand(implicit) instantiation)
				- 最常用的
				- 需要使用此模板生成的实体来**生成具体对象**时才做实例化
			- **显式实例化**(explicit instantiation)
				- 在还不需要生成对象时就显式告诉编译器去实例化一个模板
		- 两者并不是通过是否隐式传参而区分的
		- C++11之后还支持了**显式实例化声明**(explicit instantiation declaration)
		- ```
		  // t.hpp
		  template <typename T> void foo(T t) { std::cout << t << std::endl; }
		  
		  // t.cpp
		  // on-demand (implicit) instantiation
		  #include "t.hpp"
		  foo<int>(1);
		  foo(2);
		  std::function<void(int)> func = &foo<int>;
		  
		  // explicit instantiation
		  #include "t.hpp"
		  template void foo<int>(int);
		  template void foo<>(int);
		  template void foo(int);
		  ```
		- 模板在实例化的过程中，编译器会用模板的实参去替换形参，并根据一定会将生成的代码插入到某个合适的位置，该位置叫做**Point of Instantiation(POI)**
		- 因此模板对于POI一定要是**可见的**
		- 一般会把模板的定义放在头文件中，在使用时通过包含头文件来获取定义，这种编写方式叫做**包含模型**(Inclusion Model)
			- 包含模型有着一些缺陷，如果多个翻译单元(源文件)都include一个头文件，则具有相同实参的模板实例会在每一个源文件中都实例化一次，残生多个相同的类型，函数和变量
				- 如果不加以处理，会出现**重定义错误**。但是现代编译器一般会对由模板生成的代码做处理，例如GNU体系下，模板实例化生成的符号都会被标记为**弱符号**(Weak Symbol)
				- 编译时间将更加漫长
	- ## 模板实参推导
		- 模板实例化并非需要指定所有实参，编译器有时可以根据函数调用的实参来推断模板实参，此过程称作**模板实参推导**(Template Arguments Deduction)
		- 所过所有的实参都能被推导出来且推导结果无冲突，则推导成功，反之会产生错误
			- ```
			  template <typename T>
			  void foo(T, T) {}
			  
			  foo(1, 1);      // #1, deduced T = int, equivalent to foo<int>
			  foo(1, 1.0);    // #2, deduction failed.
			                  // with 1st arg, deduced T = int
			                  // with 2nd arg, deduced T = double
			  ```
		- **C++17**中引入了类模板实参推导(Class Template Arguments Deduction)，可以通过构造函数来推导模板实参
			- ```
			  template <typename T>
			  struct S { S(T, int) {} };
			  
			  S s(1, 2);     // deduced T = int, equivalent to S<int>
			  ```
	- ## 模板特化
		- 将一部分或全部的形参替换，得到一个新的模板实现，成为**模板特化**(Template Specialization)
		- 替换部分形参的叫做**偏特化(Partial Specialization)**，全部替换的成为**全特化(Full Specialization)**
		- 未经过特化的原始模板称作**主模板(Primary Template)**
		- 只有**类模板**和**变量模板**可以进行偏特化，函数模板**只能**进行全特化
		- 实例化模板时，编译器会从所有特换版本中选择**最匹配**的实现来做替换，如果没有特化模板匹配成功，则会选择主模板进行替换操作
		- ```
		  // function template
		  template <typename T, typename U> void foo(T, U)       {}     // primary template
		  template <>                       void foo(int, float) {}     // full specialization
		  
		  // class template
		  template <typename T, typename U> struct S             {};    // #1, primary template
		  template <typename T>             struct S<int, T>     {};    // #2, partial specialization
		  template <>                       struct S<int, float> {};    // #3, full specialization
		  
		  S<int, int>();      // choose #2
		  S<int, float>();    // choose #3
		  S<float, int>();    // choose #1
		  ```
		- 也可以先声明一个特化，然后在其他地方定义。需要注意的是，特化声明和现实实例化的语法非常相似，注意不要混淆：
			- ```
			  // Don't mix the syntax of "full specialization declaration" up with "explict instantiation"
			  template    void foo<int, int>;   // this is an explict instantiation
			  template <> void foo<int, int>;   // this is a full specialization declaration
			  ```
		- 二者的含义实际上也十分相近
			- 模板实例化的结果就是一个模板的全特化
			- 有的书籍和文档会使用**特化**来直接直称呼实例化之后生成的实体
			- 为了区分可以将模板实例化成为**隐式特化**(implicit specialization)，真正的模板特化成为**显式特化**(explicit specialization)
	- ## 函数模板重载
		- 函数模板不能偏特化，但是可以**重载(overloading)**，并且可以和普通函数一起重载
		- C++中的所有函数和模板只要有用不同的**签名(signature)**，就可以在程序中共存，签名包括：
			- 非限定名(Unqualified Name)，也就是一个不包含类名包名命名域之类的简单的函数(模板)名
			- 域(Scope)
			- 成员函数(模板)的CV限定符(const, volatile)
			- 成员函数(模板)的引用限定符
			- 函数形参类型列表，对于模板来说是其实例化之前的形参列表
			- 返回值类型
			- 模板形参列表
		- ```
		  template <typename T> void foo(T) {}    // #1
		  template <typename T> void foo(T*) {}   // #2
		  template <typename T> int foo(T) {}     // #3
		  template <typename T, typename U> void foo(T) {}    // #4
		  template <typename T, typename U> void foo(T, U) {} // #5
		  template <typename T, typename U> void foo(U, T) {} // #6
		  void foo(int) {}         // #7
		  void foo(float, int) {}  // #8
		  ```
		- 需要注意的是，虽然\#5和\#6被看作重载可以通过编译，但是在使用时可能会产生**歧义错误**，编译器无法确定两个函数模板中哪个的重载优先级更高
		- ```
		  foo(1);           // call #7
		  foo(new int(1));  // call #2
		  foo(1.0f, 1);     // call #8
		  foo<int, float>(1, 1.0f);  // call #5
		  foo(1, 1.0f);              // error: ambiguous
		  ```
	- ## TMP的一般规律
		- 模板像函数一样被使用，类模板的静态成员可以作为函数的返回值
		- 通过**实例化**来调用模板
		- 通过**特化**和**重载**可以实现分支选择
		- 通过递归实现循环逻辑
		- 所有过程发生在**编译时**，由编译器驱动