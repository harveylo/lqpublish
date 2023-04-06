- C++相较于C的一个新特性，分为**引用(一般引用，左值引用)**和**右值引用**
- # 引用语法
	- ## 定义引用
		- 使用类型+`&`定义引用
		- 必须在定义时初始化
		- ``int a; int &b = a;``
	- ## 作为形参
		- 在函数定义时在形参的类型后加上``&``
		- ``void func(int &a, int &b)``
- # 引用的特点
	- 实际上用作某一个变量的别名(类似于Unix系统下的硬链接)，对于引用的值得修改会直接同步到原变量
	- 引用和原变量得值和地址完全相同，对引用求地址将获得原变量的地址
	- 引用本身不是一种数据类型，不会占用储存单元
	- **不能建立数组的引用**，数组是若干元素组成的集合(类似于在Unix下无法建立到某个目录的硬链接)
	- 将引用作为形参可以避免拷贝传参，**提升性能**
- # 引用和指针的区别
	- 指针和引用的区别在一定程度上**类似于软链接和硬链接**的区别
	- 指针会占用储存空间，sizeof一个指针会是该平台下指针的大小，而sizeof一个引用会得到该变量类型的实际大小
	- 指针可以初始化为null，引用必须初始化到一个已有的对象
	- 自增自减运算符行为不一致
	- 实际上，一个引用和一个指针常量(``type* const p)``在**功能上**是一致的
- 一个常量引用表示值不能通过该引用更改
	- ``int a = 10; const int &b = a;``
	- 可以使用a来修改值，但是无法通过b修改，会报错
- # 左值引用和右值引用
	- 上述介绍的引用其实都是**左值引用**，通俗地说，能指向左值，不能指向右值就是左值引用，用**一个**``&``标识
	- ## 左值引用
		- 引用是**变量的别名**，自然无法指向没有地址的右值
		- **但是**，const左值引用**可以指向右值**
		- 因此函数的形参，如果不修改参数的值，尽量使用常量左值引用，使得可以传入右值。例如：
			- std的``vector``的``push_pack``函数，其定义为``void push_back(const value_type& val)``
			- 调用``push_back(5)``可以通过编译
	- ## 右值引用
		- 专为右值而生，可以**指向右值**，但**不能指向左值**
		- 使用``&&``标识
		- 右值引用可以修改值
	- ## 左右值引用的本质
		- ### 右值引用指向左值
			- 使用`std::move`可以使右值引用指向左值
			- ```
			  int a = 5; // a是个左值
			  int &ref_a_left = a; // 左值引用指向左值
			  int &&ref_a_right = std::move(a); // 通过std::move将左值转化为右值，可以被右值引用指向
			   
			  cout << a; // 打印结果：5
			  ```
			- ``std::move``并不如其函数名称一样会做出任何一般意义上的**移动**操作
			- 其唯一的作用是**把左值强制转化为右值**，让右值引用可以指向左值
			- 实现等同于一个**类型转换**，因此单纯的``std::move``[[$red]]==不会有性能提升==
			- [[$red]]==**事实上**==，右值引用指向右值就是通过把右值提升为一个左值，然后``std::move``将该左值再次和该右值引用绑定
				- ```
				  int &&ref_a = 5;
				  ref_a = 6; 
				   
				  //等同于以下代码：
				   
				  int temp = 5;
				  int &&ref_a = std::move(temp);
				  ref_a = 6;
				  ```
		- ### 左值引用，右值引用本身是左值还是右值？
			- **被声明出来的左右值引用都是[[$red]]==左值==**
			- 显然，左右值引用都可以用在等式左侧，且其作为一个符号在内存内自然有地址，因此是左值
			- ```
			  // 形参是个右值引用
			  void change(int&& right_value) {
			      right_value = 8;
			  }
			   
			  int main() {
			      int a = 5; // a是个左值
			      int &ref_a_left = a; // ref_a_left是个左值引用
			      int &&ref_a_right = std::move(a); // ref_a_right是个右值引用
			   
			      change(a); // 编译不过，a是左值，change参数要求右值
			      change(ref_a_left); // 编译不过，左值引用ref_a_left本身也是个左值
			      change(ref_a_right); // 编译不过，右值引用ref_a_right本身也是个左值
			       
			      change(std::move(a)); // 编译通过
			      change(std::move(ref_a_right)); // 编译通过
			      change(std::move(ref_a_left)); // 编译通过
			   
			      change(5); // 当然可以直接接右值，编译通过
			       
			      cout << &a << ' ';
			      cout << &ref_a_left << ' ';
			      cout << &ref_a_right;
			      // 打印这三个左值的地址，都是一样的
			  }
			  ```
			- 可以发现，``std::move``返回的``int&&``类型充当了右值，然而**声明的int&&**却毫无疑问地是左值
			- [[$red]]==**实际上**==，右值引用**既可以**是左值，**也可以**是右值：
				- 具名地右值引用是左值
				- 匿名地右值引用是右值
			- 通俗地说：作为表达式**返回值**的右值引用是右值，**直接声明**出来的右值引用是左值
		- ### 关于左右值本质的简单总结
			- **从性能上讲**，左右值引用没有区别，传参使用左右值都可以**避免拷贝**
			- 右值引用可以直接指向右值，也可以通过``std::move``指向左值；而左值引用除非使用使用``const``修饰，否则无法指向右值
			- 作为函数参数时，右值引用**更加灵活**，``const``左值引用虽然可以做到左右值均可接受，但无法修改
