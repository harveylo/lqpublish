title:: Porting the Unity Engine to .NET CoreCLR

- # Unity和.NET平台
	- Unity目前(2018年)大致支持3个.NET平台
		- **Mono**，是Editor和独立(Standalone)Player在Windows，Mac和Linux下的运行时。Unity在早期就已经在这些平台上使用Mono作为运行时
		- **IL2CPP**， 通过将IL翻译为C++，本来是开发来支持IOS的，目前正在快速扩展到其他平台上
		- **WinRT**，为了支持一些更受限(Restricted)的Windows Phone/Windows Store App平台。这个平台没什么好说的，因为在其上开发和维护实在是过于痛苦
	- 由于各种原因，构建一个Unity平台的工作量相当大，尤其是还要考虑操作系统，.NET平台和图形API这三者的排列组合
	- 多年来，Mono为Unity带来了.NET相关的核心体验，且因为某些原因还会持续一段时间：
		- 在过去，Mono是Unity唯一能够获得许可的**OSS(开源软件)**运行时
		- Mono运行时移植到其他平台较为便捷
			- 其主要是通过简单的C设计来实现的
		- Unity使用的是较旧版本的Mono，包括Boehm GC，这给Unity运行时开发者施加了较少的限制
	- ## Unity目前(2018年)是如何运行.NET代码的？
		- 引擎是建立在一个简化的**实体-组件系统(Entity-Component System)**上的，此系统更偏向于组合(Composition)和不是继承(Inheritance)
			- 一个**实体(Entity，在Unity中即``GameObject``)**并不**直接拥有**任何数据，且**不可被继承**，但是可以能够通过附加**组件(Component，在Unity中即``Component``)**来扩展
			- 有一些由引擎提供的组件直接通过内部``GameObject``系统来处理
				- 但是仍然存在引擎会直接调用的终端用户(End-user)组件，例如**``MonoBehavior``Unity脚本**
		- 引擎大部分都是由C++写成的，C\#是为终端用户提供可编程体验的
			- 当引擎需要处理C\# ``GameObject``和``Component``时，引擎是从C++中去访问C\#对象，且能够访问C\#的所有字段
			- 从C++到C\#的调用非常消耗资源，因为涉及到ABI(调用约定)的转换
		- 因此和常规的.NET程序不同，Unity的程序会在启动时加载C++运行时，但是**马上就会跳转进**C\#的``Program.Main()``方法
			- 且这样的跳转在之后的运行中还会被执行多次
	- ## IL2CPP AOT 运行时
		- 如果将JIT抽掉，那么C++到C\#的互相调用将会更加简单
		- 不过由于涉及到GC，这样的跳转还是会有成本。但是到目前为止IL2CPP依赖于Boehm GC，因此问题不大
	- ## 从C++到C\#
		- 从C++迁移到C\#能够提高编写游戏代码时的生产力
		- 将引擎部分的代码迁移到C\#能提升性能(可能是因为减少了C++和C\#的互相调用)
		- Burst技术更是提高了C\#代码的运行效率
- # 为什么要在Unity中使用CoreCLR
	- 三大重要因素：
		- **性能**
		- **社区**
		- **Convergence/Evolution**
	- ## CoreCLR的性能表现
		- 在相同的工作负载下，CoreCLR往往能比Mono快**2到5倍**，在某些情况下甚至能达到**10倍**
		- 对于IL2CPP来说，其已经提供了很明显的性能提升，但仍然比CoreCLR慢2倍左右
			- 一个原因就是IL2CPP团队的主要精力都花在了多平台扩展和提供新的C\#编译器和语言特性上，导致他们没有足够的时间去仔细优化代码生成和运行时
		- ### GC感知代码生成(GC Aware Codegen)
			- 对生成代码的一大影响因素就是：**所使用的GC和代码生成结合紧密程度**
			- 尽管Boehm GC宣称其具有增量/分带垃圾回收的能力，在测试中**其性能总是落后于.NET GC**
			- 通过维护一个分代管理对象可以快速地分配新的内存
				- 例如，新分配的内存划在gen0中，gen0的内存分配只是简单的加减指针，速度较快
				- 当gen0满时就需要将一些对象移动到更老的代中(gen1, gen2 ...)
					- 这一步通过write barrier完成，这是在每一次将gen0对象的引用存储到了一个较旧代对象中时都会执行的一小段代码
			- 这些对象在被移动到老代中时GC需要将这些对象移动到新的内存地址中，并更新所有在栈上指向这些对象的引用(这些引用称为**栈根(stack root)**）
				- 这就是为何没有GC感知的C++代码生成器效率不佳的原因，其不具备这些栈根的相关信息，因此更新将非常耗时
				- 而GC则一直在跟踪这些引用，当需要进行GC时可以快速定位栈根(甚至包括寄存器)并更新
			- **总结**
				- 一个非GC感知的代码生成器一般只能被迫选择保守的GC策略
				- 一个保守的GC会经常打破**分代收集(Generational Collection)**的承诺：
					- 由于不安全，其**无法将在栈上的g0对象移动到更老的代**
					- 会保留那些本来能够被收集的对象
					- 由于过多的对象被保留，gen0的空间会不断增长
					- 以上将导致**越来越多的缓存交换和缓存miss**
			- 这就是为什么一个有GC感知的代码生成器(例如CoreCLR，RyuJIT)能够更高效地进行GC并最终跑得更快，使用内存更高效
		- ### 代码生成的突破
			- 有很多顶级人才都加入到了对CoreCLR的优化中，例如：
				- 与``Span<T>``和``ReadOnlySpan<T>``相关的优化
				- 开始引入**分层JIT编译(Tiered JIT Compilation)**
		- ### Debug体验
			- 让一个C\# debugger连接到Mono运行时进行debug会导致运行速度的进一步下降，可能会比CoreCLR**慢10到50倍**
	- ## 社区
		- .NET Core超过60%的贡献来自社区
	- ## Convergence/Future
		- CoreCLR还具很多巨大的潜力，例如正在进行中的.NET AOT项目**CoreRT**
- # 如何将CoreCLR集成到Unity中？
	- 基本思想很简单：**使用CoreCLR完全实现和Mono相同的C API**
	- ## 整体计划
		- 打算只build一个standalone的player，且此player只需要播放包含一个旋转的方块的scene
		- 这将极大减少需要重新实现的Mono API，工作大致包括：
			- 使用Mono运行时的Hosting API编写一个简单的可加载一个类并调用其中静态方法的程序
				- 作为一个简单的Mono Hosting的HelloWorld程序
			- 同时，在CoreCLR上开发这个简单程序会使用到的相关API
			- 一旦我们获得了在CoreCLR下可以跑起来的东西，我们就可以继续开发确实的API
			- 使用**netcoreapp2.0**重新编译UnityEngine托管程序集
	- ## 缩小范围
		- 为了减少需要实现的函数，开发了一个很小的mono代理(Proxy)dll，让其重定向到真正的Mono运行时
			- 此代理dll会把所有调用的函数log到一个文件中，以查看到底调用了哪些函数
		- 最终的API实现情况如下：
			- ![image.png](../assets/image_1689043581959_0.png){:height 462, :width 645}
	- ## 将CoreCLR作为Mono API暴露出来