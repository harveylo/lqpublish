- 关于go中的分号：
	- go语言仍然通过分号来区分一条语句结束与否，但是**编译器会自动给那些疑似完整语句的行末添加分号**
		- 这意味着，编译器会自动在**行末为特定token的行添加分号**
		- 因此大括号的左括号不能另起一行，否则函数或者控制流会被凭空添加分号，导致编译错误
	- 如果想在一行中写下多条语句，那么就需要在行内使用分号
- # 包的概念
  collapsed:: true
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
  collapsed:: true
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
  collapsed:: true
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
	- 在类型明确的情况下可以使用**短变量声明**，此声明不需要使用`var`关键字，直接以变量名和初始化值完成声明和初始化，其符号为 ``:=``
		- 由于go明确要求在函数以外的每个语句都必须以关键字开始，短变量声明**不能在函数以外使用**
		- ```go
		  func main() {
		  	var i, j int = 1, 2
		  	k := 3  //短变量声明
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
	- ## 零值
		- 在声明时没有赋初值的变量会默认被赋予**零值**
		- 对于算术类型(整数，浮点)，零值为：**`0`**
		- 对于布尔类型，零值为：**`false`**
		- 对于字符串类型，零值为：**`""`**(空字符串)
	- ## 类型转换
		- Go中所有的类型转换都必须采用显式类型转换形式
			- 形如`T(v)`，将变量`v`的类型转换为`T`
		- ```go
		  func main() {
		  	var x, y int = 3, 4
		  	var f float64 = math.Sqrt(float64(x*x + y*y))
		  	var z uint = uint(f)
		  	fmt.Println(x, y, z)
		  }
		  ```
	- ## 类型推导
		- 在某些情况下，声明变量时变量的类型能够直接被推导出来
		- 假如变量A被另一个变量B的值初始化，那么变量A的类型就是变量B的类型
			- ```go
			  var i int
			  j := i //j的类型也是int
			  ```
		- 加入在短声明时指定了一个字面量，那么变量的类型就是字面量的类型
			- ```go
			  i := 42 //int
			  j := .12 //float64
			  k := .12 + .5i //complex128
			  ```
	- ## 常量
		- 以`const`关键字声明
		- 可以是**字符**，**字符串**，**布尔值**或**数值**
		- 不能使用`:=`短声明
		- 数值常量的精度可以做到非常高，但是如果要向一个变量赋高精度常量的值，可能会因为精度过高而无法编译通过(变量精度不够不足以承载高精度常量)
- # 注释
  collapsed:: true
	- go中有两种注释
		- 行注释：``//``
		- 块注释：`/* */`
	- ## 注释和文档的关系
		- ### 包文档
			- 在**package 语句之前的注释**会被`godoc`用作包的文档
			- ```go
			  // Package superman implements methods for saving the world.
			  //
			  // Experience has shown that a small number of procedures can prove
			  // helpful when attempting to save the world.
			  package superman
			  ```
		- ### 符号(API，常量，变量)文档
			- 紧跟在某个全局声明前的注释会被`godoc`采用做针对该符号的文档
			- 要求这样的注释必须以此符号名称开头，否则不被采纳做文档
			- ```go
			  // enterOrbit causes Superman to fly into low Earth orbit, a position
			  // that presents several possibilities for planet salvation.
			  func enterOrbit() error {
			     ...
			  }
			  ```
- # 控制流
	- ## 循环
		- go**只有`for`循环**
		- `for`循环的三部分由分号隔开
			- 初始化语句，常为短变量声明，**可选**
			- 条件表达式
			- 后置语句，**可选**
		- **没有小括号，但是循环体的大括号是必须的**
		- ```go
		  func main() {
		  	sum := 0
		  	for i := 0; i < 10; i++ {
		  		sum += i
		  	}
		  	fmt.Println(sum)
		  }
		  ```
	- 若初始化语句和后置语句都不需要，仅保留条件表达式，则不需要多余的分号，直接在`for`后放置条件表达式即可，此时的`for`循环和其它语言中的`while`循环没有区别
		- ```go
		  func main() {
		  	sum := 1
		  	for sum < 1000 {
		  		sum += sum
		  	}
		  	fmt.Println(sum)
		  }
		  ```
	- 若连条件表达式一并省略，仅余一个`for`关键字，便写出了一个无限循环，等于其它语言中的`while(true)`
		- ```go
		  func main() {
		  	for {
		  		fmt.Println("yes")
		  	}
		  }
		  ```
	- ## 条件语句
		- ### `if`语句
			- 使用关键字`if`，同样无需小括号，但必须使用大括号包裹分支代码
			- `if`语句允许在条件表达式前执行**一条**额外的语句，该语句和条件表达式需要使用分号`;`隔开
				- ```go
				  func pow(x, n, lim float64) float64 {
				  	if v := math.Pow(x, n); v < lim {
				  		return v
				  	}
				  	return lim
				  }
				  ```
			- `if`语句可以对应若干个`else`或`else if`语句，在每一次`else if`中都可以在额外语句中引入新的局部变量，后续所有的`else`或`else if`分支代码都可以使用这些被引入的额外变量
				- ```go
				  func pow(x, n, lim float64) float64 {
				  	if v := math.Pow(x, n); v < lim {
				  		return v
				  	} else if xx := 32; v== lim{
				  		fmt.Printf("%g >= %g\n", v, lim)
				  	}else {
				  		fmt.Println(v,xx)
				  	}
				  	// 这里开始就不能使用 v 了
				  	return lim
				  }
				  ```
		- ### ``switch``语句
			-