- [知乎问题：如何用C语言实现异常/状况处理机制](https://www.zhihu.com/question/20597909)
- 实际上C更建议的错误处理方式是返回**[[$red]]==错误码==**。
	- 例如在很多C库函数中，一个函数如果会出现错误，那么要么其返回值可以代表成功或错误类型，要么通过一个传入一个指针，对该指针设置错误码，要么在库函数的全局空间中引入了一个全局错误码，如果出现错误可以通过该错误码调用某些函数获得实际错误信息。
	- 这么做的好处是可以及时获取错误，一边做出调整
	- **[[$red]]==坏处==**则是在需要进行错误处理的代码中，编程会变得非常繁琐，因为对于每一个可能出错的函数调用，都必须及时获取它们可能的错误码，并作出反应，否则很容易错过错误。
- 而C++直接引入了**try-catch**，可以比较直观的处理异常。
	- 那些可能出现错误的函数，不再需要精心设计如何返回它们的错误码，而是可以直接抛出一个异常，让上一层的调用者自己去处理
	- 在出现重大错误时，这样可以让正常流程立即中断，控制逻辑必须处理好异常才能继续后续事项，减少了错误被忽略的概率
	- 但是，虽然C++会对那些进入了try block中被分配了内存的变量或对象进行析构(这一过程称作stack-unwiding)，但是仍然有批评指出，C++由于不具备真正的GC，这样的异常处理还是可能会导致异常泄露。
	- [异常与错误码](https://techsingular.net/2012/11/15/programming-in-lua%ef%bc%88%e4%ba%8c%ef%bc%89%ef%bc%8d-%e5%bc%82%e5%b8%b8%e4%b8%8e%e9%94%99%e8%af%af%e7%a0%81/)
- 但是对于那些**不是由函数调用引起的错误**，例如除0错误，或硬中断或软中断，或更直白一点，程序运行过程中收到的信号，就不可能通过错误码去处理。这种情况一般依赖于使用`signal`或`sigaction`函数注册信号处理函数。如果希望在信号处理完成之后跳转到特定的程序位置去继续执行，则需要配合`setjump`和`longjump`函数来实现。
	- 使用`setjump`和`longjump`的问题就在于，C完全不会做stack unwiding，在如果longjump跳过了一些释放资源的代码，例如`fclose`，`free`等，那这些资源就会泄露。
- # 实例：用注册信号处理函数和setjmp处理除零异常
	- ```C
	  #include <stdio.h>
	  #include <fenv.h>
	  #include <setjmp.h>
	  #include <signal.h>
	  
	  jmp_buf buf;
	  
	  void handle_divide_zero(int sig) {
	    signal(SIGFPE, handle_divide_zero);
	    longjmp(buf, 1);
	  }
	  
	  int main(int argc, char *argv[]) {
	    signal(SIGFPE, handle_divide_zero);
	    int status = setjmp(buf);
	    if (status == 0) {
	      feclearexcept(FE_ALL_EXCEPT);
	      unsigned long long result = 1 / 0;
	      printf("%llu", result);
	      return 0;
	    }
	    printf("-1\n");
	    return -1;
	  }
	  ```