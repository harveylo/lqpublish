- 一般用于配合 [[decltype关键字]] 完成类型推理
- 会返回一个用于**Unevaluated Context**的实例
- **举例**
	- ```
	  template <typename Iter>
	  class MyIteratorTraits {
	  public:
	    using ValueType = typename std::remove_reference<decltype(*Iter{})>::type;
	  };
	  ```
	- 上述代码希望``ValueType``是传入的``Iter``的解引用类型，但是问题在于，如果``Iter``没有无参构造函数，则此代码无法执行，**通用性不强**
	- +所以应该使用``std:declval``来构造一个类型实例
	- ```
	  template <typename Iter>
	  class MyIteratorTraits {
	  public:
	    using ValueType = typename std::remove_reference<decltype(*std::declval<Iter>())>::type;
	  };
	  ```
- [[$red]]==**注意：**==``std::declval``**没有函数实现**，只能用于unevaluated context等非odr-use场合
	- **odr-use**：指One definition Rule
		- 一个函数被ODR-use指此函数被调用或取地址
		- 一个变量被ODR-use指此变量被读或写或取地址
		- ODR-use涉及到具体内存(运行时)，否则就只涉及类型(编译时)
		- 因此一个被odr-use的东西必须有具体定义