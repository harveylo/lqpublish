- # 指定接下来使用的命名空间
	- ``using namespace std``
- # 使用某个命名空间中的符号
	- ``using std::cout``
- # 类型重定义
	- 可以取代``typedef``，看起来比其更直观
		- ``using fun = void (*)(int, int)``
	- 支持和模板一起使用
		- ```
		  template <typename val>
		  using int_map_t = std::map<int, val>;
		  ```