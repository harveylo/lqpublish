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
- # Vertex Shader
	- 对于vertex shader来说，每一个输入变量都是一个