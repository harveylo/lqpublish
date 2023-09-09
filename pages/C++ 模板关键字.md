# ``typename``
	- 用于定义模板形参中的类型形参(type template parameters)
		- ``template<typename T>``
	- 在模板定义的内部，用于声明一个**依赖限定名**(dependent qualified name)是一个类型
		- ``typename vector<T>::iterator *a``
		- ``vector<T>::iterator``依赖于``T``，因此是一个依赖名
		- 使用``typename``明确告知编译器，这个依赖名是一个类型，避免编译器将其理解为其他操作(乘法)
	- 也能用在非依赖名前，但是没有任何效果
	- 在C++20中引入的[requirements](https://en.cppreference.com/w/cpp/language/constraints)特性中指定类型要求