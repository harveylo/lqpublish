- # main函数
	- ## sockaddr_in结构体
		- 定义在头文件``netinet/in.h``中
		- ```
		  struct sockaddr_in {
		       short            sin_family;    // 2 字节 ，地址族，e.g. AF_INET, AF_INET6
		       unsigned short   sin_port;      // 2 字节 ，16位TCP/UDP 端口号 e.g. htons(3490)，
		       struct in_addr   sin_addr;      // 4 字节 ，32位IP地址
		       char             sin_zero[8];   // 8 字节 ，不使用
		  };
		  ```
	- ## startup函数
		-