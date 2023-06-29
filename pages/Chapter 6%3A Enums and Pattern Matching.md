- # 定义一个枚举类型
	- 使用``enum``关键字定义一个枚举类型
	- 在rust中，枚举类型一个很关键的特点是，**枚举类型中的每个类别可以定义自己包含的数据**
		- ```rust
		  enum IpAddr {
		      V4(u8, u8, u8, u8),
		      V6(String),
		  }
		  let home = IpAddr::V4(127, 0, 0, 1);
		  let loopback = IpAddr::V6(String::from("::1"));
		  ```
	- 枚举的每一种类型可以包含任何数据，**甚至是结构体和另一种枚举类型**
	- 还可以使用类似于结构体的声明方式，为数据段定义名字
		- ```rust
		  enum Message {
		      Quit,
		      Move { x: i32, y: i32 },
		      Write(String),
		      ChangeColor(i32, i32, i32),
		  }
		  ```
	- ## Option 枚举相对于空值(Null)的优势
		- rust没有其他语言中常见的**空值**概念
		- rust通过枚举类型完成了空值语义所能达到的目的：区分一个值的有效性
			- 在有空值得语言中，一个值要么是空值要么不是空值
			- rust中，如果想表达类似于空值的语义，使用``Option``枚举类型完成
		- ``Option``定义如下
			- ```rust
			  enum Option<T> {
			      None,
			      Some(T),
			  }
			  ```
		- 相较于空值，``Option``避免了空值和非空值通过同样的方式使用：
			- ```rust
			  let x: i8 = 5;
			  let y: Option<i8> = Some(5);
			  
			  let sum = x + y;
			  ```
			- 以上代码编译会报错，因为在确定``Option``的具体类型之前，并不能直接使用
		- 要使用一个``Option<T>``，就必须要将其转换为``T``类型，这个转换过程会进行编译时的检查，避免了空值可能造成的问题
- # match控制流结构
	- ``match``是rust中一个非常强大的特性
	- 允许将一个值与一系列的模式相比较，根据匹配模式执行对应代码
	- ``match``的强大在于匹配模式的表达能力和编译时检查，确保所有可能情况都能得到处理
	- 每一个分支的值是最后一个表达式的值
		- 也可以用``return``提前返回
	- 所有分支的**返回值[[$red]]==必须相同==**
	- 还可以匹配某种枚举类型的内部值
	- ```rust
	  #[derive(Debug)]
	  enum UsState {
	      Alabama,
	      Alaska,
	  }
	  
	  enum Coin {
	      Penny,
	      Nickel,
	      Dime,
	      Quarter(UsState),
	  }
	  
	  fn value_in_cents(coin: Coin) -> u8 {
	      match coin {
	          Coin::Penny => 1,
	          Coin::Nickel => 5,
	          Coin::Dime => 10,
	          Coin::Quarter(state) => {
	              println!("State quarter from {:?}!", state);
	              25
	          }
	      }
	  }
	  ```
	- ``match``必须涵盖所有可能取值，如果没有则编译无法通过
	- ## 匹配模式
		- ``other``用于表示所有其他情况
			- 此分支必须放到最后
			- **因为模式匹配是[[$red]]==按顺序进行的==**
		- ``_``占位符
			- 表示其他所有值**[[$red]]==都是非期望的==**
			- 可以匹配任何值但不会绑定该值，即无法在后续的处理语句中使用此值
			- 若不想运行任何代码，则使用``()``
- # ``if let``简洁控制流
	-
	-