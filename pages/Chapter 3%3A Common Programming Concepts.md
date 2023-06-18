- # 变量和可变性
	- ## 常量
		- 使用``const``关键字定义常量
		- 且常量的**类型(type)**必须在定义时指定，如``const THIS_IS_CONST: u32=60*60``
	- ## 覆盖(Shadowing)
		- 使用``let``重新定义一个同名变量即导致变量被覆盖
			- 如果在一个内部作用域中重新定义和**外部作用域**已有变量同名的变量，则被覆盖的变量尽在内部作用域有效，离开内部作用域后，外部作用域的变量仍然存在
			- 如果在同一个作用域中重新定义同名变量，则**原变量被覆盖**
		- 后一种覆盖往往用于**改变[[$red]]==一个变量的类型==**
			- rust作为一种强类型语言，直接改变一个变量的类型是不被允许的
			- 因此可以使用``let``重新定义并覆盖原变量
- # 数据类型(Data Types)
	- rust是**[[$red]]==静态类型==语言**，所有变量的类型都必须在编译时明确
	- 很多时候变量类型可以让编译器自行推导，若无法自行推导，则编译时会抛出错误
	- 使用**annotation**可以显式指明变量类型
	- ## 标量类型(Scalar Types)
		- 即非数组类型，rust中有四种主要的标量类型：**整数，浮点数，布尔和字符**
		- ### 整数类型
			- | Length | Signed | Unsigned |
			  | ---- | ---- | ---- |
			  | 8-bit | `i8` | `u8` |
			  | 16-bit | `i16` | `u16` |
			  | 32-bit | `i32` | `u32` |
			  | 64-bit | `i64` | `u64` |
			  | 128-bit | `i128` | `u128` |
			  | arch | `isize` | `usize` |
			- ``arch``类型取决于用户架构，即**在64位机器上是64位，在32位机器上是32位**
			- 整数字面量可以使用后最确定类型，如``57u8``，还可以使用下划线方便阅读，如``1_000``
			- | Number literals | Example |
			  | ---- | ---- | ---- |
			  | Decimal | `98_222` |
			  | Hex | `0xff` |
			  | Octal | `0o77` |
			  | Binary | `0b1111_0000` |
			  | Byte (`u8` only) | `b'A'` |
			- **整数溢出**
				- 在debug编译模式下，整数溢出会导致程序**[[$red]]==panic==**，而在release模式下编译，不会对整数溢出进行panic，而是会进行截断
				- 为了处理整数溢出，可以使用以下函数家族：
					- ``wrapping_*``函数进行显式的截断，如``wrapping_add``
					- ``checked_*``函数在溢出时返回``None``
					- ``overflowing_*``函数返回是一个布尔值表示是否会出现溢出
					- ``staturating_*``函数在溢出时使用最大或最小值，例如``u8:staturating_add(255:u8,255:u8)``返回255
		- ### 浮点数类型
			- 有两类原生(primitive)的浮点数类型，``f32``和`f64`
			- rust对于浮点数默认使用后者
		- ### 布尔类型
			- 包含`true`和`false`
		- ### 字节类型
			- 只有一个类型即``char``
			- rust中的字节类型是[[$red]]==**四字节**==，代表了一个**unicode 标量值**
	- ## 组合类型(Compound Types)
		- ### n元组(Tuple)
			- 使用括号定义一个n元组，元素之间使用逗号分隔
			- ``let tup: (i32, f64, u8) = (500, 6.4, 1);``
				- 类型声明可以省略
			- n元组中的所有数据被组合到一起，没有具体的名字，可以通过以下两种方式访问具体某一个元素
				- ```rust
				  let tup = (500, 6.4, 1);
				  let (x, y, z) = tup;//声明别名
				  ```
				- ```rust
				  let x: (i32, f64, u8) = (500, 6.4, 1);
				  let five_hundred = x.0;//通过下标访问
				  ```
			- **0元组**是一种特殊的类型，也称**[[$red]]==单元(unit)==**
				- 其类型和值都写作`()`
				- 其代表空值或空返回类型
				- 所有不返回任何值得函数都会隐式返回一个`()`
		- ### 数组类型
			- rust中的数组长度固定使用方括号定义一个数组
			- ``let a = [1,2,3,40];``
			- 数组的类型为**元素类型+长度**，如: ``let a: [i32; 5]``
				- 也可以初始化为相同值，如：``let a = [3; 5];``，意为另`a`为一个长度为5，值都为3的数组
			- 使用下标访问数组元素，如``a[0]``
				- 若访问下标无效，会产生``panic``