- # 右值引用和``std::move``的应用场景
	- ``std::move``本身无法提高效率，但是在某些场景下配合使用有奇效
	- ## 实现移动语义
		- 右值引用和std::move被广泛用于在STL和自定义类中**实现移动语义，避免拷贝，从而提升程序性能**
		- **举例：**
			- 一个类一般要有**构造函数**，**拷贝构造函数**，**赋值运算符重载**，**析构函数**等
			- ```
			  class Array {
			  public:
			      Array(int size) : size_(size) {
			          data = new int[size_];
			      }
			       
			      // 深拷贝构造
			      Array(const Array& temp_array) {
			          size_ = temp_array.size_;
			          data_ = new int[size_];
			          for (int i = 0; i < size_; i ++) {
			              data_[i] = temp_array.data_[i];
			          }
			      }
			       
			      // 深拷贝赋值
			      Array& operator=(const Array& temp_array) {
			          delete[] data_;
			   
			          size_ = temp_array.size_;
			          data_ = new int[size_];
			          for (int i = 0; i < size_; i ++) {
			              data_[i] = temp_array.data_[i];
			          }
			      }
			   
			      ~Array() {
			          delete[] data_;
			      }
			   
			  public:
			      int *data_;
			      int size_;
			  };
			  ```
			- 可以看到虽然拷贝赋值和深拷贝构造函数已经使用左值引用**避免了一次拷贝**，但是其深拷贝的内部逻辑是无法避免的。
			- 若提供一个**移动构造函数**，在被拷贝者后续**不再被使用**的情况下(因为移动构造函数为了避免被拷贝者在拷贝着被析构之前析构导致拷贝着的成员不可用，因此会提前空置被拷贝者的成员，导致被拷贝者的成员不可用)，直接将被拷贝者的资源**移动过来**，以此来避免深拷贝逻辑
			- ```
			  class Array {
			  public:
			      Array(int size) : size_(size) {
			          data = new int[size_];
			      }
			       
			      // 深拷贝构造
			      Array(const Array& temp_array) {
			          ...
			      }
			       
			      // 深拷贝赋值
			      Array& operator=(const Array& temp_array) {
			          ...
			      }
			   
			      // 移动构造函数，可以浅拷贝
			      Array(const Array& temp_array, bool move) {
			          data_ = temp_array.data_;
			          size_ = temp_array.size_;
			          // 为防止temp_array析构时delete data，提前置空其data_      
			          temp_array.data_ = nullptr;
			      }
			       
			   
			      ~Array() {
			          delete [] data_;
			      }
			   
			  public:
			      int *data_;
			      int size_;
			  };
			  ```
			- 如此实现的两个问题：
				- 表示移动语义需要一个额外的参数来和非移动构造区分开，代码臃肿不美观
				- [[$red]]==**根本无法实现**==，传入的左值引用根本无法被修改，因此置空操作无法实现；若去掉`const`则无法传入右值，导致功能缺失
			- 右值引用的出现解决了这些问题，许多实际的STL容器都实现了以右值引用问参数的**移动构造函数**和**移动通过赋值重载函数**，最常见的如``stl::vector``的``push_back``，参数为左值代表拷贝，右值代表拷贝
			- ```
			  class Array {
			  public:
			      ......
			   
			      // 优雅
			      Array(Array&& temp_array) {
			          data_ = temp_array.data_;
			          size_ = temp_array.size_;
			          // 为防止temp_array析构时delete data，提前置空其data_      
			          temp_array.data_ = nullptr;
			      }
			       
			   
			  public:
			      int *data_;
			      int size_;
			  };
			  ```
			- 使用时直接结合``std::move``便可以完成移动构造：``Array a; Array b(std::move(a));``
		- ### 一个实例：使用``std::move``提升``vector::push_back``性能
			- ```
			  int main() {
			      std::string str1 = "aacasxs";
			      std::vector<std::string> vec;
			       
			      vec.push_back(str1); // 传统方法，copy
			      vec.push_back(std::move(str1)); // 调用移动语义的push_back方法，避免拷贝，str1会失去原有值，变成空字符串
			      vec.emplace_back(std::move(str1)); // emplace_back效果相同，str1会失去原有值
			      vec.emplace_back("axcsddcas"); // 当然可以直接接右值
			  }
			   
			  // std::vector方法定义
			  void push_back (const value_type& val);
			  void push_back (value_type&& val);
			   
			  void emplace_back (Args&&... args);
			  ```
		- ### 类的可移动性
			- 大多数STL类都支持移动语义函数，即**可移动的**
			- 编译器([[$red]]==？==谁的编译器)**默认**会在**没有主动定义拷贝构造函数的情况下**为用户自定义的class和struct添加移动语义函数
			- **[[$red]]==因此建议：==**若**可移动的**对象在拷贝后不再被需要，则建议使用``std::move``提升性能
			- 某些STL类则是**仅移动的**(move-only)，如``unique_ptr``。这种类只有移动构造函数，**只能进行移动**(转义内部对象所有权，也称浅拷贝)，不能深拷贝
			- ```
			  std::unique_ptr<A> ptr_a = std::make_unique<A>();
			  
			  std::unique_ptr<A> ptr_b = std::move(ptr_a); // unique_ptr只有‘移动赋值重载函数‘，参数是&& ，只能接右值，因此必须用std::move转换类型
			  
			  std::unique_ptr<A> ptr_b = ptr_a; // 编译不通过
			  ```
		- **[[$red]]==小结==**：``std::move``本身制作类型转换，对性能无影响。在自己定义的类中实现移动语义，配合``std::move``可避免深拷贝，提升性能
