- # Printf and Logging
  collapsed:: true
	- 使用log较之使用printf的优势：
		- 可以将log输出定向到任何地方，套接字，文件甚至是远程服务器而不是单单只能输出到标准输出
		- log支持严重性分级，便于过滤输出
		- 对于新的issue，log有更大的机会包含足以检测到错误的的信息
	- ## Coloring
		- 增加log的可读性的一大举措便是改变输出的颜色
		- 在shell中，用 **ANSI escape codes**对颜色进行转义
			- **[[$red]]==使用CSI格式==**
			- **SYNOPSIS**
				- **256色**
					- 设置文字颜色（foreground）形如：``\e[38;2;<r>;<g>;<b>m <TEXT_CONTENT> \e[0m``
					- 设置背景颜色（background）形如：``\e[48;2;<r>;<g>;<b>m <TEXT_CONTENT> \e[0m```
					- 可以组合起来同时设置文字颜色和背景颜色，形如：``\e[38;2;<r>;<g>;<b>;48;2;<r>;<g>;<b>m <TEXT_CONTENT> \e[0m``
					- 还可以结合其他的一些转移代码，设置更多的文字效果，比如：
						- 背景白色，文字红色的缓慢闪烁粗体文字：
						- ``echo -e "\e[1;5;38;2;255;0;0;48;2;255;255;255mThis is red\e[0m"``
						- echo 的-e参数表示对echo的内容进行转义
				- **16色**
					- 30-37设置文字颜色，90-97设置明亮文字颜色
					- 40-47设置背景颜色，100-107设置明亮背景颜色
					- 也可以结合其他转移代码设置文字效果
					- 更见方便，支持的平台更多
			- [[$blue]]==**具体详见**==： [ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code)
	- ## 第三方log工具
		- 在每一个程序当中都自己手动写logger并不现实
		- 大多数程序都有自己的log输出，在linux下，一般是放在``/var/log``
		- 目前的趋势是使用系统日志，将所有的日志都输出到一个地方
		- 在linux下，**systemd**是一个掌管很多事务的系统守护进程，它将日志置于``/var/log/journal``
			- 该日志使用特殊的格式，使用**journalctl**指令显示其储存的消息
		- 在大多数UNIX系统下，使用dmesg访问kernel日志
		- ### 向系统日志中输出内容
			- 使用logger命令
			- 一个hello log的例子：
			- ```
			  logger "Hello Logs"
			  journalctl --since "1m ago" | grep Hello
			  ```
		- 大型的日志系统的输出往往杂乱无章，需要进行data wrangling
		- 可以使用lnav这样的工具帮助进行阅读日志
- # Debugging with debuggers
  collapsed:: true
	- 当printf不足以有效debug时，考虑借助debugger
	- ## pdb
		- python自带的一个调试工具
		- 使用``python -m pdb {{FILE_NAME}}``进入调试环境
		- **l**(ist) - Displays 11 lines around the current line or continue the previous listing.
		- **s**(tep) - Execute the current line, stop at the first possible occasion.
		- **n**(ext) - Continue execution until the next line in the current function is reached or it returns.
		- **b**(reak) - Set a breakpoint (depending on the argument provided). 可以接一个行号作为参数
		- **p**(rint) - Evaluate the expression in the current context and print its value. There’s also **pp** to display using [ `pprint` ](https://docs.python.org/3/library/pprint.html) instead. 同时可以配合python的locals()函数打印当前所有本地变量的值
		- **r**(eturn) - Continue execution until the current function returns.
		- **q**(uit) - Quit the debugger.
	- ## GDB
		- gdb可以对任何可执行二进制文件进行debug
		- ### 启动
			- gdb **[**options**]** **[**executable_file **[**core_file | process_id**]]**
			- 添加参数 -q，-quiet 或 --silent 可以去掉gdb开始执行时的一些打印信息
			- #### 命令行参数
				- gdb一般用来调试三类程序
					- **可执行程序**
						- 直接指定可执行文件进行debug
					- **正在运行的进程**
						- 在程序名称后面附加参数指定要调试的进程id或者指定core dump file的名称
						- ``gdb myprog 1001``
						- 指示让gdb连接到系统一个正在运行的名为myprog，pid为1001的进程
					- **core dump file**
						- ``gdb myprog <coreID>``
			- ### 命令行选项
			  collapsed:: true
				- 某些前跟一个连字符的选项也可以跟两个连字符，但是有的选项只能跟两个连字符比如--silent
				- #### 传递参数到被调试程序
					- ``--args``
					- 在该命令行选项之后的所有的参数只能是被调试程序和被调试程序的参数
					- 也可以在进入gdb调试环境之后利用``set args``和``run``来传递参数
				- #### 选择文件
					- 通过命令行选项指定使用文件
					- ```
					  -symbols filename, -s filename
					    单独加载一个符号表
					  -exec filename, -e filename
					    指定被调试的可执行文件
					  -se filename
					    指定包含了符号表的可执行文件，GDB的命令行参数中出现文件名，
					    这个文件名默认就是-se的参数
					  -core filename, -c filename, -c number, -pid number, -p number
					    -core 和 -pid 是等效的。用于指定进程名或核心转储文件
					  ```
					- [[$red]]==**什么是符号表？**==
						- 符号表是用于保存所有标识符（identifier）属性的一种数据结构
						- 其储存了每一个标识符的数据类型，作用域和内存地址
					- [[$red]]==**什么是核心转存？**==
						- **Core Dump**是将内存中的某一个程序在某个具体时间的执行情况记录下来，通常是从内存中转存到磁盘上
				- #### 选择用户界面
					- 可以指定其他的的终端负责被调试程序的输入输出
					- ```
					  -tty device, -t device
					    使用device作为被调试程序等标准输入输出流。例如 "gdb myprog -t /dev/tty5"
					  -windows, -w
					    使用GDB内置的GUI。若gdb没有集成GUI，该选项无效。
					  -nowindows, -nw
					    不使用GUI调试
					  -tui
					    启动文本式调试界面。
					  ```
				- #### 执行命令脚本
					- 一个命令脚本是包含若干gdb命令的文本，类似于shell脚本
					- 空白行和使用#注释的行都会被忽略
					- ```
					  -command command_file, -x command_file
					    指示GDB在启动时就执行文件中的命令
					  -batch
					    执行命令脚本后退出
					  ```
		- ### 初始化文件
		  collapsed:: true
			- 在启动时，gdb通常会加载处理一个名为``.gdbinit``的初始化文件
			- gdb会在当前目录和``$HOME``下寻找该初始化文件
				- ``$HOME``下的初始化文件优先级高于当前目录的初始化文件
			- 使用``-nx, -n``忽略所有gdb初始化文件
		- ### 使用gdb命令
			- GDB执行的每一个命令都是一个以命令关键字开始的文本行
			- 可以**[[$red]]==截断==**任何关键字，只要保证键入的关键字长度足够清楚将其与其他关键字区别
				- 例如，可以输出q(或qu、qui)来表示quit命令
			- 输入一个空行，那么GDB会重复执行上一条命令，只要该动作看似合理
				- 例如，GDB会自动重复step和next命令，但不会自动重复run命令
			- 键入help或h命令，可以查看命令类别列表
			- #### 暂停正在执行的程序
				- 在gdb调试环境下，当程序正在执行（如连接到了一个正在运行的进程），使用``<C-V>``发送暂停信号，开始调试
			- #### gdb的状态和信息
			  collapsed:: true
				- GDB提供 `info` 和 `show` 命令 ，查看被调试程序的状态信息和调试器的状态信息。
				- **被调试程序状态**
					- ```
					  (gdb) info all-reg
					    显示所有处理器寄存器的内容，包括浮点和向量寄存器。
					  (gdb) info source
					    显示当前的源文件
					  (gdb) info address radius
					    显示radius变量的地址信息
					  ```
				- **gdb状态信息**
					- ```
					  (gdb) show logging
					    显示GDB当前日志的相关信息
					  (gdb) set logging on
					    将GDB的日志重定向到gdb.txt文件
					  ```
			- #### 在GDB中执行一个程序
			  collapsed:: true
				- 使用``run``在gdb环境下执行选择的程序
				- 使用``start``开始debug执行（实际上是在方便的地方设置了一个断点，通常是程序入口）
				- 使用``starti``并在第一条指令下暂停，可以添加参数
				- ```
				  (gdb) file filename
				    如果gdb在启动时没有指定被调试程序，可以通过file指定被调试的程序。
				  (gdb) set args [arguments]
				    重新指定被调试程序的命令行参数。
				  (gdb) show args
				    显示被调试程序的命令行参数。
				  (gdb) run [arguments]
				    开始执行一个程序，并可选的为其指定命令行参数。
				  (gdb) kill
				    终止被调试的程序（不会销毁当前设置的参数）
				  ```
			- #### 显示源代码
				- 使用``list``命令在调试器中显示一个程序的源代码
				- 默认情况下gdb一次展示十行代码
				- ```
				  (gdb) list filename:line_number
				    显示源代码，并以指定的行作为中心
				  (gdb) line_number
				    跳转到指定的行(为中心)
				  (gdb) from,[to]
				    显示特定范围内的源代码
				  (gdb) function_name
				    把指定函数开始的第一行作为中心，显示源代码
				  (gdb) list, l
				    显示更多的行(遇到断点中止)
				  (gdb) set listsize number
				    设置list命令默认输出行数
				  (gdb) listsize
				    显示list命令默认输出的行数
				  ```
				- [[$red]]==**注意！**== 如果想在调试时正确显示源代码和后文中的本地变量，可执行文件必须要有符号表，或者说作为一个ELF文件必须要有一系列的.debug section。
				- 要做到这一点，在使用gcc编译时，[[$blue]]==**要加上-g**==，表示进行debug编译，编译完成的可执行文件会带有符号表
			- #### 使用断点
				- **设置和显示断点**
					- ```
					  (gdb) break [filename:] line_number
					    在（指定文件或当前文件）指定行设置断点
					  (gdb) break function
					    在指定函数的第一行设置断点
					  (gdb) break
					    在当前栈帧的下一行设置断点
					  (gdb) info breakpoints
					    显示断点信息，包括断点编号、断点位置等
					  (gdb) tbreak 16
					    设置临时断点
					  ```
				- **删除，禁用和忽略断点**
					- ```
					  (gdb) delete [bp_number | range], d [bp_number | range]
					    删除指定编号或指定范围的断点，不带参数会删除所有断点。例如"delete 1-3"
					  (gdb) disable [bp_number | range]
					    禁用断点
					  (gdb) enable [bp_number | range]
					    启用断点
					  (gdb) ignore bp_number iterations
					    忽略断点，并指定忽略次数为iterations
					  ```
				- **条件式中断**
					- ```
					  (gdb) break [position] if expression
					    position 可以是一个函数名或者一个行号，expression可以是C语言的任何表达式，
					    表达式的返回值必须是标量
					  (gdb) condition bp_number [expression]
					    更改已有端点的条件。若不带expression，则表示删除断点的条件
					  ```
				- **在中断后继续执行**
					- ```
					  (gdb) continue [passes], c [passes]
					    继续执行到下一个断点，passes表示忽略几次中断
					  (gdb) step [lines], s [lines]
					    执行多少行后再次被中断，如果遇到函数，将会进入函数，并在函数第一行停下来。
					  (gdb) next [lines], n [lines]
					    执行多少行后再次被中断，不会进入函数。
					  (gdb) finish
					    将当前函数执行结束，控制权返回给函数调用者。
					  ```
				-
			- #### 分析栈
			  collapsed:: true
				- 调用栈(call stack)是一种内存组织方式
				- 每一次函数调用都会压入一个栈帧
				- 栈帧记录着各种相关信息，如调用者地址和寄存器值，函数参数和局部变量等
				- ```
				  (gdb) backtrace, bt, where, info stack
				    多个命令都可以显示程序的调用轨迹
				  (gdb) frame [number]
				    显示当前栈帧，或者选择不同的栈帧，如果编译时加入了debug参数，使用frame能看到程序运行
				    处于源代码中的哪一行
				  (gdb) info frame
				    显示当前栈帧的相关信息
				  (gdb) info locals
				    当前栈帧的局部变量
				  (gdb) info args
				    列出对应函数调用的参数值
				  ```
			- #### 显示数据
			  collapsed:: true
				- 使用**print**命令显示变量和表达式的值``(gdb) print [/format] [expression]``
				- 随便使用不恰当的expression可能会造成意想不到的后果，如
					- ```
					  (gdb) p r
					  $1 = 1
					  (gdb) p r=7
					  $2 = 7
					  (gdb) p r*r
					  $3 = 49
					  ```
				- 可以使用p和set命令，在GDB中定义新的变量，变量名必须以 `$` 符号开头，例如 `set $var=*ptr` ，打印gdb中定义的变量值也要添加$
				- 访问其他栈帧的局部变量，应该把函数名和两个冒号(::)放在变量名之前，例如 `p main::radius` 。
				- **输出格式**
					- 在print命令中，可选参数 `/format` 由一条斜线与一个字母组成，它指定了输出格式
					- 如：``print /x``会以十六进制整数的形式显示前面的值
					- 格式参数：
						- ```
						  /d     # 十进制
						  /u     # 无符号十进制
						  /x     # 十六进制
						  /o     # 八进制
						  /t     # 二进制
						  /c     # 字符
						  /a     # 以十六进制显示地址
						  /f     # 把表达式值的位模式翻译成浮点数
						  ```
				- **显示内存区域**
					- 使用 `x` 命令可以检查未命名的内存区域
					- 该命令的参数包括该内存区域的开始地址和空间大小，以及一个可选的输出格式
					- ``x [/nfu] [address]``
					- 参数 `/nfu` 由三部分组成:
						- **n**： 一个十进制数，指定要显示多少个内存单位
						- **f**：格式修饰符，s表示以字符串显示，i表示以汇编语言方式显示，也可以使print命令中的格式选项
						- **u**：内存的单位，b表示字节，h表示两个字节，w(默认)表示四个字节，g表示八个字节
				- **观测点**
					- 通过设置观测点(watchpoint)监视针对变量的读写操作
					- 类似于断点，但并没有绑定到某一行代码，而是绑定到一块内存空间
					- 对某一变量设置观测点之后，每当该变量的值改变，程序便会暂停
					- ```
					  (gdb) watch expression
					    当 expression 的值改变时，调试器暂停程序的执行
					  (gdb) rwatch expression
					    当程序读取与expression相关的任何对象时，调试器暂停程序的执行
					  (gdb) awatch expression
					    当程序读取或修改与expression相关的任何对象时，调试器暂停程序的执行
					  ```
					- [[$red]]==**针对某局部变量设置观测点之前，必须先执行程序，直到程序流入该变量作用域内**==
					- 当程序离开一个语句块时，调试器会[[$red]]==**自动删除**==包含有[[$red]]==**不在作用域内局部变量**==的表达式的所有观测点
		- ### 分析core dump file
			- 核心转储文件是一个包含进程所使用的内存镜像的文件
			- 当一个程序异常终止时，Unix系统通常会将core文件写到工作目录下
			- 在命令行中将core文件的名称传递到GDB，可以[[$blue]]==**研究进程结束时的状态**==
			- 使用gdb分析core dump与一般调试会话类似，不过由于其并不是一个真正的可执行程序，且已经停止在某一个时间点，因此[[$red]]==**run、step、next、continue等命令无效**==
			- 格式类似：``$ gdb myprog core``
			- 分析core文件，一般通过 `bt` 命令查看调用栈，使用 `frame`  命令查看当前栈帧信息，使用 `print` 命令查看当前变量的值。
		- ### 关于gdb插件pwndbg
			- pwndbg是一款非常强大的gdb插件，可以实时打印处理器级别的信息和源代码（需要gcc编译时添加相关参数）
			- 可以在gdb中使用pi指令来运行gdb集成的python console，在此处可以运行一些pwndbg的命令，比如pwndbg.memory.read(addr, len), pwndbg.memory.write(addr, data), pwndbg.vmmap.get()等
			-
- # Specialized Tools
  collapsed:: true
	- 当程序需要完成一些只有linux kernel才能做到的动作时，程序会通过system call来进行
	- linux下通过**[[$blue]]==strace==**来跟踪系统调用
	- 当通过strace来进行跟踪系统调用又希望忽略被跟踪程序的输出时，可以把输出重定向到``/dev/null``，即null device
		- null device会丢弃掉所有写入的内容，但是汇报写入成功
	- ## 在网络编程中
		- 使用tcpdump和wireshark等工具可以获取，过滤和分析netword packet
		- 许多浏览器自带调试工具，其功能包括：
			- 检视网页的html，css，js[[$blue]]==**源码**==
			- [[$blue]]==**实时修改**==html，css，js文件：可以很方便地改变王爷的内容和行为用于测试
			- **[[$blue]]==js shel==l**：可以在此执行一些js指令
			- 分析请求时间线
			- 查找cookies和应用的本地[[$blue]]==**储存情况**==
- # 静态分析
	- linux下**python**的静态分析工具：pyflakes, mypy
	- **shell**的静态分析工具: shellcheck
	- vim的语法检查插件：ale
	- python的stylistic linter：pylint和pep8
	- bandit：能找到普遍的security issu
	- formatter：
		- black for python
		- gofmt for go
		- rustfmt for rust
		- prettier for javaScript
- # Profiling
	- ## 计时
		- 正常运行的程序可能性能并不够好
		- profiling可以帮助找到找到程序中的热点代码
		- 衡量一个程序执行时间的三种标准：
			- [[$blue]]==**real time**==：一个程序从开始执行到实际结束给出结果的时间称为 real time，也称wall clock time，然而这并不是该程序实际的cpu执行时间，因为其他程序所占用的cpu时间片也会算在其中
			- [[$blue]]==**user time**==：该程序在user mode下在cpu上的实际执行时间
			- [[$blue]]==**sys time**==：该程序在kernel mode下的执行时间
		- user + sys time才是一个程序在cpu上真正消耗的时间
		- 在linux下，在指令前加上**[[$blue]]==time==**可以看到相应的时间统计
	- ## Profiler
		- ### CPU
		  collapsed:: true
			- 大多数情况下，profiler都是指的 cpu profiler
			- cpu profiler分为两个类型：
				- **tracing**：跟踪程序的每一个函数调用
					- 开销较大，较少使用
				- **sampling**：周期性探测程序的执行情况（一般是一毫秒一次），记录该程序的stack
					- 开销较少，且一般来说一秒200次的采样已经足够分析出热点代码
			- #### python
				- 可以使用cProfile 来profile 每一个函数调用的时间
				- 一个调用的实例如下：
					- python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py
					- 但是这样的profile所得到是每一个函数调用的耗时，非常的不直观，很多时候我们是急切地想知道每一行代码的耗时，这种情况下我们便需要*line profiler*的帮助
					- 例如，使用kernprof profile一个发送http请求的python程序可以得到以下结果：
						- ```
						  $ kernprof -l -v a.py
						  Wrote profile results to urls.py.lprof
						  Timer unit: 1e-06 s
						  - Total time: 0.636188 s
						  File: a.py
						  Function: get_urls at line 5
						  - Line *#  Hits         Time  Per Hit   % Time  Line Contents*
						  **==============================================================**
						   5                                           @profile
						   6                                           def get_urls**()**:
						   7         1     613909.0 613909.0     96.5      response **=** requests.get**(**'https://missing.csail.mit.edu'**)**
						   8         1      21559.0  21559.0      3.4      s **=** BeautifulSoup**(**response.content, 'lxml'**)**
						   9         1          2.0      2.0      0.0      urls **=** **[]**
						  10        25        685.0     27.4      0.1      **for **url **in **s.find_all**(**'a'**)**:
						  11        24         33.0      1.4      0.0          urls.append**(**url['href'**])**
						  ```
		- ### 内存
			- **valgrind**工具可以用于发现程序c程序中的内存泄露
			- 带有gc的语言仍然可能需要用到内存profile，一个使用memory profileryunxingpython程序的示例如下：
				- ```
				  $ python -m memory_profiler example.py
				  Line *#    Mem usage  Increment   Line Contents*
				  **==============================================**
				       3                           @profile
				       4      5.97 MB    0.00 MB   def my_func**()**:
				       5     13.61 MB    7.64 MB       a **=** **[**1] ***** **(**10 ** 6**)**
				       6    166.20 MB  152.59 MB       b **=** **[**2] ***** **(**2 ***** 10 ** 7**)**
				       7     13.61 MB -152.59 MB       del b
				       8     13.61 MB    0.00 MB       **return **a
				  ```
		-
-