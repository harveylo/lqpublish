- 全称为**替换失败并非错误**(Substitution Failure is not an Error)
- 首先这里的Failure和Error是针对**编译时**的
	- 编译器如果遇到Error会直接终止编译
	- Failure则是某种不成功的尝试
	- ```c++
	  struct A {};
	  struct B: public A {};
	  struct C {};
	  
	  void foo(A const&) {}
	  void foo(B const&) {}
	  
	  void callFoo() {
	    foo( A() );
	    foo( B() );
	    foo( C() );
	  }
	  ```
	- 如在上面的代码中 ，foo(A()) 匹配 foo(B const&) 会Failure，而foo( C() ) 不能匹配任何函数，所以会出现error
	- 因此在很多情况下，