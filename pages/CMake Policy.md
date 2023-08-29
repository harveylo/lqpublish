- CMake的policy主要用于保证backward compatibility
- 新版本的CMake会在新的policy引入之后警告backward compatible behavior
- 每个policy都有一个编号，形式为``CMP<NNNN>``
- 其保证backward compatibility的运作机制为：
	- 新版本的CMake如果改变了某些行为，那么就会引入新的policy
		- 若这些policy被设为``NEW``，那么采用新的行为
		- 若设为``OLD``则采用旧版本行为
	- 在使用``cmake_minimum_required``命令指定最低版本时，所有该版本既之前引入的policy都会被置为``NEW``
	- 除此之外所有没有显式指明为``NEW``还是``OLD``的policy会被设为``OLD``，并且产生一个warning
- 除了下列两种情况之外，一个policy**[[$red]]==绝不能被设置为==``OLD``**
	- 在编译一个stable活frozen的项目时用于silence warning
	- temporarily as part of a larger migration path
- 所有policy的``OLD``行为都是undesired的，而且会在未来的release中被移除
- # ``cmake_policy``
	- 此指令用于显式指定相应policy的值
	- ## 通过CMake版本设置policy的值
		- [[$blue]]==**推荐使用版本号设置policy而非显式地单独设置**==
		- ``cmake_policy(VERSION <min>[...<max>])``
		- `min`和`max`的值都应该是类似于``major.minor[.path[.tweak]]``的结构
		- ``[...<max>]``中的`...`是字面量，必须给出
		- 可选的``max``版本号在3.12版本中引入
		- 将给出的版本范围内和之前的所有新策略设为`NEW`，比给出版本范围更高的策略将会设为``OLD``
			- 除非``CMAKE_POLICY_DEFAULT_CMP<NNNN>``被设置了一个默认值
		- 这会使得更新的版本去假造旧版本CMake代码写的项目时警告其所引入的新policy
		- ``cmake_minimum_required``实际上是隐式调用了一次``cmake_polikcy``
		- ### deprecation 警告
			- 3.19 版本中，如果调用``cmake_required_minimum``或``cmake_policy``设置的版本号低于`2.8.12`，则会发出一个deprecation 警告
			- 3.27版本中，设置版本号低于``3.5``会产生deprecation 警告
	- ## 单独显式设置policy
		- ``cmake_policy(SET CMP<NNNN> NEW | OLD)``
	- ## 检查Policy的值
		- ``cmake_policy(GET CMP<NNNN> <variable>)``
		- 如果policy被设置过，将`OLD`或`NEW`存入变量``variable``中
		- 否则为空
	- ## CMake policy stack
		- 为了保护父节点和兄弟节点的policy设置，CMake维护一个CMake policy stack
		- 所有的policy设置仅仅对栈顶的entry生效，每当进入子目录或调用``include``和``find_package``时都会有新的entry被压入栈，除非调用时启用了``NO_POLICY_SCOPE``选项
		- 如果想手动管理policy stack，CMake提供两个指令
			- ``cmake_policy(PUSH)``
			- ``cmake_policyu(POP)``
		- 每一个``PUSH``都必须有一个对应的``POP``来消除其增添的影响
		- 这两个指令可以用于创建临时的policy设置，但是在3.25版本中，新引入的``block()``和``endblock()``更加方便
			- block()会往policy stack中自动压入一个entry
			- endblock()会自动弹出一个entry
			- 避免了在每一个`return`之前都要手动``cmake_policy(POP)``
		- ```cmake
		  # stack management with cmake_policy()
		  function(my_func)
		    cmake_policy(PUSH)
		    cmake_policy(SET ...)
		    if (<cond1>)
		      ...
		      cmake_policy(POP)
		      return()
		    elseif(<cond2>)
		      ...
		      cmake_policy(POP)
		      return()
		    endif()
		    ...
		    cmake_policy(POP)
		  endfunction()
		  
		  # stack management with block()/endblock()
		  function(my_func)
		    block(SCOPE_FOR POLICIES)
		      cmake_policy(SET ...)
		      if (<cond1>)
		        ...
		        return()
		      elseif(<cond2>)
		        ...
		        return()
		      endif()
		      ...
		    endblock()
		  endfunction()
		  ```
- # 常用cmake policy
	-
	-