- 由于不同的处理器使用不同的字节序(x86使用小端序，ARM使用大端序)，网络通信时可能会出现问题
	- ```
	  uint16_t http_port = 80;
	  if(packert -> port == http_port)
	  ```
	- 在上述的代码中，由于序字节序不同，在某些机器上可能执行失败
- 有相关的库函数解决了这个问题，在头文件``arpa/inet.h``中定义了如下库函数：
	- ``ntohs(), htons(),ntohl(),htonl()``
	- network to host short, network to host long
	- 使用这些函数可以写出平台无关的代码
- **[[$red]]==在处理网络数据时必须要十分小心！==**
	- 不然很有可能会出现忘记转换字节序，或多次转换字节序的问题，导致花费大量时间去debug
- # 包格式
	- IP数据报的格式发布于RFC 791
	- 四字节宽度
	- 大端序