- bind用于将一个函数绑定到函数对象上，未指定的参数可以使用std::placeholders::_n占位
- 例：
	- ```
	  void funct(int a, int &b){
	  	b=4;
	  	std::cout<<a*b<<std::endl;
	  }
	  class TestClass{
	  private:
	  	int x;
	  public:
	  	void function(){
	  		std::cout<<"Calling a member function in: "<<x<<std::endl;
	  	}
	  	TestClass(int x_):x(x_){}
	  };
	  int main(){
	  	TestClass tc(3);
	  	int i=0;
	  	auto f3 = std::bind(&funct,std::placeholders::_1,std::placeholders::_2);
	  	auto f4 = std::bind(&funct,2,i);
	  	auto f5 = std::bind(funct,2,std::ref(i));
	  	auto f6 = std::bind(&TestClass::function,&tc);
	  	f4();
	  	std::cout<<"after f4: "<<i<<std::endl;
	  	i=0;
	  	f5();
	  	std::cout<<"after f5: "<<i<<std::endl;
	  	i=0;
	  	f3(4,std::ref(i));
	  	std::cout<<"after f3: "<<i<<std::endl;
	  	f6();
	      return 0;
	  }
	  ```
- 如果bind的时候直接将变量bind过去，哪怕函数本身的形参声明是引用，变量也**不会作为引用被传递**，而是被拷贝(f4)。
- 如果想在bind时就作为引用传递，则需要使用std::ref包裹变量名(f5)
- 如果使用了std::placeholders::_n，占位，那么后续调用的时候直接传入变量名也能作为引用传参(f3)
- 如果在bind时需要传递常量引用(即形参为const type&)，则使用std::cref包裹
- 绑定类成员函数则必须要在函数名前加上``&``，因为对于类成员函数取地址本来就需要加&(f6)，绑定时还必须传入一个类对象指针，函数的实际参数从bind的第三个参数开始