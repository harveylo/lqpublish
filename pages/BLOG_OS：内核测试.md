- # 退出QEMU
	- 现实中，关机需要实现**APM**或**ACPI**电源管理标准的支持
		- 很难
	- 但是QEMU支持``isa-debug-exit``特殊**设备**
		- 通过此设备可以简单地从**客户系统(guest system)**中退出
	- 使用此设备需要向QEMU传递一个``-device``参数，也可以将此参数添加到配置中
		- ```toml
		  [package.metadata.bootimage]
		  test-args = ["-device", "isa-debug-exit,iobase=0xf4,iosize=0x04"]
		  ```
	- ## 端口IO
		- 和MMIO不同，端口IO使用端口号和CPU地``in``，``out``指令访问
			- 这些指令要求一个端口号和**一个字节**的数据作为参数
		- qemu中的``isa-debug-exit``设备读取写入端口的值``value``，并将``(value << 1) | 1``作为**退出状态**并退出
		- 不直接通过内联汇编的``in``或``out``指令，而是使用``x86_64``crate中提供的抽象
	- ## 打印到控制台
		- 从内核发送数据到宿主系统需要一些方法，比如使用TCP网络接口来发送数据
		- ### 串口
			- 相较而言更加简单
			- 用来实现创行接口的芯片被称为**UARTs**(Universal Asynchronous Receiver-Transmitter)
			- 所有的UART都会兼容**16550 UART**，使用此模型即可
			-
			-
-