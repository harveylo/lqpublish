- [[在WSL中安装OpenGL开发环境]]
- # 简单理解OpenGL
	- 一种图形API标准(Specification)，实际的实现仰赖于各家显卡厂商所提供的驱动
	- ## Core-Profile和Immediate Mode
		- OpenGL的两种编码模式，或者又称风格
		- 最初opengl仅提供Immediate Mode供开发者使用
		- ### Immediate Mode
			- 使用简便，但是缺乏Flexibility
			- 抽象了，但过于抽象，使用immediate mode编程将缺乏对于opengl实际行为的具体理解
		- ### Core-Profile
			- **现代**opengl所采用的方式
			- 移除了Immediate Mode中的一些deprecated函数
		- 本书采用opengl-3.3的core-profile模式作为案例
	- ## 扩展
		- 不同的显卡厂商可能会在opengl的官方标准意外，实现一些额外的扩展函数
		- 这些扩展在使用前最好实现判断是否可用，例如：
			- ```glsl
			  if(GL_ARB_extension_name)
			  {
			      // Do cool new and modern stuff supported by hardware
			  }
			  else
			  {
			      // Extension not supported: do it the old way
			  }
			  ```
	- ## 状态机
		- 使用opengl开发类似于在和一个巨型的状态机打交道
		- 在opengl中，这样的状态被称作**[[$red]]==Context==**
			- context定义了当前opengl应该如何运作
		- 当要改变opengl的行为方式的时候，可以通过**改变一些context中的变量**的方式来更换状态，如此一来当下一个绘制命令来到时，opengl就会更改行为模式
		- 在实际使用opengl进行开发时，常会用到一些**State-Changing**函数和**State-Using**函数
			- 前者更改context，后者根据不同状态执行一些不同的操作
	- ## 对象
		- 大多数OpenGL库函数都由C实现，opengl代码有很浓重的C语言风格
		- 但是为了让编码更加简易，opengl实现了一些便捷的抽象，例如**对象(Object)**
		- opengl中的一个对象是一组**可选项（options）**的集合。
			- 表示opengl中的一个子状态
			- 例如可以用一个对象来表示绘制一个窗口时的一些设置，包括大小和支持的颜色等
		- 对象定义风格类似于C语言的结构体
			- ```glsl
			  struct object_name {
			      float  option1;
			      int    option2;
			      char[] name;
			  };
			  ```
		- 对象的使用则类似于：
			- ```glsl
			  // The State of OpenGL
			  struct OpenGL_Context {
			    	...
			    	object_name* object_Window_Target;
			    	...  	
			  };
			  ```
				- 这里表示了一个对象是Context的一个子状态
			- ```glsl
			  // create object
			  unsigned int objectId = 0;
			  glGenObject(1, &objectId);
			  // bind/assign object to context
			  glBindObject(GL_WINDOW_TARGET, objectId);
			  // set options of object currently bound to GL_WINDOW_TARGET
			  glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH,  800);
			  glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
			  // set context target back to default
			  glBindObject(GL_WINDOW_TARGET, 0);
			  ```
- # GLAD
	- glad可以看作一个显卡驱动和图形API之间的中间层
	- opengl api并不是由操作系统提供而是各家显卡驱动厂商实现，因此需要有个抽象层来方便地获取每一个硬件平台的opengl api函数地址
	- 其调用就是将``glfwGetProcAddress``传给`gladLoadGLLoder`，前者会通过后者取出若干函数保存到事先定义好的函数签名变量中，后续调用函数时不再需要通过前者根据函数名来获取函数地址，而是直接使用预先定义好的一些同名函数宏，这些宏会展开为预先定义好的函数指针
- # 创建一个窗口
	- ## 初始化GLFW
		- glfw在使用前需要被初始化，使用函数``glfwInit``来完成
		- ### 给出向新创建的window给出hint
			- 在创建一个新窗口之前可以给出一些hint，这些hint包括GLFW版本，使用的profile模式（Core Profile）等
			- hint分为**Hard Constraints**和**Soft Constraints**
				- **[[$red]]==Hard Constraints==**：创建出来的窗口必须完全符合这些constraints， 否则创建失败，包括：
					- GLFW_STEREO
					- GLFW_DOUBLEBUFFER
					- GLFW_CLIENT_API
					- GLFW_CONTEXT_CREATION_API
					- 以下两个constraints对于OpenGL context来说是hard的，但是对于OpenGL ES context来说会被忽略
				- **[[$red]]==Soft Constraints==**
					- 非hard的hint都是soft constraints
					- 这些hint，opengl在创建窗口时会尽量靠拢
					- 某些例如GLFW_CONTEXT_VERSION_MAJOR这样的hint，虽然不是hard constraints，但在某些情况下也会导致窗口创建失败，例如如果创建出来的窗口支持的api版本低于给定版本，就会失败
				-
			-