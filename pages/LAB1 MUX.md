- # 建模分类
	- 对于硬件的描述，可以分为三种类型：**数据流建模**，**结构化建模**，**行为建模**
	- 以二选一mux为例，对三种建模进行阐述
	- ## 数据流建模
		- 主要通过连续赋值语句``assign``来描述电路功能，专注于每一个输出口与输入的逻辑联系
		- 对于一个二选一mux，使用数据流建模如下：
		- ```
		  module bit2mux(a,b,s,y);
		    input   a,b,s;        // 声明3个wire型输入变量a,b,和s，其宽度为1位。
		    output  y;           // 声明1个wire型输出变量y，其宽度为1位。
		  
		    assign  y = (~s&a)|(s&b);  // 实现电路的逻辑功能。
		  
		  endmodule
		  ```
	- ## 结构化建模
		- 结构化建模主要通过逐层实例化子模块来描述电路的功能，更加接近于门级别的建模
		- ```
		  module my_and(a,b,c);
		    input  a,b;
		    output c;
		  
		    assign c = a & b;
		  endmodule
		  
		  module my_or(a,b,c);
		    input  a,b;
		    output c;
		  
		    assign c = a | b;
		  endmodule
		  
		  module my_not(a,b);
		    input  a;
		    output b;
		  
		    assign b = ~a;
		  endmodule
		  
		  module mux21b(a,b,s,y);
		    input  a,b,s;
		    output y;
		  
		    wire l, r, s_n; // 内部网线声明
		    my_not i1(.a(s), .b(s_n));        // 实例化非门，实现~s
		    my_and i2(.a(s_n), .b(a), .c(l)); // 实例化与门，实现(~s&a)
		    my_and i3(.a(s),   .b(b), .c(r)); // 实例化与门，实现(s&b)
		    my_or  i4(.a(l),   .b(r), .c(y)); // 实例化或门，实现(~s&a)|(s&b)
		  endmodule
		  ```
	- ## 行为建模
		- 通过类似于面向过程的编程语言来描述电路行为，也是对于输出口的逻辑的一种描述，不过相比数据流建模更见便于理解
		- 代码友好度较高
		- ```
		  module mux21c(a,b,s,y);
		    input   a,b,s;
		    output reg  y;   // y在always块中被赋值，一定要声明为reg型的变量
		  
		    always @ (*)
		      if(s==0)
		        y = a;
		      else
		        y = b;
		  endmodule
		  ```
	- ## 补充：Verilog相关
		- ### verilog 赋值顺序
			- 在Verilog中，各语句是并发执行的
				- 模块中所有的assign语句、always语句块和实例化语句，其执行顺序不分先后
			- 而if语句是顺序执行的语句
				- 其执行过程中必须先判断if后的条件，如果满足条件则执行if后的语句，否则执行else后的语句
			- [[$red]]==**顺序执行的语句必须包含中always块中**==，always块中的语句按照它们中代码中出现的顺序执行。
				- 所以在always块中，需要使用**非阻塞赋值**显式表示赋值不分先后顺序
		- ### always
			- always如果不带任何敏感事件列表信息，只有always这个保留字，则表示在任何情况下都执行always语句块中的语句
				- 一般用在仿真中，需要接一个延迟以生成时钟信号
			- 一个reg类型的变量**只能**在一个always中被赋值
				- 原因显而易见
		- ### localparam
			- 相当于定义一个本地可用的宏，增加代码可读性，减小复杂性
			- 使用定义的本地宏不需要加`` ` ``
		- ### generate
			- 用于生成结构相同但参数不同的verilog代码
			- 三种结构：for，if，case
			- **for**
				- 需要定义一个genvar作为一个for循环判断变量
		- ### 参数化模块
			- [参见](https://www.realdigital.org/doc/43c79714a7f3d0bbb8098d60c63fde48)
			- 形如：
				- ```
				  module an_example #(parameter PAR_A, PAR_B, PAR_C)
				  (
				  	input port_in,
				  	output port_out
				  );
				  	//module body
				  endmodule
				  ```
				- 括号中的parameter可以去除
			- 每一个参数都需要给出默认值
			- 参数的值也可以再模块实例化的时候给出
				- ```
				  A_module #( .A_PARAMETER(a_constant_value), .ANOTHER_PARAMETER(7) ) an_instance_name
				  (
				  	.a_port(a_wire),
				  	.another_port(another_wire_or_constant)
				  )
				  ```
				- 也可以直接按顺序给值，不用指定参数名称
	- ## 建议：新手不要使用行为建模
		- 强烈建议初学者远离行为建模方式，仅通过数据流建模和结构化建模方式直接描述电路
		- 需要先培养时间队列模型的思维方式