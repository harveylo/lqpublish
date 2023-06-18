- # 结构体的定义和实例化
	- 使用关键字``struct``定义结构体
	- 字段初始化可以明确每一个字段，若参数名和字段名完全一致，则不需要显式指定字段名
		- ```rust
		  fn build_user(email: String, username: String) -> User {
		      User {
		          active: true,
		          username: username,
		          email: email,
		          sign_in_count: 1,
		      }
		  }
		  ```
	- ## 使用结构体更新(update)语法从其他实例创建实例
		- ```rust
		  fn main() {
		      // --snip--
		  
		      let user2 = User {
		          active: user1.active,
		          username: user1.username,
		          email: String::from("another@example.com"),
		          sign_in_count: user1.sign_in_count,
		      };
		  }
		  ```
		- 以上代码等价于：
		- ```rust
		  fn main() {
		      // --snip--
		  
		      let user2 = User {
		          email: String::from("another@example.com"),
		          ..user1
		      };
		  }
		  ```
		- [[$red]]==**注意**==：更新语义和移动语义类似，会直接将未实现``Copy``trait的成员移动过去
			- 而任何有成员已经被移动的结构体变量，都会被无效化
			- 如果只移动实现了``Copy`` trait的成员，则原变量不会无效化，在此例中若只是用了user1的active和sign_in_count则原user1还能继续使用
			- 使用其他struct成员来手动初始化另一个结构体的成员，也是移动语法
	- ## 元组结构体
		- ```rust
		  struct Color(i32, i32, i32);
		  struct Point(i32, i32, i32);
		  
		  fn main() {
		      let black = Color(0, 0, 0);
		      let origin = Point(0, 0, 0);
		  }
		  ```
		- 即使不同的结构体有完全相同类型的成员，**结构体之间依然有着不同的类型**
		- 使用``.``加索引来访问元素
	- ## 无字段空结构体
		- 这种也称**类单元结构体(Unit-Like Structs)**
		- 在需要实现trait但不需要存储任何数据时很方便
		- ```rust
		  struct AlwaysEqual;
		  
		  fn main() {
		      let subject = AlwaysEqual;
		  }
		  ```
	- 结构体对自己的成员享有所有权，所以[[$red]]==**若结构体中有引用变量**==时，必须指明其生命周期，否则会产生错误
- # 结构体示例程序
	- 一个没有实现``Display`` trait的结构体无法直接被`println!`打印
	- 通过给一个结构体添加``#[derive(Debug)]``注解，或者手动实现Debug注解，可以使得一个结构体通过``println!("{:?}")``或``println!("{:#?}")``打印
	- 也可以通过``dbg!()``宏来打印实现了``Debug``的结构体，但是此宏[[$red]]==**会夺取所有权**==
		- 最终也会返回所有权
- # 方法(Method)语义
	- 实际上就是函数，不过**定义在结构体内部**
	- 给某个结构体定义方法，需要使用``impl``关键字，方法本身用``fn``定义
	- 任何方法的第一个参数一定是**``&self``**
		- 实际上是``self :&Self``的缩写
		- 在``impl``块中，``Self``是正在实现的类的别名
		- 如果写为``self : Self``或``self``，那么在调用一次此方法之后，变量的所有权就会被转移，无法再次使用
			- 若此方法将``self``转变为其他的东西，所以不希望原来的变量还能继续被使用，那么可以使用这种写法
		- 如果要改变自身的属性，使用``&mut self``
	- ```rust
	  impl Rectangle {
	      fn area(&self) -> u32 {
	          self.width * self.height
	      }
	  }
	  ```
	- ## 关联函数
		- 关联函数是指第一个参数不是``self``的，在``impl``块中实现的函数
		- 这种函数和方法不同，**不再依赖于某一个示例**，而是和类直接关联
		- 关联函数使用``::``调用
	- ## 多个``impl``
		- 每个结构体都允许有若干个``impl``块用于向其中添加方法和关联函数