- # CMake是什么？
	- CMake是一个高级编译配置工具，全程为**Cross Platform Make**，比``make``更高一级
	- 可以根据简单的语句来**描述**所有平台的安装(编译)过程
	- 最初是作为Makefile生成器，但现在已经能够输出各种各样的**makefile**或者**project**文件，类似于Unix下的**automake**
	- CMake的configuration file名叫``CMakeList.txt``
	- CMake并不直接建构出最终软件，而是产生标准的construct file，例如Unix的Makefile，visual C++的projects/worspaces，然后再使用一般的建构方式使用
	- 所以对于C或者C++项目来说，使用CMake的**一般流程**是：编写``CMakeList.txt``-> 运行``cmake``构建出项目 -> 运行``make``正式编译项目
	- CMake可以
		- 编译源代码
		- 制作程序库
		- 产生适配器(wrapper)
		- 以任意顺序构建可执行程序
	- CMake的**建造系统(build system)**由若干高级别的**逻辑目标(logical target)**组成。
		- 每一个目标都对应一个可执行文件或库，或包含自定义命令的自定义目标
- # CMake语言和项目组织结构
	- CMake的输入文件应该使用CMake语言编写，放置于``CMakeLists.txt``或``.cmake``文件中
	- 一个项目中的CMake源文件被组织为：
		- **目录(Dicrectories)**
			- 格式：``CMakeLists.txt``
			- 当CMake处理一个项目源树(Source Tree)时，入口就是一个位于最顶层源目录(source directory)的名为**CMakeLists.txt**的文件
			- 该文件可能包含了所有的构建specification，也可以使用``add_subdirectory()``添加建造子目录
				- 每一个使用该命令添加的建造子目录也必须包含一个``CMakeLists.txt``文件作为该目录的entry point
			- 对于每一个被处理的源目录，CMake会再建造树上生成一个对应目录作为默认的工作和输出目录
		- **脚本(Scripts)**
			- 格式：``<scripts>.cmake``
			- ``cmake``使用``-P``选项可以直接执行使用CMake语言编写的脚本
				- 仅仅执行其中的语句，不生成建造系统，因此不允许含有定义了建造目标或者行为的语句
		- **模块(Modules)**
			- 格式：``<module>.cmake``
			- 目录和脚本中的cmake语言代码都可以包含``include()``语句来在当前scope和context下装载一个cmake模块源文件
			- ``man 7 cmake-module``可以查看已安装的cmake distribution自带的模块
			- 项目也可以提供自己的模块，用``CMAKE_MODULE_PATH``来指定这些模块的位置
- # 基本语法
	- [Manual Page](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)
	- 一个CMake的源文件由若干**文件元素**组成
	- 一个**文件元素**可能是一个**命令调用(command invocation)**，也有可能是一个**括号注释**
	- **命令调用**由**命令标识符**和由括号括起来的若干参数组成
	- 参数之间可以用空格分割也可以用新起一行来分割
		- 也可以使用分号分割，但是已经**不推荐**，使用分号分隔会得到一个针对开发者的警告
		- 由于参数是用空格进行分割，如果参数名本身带有空格，要使用双引号括起来(显而易见)
- # 构建项目
	- 在shell中使用cmake构建项目构建系统时需要指定项目的源码目录
	- 也可以用选项``-B``指定build目录，若不给出，默认使用当前目录作为build目录，生成的各种临时文件也会直接存放于此，如果直接在源码目录下cmake，会扰乱当前目录的组织形式
	- 因此建议在构建一个项目时，要么在源目录下新建一个``build``目录，然后进入该目录中运行命令``cmake ..``，要么显示指定构建目录``cmake -S . -B build``
	- 在构建完成完成项目构建系统之后，可以直接运行对应的构建系统的指令进行实际的编译工作，也可以使用``cmake --build <build system directory>``进行项目编译，[[$red]]==**注意是经过cmake构建出来的项目构建系统目录，而不是源码目录**==如果新建了一个build目录来存放项目构建系统，则使用``cmake --build``时就应该指明``build``目录
