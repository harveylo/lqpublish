# 指针
	- go也具有指针类型，一个指针使用`*T`来声明
		- `var p *int`
	- 取地址操作符`&`可以获取某个变量的内存地址
		- `p = &i`
	- 在指针上使用`*`进行解引用
		- `*p=1`
	- 在go中，没有指针运算，即对指针进行加减
- # 结构体
	- 使用关键字`struct`定义一个结构体
		- ```go
		  type Vertex struct {
		  	X, Y int
		  }
		  ```
	- go中的结构体类似于C的结构体，由一组字段组成
	- 结构体变量使用`.`访问每一个字段
	- ## 结构体指针
		- 指向一个结构体变量的指针
		- 理论上，通过一个结构体指针访问某个字段需要先解引用，即：`(*p).x`，但是go中允许隐式解引用，即直接写：`p.x`
	- ## 结构体初始化
		- 使用大括号括起来为每一个字段初始化
		- 按字段声名顺序初始化：
			- ``v1 = Vertex{1, 2}``
		- 显式指明字段初始化：
			- ``v2 = Vertex{X: 1, Y: 2}``
		- 空初始化
			- ``v3 = Vertex{}``
		- 初始化时添加取地址操作符，返回的是一个结构体指针
			- ``p  = &Vertex{1, 2}``
- # 数组
	- ## 数组
		- `[n]T`表示大小为n的T类型数组
			- `var a [10]int`
		- 数组的长度是固定的，且必须在编译时确定(不能使用变量来表示长度)，使用`len(a)`来获取数组的长度
		- 在声明数组的同时可以指定数组中的初始化值
			- `var a [5]int {1,2,3,4,5}`
	- ## 切片
		- 切片的长度不固定，使用相较数组更加灵活，因此切片比数组更常用
		- 切片的类型为`[]T`
		- 切片可以从一个数组中获得，通过上界和下界来界定：`a[low : high]`
			- 这是一个半开区间，包括下标，不包括上标
			- ```go
			  func main() {
			  	primes := [6]int{2, 3, 5, 7, 11, 13}
			  
			  	var s []int = primes[1:4]
			  	fmt.Println(s)
			  }
			  
			  ```
		- 切片可以看作**数组的引用**
			- 不存储数据，而只是存储了数组中某段数据的地址范围
			- 对切片中数据的修改会实际更改数组中的数据
		- 也可以直接创建一个切片，如：`q := []int{1,2,3,4,5}`
			- 实际上是创建了一个数组，然后返回一个指向这个数组中所有数据的一个切片
		- ### 默认上下界
			- 切片的默认下界是0，默认上界是数组长度
			- 对于数组``var a [10]int``来说，以下切片是等价的：
			- ```
			  a[0:10]
			  a[:10]
			  a[0:]
			  a[:]
			  ```
		- ### 切片的长度和容量
			- 切片中所包含元素的数量就是其**长度**
			- 切片中从第一个元素开始，到其所引用的原数组最后一个元素的个数就是其**容量**
			- 分别可通过`len`和`cap`来获取
			- 一个已有的切片，只要不超过其容量就可以向后扩充(无法向前扩充)
				- ```go
				  func main() {
				  	s := []int{2, 3, 5, 7, 11, 13}
				  	printSlice(s)
				  
				  	// 截取切片使其长度为 0
				  	s = s[:0]
				  	printSlice(s)
				  
				  	// 拓展其长度
				  	s = s[2:4]
				  	printSlice(s)
				  
				  	// 舍弃前两个值
				  	s = s[0:4]
				  	printSlice(s)
				  }
				  
				  func printSlice(s []int) {
				  	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
				  }
				  ```
		- ### 空(nil)切片
			- 没有**底层数组**的切片，以及**切片的零值**，是`nil`
			- ```go
			  func main() {
			  	var s []int
			  	fmt.Println(s, len(s), cap(s))
			  	if s == nil {
			  		fmt.Println("nil!")
			  	}
			  }
			  ```
		- ### 使用`make`创建切片
			- 使用make可以动态创建数组(和切片)
			- ``a := make([]int, 5)``，第一个参数是类型，第二个参数是长度
			- 如果要指定容量，可以传入第三个参数：
				- ``a := make([]int, 0,5)``
			- 新创建的数组的元素都是默认零值
		- ### 嵌套(多维)切片(数组)
			- 例如，二维int数组的类型为``[][]int``
			- ```go
			  func main() {
			  	// 创建一个井字板（经典游戏）
			  	board := [][]string{
			  		[]string{"_", "_", "_"},
			  		[]string{"_", "_", "_"},
			  		[]string{"_", "_", "_"},
			  	}
			  
			  	// 两个玩家轮流打上 X 和 O
			  	board[0][0] = "X"
			  	board[2][2] = "O"
			  	board[1][2] = "X"
			  	board[1][0] = "O"
			  	board[0][2] = "X"
			  
			  	for i := 0; i < len(board); i++ {
			  		fmt.Printf("%s\n", strings.Join(board[i], " "))
			  	}
			  }
			  ```
		- ### 切片扩容
			- 使用``apend``可以向切片扩容
			- 如果底层数组的容量足够扩容，那么会先使用底层数组的空间，此时所谓的append实际上会改变底层数组元素的值
			- 如果扩容太多元素，超过底层数组的大小，那么根据底层数组的内存分布情况，有可能会在底层数组后方追加内存空间，此时切片仍然指向底层数组地址；如果底层数组后无法追加内存空间，那么有可能会另开一片空间，复制已有元素再追加元素，此时新的切片不再指向原底层数组的地址
			- ```go
			  func main() {
			  	var s []int
			  	printSlice(s)
			  
			  	// 添加一个空切片
			  	s = append(s, 0)
			  	printSlice(s)
			  
			  	// 这个切片会按需增长
			  	s = append(s, 1)
			  	printSlice(s)
			  
			  	// 可以一次性添加多个元素
			  	s = append(s, 2, 3, 4)
			  	printSlice(s)
			  	
			  	t := s[:2]
			  	
			  	t =append(t,9,21,22,23,24,25,26,17,18,20)
			  	
			  	t[0] = 99
			  	
			  	printSlice(t)
			  	
			  	printSlice(s)
			  }
			  
			  func printSlice(s []int) {
			  	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
			  }
			  ```
		- ### 使用``range``遍历切片
			- 使用`range`关键字遍历切片时，会同时获取下标和值
			- ```go
			  var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
			  
			  func main() {
			  	for i, v := range pow {
			  		fmt.Printf("2**%d = %d\n", i, v)
			  	}
			  }
			  ```
			- 注意：得到的值是一份副本，修改其值并不会改变切片中相应元素的值
			- 可以指获取下标或指，对于不需要的部分使用`_`表示忽略
				- ```go
				  for i, _ := range pow
				  for _, value := range pow
				  ```
			- 在只需要下标的情况下，可以只写下标
				- ``for i := range pow``
