- 上下文格式会展示不同处附近若干行的上下文
- 若要选择此格式作为``diff``输出格式，可使用``--context[=lines]``或``-C lines``或简单的``-c``选项
	- 参数lines用于指定到底**要展示多少行的context**
	- 若不指定，则默认展示3行上下文
- ``patch``需要至少**两行**上下文以支持正确的操作
- # 一个上下文格式的例子
	- ```
	  *** lao	2002-02-21 23:30:39.942229878 -0800
	  --- tzu	2002-02-21 23:30:50.442260588 -0800
	  ***************
	  *** 1,7 ****
	  - The Way that can be told of is not the eternal Way;
	  - The name that can be named is not the eternal name.
	    The Nameless is the origin of Heaven and Earth;
	  ! The Named is the mother of all things.
	    Therefore let there always be non-being,
	      so we may see their subtlety,
	    And let there always be being,
	  --- 1,6 ----
	    The Nameless is the origin of Heaven and Earth;
	  ! The named is the mother of all things.
	  ! 
	    Therefore let there always be non-being,
	      so we may see their subtlety,
	    And let there always be being,
	  ***************
	  *** 9,11 ****
	  --- 8,13 ----
	    The two are the same,
	    But after they are produced,
	      they have different names.
	  + They both may be called deep and profound.
	  + Deeper and more profound,
	  + The door of all subtleties!
	  ```
	- 以上输出是命令``diff -c lao tzu``的输出
	- ```
	  *** lao	2002-02-21 23:30:39.942229878 -0800
	  --- tzu	2002-02-21 23:30:50.442260588 -0800
	  ***************
	  *** 1,5 ****
	  - The Way that can be told of is not the eternal Way;
	  - The name that can be named is not the eternal name.
	    The Nameless is the origin of Heaven and Earth;
	  ! The Named is the mother of all things.
	    Therefore let there always be non-being,
	  --- 1,4 ----
	    The Nameless is the origin of Heaven and Earth;
	  ! The named is the mother of all things.
	  ! 
	    Therefore let there always be non-being,
	  ***************
	  *** 11 ****
	  --- 10,13 ----
	      they have different names.
	  + They both may be called deep and profound.
	  + Deeper and more profound,
	  + The door of all subtleties!
	  ```
	- 以上输出是命令``diff -C 1 lao tzu``的输出，**展现了更少的上下文**
- # 上下文格式的详细描述
	- ## 输出开头：
		- ```
		  *** from-file from-file修改时间
		  --- to-file to-file修改时间
		  ```
		- 输出开头可以通过选项``--label=label``修改
			- 使用两个`--label`选项，第一个给出的代表form-file的title，第二个给出的是to-file的title
	- ## hunks
		- 紧接着开头的是若干表示两个文件区别的文本块，每一个文本块的内容包含：
		- ```
		  ***************
		  *** from-file-行号 ****
		    from-file-line
		    from-file-line…
		  --- to-file-行号 ----
		    to-file-line
		    to-file-line…
		  ```
		- 如果一个hunk包含两行及以上内容，行号的格式为``起始行, 结束行``，否则只有结束行
		- 每一行前的字符说明其区别类别
			- `!`，表示该行有内容被改变，如果一个文件中的若干行被`!`标记，另一个文件的hunk中也会有对应的`!`标记行
			- `+`，表示此行是第二个文件相较于第一个文件被插入的行
			- `-`，表示此行是第一个文件相较于第二个文件需要删除的行
		- 如果每一个hunk中所有的改动都是插入，则来自from-file的行会被忽略
		- 如果一个hunk中所有的改动都是删除，则来自to-file的行会被忽略