- # 简单的工程目录结构
	- 一个工程项目的目录结构并没有定式，但是一般会遵守一些固有的习惯
	- 一般会有一个``src``目录用于存放源代码
	- 一般会有一个``doc``目录用于存放工程文档
	- 一般会有一个README文件对工程项目做简单的介绍
	- 可能会有一个``COPYRIGHT``文件介绍工程的知识产权信息
	- 可以添加一些``.sh``脚本用于调试，可以放在``scripts``目录下
	- 在这样的工程目录结构下，一般希望将构建后的目标文件放入构建目录(``build``)下的``bin``目录，将``doc``目录下的内容和COPYRIGHT，README一起安装到``/usr/share/doc/cmake``下
- # 一些简单技巧
	- ## 同时设置名称相同的动态库和静态库
		- 直接使用两个``ADD_LIBRARY``指令是无法同时创建同名的静态库和动态库的
		- 虽然可以通过更改其中一个库的名字，但是还是期望能够让静态库和动态库的名称相同，后缀名不同
		- 通过修改目标属性可以完成这项操作
			- 首先添加两个项目内名称不同的库
			- 将它们的输出名字设为相同
			- 将它们的``CLEAN_DIRECT_OUTPUT``属性设置为1
				- [[$red]]==**注意！**==在版本2.6之后，此属性已经被移除，然后此属性的相关行为一直开启
				- **即：**不再需要这一步，此步骤在2.6之后的版本可以直接被忽略
	- ## 设置版本号的关联
		- 一个动态库可以拥有一个build版本号和一个API版本号，分别使用``VERSION``和``SOVERSION``属性来设置
