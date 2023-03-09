- 和``auto``一样用于**编译时**类型推导，主要用于泛型
- 已一个普通表达式作为参数，返回该表达式的类型
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
		-