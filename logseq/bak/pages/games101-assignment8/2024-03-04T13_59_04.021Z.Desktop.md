- 在`CGL/deps/glfw/src/x11_init.c`的245行附近，框架调用了`XkbGetKeyboard`，然而这个函数会返回`nullptr`导致segmentation fault
	- 再次应证了我说games101框架是一坨狗屎的说法
	- 不过也不全怪框架编写这，也许是因为他们根本就没考虑到库函数的版本变动？
	- 具体导致的原因不清楚，猜测是库函数版本更新导致的接口不兼容
	- 具体改动方法已经在代码中做了注释，此处再此粘贴一次
	- ```c++
	  // ! should not use the XkbGetKeyboard function, it returns nullptr
	  // ? check this issue here: https://github.com/glfw/glfw/issues/389
	  // ? 'The reason this returns NULL is because we don't send the map name at all in Wayland; 
	  // ? in fact, it may not have originally been XKB. 
	  // ? Using XkbGetMap instead of XkbGetKeyboard will fix that.'
	  // XkbDescPtr descr = XkbGetKeyboard(_glfw.x11.display,
	  //                                   XkbAllComponentsMask,
	  //                                   XkbUseCoreKbd);
	  // * So the solution is to use XkbGetMap instead of XkbGetKeyboard
	  // * check this commit https://github.com/glfw/glfw/commit/2d39dff84af0f51e5c11c3e9b953cd2e2af166ef
	  XkbDescPtr descr = XkbGetMap(_glfw.x11.display,
	                                    XkbAllComponentsMask,
	                                    XkbUseCoreKbd);
	  
	  XkbGetNames(_glfw.x11.display, XkbKeyNamesMask, descr);                             
	  ```