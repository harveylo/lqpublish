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
	- cargo是一个rust的**建造系统**和**包管理器**
	- 使用``cargo new {project_name}``直接新建一个rust项目目录，同时初始化git仓库
		- 如果git相关文件已经存在，则不会初始化git，使用``git new --vcs=git``来覆盖这一行为
		- 也可以指定除了git以外的其他版本管理工具，使用``cargo new --help``查看详情
	- rust项目的配置文件为``Cargo.toml``，使用[toml(Tom's Obvious, Minimal Language)](https://toml.io/en/v1.0.0)作为配置语言
	- 一个典型的配置文件包含至少两个表(Table)
		- **`[package]`**，此表存放对于一个包的配置
			- 其中``edition``指定编译时使用的编译器版本，若缺失会基于兼容性考虑，使用2015版本
		- **`[dependencies]`**，此表存放项目的所有依赖，在rust中，包被称为package
	- 一个典型的项目目录结构为：
		- 所有的代码文件置于``src``中
		- 顶层目录中仅存放README文件，许可证信息和配置文件
	- ## 使用cargo建造和运行程序
		- 使用cargo一个很重要的优势时，**cargo的所有命令都是跨平台的**
		- 在项目目录下使用``cargo build``可以建造项目
			- cargo的默认建造是``debug``建造，因此默认情况下， 建造完毕的可执行文件位于``\target\debug\``下
			- cargo build建造还会再项目的顶层目录下创建一个``Cargo.lock``文件，此文件用于跟踪项目的依赖版本
		- 在项目目录下使用``cargo run``可以方便地运行已建造程序
		- 使用``cargo clean``可以清除已建造文件
		- 使用``cargo check``检查项目是否可以成功编译，但是不生成可执行文件
			- 此命令比``cargo build``要快很多，因此仅仅是检查项目可否编译时，使用此命令可以节省时间
	- ## 建造发行版本(Release)
		- 像建造正式版本而不是debug版本时，使用``cargo build --release``命令编译，此命令编译会引入优化
			- 此命令会在``\target\release``下建造可执行文件