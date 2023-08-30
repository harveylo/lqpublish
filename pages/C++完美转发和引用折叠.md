- 引用折叠和完美转发随着万能引用出现和具有实际作用
- 给定一个万能引用函数模板，根据传入的值种类，编译器会推导出不同的类型
	- 假设一个函数模板为：
		- ```c++
		  template<typename T>
		  void func(T&& a);
		  ```
	- 现假设传入一个``int``类型的值，根据这个值的类型，有
		- 传入左值，`T`被推导为``int &``
		- 传入左值引用，`T`被推导为``int &``
		- 传入右值，`T`会被推导为`int`
- 而模板中的形参形式为``T&&``，那么和推导出的类型结合，就会的到如下结论：
	- int& & = 左值引用
	- int& && = 左值引用
	- int&& & = 左值引用
	- int&& && = 右值引用
	- **概括来说：**只有两个右值引用结合在一起才是右值引用，否者两个任何引用结合在一起都是左值引用
- # 完美转发
	- 需要**和万能模板配合**使用
	- [[$red]]==**重复强调：**==**任何变量**，不管其值类型是左值引用还是右值引用，其本身都是左值
	- 因此如果想针对左值引用和右值引用分开做处理，直接传递值无法得到预期结果：
		- ```
		  template<typename T>
		  void fun(T& x){
		  	cout<<"&: "<<x<<endl;
		  }
		  
		  template<typename T>
		  void fun(T&& x){
		  	cout<<"&&: "<<x<<endl;
		  }
		  
		  template<typename T>
		  void printT(T&& x){
		  	fun(x);
		  }
		  ```
		- 调用``printT(a);``和``printT(2)``都将被转发到左值引用的``fun(T&)``(a是一个左值变量)
			- 因为对于``printT``来说，它的形参``x``在是一个变量，直接作为实参传递给``fun``肯定是会调用左值引用的``fun``
	- 如果想保留形参``x``的值类型信息，需要使用``std::forward``函数
		- ```
		  template<typename T>
		  void fun(T& x){
		  	cout<<"&: "<<x<<endl;
		  }
		  
		  template<typename T>
		  void fun(T&& x){
		  	cout<<"&&: "<<x<<endl;
		  }
		  
		  template<typename T>
		  void printT(T&& x){
		  	fun(std::forward<T&&>(x));
		  }
		  ```
		- 如此一来调用``printT(a);``和``printT(2)``都将被转发到正确的``fun``函数
	- ## ``std::forward``的原理
		- ``std::forward``的原理也仰赖于引用折叠
		- 其源码类似于：
			- ```
			  template<typename T>
			  T&& forward(T &param)
			  {
			  	return static_cast<T&&>(param);
			  }
			  ```
			- [[$red]]==**注意：**==此代码片段只是为了方便理解，细节是完全错误的，根本无法正常工作，但是其实现确实使用了``static_cast``
			- **当传入右值时**：`T`推导为非引用值，即T，再结合``T&&``返回，引用折叠便得到一个右值
			- **当传入左值时**：``T``推导为左值引用，即T&，再结合``T&&``返回，引用折叠便得到一个左值
		- 因此，可以预料到以下两种操作的结果：
			- 如果forward语句在实际使用时写作：**``std::forward<T&>(arg);``**，则不管是左值还是右值都将变为左值
				- [[$red]]==个人理解==：其实已经发生了一次引用折叠，因为T本身已经是一个左值引用，因此static_cast语句的实际就变成了``static_cast<T& &&>`` = ``static_cast<T&>``，如此一来，传入不论左值还是右值，基于引用折叠的规则，最终都只会得到左值
			- 如果forward语句在实际使用时写作：**``std::forward<T&&>(arg)``**，则最终转发结果和``std::forward<T>(arg)``一致
				- [[$red]]==个人理解==：原理同上，不过第一次发生的引用折叠根据饮用者折叠规则得到的实际语句是``static_cast<T&&>``，因此结果保持正确