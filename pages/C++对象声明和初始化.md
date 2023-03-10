- C++的对象初始化语法**相当混乱**，总的来说，`{}`，`()`和`=`都可以用于赋初始值
- ```
  int x(0);
  int y = 0;
  int z{0};
  ```
- 对于基本类型的初始化赋值，这些语法的区别**是模糊的**
- 但是对于用户自定义的类，不同的初始化赋值**区别很大**
- ```
  Widget w1 //调用默认构造函数
  Widget w2 = w1 //调用拷贝构造函数
  w1 = w2 //赋值操作，调用operator=函数
  ```
- 因为初始化语法的混乱，C++11引入了**统一初始化**语法，语法上使用大括号，有人成为**大括号初始化**
- **大括号**可以对容器进行**列表初始化**``std::vector<int> v{1,2,3};``
- **大括号**也可以指定**类成员默认初始值**，在C++11中，此功能也可以通过``=``实现，``()``不行
	- ```
	  class widget {
	  public:
	  	int x{0};
	      int y = 0;
	      int z(0); //编译器报错
	  }
	  ```
- **不可拷贝对象**(如std::atomic)可以用**大括号**和**小括号**初始化，**不能**使用等号
	- ```
	  std::atomic<int> a1{0};
	  std::atomic<int> a2(0);
	  std::atomic<int> a3 = 0; //编译报错
	  ```
- [[$red]]==**注意**==：大括号用于基本类型初始化时，如果存在**精度丢失问题**，编译器会报错
	- ```
	  double d = 3.14;
	  int a{d}; //编译器报错
	  int b(d); //无报错
	  ```
- 使用大括号可以避免**调用无参构造函数**的歧义问题
	- ```
	  Widget w1(0); //带参构造函数
	  Widget w2(); //歧义，声明了一个函数
	  Widget w3; //调用默认初始化函数
	  Widget w4{}; //无歧义
	  ```
- 但是大括号初始化会导致**[[$red]]==构造函数劫持==**的问题！
	- 如果定义了使用``std::initializer_list``作为形参的构造函数，大括号初始化会**强制**将实参转化为初始化列表然后调用使用``std::initializer_list``的构造函数
		- ```
		  class Widget {
		  public:
		    Widget(int i, bool b);
		    Widget(int i, double d);
		    Widget(std::initializer_list<long double> il);
		    ...
		  };
		  Widget w1(10, true);   // 使用圆括号，调用第一个构造函数
		  
		  Widget w2{10, true};   // 使用大括号，强制调用第三个构造函数，10和true被转换为long double                    
		  
		  Widget w3(10, 5.0);    // 使用圆括号，调用第二个构造函数
		  
		  Widget w4{10, 5.0};    // 使用大括号，强制调用第三个构造函数，10和5.0被转换为long double
		  ```
	- 甚至会劫持**拷贝构造函数**和**移动构造函数**
		- ```
		  class Widget {
		  public:
		    Widget(int i, bool b);
		    Widget(int i, double d);
		    Widget(std::initializer_list<long double> il);
		    operator float() const;   // 支持隐式转换为float类型
		    ...
		  };
		  
		  Widget w5(w4);    // 使用圆括号，调用拷贝构造函数
		  
		  Widget w6{w4};    // 使用大括号，调用第三个构造函数
		                    // 原因是先把w4转换为float，再把float转换为long dobule
		  
		  Widget w7(std::move(m4));  // 使用圆括号，调用移动构造函数
		  
		  Widget w8{std::move(m4)};  // 使用大括号，调用第三个构造函数，理由同w6
		  ```
	- 即使给出的实参无法匹配``std::initializer_list``，大括号初始化也会**义无反顾地选择**``std::initializer_list``构造函数，忽略其他构造函数，哪怕是参数精确匹配的
		- ```
		  class Widget {
		  public:
		    Widget(int i, bool b);
		    Widget(int i, double d);
		    Widget(std::initializer_list<bool> il);  // long double 改为 bool
		    ...
		  };
		  
		  Widget w{10, 5.0};  // 报错，因为发生范围窄化转换
		                      // 编译器会忽略另外两个构造函数(第二个还是参数精确匹配的！)
		  ```
	- 只有当给出的实参无法转化为``std::initializer_list``的参数类型时，才会考虑其他构造函数
		- ```
		  class Widget {
		  public:
		    Widget(int i, bool b);
		    Widget(int i, double d);
		    Widget(std::initializer_list<std::string> il);  // bool 改为 std::string
		    ...
		  };
		  
		  Widget w1(10, true);  // 使用圆括号，调用第一个构造函数
		  
		  Widget w2{10, true};  // 使用大括号，不过调用第一个构造函数，因为无法转换为string
		  
		  Widget w3(10, 5.0);   // 使用圆括号，调用第二个构造函数
		  
		  Widget w4{10, 5.0};   // 使用大括号， 不过调用第二个构造函数，因为无法转换为string
		  ```
	- 但是**无参构造函数**并不会受到影响，``Widget w1{};``仍然会调用默认构造函数
	- 而如果想要将一个空的``std::initializer_list``作为参数而调用``std::initializer_list``构造函数，应该使用：``Widget w1({});``
- 要**注意正确区分**小括号和大括号可能会调用的不同的构造函数，例如对于``std::vector``来说
	- ```
	  std::vector<int> v1(10, 20);   // 使用不带std::initializer_list的构造函数
	                               // 创建10个元素的vector，每个元素的初始值为20
	  - std::vector<int> v2{10, 20};   // 使用带std::initializer_list的构造函数
	                               // 创建2个元素的vector，元素值为10和20
	  ```