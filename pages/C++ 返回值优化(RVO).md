- 在C++语境下，**拷贝省略(Copy Elision)**和RVO基本是同义词，前者是字面意义，后者既是适用场景，也是实现细节
- 拷贝省略是在C++标准中的一种[优化方式](https://en.cppreference.com/w/cpp/language/copy_elision)
- 若函数返回类型为变量而非引用和指针，且实际返回时返回了一个在自己栈帧上分配的本地变量或临时变量，大多数情况下编译器都会尝试进行进行**[[$red]]==RVO==**(Return Value Optimization)。且如果**返回对象在编译时就可确定**，则大概率会进行RVO。
- 所返回对象谓编译时可确定是指：
	- return的对象类型就是函数返回的类型
	- 没有复杂的返回逻辑，即只有一个return
- RVO还可以细分为URVO(Unnamed Return Value Optimization)和NRVO(Named Return Value Optimization)
	- 前者是返回匿名对象(通过调用构造函数或返回对象的函数)
		- ```C++
		  T func{
		    return T();
		  }
		  ```
	- 后者是已经在先创建了对象，然后返回
		- ```C++
		  T func{
		    T t;
		    return t;
		  }
		  ```
- RVO的**原理**：直接在调用者的栈帧上分配内存，而不会先在函数的栈帧中初始化一次，再通过拷贝构造函数复制到调用者的栈帧中
	- 实际上就是函数内部会拿到一个函数外部(调用者)的地址指针，在此地址上构建对象
- 在以下情况**编译器大概率不会进行RVO**
	- 函数的具体返回值在运行时动态确定(条件判断或三元运算符如return b?a:c)
	- 函数的返回类型是基类或虚基类
	- 函数的返回类型是数组类型([[$red]]==?==)
	- 函数的返回类型是``std::intitializer_list``[[$red]]==(?)==
	- 函数的返回类型是非class类型(基础类型，枚举，void等)
- **[[$red]]==使用移动语义也会干扰RVO==，导致额外移动构造函数的调用**
- 可以手动关闭编译器的返回值优化
	- 添加选项``gcc：-fno-elide-constructors``