- # 映射
	- 键值对的映射也可以看作一种数组，不过下标不再一定是int，也不再连续
	- 类型为`map[T1]T2`
	- 映射的零值为`nil`，此类映射没有键，也无法添加键值对
		- 创建一个新的映射需要使用`make`，若只是声明，将得到一个零值映射
	- 如果一个键尚未添加，那么访问将会获得初始零值
	- ```go
	  type Vertex struct {
	  	Lat, Long float64
	  }
	  
	  var m map[string]Vertex
	  
	  func main() {
	  	m = make(map[string]Vertex)
	  	m["Bell Labs"] = Vertex{
	  		40.68433, -74.39967,
	  	}
	  	fmt.Println(m["Bell Labs"], m["Harvey Labs"])
	  }
	  ```
	- ## 映射的初始化
		- 在初始化时需要指定键值对的值
		- ```go
		  type Vertex struct {
		  	Lat, Long float64
		  }
		  
		  var m = map[string]Vertex{
		  	"Bell Labs": Vertex{
		  		40.68433, -74.39967,
		  	},
		  	"Google": Vertex{
		  		37.42202, -122.08408,
		  	},
		  }
		  
		  func main() {
		  	fmt.Println(m)
		  }
		  ```
		- 也可以忽略映射类型
			- ```go
			  var m = map[string]Vertex{
			  	"Bell Labs": {40.68433, -74.39967},
			  	"Google":    {37.42202, -122.08408},
			  }
			  ```
	- ## 使用映射
		- 删除元素：`delete(m,key)`
		- 检查某个键是否存在：`elem, ok := m[key]`
			- 如果不存在，则`ok`为false，反之为true
- # 函数作为参数和闭包
	- 在go中，函数可以作为参数被传递，甚至作为返回值返回
	- ## 作为形参
		- 在定义一个函数时，某一个形参可以是函数
		- 使用``func(T1, T2, ...)T``来定义一个函数形参
		- ```go
		  func compute(fn func(float64, float64) float64) float64 {
		  	return fn(3, 4)
		  }
		  ```
	- ## 作为实参
		- 直接将函数名作为参数传入即可，也可以传入一个函数闭包
		- ```go
		  func compute(fn func(float64, float64) float64) float64 {
		  	return fn(3, 4)
		  }
		  
		  func fn(a,b float64) float64{
		  	return a+b
		  }
		  
		  func main() {
		  	hypot := func(x, y float64) float64 {
		  		return math.Sqrt(x*x + y*y)
		  	}
		  	fmt.Println(hypot(5, 12))
		  
		  	fmt.Println(compute(fn))
		  	fmt.Println(compute(math.Pow))
		  }
		  ```
	- ## 函数闭包
		- 函数闭包的声明和函数声明别无二致，也可以看作给函数赋予了一个别名
		- 函数闭包还可以指向已经声明的函数
		- 闭包的一大特点是可以使用形参和本地变量以外的外部值，做到**类似于局部静态变量的效果**
		- ```go
		  func adder() func(int) int {
		  	sum := 0
		  	return func(x int) int {
		  		sum += x
		  		return sum
		  	}
		  }
		  
		  func main() {
		  	pos, neg := adder(), adder()
		  	for i := 0; i < 10; i++ {
		  		fmt.Println(
		  			pos(i),
		  			neg(-2*i),
		  		)
		  	}
		  }
		  ```
	- 在上面的代码中，`pos`和`neg`都指向一个函数，但是各自都有一个静态的`sum`变量
		- 因为`adder`返回的函数对象指向了一个外部的`sum`，这使得`sum`不再分配在栈上，而是逃逸到了堆上，使得在`adder`函数返回之后，其返回的函数能继续使用`sum`