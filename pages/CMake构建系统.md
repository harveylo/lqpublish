- 一个基于CMake的**构建系统(Buildsystem)**被组织为一些**高级逻辑(logical target)**目标的集合
- 每一个逻辑目标对应一个可执行文件或一个库文件，也可以是一个包含自定义命令的自定义目标
- 建在系统会使用目标之间的依赖关系来确定不同目标的建造顺序和为了应对改动而重新生成(regeneration)的规则
- # 二进制目标
	- 可执行目标和库目标属于二进制目标
	- 可分别使用``add_executable``和``add_library``命令定义
	- 这些目标在实际建造时所对应的生成文件会具有相关平台所对应的**前缀(prefix)**，**后缀(suffix)**以及**扩展名(extension)**
	- 二进制目标之间的依赖关系通过``target_link_libraries()``命令指定
	- 例如：
		- ```
		  add_library(archive archive.cpp zip.cpp lzma.cpp)
		  add_executable(zipapp zipapp.cpp)
		  target_link_libraries(zipapp archive)
		  ```
	- ## 二进制可执行目标
		- 使用``add_executable``命令定义
		- 且可执行目标能用在后续的指令中，例如在``add_custom_command``命令中，``COMMAND``可以是一个可执行目标
			- 生成的构建系统会在完成该可执行目标的编译后再执行自定义的command
	- ## 二进制库目标的类型
		- ### 普通库
			- 可以查看``add_library``指令的[章节](logseq://graph/Logseq?block-id=63fb1780-70ba-4128-9ff4-9ca146352792)
		- ### 苹果framework
			-