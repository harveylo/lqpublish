- metaprogramming，可以看作用一种语言去**[[$red]]==生成==**（操作）另外一种语言（或程序）。译作**元编程**
- 元编程的一些使用例：
	- **代码生成**
		- ```
		  #!/bin/sh
		  # metaprogram
		  echo '#!/bin/sh' > program
		  for i in $(seq 992)
		  do
		      echo "echo $i" >> program
		  done
		  chmod +x program
		  ```
		- 使用几行代码便生成了千行代码
	- **代码仪表化**
		- 使用元编程 完成[dynamic program analysis](https://en.wikipedia.org/wiki/Dynamic_program_analysis)
	- **行为改变**
		- 可以改变程序的行为，如同[aspect-oriented programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming)一样
		- 例如，元编程可以用于注入feature flags或者bug修复
- # 生成系统（Build systems）
	- 一个生成系统可以用来帮助一个项目不同的生成目标
	- 生成系统往往具有一些共性，一般来说都是定义一些依赖（dependence），和目标，构建目标之前需要先完成该目标的依赖。
	- **make**是最常用的依赖系统，详情见： [[Makefile]]
- # 依赖管理
	- 许多语言都有自己的依赖管理工具，比如ruby的rubygems，python的pypi等
	- 不同的依赖管理工具机制会有所不同，但是都有一些共性
	- ## versioning
		- 被依赖的程序或库一般都每一个release都会有一个版本号
		- 版本号可以将依赖固定下来，当一个项目依赖于某一版本的库，库的后续更新不会影响该程序的稳定性
		- 为了让使用者直观地知道更新内容，版本号需要遵守一定的规则，目前普遍被接受地一种规则是[*semantic versioning*](https://semver.org/)
		- 在**semantic versioning**下，版本号形如：major.minor.patch
			- 如果新的release没有更改api，增加patch version
			- 如果增加了新的向后兼容的功能，增加minor version
			- 如果对api进行了不可向后兼容的更改，增加major version
		- 在这样的版本命名规范下，依赖高版本的程序不一定能在低版本的库上正确运行，但是一般可以在major version相同的更高的minor version上运行
	- ## lock files
		- lock file 给出当前项目对于每个依赖的具体版本号
		- 如果需要更新某个以来的版本号，使用具体的upgrade命令来显式地进行，这样可以避免自动更新带来的版本兼容问题
		- 这种方法的极端例子叫做vendoring，即手动复制所有依赖的版本，完全掌控所有以来的版本，但是也更加麻烦
- # 持续集成（Continuous integration）系统
	- 持续集成总的来说就是当代码改变时就会自动运行的系统
	- 市面上有很多面向开源地免费持续集成服务提供商，诸如Travis CI，Azure piplines 和 Github Action等
	- 以上服务都有一个共性，那就是添加一个秒速文件，指定每当仓库发生各种改变时需要做的动作
	- ## 测试术语简介
		- Test Suite：所有测试的集合
		- Unit Test：隔离测试单个特性的微型测试
		- Integration Test：测试更大的系统中各个模块是否合作正常的测试
		- Regression Test：用曾经会产生bug的测试集来测试，确保bug不会再现
		- Mocking：用假的实现替换一些和当前被测功能无关的函数，模块或类型
-
	-
	-