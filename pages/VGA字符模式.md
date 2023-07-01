- # VGA字符缓冲区
	- 通常情况下，VGA字符缓冲区是一个**25行，80列**的**二维数组**
	- 其内容会被**实时渲染到屏幕上**
	- 此数组的元素被称为**字符单元(Character Cell)**
	- 每个大小为**2字节**，其格式如下：
		- | Bit(s) | Value |
		  | ---- | ---- | ---- |
		  | 0-7 | ASCII code point |
		  | 8-11 | 前景(字符)颜色(Foreground color) |
		  | 12-14 | 背景颜色(Background color) |
		  | 15 | 是否闪烁(Blink) |
		- 第一个字节更准确地说是类似于**437字符编码**
	- 可用颜色列表如下：
		- | 0x0 | Black | 0x8 | Dark Gray |
		  | 0x1 | Blue | 0x9 | Light Blue |
		  | 0x2 | Green | 0xa | Light Green |
		  | 0x3 | Cyan | 0xb | Light Cyan |
		  | 0x4 | Red | 0xc | Light Red |
		  | 0x5 | Magenta | 0xd | Pink |
		  | 0x6 | Brown | 0xe | Yellow |
		  | 0x7 | Light Gray | 0xf | White |
	- 通过**内存映射IO**，向地址``0xb8000``读取或写入即可访问VGA字符缓冲区
	- 字符缓冲区**支持便准读写操作**，可以像正常
- # volatile
	- volatila操作可以保证一些操作不会被编译器激进的优化给破坏
		- 例如，向VGA缓冲区写入字符就有可能被编译器优化掉，因为编译器并不知道``0xb8000``是一块特殊的MMIO地址，只知道我们向一块内存写入了数据但从来没有使用过
	- 使用volatile需要添加volatile依赖
-