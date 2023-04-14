- 用于在一个类中声明**友元函数**和**友元类**
- # 友元函数
	- 表示此函数可以访问类的private成员
	- 可以是一个全局函数，也可以是其他类的函数**[[$red]]==(但不能是其他类的私有函数)==**
	- 友元函数**并不是这个类的成员函数**
		- 因此在实际定义的时候，不能使用``Class::function``的作用域指明方式来定义函数，而是因该直接定义函数
	- ```
	  #include<iostream>
	  using namespace std;
	  class CCar;  //提前声明CCar类，以便后面的CDriver类使用
	  class CDriver
	  {
	  public:
	      void ModifyCar(CCar* pCar);  //改装汽车
	  };
	  class CCar
	  {
	  private:
	      int price;
	      friend int MostExpensiveCar(CCar cars[], int total);  //声明友元
	      friend void CDriver::ModifyCar(CCar* pCar);  //声明友元
	  };
	  void CDriver::ModifyCar(CCar* pCar)
	  {
	      pCar->price += 1000;  //汽车改装后价值增加
	  }
	  int MostExpensiveCar(CCar cars[], int total)  //求最贵气车的价格
	  {
	      int tmpMax = -1;
	      for (int i = 0; i<total; ++i)
	          if (cars[i].price > tmpMax)
	              tmpMax = cars[i].price;
	      return tmpMax;
	  }
	  int main()
	  {
	      return 0;
	  }
	  ```
- # 友元类
	- 被指定为友元的类可以访问此类的所有私有成员
	- ```
	  class CCar
	  {
	  private:
	      int price;
	      friend class CDriver;  //声明 CDriver 为友元类
	  };
	  class CDriver
	  {
	  public:
	      CCar myCar;
	      void ModifyCar()  //改装汽车
	      {
	          myCar.price += 1000;  //因CDriver是CCar的友元类，故此处可以访问其私有成员
	      }
	  };
	  int main()
	  {
	      return 0;
	  }
	  ```