- # 函数
	- rust的函数命名采取**snake case**风格，即全部小写，使用下划线分割单词
	- 通过关键字``fn``定义函数
	- rust**[[$red]]==不关心==**函数定义的顺序，只要是定义在可见作用域内的函数都可以调用
	- ## 形参(Parameters)
		- 每一个形参必须明确其类型，如: ``fn function(a: i32)``
		- 如果想改变形参的值，需要定义为**可变引用**：``fn fun(a :&mut i32)``
	- ## 语句和表达式
		- rust中**语句(Statement)**和**表达式(Expression)**是分开的概念，且rust是**基于表达式(expression-based)**的语言
			- **语句**是一组执行某些操作但是**没有返回值**的语句
			- **表达式**计算并**产生一个值**
		- ``let``语句就是典型的语句，并没有返回值，因此不能有如下写法：``let y = (let x = 5)``
		- 表达式会计算的出一个值，包括：
			- 字面量和计算表达式
			- 函数调用
			- 宏调用
			- **大括号创建的一个块作用域**
				- ```rust
				  fn main() {
				      let y = {
				          let x = 3;
				          x + 1
				      };
				      println!("The value of y is: {y}");
				  }
				  ```
				- 表达式的结尾**[[$red]]==不能有分号==**，如果加上分号就变成了语句，没有返回值
		- ### 具有返回值的函数
			- 使用``->``声明函数的返回类型
			- 函数的返回值**等同于函数体最后一个表达式的值**
				- ``fn five() -> i32 {5}``
				- 若要使用这种方式隐式返回值，最后一条语句不能有分号，例如以下写法是错的
				- ```rust
				  fn plus_one(x: i32) -> i32 {
				      x + 1;
				  }
				  ```
			- 也可以使用``return``关键字**提前返回**
- # 注释
	- rust只有一种注释方式，即``//``
	- 但对于需要进行publish的crate来说，使用``///``注释可以支持markdown语法，方便编写文档
- # 控制流
	- ## if表达式
		- rust中if表达式后直接跟条件，不需要括号
		- 条件必须是一个返回值为布尔的表达式，且语句必须要用大括号包裹，哪怕只有一条语句
		- 使用`let`语句进行变量声明时，可以使用`if`进行条件初始化，如``let a = if condition {5} else {6};``
			- **两个语句的返回值必须保持一致**，否则会出现编译问题
	- ## 循环
		- ### ``loop``循环
			- loop循环在主动调用``break``之前会无限运行
			- rust中，loop语句**可以使用``break``语句返回值**
				- ```rust
				  fn main() {
				      let mut counter = 0;
				      let result = loop {
				          counter += 1;
				          if counter == 10 {
				              break counter * 2;
				          }
				      };
				      println!("The result is {result}");
				  }
				  ```
			- rust可以给不同的`loop`添加不同的**标签(label)**，来区分**不同层级的`loop`**以方便地``continue``和``break``
				- ```rust
				  fn main() {
				      let mut count = 0;
				      'counting_up: loop {
				          println!("count = {count}");
				          let mut remaining = 10;
				          loop {
				              println!("remaining = {remaining}");
				              if remaining == 9 {
				                  break;
				              }
				              if count == 2 {
				                  break 'counting_up;
				              }
				              remaining -= 1;
				          }
				          count += 1;
				      }
				      println!("End count = {count}");
				  }
				  ```
				- 以上地代码中，``break `counting_up`语句直接从内部循环中中断了外部循环
		- ### ``while``循环
			- 可以指定循环的条件，当条件不再成立时立即终止循环
			- ```rust
			  fn main() {
			      let mut number = 3;
			      while number != 0 {
			          println!("{number}!");
			          number -= 1;
			      }
			      println!("LIFTOFF!!!");
			  }
			  ```
		- ### `for`遍历集合
			- ```rust
			  fn main() {
			      let a = [10, 20, 30, 40, 50];
			      for element in a {
			          println!("the value is: {element}");
			      }
			  }
			  ```
			- 如果要遍历下标，使用``in (1..4)``的方式来遍历，若要逆序，则在范围后调用``.rev()``
				- ```rust
				  fn main() {
				      for number in (1..4).rev() {
				          println!("{number}!");
				      }
				      println!("LIFTOFF!!!");
				  }
				  ```
				- 注意``(1..4)``不包含4，若要包含，需要``(1..=4)``
				-