- # PROJECT
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
- # SET
  collapsed:: true
	- [Manual Page](https://cmake.org/cmake/help/latest/command/set.html)
	- ## 设定一般变量
		- ``set(<variable> <value>... [PARENT_SCOPE])``
		- 在当前函数或目录作用域中设置变量的值
		- 如果给出PARENT_SCOPE，则此次对变量的修改，不在本作用域生效，而是在父作用域
			- 每一个函数和目录都会创建一个新的作用域
			- ``block()``命令也可以
			- 也就是说，如果给出此参数，设置的是父作用域中相应变量的值，在本作用域中的同名变量保持不变
	- ## 设定缓存条目(Cache Entry)
		- ``set(<variable> <value>... CACHE <type> <docstring> [FORCE])``
	- ## 设定环境变量
		- ``set(ENV{<variable>} [<value>])``
- # MESSAGE
  collapsed:: true
	- [Manual Page](https://cmake.org/cmake/help/latest/command/message.html)
	- ## 一般信息
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
	- ## 报告检查结果
		- ``message(<checkState> "message" ...)``
	- ## 配置记录
		- ``message(CONFIGURE_LOG <text>...)``
- # ADD_EXECUTABLE
	- [Manual Page](https://cmake.org/cmake/help/latest/command/add_executable.html)
	- ## 普通可执行文件
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
	- ## 引入的可执行文件(Imported Executable)
		- ``add_executable(<name> IMPORTED [GLOBAL])``
	- ## 别名可执行文件(Alias Executables)
		- ``add_executable(<name> ALIAS <target>)``
		- 在之后的命令中``<name>``可以作为``<target>``的别名使用，但是``<name>``可执行文件并不会出现在建造系统中
- # IF
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
- # ADD_SUBDIRECTORY
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
- # INSTALL
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
	- ## 安装文件
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
	- ## 安装目录
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
	- ## 安装目标
		- ```
		  install(TARGETS targets... [EXPORT <export-name>]
		          [RUNTIME_DEPENDENCIES args...|RUNTIME_DEPENDENCY_SET <set-name>]
		          [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
		            PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE|FILE_SET <set-name>|CXX_MODULES_BMI]
		           [DESTINATION <dir>]
		           [PERMISSIONS permissions...]
		           [CONFIGURATIONS [Debug|Release|...]]
		           [COMPONENT <component>]
		           [NAMELINK_COMPONENT <component>]
		           [OPTIONAL] [EXCLUDE_FROM_ALL]
		           [NAMELINK_ONLY|NAMELINK_SKIP]
		          ] [...]
		          [INCLUDES DESTINATION [<dir> ...]]
		          )
		  ```
		- 可一次性安装多个target，并给每一个target依次指定其种类和目标路径之类的属性
		- 有多种目标**Output Artifacts**可以安装
			- ``ARCHIVE``，包括：
				- 静态库(除了在macos上被标记为``FRAMEWORK``的库)
				- DLL import 库 (.lib库，.dll库应该使用``RUNTIME``安装)
				- 在AIX上，连接器输入文件(linker import file)，需要开启``ENABLE_EXPORTS``
			- ``LIBRARY``，包括
				- 共享库，除了：
					- DLL库，因该使用``RUNTIME``
					- 在macos上被标记为``FRAMEWORK``的库
			- ``RUNTIME``，包括
				- 可执行文件，除了macos上被标记为``MACOS_BUNDLE``的可执行文件，应该使用``BUNDLE``
				- DLL库，.dll文件，注意和.lib的DLL import库分开
- # ADD_LIBRARY
  collapsed:: true
	- [Manual Page](https://cmake.org/cmake/help/latest/command/add_library.html)
	- ## 普通库
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
- # SET_TARGET_PROPERTIES
  collapsed:: true
	- ```
	  set_target_properties(target1 target2 ...
	                        PROPERTIES prop1 value1
	                        prop2 value2 ...)
	  ```
	- 目标可以拥有一些影响自身建造过程和结果的属性
	- 简单列出所有想设置属性的目标和所有属性键值对即可
	- 后续可以通过``get_property()``和``get_target_property()``获取设置的属性
- # CMAKE_MINIMUM_REQUIRED
  collapsed:: true
	- 指定构造该项目所需的最低cmake版本
	- ``cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])``
	- 此命令会将变量``CMAKE_MINIMUM_REQUIRED_VERSION``设置为``<min>``的值
	- 可选参数``FATAL_ERROR``会被2.6以上的版本忽略，但是2.4以下版本如果不指定该参数，则未达到此最低要求版本只会给出一个警告
- # INCLUDE_DIRECTORIES
  collapsed:: true
	- 增添头文件目录，以便编译器搜寻头文件时使用
	- ``include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])``
	- 将目录添加到目录和目标的``IINCLUDE_DIRECTORIES``属性中
	- 可选项``AFTER``和``BEFORE``指明是将目录附到头文件目录列表的头部还是尾部
	- 可选项``SYSTEM``告诉编译器此头文件目录是系统头文件目录
