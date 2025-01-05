- 一堆build错误，我都不知道是不是故意的
- # TasAction相关
	- ```Cannot use @TaskAction annotation on method IncrementalTask.taskAction$gradle_core() because interface org.gradle.api.tasks.incremental.IncrementalTaskInputs is not a valid parameter to an action method.```
	- Android Gradle Plugin版本太低导致的，修改项目根目录下的``build.gradle``中的``classpath 'com.android.tools.build:gradle:7.0.0'``中的版本7.0.0为最新，例如8.7.3
- # 缺少namespace
	- 报错会直接报错少namespace，在`TriangleBinNative/build.gradle`的android中新增一个namespace，类似于：
	- ```gradle
	  apply plugin: 'com.android.application'
	  
	  android {
	      namespace 'com.harvey.trianglebin'
	      compileSdkVersion 31
	      defaultConfig {
	      
	      ......
	  ```
- # 错误配置AndroidManifest.xml
	- ``TriangleBinNative/src/main/AndroidManifest.xml``中错误设置了package
	- 将开头的package去掉，及删除：
		- ```xml
		  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
		  
		      package="com.socpk.trianglebin"
		      
		      >
		  ```
	- 中的pakcage一列
- # CMake版本设置错误
	- 使用NDK时，对cmake版本有最低要求，否则某些特性无法使用导致cmake编译不过，然而项目的主CMakeList文件居然将最低版本需求设置到了3.1，将该版本设置到任意大于3.3的版本即可
	- 修改``TriangleBinNative/src/main/cpp/CMakeLists.txt``中`cmake_minimum_required`中的版本为任意大于等于3.3的值
-