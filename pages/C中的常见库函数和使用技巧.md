# stdin.h
	- ## popen, pclose
		- 可以打开将一个指令当作文件打开，且可以通过`fprintf`与`fscnaf`向其中写数据或读数据
		- `pclose`会返回指令的状态码(Status code)和一些其他的信息，例如是否是被信号终止等。状态码只有八位，被放置在0xff00这八位上，如果要从pclose的返回值中获取状态码，则可以使用`WEXITSTATUS`宏，如果想知道其他信息还可以使用`WIFSIGNALED`等其他宏。
		- 如果在main函数中`return -1`，那么正常运行结束的进程在`pclose`中会返回多少？答案是65280，因为在类unix系统中状态码最多只有8位，返回的-1被转换为255，255再被置于0xff00位上了，就变成了658280。
			- 因此并不建议在main函数中return 0-255以外的值。
		- ### 使用popen,pclose的实例
			- ```C
			  int main(int argc, char *argv[]) {
			      FILE* fp = popen("./a.out", "r");
			      unsigned long long result;
			      fscanf(fp, "llu",&result);
			      int status = pclose(fp);
			      printf("%d\n", status);
			      return 0;
			  }
			  ```