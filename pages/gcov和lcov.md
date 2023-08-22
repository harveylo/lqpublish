# gcov
	- gcov用于生成代码的覆盖度报告
	- 在使用时需要和编译器配合使用
	- 在编译时需要加上选项
		- ``-ftest-coverage -fprofile-arcs``
		- 最好再加上一个``--coverage``
		- 为了避免优化导致的误差，建议再加上``-g3 -O0``
			- 开启``g3``会致使gcc编译时提供额外的debug信息，因此可能某些debugger无法正常工作
			- 对于level 3级来说，会额外提供和宏相关的信息，使某些debugger支持宏展开
			- 默认等级为2
			- level 0不会有任何debug信息输出，``-g0``相当于取消了``-g``
	- 如果是分步编译(先获取目标文件再链接)，那么在链接时可能需要加上如下参数之一
		- [[$red]]==**没碰到过，因此也不知道实际情况如何，需要补充实验**==
		- ``--coverage ``
		- ``-lcov``
		- ``-fprofile-arcs``
	- 编译结束之后会生成`gcno`文件，此文件包含重建基本块图并将源代码行标和块对应起来的信息
	- 编译生成可执行文件之后，直接运行可执行文件即可生成相关报告，格式为`gcda`
	- ## 补充
		- 在项目中实际接入并生成测试度覆盖报告时一般不需要手动实行``gcov``，`lcov`在整合``gcov``的文件信息时，如果发现没有``gcda``文件会自动调用``gcov``生成
		- [[$red]]==**注意，编译所使用的gcc版本必须和gcov版本对应**==
			- 意味着，如果使用的是**`clang`**编译，那么很有可能无法使用gcc提供的`gcov`
			- 解决方法是，安装``llvm``使用``llvm``提供的类似于`gcov`功能的``llvm-cov gcov``
			- 如果确定了项目会使用``clang``，那么建议生成一个sh脚本，然后将``/usr/bin``下的``gcov``或``gcov-<version>``软连接链到这个脚本
				- ```shell
				  #!/bin/bash
				  llvm-cov gcov $@
				  ```
- # lcov
	- 从``gcov``的数据中生成整体的覆盖度测试报告
	- 需要把所有的**源码文件和``gcno``文件所在的目录**使用``-d``包括进来
		- 不然在加上``--no-external``选项之后，``lcov``看不到源文件或是``.gcno``文件都会导致最终的生成报告中没有相关文件
		- 不加上`--no--external`选项则会导致把所有文件甚至库函数(stl)都包括进来，导致报告庞杂且影响整体覆盖率
	- 最新的(至少在2013年已是如此)`lcov`已经默认不再生成分支覆盖度报告，如果需要这部分信息，需要加上选项``--rc lcov_branch_coverage=1``
	- ``lcov`` 最终会生成`.info`文件，此文件已经包含了所有所需信息，但是阅读仍然困难，因此需要使用``genhtml``生成html格式的图形化覆盖度报告
		- 此工具``lcov``自带
		- 此工具生成的html覆盖度报告默认也不带分支覆盖率，如果需要，加上``--rc genhtml_branch_coverage=1``选项
- # CMake中自定义命令一键生成覆盖度报告示例
	- ```cmake
	  SET(TEST_P ${CMAKE_BINARY_DIR}/test)
	  SET(ROOT_P ${CMAKE_SOURCE_DIR})
	  
	  add_custom_target(cov_report
	      COMMAND ctest
	      COMMAND echo "=================== LCOV ===================="
	      COMMAND lcov --capture --d ${ROOT_P}/src 
	      -d ${ROOT_P}/include -d ${TEST_P} -t test 
	      -b ${TEST_P} -o ${TEST_P}/cover.info 
	      --rc lcov_branch_coverage=1 --no-external
	      COMMAND echo "-- Generating HTML output files"
	      COMMAND genhtml ${TEST_P}/cover.info -o ${TEST_P}/report --rc genhtml_branch_coverage=1
	  )
	  ```
	- 只需要在``build``目录下使用``make cov_report``即可生成覆盖度报告