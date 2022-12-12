- 补充： [[menuconfig简介]]
- 补充： [[地址无关代码简介]]
- 补充： [[c语言的缓冲区]]
- 补充： [[C语言为什么要关闭文件指针]]
- 补充： [[左值和右值]]
- 补充： [[gcc编译过程简介]]
- 补充： [[ELF文件格式和链接过程简介]]
- gcc支持代码块有返回值，返回值位最后的计算操作
	- 不能只有声明语句
- # 运行时环境
	- 运行时环境很多时候和硬件架构密切相关
	- 为了解耦，降低维护成本，可以抽象出程正确执行所需要的API，硬件层面提供对这些api的支持
		- 这些API可以被封装为库函数
	- ## Abstract machine
		- 一组统一抽象的API，代表了程序运行对于计算机的需求，因此被称为抽象计算机
		- AM是一个向程序提供运行时环境的库
		- AM=TRM+IOE+CTE+VME+MPE
			- TRM：图灵机，最简单的运行时环境，提供基本的计算能力
			- IOE：IO Extension，输入输出扩展，为程序提供输入输出的能力
			- CTE：Context Extension，上下文扩展，提供上下文管理的能力
			- MPE：Multi-processor Extension，多处理器扩展，为程序提供多处理器通信的能力
		- 因此NEMU的开发流程：
			- (在NEMU中)实现硬件功能 -> (在AM中)提供运行时环境 -> (在APP层)运行程序
- # Trace
	- ## 环形buf
		- ### 实现思路
			- 修改nemu对于参数的处理，根据输入的参数确认是否开启环形buff(可选)
			- 增加初始化环形buf的函数(可选)
			- 增加环形buf对象
			- 每次traceanddifftest的时候，将当前指令的内容输入进buff
		- ### 实际实现
			- 没有设置额外的参数，ringbuf的内容直接输出到nemu-log.txt文件的最后位置
			- 由于isa的取值和执行是一并执行的，所以ringbuf没法取到最新的inst的值，记录停留在出错前的一条指令上，解决办法：
				- 将取值和执行分开，增加一个prefetch函数，先将指令的值取到Decode结构体中，完成记录之后再执行
			-
- # 补充
	- [[C内联汇编]]
	- [[库函数stdarg]]
	- ## C和C++的混编
		- ``extern "C"``，用于让C能够连接C++的函数
			- C++中有函数名重载，因此C++编译器不能将函数名作为唯一的ID进行连接
				- 因此C++编译器会给每一个函数附加诸如参数类型的额外信息
			- 而C不能进行函数名重载，因此没有这方面的考虑
			- 在C++中加上``extern "C"``的关键字之后，C++编译器将不会再给该函数附加额外信息
			- 可以单独使用，也可以作为代码块使用
				- ```
				  extern "C" void foo(int);
				  extern "C"
				  {
				     void g(char);
				     int i;
				  }
				  ```
	- ## 定义是如何从Kconfig最终到c文件中的？
		- Make会调用生成menuconfig的命令
		- 该命令从Kconfig文件生成图形化菜单
		- 在菜单中配置之后，生成.config文件
		- 通过.config文件中的东西生成autoconf.h头文件，所有定义都在其中
			- 应该是在执行mconf或conf是完成的头文件生成
			- 具体生成函数在confdata.c文件中
	- ## ELF文件中的符号表
		- 位于文件的``.symtab``段
	- ## 什么时候getopt-long会返回1？
		- 在指定optstring的时候，如果在开头加一个``-``，则如果处理到一个不匹配的字符串，则返回值则会是1
		- 似乎只会对遇到的第一个不匹配参数有用
- # 部分问题回答
	- ## 错误码为什么是1？make程序如何得到这个错误码
		- 测试用例调用的check函数在测试不通过时会调用``halt(1)``
		- 该函数的具体实现为调用相关平台的``nemu_trap(code)``宏，且调用时会把1作为参数传入
		- ``nemu_trap(code)``在riscv平台下的具体定义为：
			- ``asm volatile("mv a0, %0; ebreak" : :"r"(code)``
			- 为内联汇编代码
		- nemu在执行ret语句时会调用宏``NEMUTRAP(s->pc,R(10))``
			- R(10)为读取寄存器a0的值
			- 该宏的详细定义为``#define NEMUTRAP(thispc, code) set_nemu_state(NEMU_END,thispc,code)``
		- ``set_nemu_state(int state, vaddr_t pc, int halt_ret)``函数会设置各种nemu_state结构体中的各种属性，具体的：
			- ```
			  nemu_state.state = state
			  nemu_state.halt_pc = pc;
			  nemu_state.halt_ret=halt_ret
			  ```
		- 而在nemu对于每一条指令的执行过程中，如果nemu_state的状态变味了END，nemu将会停止执行，并且在nemu-main.c的main函数中，返回值时会判断nemu_state中的halt_ret，如果不为0，则会返回1