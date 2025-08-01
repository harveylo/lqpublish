- **安装nightly rust**
	- ``rustup override add nightly``
	- ``rustc --verison``查看当前``rustc``版本
- **安装`bootimage`**
	- ``cargo install bootimage``
	- bootimage依赖于llvm-tools：``rustup component add llvm-tools-preview``
- **安装QEMU**
	- ``sudo apt install qemu qemu-system-x86``
- **安装窗口支持**
	- 如果无法直接调用窗口(可能是win10问题)，则需要完成以下操作：
	- 安装[VcXsrv](https://sourceforge.net/projects/vcxsrv/)
	- ``export DISPLAY="$(grep nameserver /etc/resolv.conf | sed 's/nameserver //'):0"``
	- ``sudo apt install x11-apps -y``
	- ``sudo apt install xserver-xorg-core``
	- `sudo apt install x-window-system-core`
	- 配置windows防火墙 ：
		- Control Panel
		- \> System and Security
		- \> Windows Defender Firewall
		- \> Advanced Settings
		- \> Inbound Rules
		- \> New Rule...
		- \> Program
		- \> %ProgramFiles%\VcXsrv\vcxsrv.exe
		- \> Allow the connection
		- \> checked Domain/Private/Public
		- \> Named and Confirmed Rule
	- 通过XLaunch启动VcXsrv，并勾选"**Disable access control**"