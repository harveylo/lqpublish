# 下载android cmdline-tools
	- 去android开发者官网下载，不要下载android studio(如果要用android studio来开发，那这些设置都没必要)，仅下载android cmdline-tools
- # 重整目录结构
	- 可参考这篇[doc](https://developer.android.com/tools/sdkmanager)
	- 在合适位置创建一个SDK Root目录，在该目录中创建`cmdline-tools/latest`这样的结构，将下载下来的cmdline-tools中的所有内容拷贝到这个目录中
- # 添加环境变量
	- ```bash
	  export ANDROID_SDK_ROOT=/home/harveylwo/android_sdk_root
	  export ANDROID_HOME=/home/harveylwo/android_sdk_root
	  export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
	  export PATH=$PATH:$ANDROID_HOME/emulator
	  export PATH=$PATH:$ANDROID_HOME/platform-tools
	  ```
- # 使用`sdkmanager`下载sdk和所需package
	- 使用`sdkmanager --list`可以看到所有可下载的package，包括各种平台工具，sdk版本等
	- 下载所需的package即可
- # 搭建测试emulator
	- 下载系统镜像
		- ```bash
		  # List all available system images
		  sdkmanager --list | grep system-images
		  
		  # Download a specific system image (example for Android 33 with Google APIs for x86_64)
		  sdkmanager "system-images;android-35;google_apis;x86_64"
		  
		  # Accept licenses if needed
		  sdkmanager --licenses
		  ```
	- 创建新的emulator
		- ``avdmanager create avd -n [emulator_name] -k "[system_image_name]" -d [device_model]``
	- 启动emulator
		- `emulator -avd [emulator_name]`
	- 使用adb查看连接的设备
		- `adb devicecs`
	- 启动过程中可能会遇到kvm问题，自行解决
	- vscode中下载android ios emulator插件，在插件设置中正确设置emulator的路径，具体详见插件描述页