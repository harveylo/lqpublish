title:: Domain Reload Research for .NET 6

- [域重加载](https://docs.unity3d.com/cn/2023.2/Manual/DomainReloading.html)
- # 背景
	- 微软的Jan  Kotas表达了帮助我们实现所需上游的兴趣
	- Jonas为Unity在CoreCLR原型中实现了类似于**域重加载(Domain Reloads)**的技术。
		- 此技术使用了**程序集加载上下文(Assembly Load Context)**
		- 此原型存在于.NET Core 3
- # 可选域重加载技术
	- ## 使用CoreCLR的程序集加载上下文
	- ## 使用CoreCLR关闭并重启运行时
		- 此选项没有被探索过，但是值得考虑
	-