- go不使用传统面向对象方法中的类和成员函数的语法，但是仍然可以给结构体添加函数，即**方法**
- # 方法
	- 在go中，方法其实就是**带有接收者(receiver)参数的函数**
	- 其语法为在函数名前用括号括起接收者参数
		- ``func (v Vertex) Abs() float64{``
		- 有了接收者之后，函数体中的代码可以直接访问接收者的每一个字段
	- 非结构体类型也可以有方法，但是不能给内建的基本类型声明方法，除非是用类型别名
		- ```go
		  type MyFloat float64
		  
		  func (f MyFloat) Abs() float64 {
		  	if f < 0 {
		  		return float64(-f)
		  	}
		  	return float64(f)
		  }
		  
		  func main() {
		  	f := MyFloat(-math.Sqrt2)
		  	fmt.Println(f.Abs())
		  }
		  ```
	- **[[$red]]==注意==**：接收者的类型定义和方法实现必须在同一包内
	- ## 指针接收者
		- 单纯的变量接收者由于go传参默认是值传递，因此无法修改接收者的值
		- 如果某个方法需要修改接收者内部字段的值，那么需要使用指针接收者，在实际编码中，指针接收者也更常用
		- ```go
		  func (v *Vertex) Scale(f float64) {
		  	v.X = v.X * f
		  	v.Y = v.Y * f
		  }
		  ```
		- 有趣的点在于，如果函数的形参指定某个参数为指针，那么在实际调用该函数时，实参需要添加取地址运算符`&`，例如
			- ``f(&a,b)``
		- 但是对于指针接收者方法，仍然可以直接使用变量来调用，实际上，go将调用自动补齐为了`(&v).Scale()`，也可以用一个指针来调用方法，不过和变量调用是等价的
			- ```go
			  var v Vertex
			  v.Scale(5)  // OK
			  p := &v
			  p.Scale(10) // OK
			  ```
		- 相反的，对于值接收者方法，也可以用指针来调用，不过等价于`(*v).Abs`
			- ```go
			  var v Vertex
			  fmt.Println(v.Abs()) // OK
			  p := &v
			  fmt.Println(p.Abs()) // OK
			  ```
- # 接口
	- 接口(interface)是一系列方法签名的集合
	- 一个类型为某个接口的变量可以指向任何一个实现了接口中**所有方法**类型的变量
	- 定义一个接口的语法为``type Name interface {}``
	- ```go
	  type Abser interface {
	  	Abs() float64
	  }
	  
	  func main() {
	  	var a Abser
	  	f := MyFloat(-math.Sqrt2)
	  	v := Vertex{3, 4}
	  
	  	a = f  // a MyFloat implements Abser
	  	a = &v // a *Vertex implements Abser
	  
	  	// In the following line, v is a Vertex (not *Vertex)
	  	// and does NOT implement Abser.
	  	//a = v
	  
	  	fmt.Println(a.Abs())
	  }
	  
	  type MyFloat float64
	  
	  func (f MyFloat) Abs() float64 {
	  	if f < 0 {
	  		return float64(-f)
	  	}
	  	return float64(f)
	  }
	  
	  type Vertex struct {
	  	X, Y float64
	  }
	  
	  func (v *Vertex) Abs() float64 {
	  	return math.Sqrt(v.X*v.X + v.Y*v.Y)
	  }
	  
	  ```
	- ## 接口是隐式实现的
		- 在别的语言中，实现一个接口的一般流程是显式声明实现某个接口，然后实现该接口的所有函数
			- ``class A implements B``
		- 但是在GO中，不需要这种显式地实现声明，任何一个类型都可以自由地实现一些方法，只要某个类型实现了某个接口中的所有方法，该类型就自动地被视作实现了该接口
		- 隐式接口实现的好处在于将接口的定义和实现解耦，你可以在任何地方定义一个接口，而所有实现了接口中所有方法的类型都自动实现了该接口，哪怕该类型的实际定义和声明**在另一个包**
	- ## 接口变量
		- 接口变量在使用上和一般变量没有什么区别
		- 其可以看作一个二元组，`(variable, type)`
			- `variable`是一个**实际变量**(concrete variable)
			- `type`是这个实际变量的类型
		- 其会存储一个实际变量，当在接口变量上调用方法时，调用的是实际变量的同名函数
		- 当一个接口变量被赋予一个别的实现了此接口的变量时，会拷贝一分存储于实际变量中
		- ### 接口变量的零值
			- 一个只有声明，没有初始化或没有经过赋值的接口变量是一个零值变量
			- 零值接口变量没有存储实际变量，也没有存储类型，因此无法调用函数
			- 零值接口变量的值是`nil`
	- ## 空接口
		- 空接口变量可以承载任何类型的变量
		- 因此空接口可以用于处理未知类型的变量，例如，`fmt.Println()`接收任意数量的空接口类型参数
	- # 类型断言
		- 用于从接口变量中取出具体的变量
		- ``t := i.(T)``
		- 如果接口变量的具体类型不是`T`，`t`会接到一个`T`类型的默认零值
		- 可以同时用一个变量接下是否是`T`的判断值：
			- `s,ok := i.(T)`
			- 如果`i`的实际变量是`T`类型，那么`ok`的值为true，反之为false
		- 如果在非短变量声明中使用类型断言，那么在类型不匹配的情况下会直接runtime error
		- ```go
		  func main() {
		  	var i interface{} = "hello"
		  
		  	s := i.(string)
		  	fmt.Println(s)
		  
		  	s, ok := i.(string)
		  	fmt.Println(s, ok)
		  
		  	f, ok := i.(float64)
		  	fmt.Println(f, ok)
		  	
		  	var ff float64
		  	ff = i.(float64) // panic
		  	fmt.Println(ff)
		  }
		  ```
	- ## 类型switch语句
		- 根据一个接口变量的类型来确定switch分支
		- ```go
		  switch v := i.(type) {
		  case T:
		      // here v has type T
		  case S:
		      // here v has type S
		  default:
		      // no match; here v has the same type as i
		  }
		  ```
		- **注意：**`fallthrough`不能用在类型switch语句中
- # 错误
	- go使用一个`error`接口来表示错误
		- ```go
		  type error interface {
		      Error() string
		  }
		  ```
	- 自定义的错误类都应该实现这个接口
	- 有的函数会返回错误变量，通过判断这个错误变量的值是否为`nil`来确定是否有错误发生
		- ```go
		  i, err := strconv.Atoi("42")
		  if err != nil {
		      fmt.Printf("couldn't convert number: %v\n", err)
		      return
		  }
		  fmt.Println("Converted integer:", i)
		  ```
	- ## `io.Reader`
		- io包中有一个`Reader`接口，可以存放数据流，并在读到末尾是给出一个`io.EOF`错误表示到达末尾
		- `Reader`接口中有一个`Read`方法
			- ``func (T) Read(b []byte) (n int, err error)``
			- 此方法将数据填充到给出的字节切片中，并返回填充的字节数并在碰到数据末尾时给出一个`io.EOF`错误
		-