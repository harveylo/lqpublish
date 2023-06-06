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
	-