- # TARGET_LINK_DIRECTORIES
  collapsed:: true
	- [Manual Page](https://cmake.org/cmake/help/latest/command/target_link_directories.html)
	- ```
	  target_link_directories(<target> [BEFORE]
	    <INTERFACE|PUBLIC|PRIVATE> [items1...]
	    [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
	  ```
	- 给相应的目标制定链接时搜索库的目录
	- 给出的目标必须已经在之前的``add_executable``或``add_library``命令定义
	- 如果可选项``BEFORE``给出，库目录会被附加到库列表头而非尾部
- # TARGET_LINK_LIBRARIES
  collapsed:: true
	- [Manual Page](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)
	- ``target_link_libraries(<target> ... <item>... ...)``
	- 连接的目标必须是一个之前定义过的目标，且不能是别名目标
	- 每一个``item``必须是下列之一：
		- **一个库目标的名称**
			- 项目中的一个库的名称，必须在之前的脚本中被``ADD_LIBRARY()``定义
			- 也可以是一个引入的库
		- **一个库文件的绝对路径**
		- **一个简单的库名称**
		- **一个链接flag**
			- 以``-``开头的item会被作为一个linker flag对待
			- 注意它们也有依赖传递，因此如果使用``PRIVATE``是比较保险的
		- **一个生成器表达式**
	- 每一个``item``之前可以跟一个``debug``，``optimized``，``general``关键字
		- 只有在相关configuration建造下才会使用这些库
	- ## 给目标添加库
		- ```
		  target_link_libraries(<target>
		                        <PRIVATE|PUBLIC|INTERFACE> <item>...
		                       [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
		  ```
		- ``PUBLIC``, ``PRIVATE``, ``INTERFACE``是作用域关键字，用于指明链接依赖类型
		- ### PRIVATE
		  collapsed:: true
			- 若A使用B进行其功能实现，但是不在其提供的公共API中出现，则这样的依赖关系是**PRIVATE**的
			- 任何对于A中代码的调用都不需要对B中的任何东西进行直接引用
			- **举例**
				- 假设A是一个在各种SSL库(B)上实现的网络库
				- A提供给客户统一的接口，但这些接口并不直接引用(形参或返回类型([[$red]]==个人理解==))任何内部的SSL数据结构或函数
				- 那么客户代码将对于A使用的实际的SSL实现(B)完全不知情，也不需要关心
		- ### INTERFACE
		  collapsed:: true
			- 如果A并不使用B完成自己的实现，但是B中的某些东西出现在了A的公共API中， 则称A对于B的依赖是**INTERFACE**
			- 对于A的某些调用可能需要引用B中的一些东西
			- **举例**
				- 假设A只是一个简单地转发一个调用进入库B，且在转发的过程中，其接口可能并不实际引用B中的一些类型和数据结构，而是单纯地将其视作一个指针或引用(因此在A的实际代码中没有出现B中的任何元素，但是如果有人想调用A的这个接口，则必须对B的实际实现和数据结构有一定了解才能完成调用)
				- 另一个例子是A是在CMake中定义的接口库，其本身没有任何实际实现，只作为其他库的一个集合
		- ### PUBLIC
		  collapsed:: true
			- 实质上是``PRIVATE``和``INTERFACE``的集合
			- A既在实际实现中使用了B，也在公共接口中使用了B
		- 一个浅显但不一定完全正确的例子：
		  collapsed:: true
			- 如果一个库的源文件中include了一个其他库的头文件，但是在他自己的头文件中并没有include这个头文件，那么这个属于``PRIVATE``
			- 如果一个库的所有源文件都没有include某个其他库的头文件，但是在自己的头文件中include了一个其他库的头文件，这属于``INTERFACE``
			- 如果一个库在自己的头文件和源文件中都include了一个其他库的头文件，则属于``PUBLIC``
		- 如果不给出依赖类型，那么链接依赖会**默认传递**，即若另一个目标需要链接这个目标，这该目标的所有链接的库都会出现在另一个目标的链接列表里
- # FILE
	- ```
	  Reading
	    file(READ <filename> <out-var> [...])
	    file(STRINGS <filename> <out-var> [...])
	    file(<HASH> <filename> <out-var>)
	    file(TIMESTAMP <filename> <out-var> [...])
	    file(GET_RUNTIME_DEPENDENCIES [...])
	  
	  Writing
	    file({WRITE | APPEND} <filename> <content>...)
	    file({TOUCH | TOUCH_NOCREATE} [<file>...])
	    file(GENERATE OUTPUT <output-file> [...])
	    file(CONFIGURE OUTPUT <output-file> CONTENT <content> [...])
	  
	  Filesystem
	    file({GLOB | GLOB_RECURSE} <out-var> [...] [<globbing-expr>...])
	    file(MAKE_DIRECTORY [<dir>...])
	    file({REMOVE | REMOVE_RECURSE } [<files>...])
	    file(RENAME <oldname> <newname> [...])
	    file(COPY_FILE <oldname> <newname> [...])
	    file({COPY | INSTALL} <file>... DESTINATION <dir> [...])
	    file(SIZE <filename> <out-var>)
	    file(READ_SYMLINK <linkname> <out-var>)
	    file(CREATE_LINK <original> <linkname> [...])
	    file(CHMOD <files>... <directories>... PERMISSIONS <permissions>... [...])
	    file(CHMOD_RECURSE <files>... <directories>... PERMISSIONS <permissions>... [...])
	  
	  Path Conversion
	    file(REAL_PATH <path> <out-var> [BASE_DIRECTORY <dir>] [EXPAND_TILDE])
	    file(RELATIVE_PATH <out-var> <directory> <file>)
	    file({TO_CMAKE_PATH | TO_NATIVE_PATH} <path> <out-var>)
	  
	  Transfer
	    file(DOWNLOAD <url> [<file>] [...])
	    file(UPLOAD <file> <url> [...])
	  
	  Locking
	    file(LOCK <path> [...])
	  
	  Archiving
	    file(ARCHIVE_CREATE OUTPUT <archive> PATHS <paths>... [...])
	    file(ARCHIVE_EXTRACT INPUT <archive> [...])
	  ```
	- ## 文件系统(Filesystem)
		- ```
		  file(GLOB <variable>
		       [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
		       [<globbing-expressions>...])
		  file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
		       [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
		       [<globbing-expressions>...])
		  ```
		- 生成和``globbing-expression``匹配的文件列表，并将值储存在``variable``中
			- ``globbing-expression``类似与正则表达式，不过简单得多
			- google一下glob
		- ``RELATIVE <path>``，如果给出， ``globbing-expression``则会作为相对于``<path>``的相对路径返回
		- **在3.6以后**：结果会以字典序返回
		- 在windows和macos下，如果所使用的文件系统是大小写敏感的，则globbing也是大小写敏感的
			- 在其他平台下，globbing是大小写敏感的
		- **在3.3以后**：默认情况下``GLOB``会列出目录
			- 如果``LIST_DIRECTORIES``设会false，则会忽略目录
		- **在3.12以后**：如果给出了``CONFIGURE_DEPENDS``标识，CMake会在主建造系统中增添检查