- # 杂项
	- ## 编译器优化
		- ``#pragma GCC optimize(2)``，O2优化
		- ``#pragma GCC optimize(3,"ofast","inline")``，O3优化
		- [[$red]]==**注意**==：O2和O3优化都会改变代码原有的执行顺序，给调试带来困难
			- 且开启高等级优化之后可能会导致Runtime Error，可能是负数数组下标导致
	- ##  万能头文件
		- ``#include <bits/stdc++.h>``
- # 快读
	- ## int
		- id:: 646de3ae-9028-43be-8d35-23f46c36b8b6
		  ```
		  inline int intRead() // 内联函数稍微快一点点
		  {
		      char ch = getchar();
		      int s = 0, w = 1;
		      while (ch < '0' || ch > '9')
		      {
		          if (ch == '-')
		              w = -1;
		          ch = getchar();
		      }
		      while (ch >= '0' && ch <= '9')
		      {
		          s = s * 10 + ch - '0', ch = getchar();
		      }
		      return s * w;
		  }
		  ```
	- ## long
		- ```
		  inline long long longRead()
		  {
		      char ch = getchar();
		      long long s = 0LL, w = 1LL;
		      while (ch < '0' || ch > '9')
		      {
		          if (ch == '-')
		              w = -1;
		          ch = getchar();
		      }
		      while (ch >= '0' && ch <= '9')
		      {
		          s = s * 10LL + ch - '0', ch = getchar();
		      }
		      return s * w;
		  }
		  ```
	- ## string
		- ```
		  inline string stringRead()
		  {
		      string str;
		      char s = getchar(); // 处理多余回车或空格
		      while (s == ' ' || s == '\n' || s == '\r')
		      {
		          s = getchar();
		      } // 不断读入直到遇到回车或空格
		      while (s != ' ' && s != '\n' && s != '\r')
		      {
		          str += s;
		          s = getchar();
		      }
		      return str;
		  }
		  ```
	- ## double
		- [[$red]]==**数据过大时可能会出错**==
		- ```
		  inline double doubleRead()
		  {
		      // double的值可能很大，所以开long long
		      long long s = 0, w = 1, k = 0, n = 0, m = 0;
		      char ch = getchar(); // 和int一毛一样有木有
		      while (ch < '0' || ch > '9')
		      {
		          if (ch == '-')
		              w = -1;
		          ch = getchar();
		      }
		      while ((ch >= '0' && ch <= '9') || ch == '.')
		      {
		          // n = 0代表读入整数，= 1代表读入小数
		          if (ch == '.')
		              n = 1;
		          else if (n == 0)
		              s = s * 10 + ch - '0';
		          else
		              k = k * 10 + ch - '0', m++;
		          ch = getchar();
		      }
		      return (pow(0.1, m) * k + s) * w;
		  }
		  ```
-