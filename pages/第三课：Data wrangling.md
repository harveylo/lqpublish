- **将某些数据从一种格式转换到另一种格式叫做data wrangling**
- # Pager
	- 一类用于让输入数据更加便于阅览的数据处理程序
	- **less**
		- linux下自带的一种pager，将输入数据分页，用d和u键翻页，同时还带有搜索功能
- # Regex（regular expression）
	- 非常好用的匹配机制
	- ^：表示开头
	- $：表示结尾
	- 括号：
		- ()：表示组
			- 可以用来将复杂的结构括起来以便形成更复杂的结构
			- 也可以用于提取希望获取的内容
		- []：表示匹配字符的范围
			- 可用连接符号-表示多少到多少，仅能用于字母和数字之间
		- {}：表示匹配长度[0-9]{9}表示长度为9的数字，[0-9]{0,9}表示长度为0到9的数字
	- |：或
	- ?：出现一次或0次
	-
- # Stream Editor
	- ## sed
		- 's/{pattern}/{target}/g'
			- 表示将所有匹配{pattern}的字符串替换为{target}。若去掉g，则制作一次匹配替换，即将第一个匹配的字符串替换
		- -E：使用扩展的正则表达式语法