- 补充：[[Base64编码]]
- # 使用telnet发送电子邮件
	- telnet是一个用于在两个程序之间建立网络上的可靠字节流的工具，且可以指定协议
	- 先使用telnet向支持smtp服务的电子邮件服务器发送连接请求，已qq邮箱为例：
		- ``telnet smtp.qq.com smtp``
	- 然后向服务器发送问候信息``helo``
		- ``helo qq.com``
		- 问候信息的内容随意
	- 使用``auth login``进行登录操作
		- 服务器会发送``344 VXNlcm5hbWU6``，是username的base64码
		- 在看到上述信息之后发送自己账户的base64码
		- 服务器会继续发送``334 UGFzc3dvcmQ6``，要求输入密码
		- 发送自己授权码的base64码完成登录
	- 接下来可以尽心消息发送
		- ```
		  mail from: <xxxxxxxxx@qq.com>
		  rcpt to: <**********@qq.com>
		  
		  DATA
		  From: xxxxxxxxx@xx.xxx
		  To: xxxxxxxxxx@xx.xxx
		  Subject: xxxxxxxxxx
		  Mail Content
		  ...................
		  ```
		- 正文内容已单独一行的``.``结束
	- 输入``QUIT``关闭连接
-