- 合并格式是上下文格式的变种，相较于上下文格式更加简洁，其忽略了冗余的上下文行
- 使用选项``--unified[=lines](-U lines)``或简单的``-u``选择此输出格式
	- 参数lines指定上下文的行数
	- 如果不给出默认是3行
- # 合并格式的输出例子
	- ```
	  --- lao	2002-02-21 23:30:39.942229878 -0800
	  +++ tzu	2002-02-21 23:30:50.442260588 -0800
	  @@ -1,7 +1,6 @@
	  -The Way that can be told of is not the eternal Way;
	  -The name that can be named is not the eternal name.
	   The Nameless is the origin of Heaven and Earth;
	  -The Named is the mother of all things.
	  +The named is the mother of all things.
	  +
	   Therefore let there always be non-being,
	     so we may see their subtlety,
	   And let there always be being,
	  @@ -9,3 +8,6 @@
	   The two are the same,
	   But after they are produced,
	     they have different names.
	  +They both may be called deep and profound.
	  +Deeper and more profound,
	  +The door of all subtleties!
	  ```
	- 以上是命令``diff -u lao tzu``的输出
- # 格式具体描述
	- ## 输出开头
		- 和上下文格式的开头保持一致
	- ## hunks
		- 开头之后就是若干个代表区别内容的hunks
		- ```
		  @@ from-file-line-numbers to-file-line-numbers @@
		   line-from-either-file
		   line-from-either-file…
		  ```
		- 行号的格式为`开始行号，结束行号`
			- 如果hunk只包含一行，则只给出开始行号
		- 行号都会有一个字符打头，
			- `+`表示一行新的内容被插入到了第一个文件
			- `-`表示从第一个文件中删去一行