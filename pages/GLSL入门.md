- GLSL脚本的一般形式为：
	- ```C
	  #version version_number
	  in type in_variable_name;
	  in type in_variable_name;
	  
	  out type out_variable_name;
	    
	  uniform type uniform_name;
	    
	  void main()
	  {
	    // process input(s) and do some weird graphics stuff
	    ...
	    // output processed stuff to output variable
	    out_variable_name = weird_stuff_we_processed;
	  }
	  ```
- # GLSL类型(Types)
	- GLSL基本上拥有所有C语言中的基本类型，包括`int`, `float`, `double`, `uint`, `bool`等
	- GLSL有两种常见的容器类型，主要是`vector`和`matrices`
	- ## Vectors
		- 在GLSL中，vector是一个可以包含2,3或4个上述基础类型的容器，定义形式为：
			- `vecn`，包含`n`个`float`的vector
				- 例如常用的`vec2`, `vec3`, `vec4`
			- `bvecn`，bool值n元vector
			- `ivecn`，int值n元vector
			- `uvecn`，uint值n元vector
			- `dvecn`，double值n元vector
		- 使用`x`, `y`, `z`, `w`来分别访问一个vector变量的第1，2，3或4个值
		- Vector在GLSL中的元素使用非常灵活，其中有一种被叫做**[[$red]]==swizzling==**的用法
			- 使用至多四个字母下标来构造出一个新的vector，只要原有的vector有这些维度
			- ```GLSL
			  vec2 someVec;
			  vec4 differentVec = someVec.xyxx;
			  vec3 anotherVec = differentVec.zyw;
			  vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
			  ```
			- 同时还可以将一个vector作为其他vector构造函数的一个参数传入
				- ```GLSL
				  vec2 vect = vec2(0.5, 0.7);
				  vec4 result = vec4(vect, 0.0, 0.0);
				  vec4 otherResult = vec4(result.xyz, 1.0);
				  ```
	- ## Opaque Type
		- glsl中有一些类型是不能在定义时初始化的，只能定义为uniform由cpu传入数据，``sampler2D``就是典型的opaque type
		- 任何包含有opaque type的struct也不能初始化
- # GLSL输入输出
	- 每一个Shader程序都需要定义**输入（in）**和**输出（out）**
	- 当前一个阶段的Shader的输出的类型和变量名都对上后一个阶段的Shader的输入变量的类型和变量名时，OpenGL就会将前后两阶段的输入和输出连接起来，以完成shader之间的通信
	- ## Uniforms
		- uniform是另一种将数据从CPU上的应用程序传输到GPU上的Shader的方法
		- 对于某一个**Shader程序**，一个uniform变量是**[[$red]]==全局唯一==**的
			- 在这个shader程序的**任意阶段**shader中，都可以访问这个全局变量，不论是读值还是写入
		- 可以在任何阶段的shader中定义uniform变量，只要定义了就可以使用
			- 例如，可以在fragment shader中定义一个uniform变量，如果在vertex shader中不需要这个变量，那么在vertex shader中就不必定义。但如果需要使用，则直接定义同名uniform变量即可
		- 如果一个uniform 变量被定义了但没有在任何阶段的shader中使用，那么编译器会自动移除这个未使用的变量
	- 在应用程序里，可以使用``glUniform``函数家族来设置uniform变量的值
	  collapsed:: true
		- 一些可能的后缀：
			- `f`: the function expects a `float` as its value.
			- `i`: the function expects an `int` as its value.
			- `ui`: the function expects an `unsigned int` as its value.
			- `3f`: the function expects 3 `float`s as its value.
			- `fv`: the function expects a `float` vector/array as its value.
		- 但在设置值之前需要先通过`glGetUniformLocation`来获取一个变量位置
		- ```C++
		  auto uColorLocation = glGetUniformLocation(shaderProgram, "uColor");
		  glUniform4f(uColorLocation, r, g, b, 1.0f);
		  ```
	- ## 输入插值
		- 在调用api绘制一个图形，例如triangle时，虽然给出的数据往往只有几个vertices，但是实际将该图形绘制在屏幕上显然需要很多fragment，因此opengl会对输入数据进行插值得出额外的fragment输入数据，并对每一个插值出来的fragment都走一遍渲染管线
		- ![image.png](../assets/image_1745126416535_0.png)
			- 例如此图中，只指定了该三角三个顶点的方位和颜色，但是opengl插值出了其他三角形内部的所有fragment，包括颜色和位置数据
- # Vertex Shader
	- 对于vertex shader来说，每一个输入变量也可以被看作是一个**Vertex Attribute**
	- 可以被定义的Vertex Attributes**数量有一个[[$red]]==最大值==**，一般由硬件决定
		- OpenGL保证至少有**[[$red]]==16==**个**4-component**的vertex attributes
		- 某些硬件提供额外可用的vertex attributes，可以通过
		  ``glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);``来查询
	- ## 输入输出
		- ### 输入
			- Vertex Shader的输入来自于在CPU上的配置，因此在指定输入时需要指明**layout**
				- `layout(location = 0)`表明是第0个attribute
				- 只有Vertex shader需要如此
				- 也可以不显式指明layout而使用`glGetAttributeLocation`函数去获取attribute的位置
		- ### 输出
- # Fragment Shader
	- ## 输入输出
		- ### 输出
			- Fragment Shader必须要输出一个`vec4`，因为这对应的是一个颜色输出。
				- 如果没有对应的`vec4`输出，那么最终的颜色缓冲就全是未定义的值，表现出的结果就是屏幕全白或全黑
			-
	-