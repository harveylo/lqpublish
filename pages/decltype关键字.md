- 和``auto``一样用于**编译时**类型推导，主要用于泛型
- 以**一个普通表达式**作为参数，返回该表达式的**类型和值类别**(左值或右值)
- 属于**Unevaluated Context**(下文会简单解释)
- # 规则
	- 如果 expr 是一个 **id-expression**，那么decltype 不会推导表达式的值类别。即：
		- 如果expr 中只包含一个变量名，此时 decltype 会返回这个变量的类型
		- 如果expr 只包含一个访问类数据成员的表达式，此时 decltype 会返回这个数据成员的类型
	- 否则，decltype 会同时推导 expr 的类型（假设为 T）和值类别。具体地：
		- 如果 expr 是一个 lvalue，那么 decltype 将返回 T &
		- 如果 expr 是一个 xvalue，那么 decltype 将返回 T &&
		- 如果 expr 是一个 prvalue，那么 decltype 将返回 T。
	- **例：**
		- ```
		  int a;
		  
		  struct Foo {
		    double b;
		  };
		  Foo foo;
		  
		  std::string s;
		  
		  decltype(a);     // id-expr, int
		  decltype(foo.b); // id-expr, double
		  decltype((a));   // lvalue, int &, note the parenthesis
		  decltype(s[0]);  // lvalue, char &
		  decltype(std::move(s));  // xvalue, std::string &&
		  decltype(a + foo.b);     // prvalue, double
		  ```
	- expr并不会在运行时被估值(evaluate)，只是在**编译时**对其进行类型推导
		- 所以``decltype``属于**unevaluated** context
- # 应用
	- ## 推导表达式类型
	- ## 与using/typedef何用定义类型
		- ```
		  vector<int >vec;
		  typedef decltype(vec.begin()) vectype;
		  for (vectype i = vec.begin; i != vec.end(); i++)
		  {
		  	//...
		  }
		  ```
		- 可提高代码可读性
	- ## 重用匿名类型
		- ```
		  struct 
		  {
		      int d ;
		      doubel b;
		  }anon_s;
		  
		  decltype(anon_s) as;
		  ```
	- ## 泛型中结合auto，用于追踪函数的返回值类型
		- 最常用
		- ```
		  template <typename _Tx, typename _Ty>
		  auto multiply(_Tx x, _Ty y)->decltype(_Tx*_Ty)
		  {
		      return x*y;
		  }
		  ```
		- ### 为什么会有这种写法？
			- C++11的函数声明有两种写法
				- ``return-type identifier (paramters...)``
				- ``auto identifier (parameters...) -> return-type``
			- C++11中的``auto``推理能力实际上并不强，因此往往还需要显式地给出
			- ```
			  template <typename L, typename R>
			  auto add(L lhs, R rhs) {
			    return lhs + rhs;
			  }
			  ```
			- 上述代码就无法编译通过，因为不知道lhs+rhs地类型
			- 如果直接在返回类型处使用``decltype``，则会因为参数出现在后面而不知到参数符号的意义而出现编译错误
				- ```
				  template <typename T1, typename T2>
				  decltype(a + b) compose(T1 a, T2 b); //错误：不知道decltype中的a和b为何物
				  ```
			- 当然也可以借助``std::declval``，但是其代码风格就有点臃肿了
				- ```
				  template <typename T1, typename T2>
				  decltype(std::declval<T1>() + std::declval<T2>())
				  compose(T1 a, T2 b);
				  ```
			- 因此C++11引入了第二种声明方式，就是为了解决使用`decltype`时参数还未给出的问题
				- ```
				  template <typename T1, typename T2>
				  auto compose(T1 a, T2 b) -> decltype(a + b);
				  ```
			- [[$red]]==**注意：**==在C++14之后，只要函数是``well-defined``的，使用auto作为返回类型即可自动推导类型，但显式给出代码可读性更强