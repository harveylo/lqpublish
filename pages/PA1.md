- ## [[补充：编程模型]]
- ## [[GNU Unified Format]]
- **调用多线程编译** 在make后使用-j，可以指定使用多少个线程来进行编译，如``-j2``
	- 使用``lscpu``查看有几核+
- **使用ccache降低编译时间**
	- ccache可以把每一次编译产生的文件储存起来，当下一次进行编译，而文件没有产生变化时，能极大提升编译性能
	- 在环境变量的``PATH``中增添``/usr/lib/ccache``
- [RISC-V ELF psABI Document](https://github.com/riscv-non-isa/riscv-elf-psabi-doc)
- x86中，程序计数器叫EIP(extended instruction pointer)
- 由于一台电脑可以看作一个状态机，而一个程序加载进内存以后就是确定了一个唯一的状态，在该种太后的迁移方式也是唯一确定的
- 某种程度上来说，程序也是个状态机
- 匿名union可以直接用union中定义的名字去解释内存，非匿名需要先使用union的名字再指定解读方式
- gcc可以编译cpp文件，如果使用到了c++的库，需要加上``-lstdc++``
- ``__attribute__``是用来给编译器提示的，类似于java的注解
- # api
	- 加载用户程序在init_isa中完成
	- 初始化寄存器的工作在nit_rsa中的restart完成
- # 调试工具与原理
	- **Fault**：实现错误的代码，如``if(p=NULL)``
	- **Error**：程序执行时不符合预期的状态，例如，p本不应该在某一个时间节点为NULL，但是却被赋值为NULL
	- **Failure：**：能直接观测到的错误，例如程序发生了段错误
	- **调试**就是从观测到failure开始，逐步回溯寻找到fault的过程
	- 调试的[[$red]]==难点==一般在与：
		- fault不一定马上触发error
		- 触发的error也不一定马上转变为可观测的failure
		- error为逐渐积累，逐渐改变程序行为，当观测到failure时，可能已经离fault十分遥远了
	- **测试**就是为了尽可能把fault转变成error，但是效果受到测试覆盖率的影响
	- **尽早观测到error的存在**
		- 使用``-Wall, Werror``，在编译时寻找未定义行为
		- ``assert``在运行时把error 直接转变为failure，在针对数组访问越界和空指针引用时十分有用
			- **sanitizer**
				- 是一种底层的assert
				- 编译时自动在指针和数组访问之前插入用来检查是否越界的代码
				- 在gcc中使用``-fsanitize=address``开启
				- 在man中查阅关于``fsanitize``选项的更多内容
		- ``printf``通过在运行时输出一些debug信息来帮助获取error的相关信息
		- 使用调试工具，例如``gdb``
	- 一些[[$blue]]==调试建议==：
		- 总是使用 `-Wall` 和 `-Werror`
		- 尽可能多地在代码中插入 `assert()`
		- 调试时先启用sanitizer
		- `assert()` 无法捕捉到error时, 通过 `printf()` 输出可疑的变量, 期望能观测到error
		- `printf()` 不易观测error时, 通过GDB理解程序的精确行为
- # 断点
	- 通过观测点可以模拟断点，例如：``w $pc == ADDR``
	- TODO 阅读文章[How debuggers work: Part 2 - Breakpoints](https://eli.thegreenplace.net/2011/01/27/how-debuggers-work-part-2-breakpoints)
- # 阅读手册
	-