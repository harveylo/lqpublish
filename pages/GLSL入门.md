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
- # Vertex Shader
	- 对于vertex shader来说，每一个输入变量也可以被看作是一个**Vertex Attribute**
	- 可以被定义的Vertex Attributes**数量有一个[[$red]]==最大值==**，一般由硬件决定
		- OpenGL保证至少有**[[$red]]==16==**个**4-component**的vertex attributes
		- 某些硬件提供额外可用的vertex attributes，可以通过
		  ``glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);``来查询
	-
	-