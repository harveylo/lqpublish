- # 本地和静态变量
	- 目前kernel中有两中变量
		- 本地变量储存在call stack上，随着函数返回而失效
		- 静态变量存储在内存中的固定位置，且其生命周期贯穿整个程序执行过程
			- 静态变量的**地址在编译时便已确定**
			- rust中的静态变量为了避免**数据竞争**，所有的静态变量都是**只读**的，除非使用``Mutex``包装
- # 动态内存(Dynamic Memory)
	- 静态和本地变量都有其局限性
		- 本地变量生命周期太短
		- 静态变量生命周期又过长，过多的静态变量会严重限制内存利用
		- 两者都是固定大小
	- **堆**就是大多数编程语言用于绕过上述两种变量缺点的方法
	- ## 使用动态内存的常见错误
		- 当程序员忘记释放申请的动态内存时就会产生内存泄漏，这往往是内存消耗过大的原因
		- 然而还有两种更为致命的常见错误
			- **释放后使用(use-after-free)**，访问已经被释放的内存地址，往往会导致未定义行为且常被攻击者利用
			- **双重释放(double-free)**，对已经释放的内存地址再次进行释放操作，可能导致释放后使用的问题
		- rust使用**所有权**(ownership)概念来避免这些问题，程序员无需再手动管理动态内存
			- 但是也提供了手动管理内存的``allocate``和``deallocate`` api
	- ## Rust中的内存分配(Allocation)
		- Rust std提供一个类``Box``，其``new``函数隐式调用``allocate``，为其分配足够使用的空间，并且实现了``drop`` trait，在走出有效域(scope)时隐式调用``dealocate``释放空间
			- 也就是**RAII(Resource acquisition is initialization)**
	- ## 内核中的动态内存分配
		- 由于动态内存分配必然伴随overhead，例如寻找可用的heap空间，因此对于性能敏感的内核代码，使用本地变量必然是更优解
		- 但是动态内存分配能带来无法替代特性，其最突出的就是动态生命周期和可变大小
			- 前者由**引用计数器**带来
			- 后者让我们可以很方便地使用一些**collection types**，例如``Vec``, ``String``等，这些类都可以自由地增长和减少大小
		- 因此在内核中使用动态内存分配是必不可少的
	- ## Allocator Interface
		- 实现堆分配器的第一步是增添对内置``alloc`` crate的依赖
		- 由于我们是自定义目标，还需要将此crate添加入重新编译列表
		- Rust需要一个实现了``GlobalAloc`` trait的静态变量来提供``alloc``和``dealloc``函数
			- ```rust
			  pub unsafe trait GlobalAlloc {
			      unsafe fn alloc(&self, layout: Layout) -> *mut u8;
			      unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);
			  
			      unsafe fn alloc_zeroed(&self, layout: Layout) -> *mut u8 { ... }
			      unsafe fn realloc(
			          &self,
			          ptr: *mut u8,
			          layout: Layout,
			          new_size: usize
			      ) -> *mut u8 { ... }
			  }
			  ```
	- ## 创建内核堆
		- 在创造一个合适的分配器之前，我们首先需要创建一个可供分配器进行内存分配的内存区域，也即一块虚拟内存空间
		-