- # 基本命令
  collapsed:: true
	- ## PROJECT
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/project.html#command:project)
		- ```
		  project(<PROJECT-NAME> [<language-name>...])
		  project(<PROJECT-NAME>
		          [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
		          [DESCRIPTION <project-description-string>]
		          [HOMEPAGE_URL <url-string>]
		          [LANGUAGES <language-name>...])
		  ```
		- 可以用来指定工程项目的名称和支持的语言，若不指定支持的语言，则默认支持所有语言
		- **举例：**
			- ``PROJECT(HELLO)``，指定了工程名称为HELLO，且支持所有语言
			- ``PROJECT(HELLO CXX)``，指定了工程名称，且支持的语言仅有C++
			- ``PROJECT(HELLO C CXX)``，指定了工程名称，且支持C，C++
		- 此关键字还会隐式定义若干CMake变量
			- ``PROJECT_NAME``
				- 等于在project中给出的项目名称
			- ``CMAKE_PROJECT_NAME``
				- 在最顶层的``CMakeLists.txt``中的project会定义此变量
			- ``<PROJECT-NAME>_BINARY_DIR``和``PROJECT_BINARY_DIR``
				- 该变量的指明项目的编译文件目录，后续使用``add_subdirectory()``时该变量非常有用
				- 可能会随着``cmake``指令的option而改变
			- ``<PROJECT-NAME>_SOURCE_DIR``和``PROJECT_SOURCE_DIR``
				- 同上，但是指明的是源文件目录
				- 实际值就是当前``CMakeLists.txt``所在的目录
			- ``PROJECT_IS_TOP_LEVEL``和``<PROJECT-NAME>_IS_TOP_LEVEL``
				- 用于指明当前项目是否处于最顶层的布尔值
		- 以上两个变量名如果在``project()``中更改了项目名称，变量名也会跟着变，如果想使用不会随项目名称改变的变量，请使用``PROJECT_BINARY_DIR``等变量
			- 这些变量都是指向[[$red]]==**最近的**==``project()``指向的目录
	- ## SET
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/set.html)
		- ### 设定一般变量
			- ``set(<variable> <value>... [PARENT_SCOPE])``
			  id:: 63f5b9db-3c49-4a59-8ee7-de940034ad19
			- 在当前函数或目录作用域中设置变量的值
			- 如果给出PARENT_SCOPE，则此次对变量的修改，不在本作用域生效，而是在父作用域
				- 每一个函数和目录都会创建一个新的作用域
				- ``block()``命令也可以
				- 也就是说，如果给出此参数，设置的是父作用域中相应变量的值，在本作用域中的同名变量保持不变
		- ### 设定缓存条目(Cache Entry)
			- ``set(<variable> <value>... CACHE <type> <docstring> [FORCE])``
		- ### 设定环境变量
			- ``set(ENV{<variable>} [<value>])``
	- ## MESSAGE
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/message.html)
		- ### 一般信息
			- ``message([<mode>] "message text" ...)``
			- **mode**有如下种类
				- ``FATAL_ERROR``
					- CMake error，终止处理和生成
					- ``cmake``命令在执行到这样的message指令之后会返回一个非零的退出码
				- ``SEND_ERROR``
					- CMake error，继续处理但是终止生成
				- ``WARNING``
					- CMake warning，继续处理
				- ``AUTHOR_WARNING``
					- 开发人员给出的CMake warning，继续处理
				- ``DEPRECATION``
					- CMake Deprecation Error 或者 Warning，取决于变量``CMAKE_ERROR_DEPRECATED``和``CMAKE_WARN_DEPRECATED``是否启用
					- 如果都没启用则不发送消息
				- ``NOTICE``
					- 打印到**标准错误输出**的重要信息
					- 如果mode参数留空，则默认为NOTICE
				- ``STATUS``
					- 项目用户主要感兴趣的信息
					- 这样的信息应该精简，不超过一行但是信息丰富
					- 输出到**标准输出**
				- ``DEBUG``
					- 为此项目的开发者自身输出的详细信息
					- 用户对这些信息兴趣不大
					- 往往涉及到内部实现的细节
				- ``VERBOSE``
					- 为项目用户输出的详细信息
					- 这些信息应该提供一般情况下并不关心的额外细节，但这些信息对于那些想对项目建造有更深洞见的用户相当有用
				- ``TRACE``
					- 有着非常细粒度的底层级实现与细节的信息
					- 使用这种记录级别的信息应该是临时的，且在正式的发布时应该被省去
					- 输出到**标准输出**
		- ### 报告检查结果
			- ``message(<checkState> "message" ...)``
		- ### 配置记录
			- ``message(CONFIGURE_LOG <text>...)``
	- ## ADD_EXECUTABLE
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/add_executable.html)
		- ### 普通可执行文件
			- ```
			  add_executable(<name> [WIN32] [MACOSX_BUNDLE]
			                 [EXCLUDE_FROM_ALL]
			                 [source1] [source2 ...])
			  ```
			- 增添一个根据列出的源文件构造的名为``<name>``的可执行文件
			- ``<name>``必须在**整个项目中全局唯一**
			- 可执行文件的后缀由平台决定，可能是``.exe``也有可能没有后缀
			- 在**版本3.1**之后，源文件参数可以使用“生成器表达式(generator expression)”，语法类似于``$<...>``
			- 在**版本3.11**之后，源文件可以忽略，只要在后续通过``target_sources()``命令给出
			- 一般来说生成的可执行文件会被存放在build目录下和source tree对应的位置，可以参考``RUNTIME_OUTPUT_DIRECTORY``目标属性以修改生成位置
			- 参考``OUTPUT_NAME``目标属性以改变可执行文件最终的文件名
			- ``WIN32``参数如果给出，该目标会被赋予``WIN32_EXECUTABLE``的属性
			- ``MACOSX_BUNDLE``参数如果给出，该目标会被赋予对性的属性
			- 更多详情参见manual page
		- ### 引入的可执行文件(Imported Executable)
			- ``add_executable(<name> IMPORTED [GLOBAL])``
		- ### 别名可执行文件(Alias Executables)
			- ``add_executable(<name> ALIAS <target>)``
			- 在之后的命令中``<name>``可以作为``<target>``的别名使用，但是``<name>``可执行文件并不会出现在建造系统中
	- ## IF
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/if.html)
		- ```
		  if(<condition>)
		    <commands>
		  elseif(<condition>) *# optional block, can be repeated*
		    <commands>
		  else()              *# optional block*
		    <commands>
		  endif()
		  ```
		- 在if控制语句中，对变量的取值不需要加``${}``，直接使用变量名即可
	- ## ADD_SUBDIRECTORY
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/add_subdirectory.html)
		- ``add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])``
		- ``source_dir``指明``CMakeLists.txt``源文件和代码文件所在的目录
			- 如果是相对目录，则从当前目录开始寻址
		- ``binary_dir``指明``source_dir``目录下的文件被构建之后的输出目录
			- 如果该目录没有指明，则使用``source_dir``用作输出目录
		- 调用该指令之后会立即开始处理``source_dir``下的``CMakeLists.txt``文件，执行完毕之后再重新执行调用此命令的``CMakeLists.txt``文件
		- 如果``EXCLUDE_FROM_ALL``参数被给出，则子目录中的所有文件会从父目录中默认的``ALL`` target重被排除，IDE也会排除这些文件，用户必须进入该子目录手动build
			- 因此这一参数一般给那些和主项目关系不大的独立部分使用，例如测试用例
			- 被exclude的目录的``CMakeLists.txt``中往往会有自己的``project()``命令
			- 但是此选项会被跨target依赖(inter-target dependencies)覆盖，如果父项目的一个目标构造依赖于一个此子目录的目标，那么子目录中的被依赖目标会被包含进父项目的建造系统中
		- 如果``SYSTEM``参数被给出，那么此子目录的``SYSTEM``目录属性会被置为true
	- ## INSTALL
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/install.html)
		- ```
		  install(TARGETS <target>... [...])
		  install(IMPORTED_RUNTIME_ARTIFACTS <target>... [...])
		  install({FILES | PROGRAMS} <file>... [...])
		  install(DIRECTORY <dir>... [...])
		  install(SCRIPT <file> [...])
		  install(CODE <code> [...])
		  install(EXPORT <export-name> [...])
		  install(RUNTIME_DEPENDENCY_SET <set-name> [...])
		  ```
		- ``INSTALL``是一个高度复杂且有很多选项的指令
		- ``INSTALL``生成项目的安装规则
		- 在安装时，所有脚本中的``INSTALL``按顺序执行
		- 在**版本3.14**中，通过``add_subdirectory``添加的子目录下的安装指令会和父目录下的安装指令穿插执行以达到按照声明顺序执行的效果
			- 在之前的版本中，``add_subdirectory``中的安装指令会在父目录下的所有安装指令执行之后才执行
		- 在**版本3.22**中，通过环境变量``CMAKE_INSTALL_MODE1``可以改变``install``指令的默认行为
		- 如语法所示，此命令有很多的signature，不同的signature支持不同的**选项**
			- ``DESTINATION``
				- 指定文件将会被安装到的目录
				- 如果是相对路径，则相对变量``CMAKE_INSTALL_PREFIX``寻址
				- 由于``cpack``并不支持绝对路径，因此建议全程使用相对路径
			- ``PERMISSIONS``
			  collapsed:: true
				- 指定安装文件的权限
				- 包括
					- ``OWNER_READ``
					- ``OWNER_WRITE``
					- ``OWNER_EXECUTE``
					- ``GROUP_READ``
					- ``GOURP_WRITE``
					- ``GROUP_EXECUTE``
					- ``WORLD_READ``
					- ``WORLD_WRITE``
					- ``WORLD_EXECUTE``
					- ``SETUID``
					- ``SETGID``
				- 某些平台可能不支持某些权限，这些权限将会被忽略
			- ``CONFIGURATIONS``
			  collapsed:: true
				- 指定建造的配置
				- 指定配置之后只对``CONFIGURATION``选项之后的选项有用
					- 例如：
					- ```
					  install(TARGETS target
					          CONFIGURATIONS Debug
					          RUNTIME DESTINATION Debug/bin)
					  ```
					- ``CONFIGURATION``在``RUNTIME DESTINATION``之前
			- ``COMPONENT``
				- 指定此安装命令所关联的安装组件名称，例如Runtime或Development
				- 在component-specific的安装时，只执行和相关组件关联的安装命令
				- 在完整安装(full install)时，除了``EXCLUDE_FROM_ALL``以外的所有组件都将被安装
				- 若此选项不给出，则其组件名称默认为``Unspecified``，默认组件名称可以通过变量``CMAKE_INSTALL_DEFAULT_COMPONENT_NAME``更改
			- ``EXCLUDE_FROM_ALL``
				- 指明此文件在完整安装时不会被安装，只有在被指明时安装
			- ``RENAME``
				- 指明被安装的文件在安装时可能会被重命名，不与源文件的名称保持相同
				- 仅在对单个文件进行安装时有效
			- ``OPTIONAL``
				- 指明当此安装指令被执行时，待安装文件不存在不属于错误
		- ### 安装文件
		  collapsed:: true
			- ```
			  install(<FILES|PROGRAMS> files...
			          TYPE <type> | DESTINATION <dir>
			          [PERMISSIONS permissions...]
			          [CONFIGURATIONS [Debug|Release|...]]
			          [COMPONENT <component>]
			          [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
			  ```
			- 在``FILES``形式下，如果``PERMISSION``选项没有给出，则安装的文件会默认拥有OWNER_WRITE, OWNER_READ, GROUP_READ, WORLD_READ权限
			- ``PROGRAMS``形式类似，但是默认多了OWNER_EXECUTE, GROUP_EXECUTE, and WORLD_EXECUTE权限
				- 此形式适合安装非目标程序，例如shell脚本
				- 对于安装项目中生成的目标，应该使用``TARGET``
			- 文件列表可以使用生成器表达式，生成的路径必须是完成路径
			- ``TYPE``或``DESTINATION``选项必须出现一个，但不能两个都出现
			- ``TYPE``选项指明文件的通用类型
				- | TYPE Argument | GNUInstallDirs Variable | Built-In Default |
				  | ---- | ---- | ---- |
				  | BIN | ${CMAKE_INSTALL_BINDIR} | bin |
				  | SBIN | ${CMAKE_INSTALL_SBINDIR} | sbin |
				  | LIB | ${CMAKE_INSTALL_LIBDIR} | lib |
				  | INCLUDE | ${CMAKE_INSTALL_INCLUDEDIR} | include |
				  | SYSCONF | ${CMAKE_INSTALL_SYSCONFDIR} | etc |
				  | SHAREDSTATE | ${CMAKE_INSTALL_SHARESTATEDIR} | com |
				  | LOCALSTATE | ${CMAKE_INSTALL_LOCALSTATEDIR} | var |
				  | RUNSTATE | ${CMAKE_INSTALL_RUNSTATEDIR} | <LOCALSTATE dir>/run |
				  | DATA | ${CMAKE_INSTALL_DATADIR} | <DATAROOT dir> |
				  | INFO | ${CMAKE_INSTALL_INFODIR} | <DATAROOT dir>/info |
				  | LOCALE | ${CMAKE_INSTALL_LOCALEDIR} | <DATAROOT dir>/locale |
				  | MAN | ${CMAKE_INSTALL_MANDIR} | <DATAROOT dir>/man |
				  | DOC | ${CMAKE_INSTALL_DOCDIR} | <DATAROOT dir>/doc |
			- 对于要指明安装路径而不是类型的文件，建议使用``GNUInstallDirs``组件中的路径来宝成兼容性
				- 举例：
				- ```
				  include(GNUInstallDirs)
				  install(FILES logo.png
				          DESTINATION ${CMAKE_INSTALL_DOCDIR}/myproj
				  )
				  ```
		- ### 安装目录
		  collapsed:: true
			- ```
			  install(DIRECTORY dirs...
			          TYPE <type> | DESTINATION <dir>
			          [FILE_PERMISSIONS permissions...]
			          [DIRECTORY_PERMISSIONS permissions...]
			          [USE_SOURCE_PERMISSIONS] [OPTIONAL] [MESSAGE_NEVER]
			          [CONFIGURATIONS [Debug|Release|...]]
			          [COMPONENT <component>] [EXCLUDE_FROM_ALL]
			          [FILES_MATCHING]
			          [[PATTERN <pattern> | REGEX <regex>]
			           [EXCLUDE] [PERMISSIONS permissions...]] [...])
			  ```
			- 此形式将一个或多个目录安装到目标路径下
			- 源目标目录的结构将会原封不动地拷贝到目标目录
			- 每一个源目录的末尾如果**加上斜杠**则表示将源目录下的所有内容拷贝到目标目录下，否则是将源目录拷贝到目标目录
				- 如：``scripts/``表示将scripts下的所有内容拷贝到目标目录下，而``scripts``则是将scripts目录直接搬过去
			- ``USE_SOURCE_PERMISSION``选项如果给出，则连带权限一并拷贝
			- | TYPE Argument | GNUInstallDirs Variable | Built-In Default |
			  | ---- | ---- | ---- |
			  | BIN | ${CMAKE_INSTALL_BINDIR} | bin |
			  | SBIN | ${CMAKE_INSTALL_SBINDIR} | sbin |
			  | LIB | ${CMAKE_INSTALL_LIBDIR} | lib |
			  | INCLUDE | ${CMAKE_INSTALL_INCLUDEDIR} | include |
			  | SYSCONF | ${CMAKE_INSTALL_SYSCONFDIR} | etc |
			  | SHAREDSTATE | ${CMAKE_INSTALL_SHARESTATEDIR} | com |
			  | LOCALSTATE | ${CMAKE_INSTALL_LOCALSTATEDIR} | var |
			  | RUNSTATE | ${CMAKE_INSTALL_RUNSTATEDIR} | <LOCALSTATE dir>/run |
			  | DATA | ${CMAKE_INSTALL_DATADIR} | <DATAROOT dir> |
			  | INFO | ${CMAKE_INSTALL_INFODIR} | <DATAROOT dir>/info |
			  | LOCALE | ${CMAKE_INSTALL_LOCALEDIR} | <DATAROOT dir>/locale |
			  | MAN | ${CMAKE_INSTALL_MANDIR} | <DATAROOT dir>/man |
			  | DOC | ${CMAKE_INSTALL_DOCDIR} | <DATAROOT dir>/doc |
			-
	- ## ADD_LIBRARY
	  collapsed:: true
		- [Manual Page](https://cmake.org/cmake/help/latest/command/add_library.html)
		- ### 普通库
			- ```
			  add_library(<name> [STATIC | SHARED | MODULE]
			              [EXCLUDE_FROM_ALL]
			              [<source>...])
			  ```
			- 目标库名``<name>``必须在整个项目中全局唯一
			- 实际构建出的库的扩展名由原生平台的惯例决定(.a或.lib)
			- ``STATIC``表示构建静态库
			- ``SHARED``表示构建编译时**动态连接**，运行时加载的动态库
			- ``MODULE``表示构建编译时**不连接**，运行时通过类似于dlopen的函数进行加载和运行的动态库
			- 如果以上三个选项一个都不给出，则根据变量``BUILD_SHARED_LIBS``变量的值是否为``ON``确定创建``SAHRE``或``STATIC``库
			- 对于``SAHRE``和``MODULE``库，目标``POSITION_INDEPENDENT_CODE``的属性被自动设置为``ON``
			- ``SHARED``库可以被标记上``FRAMEWORK``目标属性来构建一个macos框架
			- 在**版本3.8**之后，``STATIC``也可以被标记上``FRAMEWORK``属性以构建静态框架
			-
	- ## SET_TARGET_PROPERTIES
	  collapsed:: true
		- ```
		  set_target_properties(target1 target2 ...
		                        PROPERTIES prop1 value1
		                        prop2 value2 ...)
		  ```
		- 目标可以拥有一些影响自身建造过程和结果的属性
		- 简单列出所有想设置属性的目标和所有属性键值对即可
		- 后续可以通过``get_property()``和``get_target_property()``获取设置的属性