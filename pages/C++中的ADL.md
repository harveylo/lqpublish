- 参看[这篇博客](https://quuxplusone.github.io/blog/2019/04/26/what-is-adl/)
- **Argument-Dependent Lookup**
- 也叫做**Koenig Lookup**
- # C++中的命名空间
	- 在C中是没有命名空间的，两个不同翻译单元不能定义同名函数，否则会链接失败
		- 为了规避这种情况，C程序员可能会约定，某个翻译单元中的所有函数名都具有某个特定的前缀(后缀也行)
	- 在设计C++时，这种在函数名前加前缀避免冲突的做法直接通过命名空间和函数名重整(name mangling)整合到了语言的语义中
		- C++的某个函数如果是定义在某个命名空间中的，那么在函数名重整生成函数签名时，命名空间也会被整合到函数的签名中
- # 命名空间和运算符之间的冲突
	- 假设不存在ADL，那么冲突的关键在于运算符并不能显式地使用命名空间来调用，即不存在``a <namespace>::+ b``的写法
	- ```c++
	  // SakBigNum.h
	  namespace sak {
	      struct bignum {
	          bignum operator++();
	      };
	      std::ostream& operator<<(std::ostream&, bignum);
	  }
	  
	  // AjoBigNum.h
	  namespace ajo {
	      struct bignum {
	          bignum operator++();
	      };
	      std::ostream& operator<<(std::ostream&, bignum);
	  }
	  
	  // AjoExtra.cpp
	  #include "AjoBigNum.h"
	  #include "SakBigNum.h"
	  namespace ajo {
	      void foo(int& x) {
	          bignum b;        // refers to ajo::bignum
	          ++b;             // calls ajo::bignum::operator++()
	          std::cout << b;  // calls ajo::operator<<(ostream&, bignum)
	      }
	  
	      void bar(int& x) {
	          sak::bignum b;   // refers to sak::bignum
	          ++b;             // calls sak::bignum::operator++()
	          std::cout << b;  // UH-OH!
	      }
	  }
	  ```
	- 以上代码的第30行会出现二义性
- # 使用ADL解决冲突
	- 最开始在C++98中被引入，当时被称为**Koenig Lookup**
	- 本来ADL只打算应用在运算符上，但是在96年的C++标准制定过程中，决定将其扩展到函数调用上
	- ## 核心思想
		- 所有没有添加命名空间的运算符和函数调用，在寻找这些符号时，不仅会在当前命名空间下寻找，还会**在实参类型所属的命名空间**下寻找
		- 然然以上一节中的代码片段做例子，第30行在调用被重载的操作符时，不仅会在命名空间``ajo``下寻找被重载的运算符，还会在两个参数的命名空间下，即`std`和`sak`下寻找，最终调用到正确版本的运算符
	- **ADL仅在[[$red]]==函数名没有命名空间限定==的情况下介入**
	- ADL在实参的相关命名空间中只会寻找函数符号，忽略所有非函数符号
- # 不会引起ADL的情况
	- 仅在``unqualified-id``上起作用，因此如果把函数用一个括号括起来，则不会发生，因为此时被括起来的函数是一个``primary-expression``
		- ```C++
		  namespace A {
		      struct A { operator int(); };
		      void f(A);
		  }
		  namespace B {
		      void f(int);
		      void f(double);
		      void test() {
		          A::A a;
		          void (*fp)(int) = f;  // OK, no ADL
		          void (*gp)(A::A) = f; // ERROR, no ADL, fails to find A::f
		          f(a);     // ADL, calls A::f(A)
		          (&f)(a);  // no ADL, calls B::f(int)
		          (f)(a);   // no ADL, calls B::f(int)
		      }
		  }
		  ```
	- 如果在**当前命名空间**下找到了[[$red]]==**非函数的对应符号**==，则不会触发ADL。
		- 仅在找到了函数符号或什么都没找到的情况下会触发ADL
		- ```C++
		  namespace A {
		      struct A { operator int(); };
		      void f(A);
		      void g(A);
		      void h(A);
		      int i(A);
		      int j(A);
		  }
		  namespace B {
		      void f(int);
		      auto h = [](int) {};
		      using i = int;
		      void test() {
		          A::A a;
		          f(a);           // ADL, calls A::f(A)
		          g(a);           // ADL, calls A::g(A)
		          h(a);           // no ADL: lookup found B::h which is not a function
		          int ia = i(a);  // no ADL: lookup found B::i which is not a function
		          int j = j(a);   // no ADL, and ERROR! lookup found local variable j
		      }
		  }
		  ```
	- 仅考虑**函数实参**，**[[$red]]==不会考虑模板实参==**
		- ```C++
		  namespace A {
		      struct A { operator int(); };
		      struct X {};
		  
		      template<class T>
		      void f(int);
		  }
		  namespace B {
		      template<class T>
		      void f();
		  
		      void test() {
		          A::A a;
		          f<A::X>();   // OK, ADL doesn't consider A::f, calls B::f
		          f<A::X>(a);  // OK, ADL considers A::f because of A::A, calls A::f
		          f<A::X>(42); // ERROR: ADL doesn't consider A::f
		      }
		  }
		  ```
-