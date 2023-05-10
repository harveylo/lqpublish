- 虚函数是[[多态]]的重要实现方式
- # 基本特性
	- 虚函数在被调用时，**会匹配[[$red]]==派生最远的重写版本==**
		- 匹配规则是**相同函数签名(函数名，参数个数与类型和返回类型)**
			- [[$red]]==**返回类型可以不相同**==，但必须存在派生关系
	- 虚函数声明时只需在前面加上``virtual``关键字即可
		- 在类生命之外的实际函数实现时不能加``virtual``关键字
		- 在C++中，基类也是可以实现虚函数的，但是若指向的是一个重写了此虚函数的派生类，并不会调用
	- 在派生类实际实现虚函数时，如果不加上``override``关键字，则必须保证子类中实现的虚函数和基类的对应虚函数的函数签名完全相同
		- 否则就会变为子类自己声明了一个新的虚函数
	- 为了避免因为小失误导致函数签名不匹配而实现失败，加上``override``关键字之后可以**显式告知编译器，正在写的虚函数是基类中某个虚函数的重写而不是一个新的虚函数**，此时若再出现函数签名不匹配的情况，则编译无法通过
	- **``final``**关键字修饰的虚函数[[$red]]==**不允许**==子类进行重写：``virtual void fun() const override final``
		- ``final``还可以直接修饰类，表示此类不允许继承：``class B final: public A``
	- ## 协变返回类型(Covariant Return Type)
		- 上述提到的重写虚函数必须匹配函数签名有一个例外情况，那就是：
			- 父类虚函数的返回值是一个类的指针
			- 子类重写虚函数的**返回值**，是父类中对应虚函数**返回值**的**[[$red]]==子类==**的**指针或引用**
	- ## 将析构函数声明为虚函数
		- 对于会进行继承的类，如果析构函数不声明为虚函数，则在``delete``一个实际指向子类的父类指针时，只会调用父类的析构函数，将有可能导致[[$red]]==**内存泄漏**==
