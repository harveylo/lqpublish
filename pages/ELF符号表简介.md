- 本质是一个符号表项数组
- 每一个符号表项本质上是一个Elf64_Symbol结构体
- ```
  typedef struct { 
      int   name;      /* String table offset */ 
      char  type:4,    /* Function or data (4 bits) */ 
      binding:4;       /* Local or global (4 bits) */ 
      char  reserved;  /* Unused */  
      short section;   /* Section header index */
      long  value;     /* Section offset or absolute address */ 
      long  size;      /* Object size in bytes */ 
  } Elf64_Symbol;
  ```
	- 该结构体定义在系统头文件``elf.h``中
- 有三类**伪节**是没有节头表条目的
	- **ABS**
		- 不需要重定位符号，比如源代码的路径名
	- **COMMON**
		- 未初始化全局变量
		- .bss保存的是未初始化的静态变量，初始化为0的全局变量和静态变量
	- **UND**
		- 在目标文件中引用，但定义在别的文件中的符号
- 注意：name成员并不是一个字符串，而是在strtab中的偏移量