- # 完美转发``std::forward``
	- 类似于move，但是更强大，**既可以**转换为右值**又能**转换为左值
	- std::forward<T>(u)有两个参数：
		- 当T为左值引用类型时，u将被转换为T类型的左值；
		- 否则u将被转换为T类型右值。
	- ## 两个例子
		- ```
		  void B(int&& ref_r) {
		      ref_r = 1;
		  }
		   
		  // A、B的入参是右值引用
		  // 有名字的右值引用是左值，因此ref_r是左值
		  void A(int&& ref_r) {
		      B(ref_r);  // 错误，B的入参是右值引用，需要接右值，ref_r是左值，编译失败
		       
		      B(std::move(ref_r)); // ok，std::move把左值转为右值，编译通过
		      B(std::forward<int>(ref_r));  // ok，std::forward的T是int类型，属于条件b，因此会把ref_r转为右值
		  }
		   
		  int main() {
		      int a = 5;
		      A(std::move(a));
		  }
		  ```
		- ```
		  void change2(int&& ref_r) {
		      ref_r = 1;
		  }
		   
		  void change3(int& ref_l) {
		      ref_l = 1;
		  }
		   
		  // change的入参是右值引用
		  // 有名字的右值引用是 左值，因此ref_r是左值
		  void change(int&& ref_r) {
		      change2(ref_r);  // 错误，change2的入参是右值引用，需要接右值，ref_r是左值，编译失败
		       
		      change2(std::move(ref_r)); // ok，std::move把左值转为右值，编译通过
		      change2(std::forward<int &&>(ref_r));  // ok，std::forward的T是右值引用类型(int &&)，符合条件b，因此u(ref_r)会被转换为右值，编译通过
		       
		      change3(ref_r); // ok，change3的入参是左值引用，需要接左值，ref_r是左值，编译通过
		      change3(std::forward<int &>(ref_r)); // ok，std::forward的T是左值引用类型(int &)，符合条件a，因此u(ref_r)会被转换为左值，编译通过
		      // 可见，forward可以把值转换为左值或者右值
		  }
		   
		  int main() {
		      int a = 5;
		      change(std::move(a));
		  }
		  ```
		- 以上两个例子只是单纯为了展示，实际编程中不会有人这么无聊
		- ``std::forward``实际因公用主要运用在**模板编程的参数转发**中
		- DONE 学习万能引用``T &&``
		- DONE 学习引用折叠 ``& && -> ?``