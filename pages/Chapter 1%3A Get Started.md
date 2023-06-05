- 使用``rustfmt``工具可以格式化一个.rs文件
- rust的缩进为**四个空格**，不是制表符
- # 从hello程序说起
	- ```rust
	  fn main() {
	      println!("Hello, rust World!");
	  }
	  ```
	- 使用关键字``fn``定义一个函数，形如:
		- ``fn  <func_name> (){}``
	- ``println!()``中的`!`表明时调用了一个**rust宏(macro)**，**调用函数不需要`!`**
	- 分号用于结束一条语句
- # Cargo
	- cargo是一个rust的建造系统和包管理器
	-