- 用于将一个构造函数或copy-initializer函数标记为必须显式调用，避免出现一些反直觉的隐式调用。
- 例如：
	- ```c++
	  class mvector{
	  private:
	    vector<int> v;
	  public:
	    mvector(int size):v(){
	      v.resize(size);
	    }
	  }
	  ```
	- 如果向上述例子一样，构造函数不加``explicit``关键字，那么可能会导致如下调用：
	- ```c++
	  mvector v = 3;
	  ```
	- 此调用实际上会将v的``vector``成员变量resize到3，但是其形式非常地反直觉
	- 为了避免这样的情况可以在构造函数前添加``explicit``关键字，则此情况下写出上述调用会导致编译器报错
- 没有``explicit``关键字的构造函数都是默认可以隐式调用的，这样的构造函数也被称为**[[$red]]==转换函数(Conversion Function)==**
- # 例子
	- 来自cppreference.com
	- ```c++
	  struct A
	  {
	      A(int) { }      // converting constructor
	      A(int, int) { } // converting constructor (C++11)
	      operator bool() const { return true; }
	  };
	   
	  struct B
	  {
	      explicit B(int) { }
	      explicit B(int, int) { }
	      explicit operator bool() const { return true; }
	  };
	   
	  int main()
	  {
	      A a1 = 1;      // OK: copy-initialization selects A::A(int)
	      A a2(2);       // OK: direct-initialization selects A::A(int)
	      A a3 {4, 5};   // OK: direct-list-initialization selects A::A(int, int)
	      A a4 = {4, 5}; // OK: copy-list-initialization selects A::A(int, int)
	      A a5 = (A)1;   // OK: explicit cast performs static_cast
	      if (a1) { }    // OK: A::operator bool()
	      bool na1 = a1; // OK: copy-initialization selects A::operator bool()
	      bool na2 = static_cast<bool>(a1); // OK: static_cast performs direct-initialization
	   
	  //  B b1 = 1;      // error: copy-initialization does not consider B::B(int)
	      B b2(2);       // OK: direct-initialization selects B::B(int)
	      B b3 {4, 5};   // OK: direct-list-initialization selects B::B(int, int)
	  //  B b4 = {4, 5}; // error: copy-list-initialization does not consider B::B(int, int)
	      B b5 = (B)1;   // OK: explicit cast performs static_cast
	      if (b2) { }    // OK: B::operator bool()
	  //  bool nb1 = b2; // error: copy-initialization does not consider B::operator bool()
	      bool nb2 = static_cast<bool>(b2); // OK: static_cast performs direct-initialization
	   
	      [](...){}(a4, a5, na1, na2, b5, nb2); // may suppress "unused variable" warnings
	  }
	  ```