- # 进阶
	- ## 函数捆绑调用
		- 捆绑(binding)指将标识符(identifier)转化为地址
		- ### 静态绑定(static binding)
			- 也称**早期绑定(early binding)**
			- 一般的函数调用都是如此绑定，发生在编译时
			- 可以看作**链接**的同义词
		- ### 动态绑定(dynamic binding)
			- 也称**后期绑定(late binding)**
			- 在编译时并不知道到底在调用哪个函数的时候，就只能在运行时进行绑定，包括两种典型情况：
				- 虚函数
				- 函数指针
	- ## 虚函数表(Vtable)
		- 每个具有一个或多个虚函数的类都有一张虚表
		- 虚表是在**编译时**建立的**静态数组**，包含了每个虚方法的**函数指针**
			- 每一个指针指向该类可见派生最远的函数实现
		- 编译器会在基类对象添加一个**隐含指针**``*__vptr``指向虚表
			- 当使用某个对象调用虚方法时，通过此指针查找虚表
			- **能够被子类继承**
		- 每一个实例的虚表中的entry指向自己所能看到的最远派生，例如对于例子：
			- ```
			  class Base
			  {
			  public:
			      virtual void function1() { }
			      virtual void function2() { }
			  }
			  
			  class D1: public Base
			  {
			  public:
			      virtual void function1() override { }
			  }
			  
			  class D2: public Base
			  {
			  public:
			      virtual void function2() override { }
			  }
			  ```
		- 虚表结构如下：
			- ![](https://pic4.zhimg.com/v2-9bfc157fe7324e86a095fc58b03555e7_r.jpg)
	- ## 纯虚函数与抽象基类
		- 若并不希望一个基类的虚函数在基类中有相关实现，可以将其定义为**纯虚函数**，语法为：
			- ``virtual T function() =0;``
			- 即在函数声明后紧跟一个``=0``
		- 如果一个类含有**一个或以上**的纯虚函数，则该类为**抽象类**，[[$red]]==**抽象类无法实例化**==
		- 继承抽象类的子类**[[$red]]==必须==**重写所有纯虚函数，否则该子类[[$red]]==**也将成为一个抽象类**==
		- 抽象类的指针可以指向一个可实例化的子类对象
	- ## 接口类
		- Java和C\#等语言中存在接口的概念和语法，C++不提供原生的接口类语法，但是可以用[[$red]]==**只有纯虚函数**==的抽象类来模拟类似地效果
		- 所以C++中可以称**只有纯虚函数**的抽象类为**接口类**或**纯抽象类**
	- ## 虚基类
		- 在菱形继承中可能出现歧义问题
		- ![](https://pic1.zhimg.com/80/v2-d2abdaa786fdce1a432dfc9c1dc3ae94_720w.webp)
		- 再上图中，Copier中实际会出现两个PoweredDevice对象，此时若调用一个PowerDevice中的虚函数则会出现错误，不知道使用哪个PoweredDevice对象的函数
		- 解决方法是在继承时使用虚继承，语法如：``class A : virtual public B``
		- 被虚继承的类只会在子类中被实例化一次，但是虚继承的基类需要由派生最远的类实例化否则没有基类对象被创建
		- ```
		  class PoweredDevice
		  {
		  public:
		      PoweredDevice(int power)
		      {
		          cout << "PoweredDevice: " << power << endl;
		      }
		  
		      virtual void reportError() { cout << "Error" << endl; }
		  };
		  
		  class Scanner : virtual public PoweredDevice
		  {
		  public:
		      Scanner(int scanner, int power) :
		          PoweredDevice(power)
		      {
		          cout << "Scanner: " << scanner << endl;
		      }
		  };
		  
		  class Printer : virtual public PoweredDevice
		  {
		  public:
		      Printer(int printer, int power) :
		          PoweredDevice(power)
		      {
		          cout << "Printer: " << printer << endl;
		      }
		  };
		  
		  class Copier : public Scanner, public Printer
		  {
		  public:
		      // Note: 虚基类是由派生最远的类负责创建，所以，
		      //       构造函数初始化列表中需要增加虚基类的构造函数调用
		      Copier(int scanner, int printer, int power):
		          Scanner(scanner, power), Printer(printer, power),
		          PoweredDevice(power)
		      {}
		  
		  };
		  ```
		- 实际上多重继承并不推荐，会使继承结构变得复杂难测，在Java，C\#之类的语言中，不允许继承多个类，仅允许继承多个接口，C++中可能也可以模仿相关编程风格
	- ## 对象切片
		- 使用基类引用或指针指向一个子类不会造成问题，也是多态实现的一般操作
			- ``A a; B& b1 = a;B* b2 = &a``
		- 但是如果直接将一个子类**[[$red]]==赋值==**给基类，则可能会导致基类实例并不能获取实例的所有信息，后续可能会造成意想不到的情况
		- 之中因为子类赋值给基类导致只获取部分信息的情况叫做**对象切片(Object Slicing)**
	- ## 动态转型
		- 一般来说，**向上转型(Upcasting)**，即子类类型转换为父类甚至更向上的基类是不会产生问题的，因为子类本来就包含了其所继承的所有类的信息
		- 但是**向下转型(Downcasting)**，则不然，如果一个基类指针真的是指向的一个基类实例，则将其向下转型为子类指针则有可能产生不可控行为
		- 可以使用``dynamic_cast``动态转型操作符完成向下转型，在无法进行向下转型时，将返回**[[$red]]==空指针==**(对于指针类型)或**抛出[[$red]]==std::bad_cast==**异常(对于引用类型)，例：
			- ```
			  void process(Base* ptr)
			  {
			      Derived* derived = dynamic_cast<Derived*>(ptr);
			      if (derived == nullptr)
			      {
			          // 后序处理
			          // ...
			      }
			  }
			  ```
		- 相应的，也存在**静态转换操作符**``static_cast``，使用方法和动态转换基本一致，但是其在编译时完成。
			- 和直接类型转换相比，``static_cast``更安全，其会在编译时进行类型检查，若类型之间并不存在转换规则，则会在编译期报错。而直接转换则会略去编译时检查这一步
		- [[$red]]==**注意**==：动态转型的运行时开销可能会比较高，性能敏感程序慎用