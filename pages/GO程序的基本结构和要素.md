# 包的概念
	- 一个包由多个`.go`源文件组成
	- 每个源文件只可从属于一个包
		- 每个源文件必须在**非注释的第一行**使用`package <pkg_name>`指明此文件所属的包
		- 每个go程序都包含一个名为`main`的包
	- 所有的包名都应当**使用小写字母**
	- 属于同一个包的所有文件必须被一起编译，即一个包是一个编译时单元，因此每个目录应当只包含同在一个包的文件
	- ## 导入包
		- 通过关键字``import``导入包，包名用双引号括起
		- 可以通过括号，一次导入多个包：
			- ```go
			  import(
			  	"fmt"
			    	"os"
			  )
			  ```
			- 或更简便地：``import ("fmt"; "os)``，但是源文件格式化之后会被替换为上面的格式
		- 导入包时可以指定此包在后续程序中被使用的别名
			- ``import fm "fmt"``
			- 在之后的代码中可以使用`fm`来代指包`fmt`
		- ### 包名与路径
			- 在引入包是如果包名开头没有`.`或`/`则编译器会在注册目录下寻找包
			- 如果开头带有`./`则会在当前目录下寻找
				- 仅在将当前目录加入``GOPATH``后有效
			- 如果开头带有`/`则将其视作绝对路径(Windows下同样可用)
		- ### 包与项目结构
			- 有两种项目结构的组织方式，页分别对应了两种依赖管理方法
			- **GOPATH**
				- go1.14版本之前唯一的方式，其使用较为复杂，且需要将当前开发目录添加到`GOPATH`环境变量中
				- 详细的由于已经过时，则不多赘述，其在项目结构上还会存在`src`目录
			- **GOMODULE**
				- go1.14版本之后官方强烈推荐的方式，使用gomodule不需要在环境变量上做出改变，秩序开启gomodule功能即可
					- ``go env -w GO111MODULE=on``
				- 在新建一个项目时，通过指令`go mod init <module_name>`初始化项目，其中`module_name`的起名方式有一定讲究
					- 如果只是本地开发，那么`module_name`可以任起
					- 如果有发布此项目作为一个module的打算，那`module_name`需要具备一个合理的前缀，此前缀一般是自己github仓库的前缀或者自己的一个域名，或人名或公司名
					- 详细可见[官方文档](https://go.dev/doc/modules/managing-dependencies#naming_module)
				- 初始化这个项目之后会出现一个`go.mod`文件，此文件用于记录当前项目的module名和项目依赖
				- 依赖可以在这个文件中手动加入，也可以在开发过程中直接使用`import`关键字引入一个module，然后通过`go mod tidy`手动解析依赖并获取依赖
				- 对于此项目内部的包，每个包的所有文件至于一个目录内，该目录的所有源文件都应当在非注释的第一行指明此源文件所从属的包名
				- 主源文件置于项目根目录下，例如`main.go`
				- 如果需要引用项目内部的包，直接使用`import "<module_name>/<package_name>"`即可
					- `module_name`是带路径的完整报名，即`go.mod`文件中module的名称是什么就用什么
				- 如果要使用本地另一个项目中的包，那么可以手动引入此包
					- 添加：`require "<module_name>" v0.0.0`
						- 由于是本地项目，因此版本号使用`v0.0.0`
					- 重定向项目位置：`replace "<module_name>" => "<path_to_module>"`
		- ### 可见性
			- 某一个包中的标识符，如果**以大写字母开头**，则此包被import的时候这些标识符对于引入者来说是可见的，反之则不可见
			- 可以利用这一点将包机制作为命名空间来使用，避免命名冲突
- # 函数
	- 使用关键字`func`定义函数
	- 在go中，函数形参的类型定义在参数名之后，形如
		- ``func funcName(a int, b int)``
	- 多个相同类型的形参名可以放在一起，只记一次形参类型
		- ``func funcName(a, b int)``
	- 返回类型置于形参列表之后，函数体放在紧跟返回类型后的大括号中。返回类型可以是一个列表
		- ```go
		  func funcName(parameter_list) (return_value_list){
		    ...
		  }
		  ```
	- 返回列表中的返回值可以有名称，但是主要是作为文档用途。
		- 没有参数的`return`语句返回已命名的返回值，但只建议在短函数中使用，否则将极大影响可读性
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func split(sum int) (x, y int) {
		  	x = sum * 4 / 9
		  	y = sum - x
		  	return
		  }
		  
		  func main() {
		  	a,b:=split(20)
		  	fmt.Println(a,b)
		  	fmt.Println(split(17))
		  }
		  ```
- # 变量
	- 使用`var`关键字来声明变量
		- `var a,b,c int`
	- 一次性声明多个不同类型的变量，需要使用括号
		- ```go
		  var (
		  	c            int
		  	python, java bool
		  )
		  ```
	- 变量在声明时可以进行初始化， 每个变量对应一个初始值。能够推导类型的初始值可以省略类型，且不同类型的变量可以一起初始化
		- ```go
		  var i, j int = 1, 2
		  var k = 2.9
		  
		  func main() {
		  	var c, python, java = true, false, "no!"
		  	fmt.Println(i, j, c, python, java, k)
		  }
		  ```
	- 在类型明确的情况下可以使用**短变量声明**，此声明不需要使用`var`关键字，直接以变量名和初始化值完成声明和初始化
		- 由于go明确要求在函数以外的每个语句都必须以关键字开始，短变量声明**不能在函数以外使用**
		- ```go
		  func main() {
		  	var i, j int = 1, 2
		  	k := 3
		  	c, python, java := true, false, "no!"
		  
		  	fmt.Println(i, j, k, c, python, java)
		  }
		  ```
	- ## Go的基本类型
		- `bool`
		- `string`
		- ```go
		  int  int8  int16  int32  int64
		  uint uint8 uint16 uint32 uint64 uintptr
		  ```
		- `byte`，等于`uint8`
		- `rune`，等于`int32`，表示一个unicode码点
		- `float32  float64`
		- `complex64  complex128`
		- `int`和`uint`在32位机上是32位，在64位机上是64位
		- 在使用库函数`fmt.Printf`时，可以使用`%T`直接打印类型信息
			- ```go
			  var (
			  	ToBe   bool       = false
			  	MaxInt uint64     = 1<<64 - 1
			  	z      complex128 = cmplx.Sqrt(-5 + 12i)
			  )
			  
			  func main() {
			  	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
			  	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
			  	fmt.Printf("Type: %T Value: %v\n", z, z)
			  }
			  ```
		- ### 关于整形溢出
			- 在go中，算术常量和字面量的值可以远大于64位，但是最多不能超过512位
				- 这些常量在编译时确认数值，运行时看不到常量
			- 当需要使用这些远超内置类型的大小的常量时，除非将常量减小到内置类型的范围内，不然会报溢出错误
			- ```go
			   const x = 1<<100
			  var y = x>> 98 //correct
			  var z = x-1 //wrong
			  ```