- # 第一版本
	- ```rust
	  use std::io;
	  
	  fn main() {
	      println!("Guess the number!");
	      println!("Please input your guess.");
	      let mut guess = String::new();
	      io::stdin()
	          .read_line(&mut guess)
	          .expect("Failed to read line");
	      println!("You guessed: {guess}");
	  }
	  ```
	- 为了获取用户输入，需要使用``io``库，而``io``库又处于``std``库中，因此引入``io``库，使用：
		- ``use std::io;``
		- rust中存在**[[$red]]==prelude==**项的概念，而rust语言级别的预定义项在所有项目中都能使用，[参看](https://doc.rust-lang.org/std/prelude/index.html)
		- 某些模块或者包也可能会有自己的prelude项
		- 对于不属于prelude项中的东西，需要通过``use``引用
	- 使用``let``定义变量
		- 单纯使用``let``定义的变量是**[[$red]]==不可变的==(immutbale)**
		- 要定义可变变量需要使用``let mut``显式定义
		- 使用`::`调用某个type中实现的函数，例如`String::new`
	- `&mut guess`表示传入的是`guess`的可变引用
		- 默认情况下(不使用`mut`，如`&guess`)传入的是不可变引用
	- `read_line`函数会返回一个`Result`类型的值，这个类型是一个枚举类型
		- `Result`有两个变种，`Ok`和`Err`
			- `Ok`标识操作成功，`Err`标识操作失败且会包含失败信息
		- `Result`类型有一个`expect`函数
			- 如果`Result`是`Err`类型，调用此函数会导致程序崩溃并打印用户给出的错误信息
			- 如果`Result`是`Ok`类型，`expect`函数会直接返回`Result`中所包含的信息，对于`read_line`函数来说，是读取的字节数
	- 使用`println!`输出时，使用`{}`作为占位符
		- 如果是变量本身，可以直接把变量包裹在`{}`中
		- 如果要做计算，则使用`{}`做占位符，计算式跟在格式化字符串后，如
			- `println!("x = {x} and y + 2 = {}", y + 2);`
- # 第二版本
	- ```rust
	  use rand::Rng;
	  use std::cmp::Ordering;
	  use std::io;
	  
	  fn main() {
	      println!("Guess the number!");
	      let secret_number = rand::thread_rng().gen_range(1..=100);
	      loop {
	          println!("Please input your guess.");
	          let mut guess = String::new();
	          io::stdin()
	              .read_line(&mut guess)
	              .expect("Failed to read line");
	          let guess: u32 = match guess.trim().parse(){
	              Ok(num) => num,
	              Err(_) => continue
	          };
	          println!("Your guess: {guess}");
	          match guess.cmp(&secret_number) {
	              Ordering::Less => println!("Too small!"),
	              Ordering::Greater => println!("Too big!"),
	              Ordering::Equal => {
	                  println!("You win!");
	                  break;
	              }
	          };
	      }
	  }
	  ```
	- ## 依赖控制
		- cargo可以理解一个crate的**语义版本(Sematic Version)**
		- ``rand="0.8.5"``实际上是``rand=^0.8.5``的略写，意味着**至少0.8.5，低于0.9.0**
			- 因为**0.9.0**之前的版本都保证不会改变API
			- 这样可以使得在保证兼容性的情况下，获取最新的crate
		- **cargo**在建造时会算出所有的依赖，然后生成一个``Cargo.lock``文件，确保每次build的正确性
			- 在手动更改配置文件中的依赖版本之前，cargo都会使用``Cargo.lockl``文件中的依赖来建造
			- 使用``cargo update``可以在不影响api使用的情况下更新crate，例如**从0.8.5到0.8.6**，但不会到0.9.0
			- 要升级到0.9.0需要显式在``Cargo.toml``中更改依赖版本
	- ``1..=100``定义了一个闭区间范围，一般形式为``<start>..=<end>``
	- 使用``cargo doc --open``可以构建所有依赖的文档
	- 引入 ``Ordering``，此类型是另外一个枚举类型，包括``Less, Greater, Equal``
	- 任何可比较的值都有``cmp``函数
	- 使用``match``语句，根据不同的结果进行不同的处理
		- ``match``语句包含不同的``arm``，match会根据给出的``arm``做匹配
	- 默认情况下，rust使用的整数类型为``i32``
	- rust允许重新定义同名变量，重新定义的变量会覆盖之前的变量
	- ``trim``函数会去掉字符串开头和末尾的所有空白字符
	-
	-
	-