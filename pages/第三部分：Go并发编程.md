- **GO语言就是为并发而生的**，其在语言层面为并发提供了很多便捷有效的特性
- # GO携程(goroutine)
	- go携程是由go运行时管理的**线程**
	- 使用`go`关键字可以直接开启一个携程并运行
	- ```go
	  func say(s string) {
	  	for i := 0; i < 5; i++ {
	  		time.Sleep(100 * time.Millisecond)
	  		fmt.Println(s)
	  	}
	  }
	  
	  func main() {
	  	go say("world")
	  	say("hello")
	  }
	  ```
- # 信道(channel)
	- go为携程提供了语言层面支持的通信方式，即**信道**
	- 信道需要在创建时指定信道所传输的变量类型
		- ``ch := make(chan int)``
		- [[$red]]==注意==：一个信道的完整定义必须带有类型，因此此处的`make`语句的参数中，`chan`和`int`之间并没有逗号，因为`chan int`合起来构成一个对于信道的完整定义
	- 通过信道，不同的携程之间可以发送和接收值，传输时使用`<-`符号指向数据流通的方向
		- 往信道上发送信息：`ch <- v`
		- 从信道上接收信息：`v := <- ch`
	- 默认情况下，**在信道上收发数据都会造成阻塞**，go通过这种方式在没有显式锁和条件变量的情况下**完成同步**
		- ```go
		  func sum(nums []int, c chan int) {
		  	cur := 0
		  	for _, i := range nums {
		  		cur += i
		  	}
		  	c <- cur
		  }
		  
		  func main() {
		  	nums := []int{2, 3, 4, 5, 6, -1}
		  	ch := make(chan int)
		  	go sum(nums[:len(nums)>>1], ch)
		  	go sum(nums[len(nums)>>1:], ch)
		  	x, y := <-ch, <-ch
		  	fmt.Println(x, y, x+y)
		  }
		  ```
	- ## 带缓存的信道
		- 信道可以带有缓存，在`make`创建信道时，可以传入第二个参数表示信道的buffer大小
			- `make(chan int, 100)`
		- 带有缓存的信道只在以下情况成立时会造成阻塞：
			- 从信道读取信息，但是**信道缓存为空**
			- 向信道发送信息，但是**信道缓存已满**
		- 除此之外对于信道的操作都不会造成阻塞
	- ## 信道的关闭和数据边界
		- 信道可以通过`close(ch)`来关闭
		- 应当在代码中保证：**只有发送方才能关闭信道**
			- 因为在已经关闭的信道上发送消息会导致panic
		- 接收方在信道关闭之后会收到相关提示
			- 通过`v, ok := <-ch`中的`ok`来判断是否信道已关闭
			- 如果`ok`为false则表明信道已关闭，不会再有数据通过此信道发送
		- 如果要持续从某个信道接收数据直到信道被发送官方关闭，则可以使用`range`来便捷地表示：
			- `for data := range ch`
		- [[$red]]==注意==：信道不同于文件，大多数情况下关闭一个信道是没必要的。
			- 一般只会在接收方需要被明确告知是否还有更多的数据待处理时才需要发送方做出关闭信道的操作
	- ## `select`语句
		- `select`语句可以让一个携程等待多个信道上的操作
		- `select`语句会阻塞直到列出的通信操作之一发生，然后执行对应case的语句
			- 如果多个操作同时发生，随机选择一个case执行
		- ```go
		  func fibonacci(c, quit chan int) {
		  	x, y := 0, 1
		  	for {
		  		select {
		  		case c <- x:
		  			x, y = y, x+y
		  		case <-quit:
		  			fmt.Println("quit")
		  			return
		  		}
		  	}
		  }
		  
		  func main() {
		  	c := make(chan int)
		  	quit := make(chan int)
		  	go func() {
		  		for i := 0; i < 10; i++ {
		  			fmt.Println(<-c)
		  		}
		  		quit <- 0
		  	}()
		  	fibonacci(c, quit)
		  }
		  ```
		- 注意上述例子中还使用了匿名函数
		- ### `select`语句的默认case
			- `select`语句中的`default`分支会在其他时间都没有发生时被执行，且**不会被block**
			- 常用于尝试在不被block的情况下发送或接受数据
			- ```go
			  	tick := time.Tick(100 * time.Millisecond)
			  	boom := time.After(500 * time.Millisecond)
			  	for {
			  		select {
			  		case <-tick:
			  			fmt.Println("tick.")
			  		case <-boom:
			  			fmt.Println("BOOM!")
			  			return
			  		default:
			  			fmt.Println("    .")
			  			time.Sleep(50 * time.Millisecond)
			  		}
			  	}
			  ```
			-