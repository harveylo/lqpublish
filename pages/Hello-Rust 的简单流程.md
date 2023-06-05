- 通过``cargo new <project-name>``可以快速创建一个新的rust项目
- 一个新的空白rust项目的典型结构为：
	- ```
	  <project-name>
	  |- Cargo.toml
	  |- src
	    |- main.rs
	  ```
	- ``Cargo.toml``为rust的**清单文件**，包含项目的元数据和依赖库
	- ``main.rs``即为主逻辑文件
- 在一个rust的项目目录中使用``cargo run``构造并运行项目
- # 添加依赖
	- 在[crates.io](https://crates.io/)上可以找到rust的所有包
	- 在rust中，一个包通常被叫做**[[$red]]==crates==**
	- 将需要添加的依赖写入``Cargo.toml``文件即可