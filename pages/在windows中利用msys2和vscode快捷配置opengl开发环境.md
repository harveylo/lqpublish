# 安装MSYS2
	- [官网链接](https://www.msys2.org/)
	- 便捷的windows toolchains以及包管理工具
	- 能够在windows下获得类似于linux下直接通过sudo apt安装开发package的体验
	- 安装好之后可以直接在其terminal中安装clang工具链，推荐使用ucrt64工具链，顺带安装一下glfw
		- ```bash
		  pacman -S make
		  mingw-w64-ucrt-x86_64-clang 
		  pacman -S mingw-w64-ucrt-x86_64-clang-tools-extra
		  pacman -S mingw-w64-ucrt-x86_64-glfw
		  ```
- # 配置环境变量
  id:: 68275f4b-4583-4515-9d4b-e4c557b3f19c
	- 需要将`C:\msys64\usr\bin`添加到环境变量中，因为其中包含make等工具的可执行文件
- # VSCode下的配置
	- ## 设置项目专用环境变量
		- 把工具链的bin添加到vscode 的terminal的环境变量中，方便在vscode的terminal中直接使用该工具链的各种工具
		- ```json
		  "terminal.integrated.env.windows": {
		      "PATH": "${env:PATH};C:\\msys64\\ucrt64\\bin"
		  },
		  ```
	- ## 指定clangd路径
		- 需要指定clangd路径，否则无法使用lsp
		- ```json
		  "clangd.path": "C:\\msys64\\ucrt64\\bin\\clangd.exe",
		  ```
- # 配置CMake toolchain file
	- ```cmake
	  # CMake toolchain file for MSYS2 MinGW-w64 UCRT Clang
	  # Toolchain root: C:/msys64/ucrt64
	  
	  # Set the system name (target system)
	  set(CMAKE_SYSTEM_NAME Windows)
	  
	  # Specify the C and C++ compilers
	  set(TOOLCHAIN_ROOT C:/msys64/ucrt64)
	  set(CMAKE_C_COMPILER ${TOOLCHAIN_ROOT}/bin/clang.exe)
	  set(CMAKE_CXX_COMPILER ${TOOLCHAIN_ROOT}/bin/clang++.exe)
	  
	  # Optional: Specify the resource compiler (windres)
	  # Check if it exists, as it might not always be needed or present
	  if(EXISTS "${TOOLCHAIN_ROOT}/bin/windres.exe")
	      set(CMAKE_RC_COMPILER ${TOOLCHAIN_ROOT}/bin/windres.exe)
	  endif()
	  
	  # Optional: Specify the archiver (llvm-ar)
	  if(EXISTS "${TOOLCHAIN_ROOT}/bin/llvm-ar.exe")
	      set(CMAKE_AR ${TOOLCHAIN_ROOT}/bin/llvm-ar.exe)
	  endif()
	  
	  # Optional: Specify ranlib (llvm-ranlib)
	  if(EXISTS "${TOOLCHAIN_ROOT}/bin/llvm-ranlib.exe")
	      set(CMAKE_RANLIB ${TOOLCHAIN_ROOT}/bin/llvm-ranlib.exe)
	  endif()
	  
	  # Set the target architecture (optional, but good practice)
	  set(CMAKE_C_COMPILER_TARGET x86_64-w64-windows-gnu)
	  set(CMAKE_CXX_COMPILER_TARGET x86_64-w64-windows-gnu)
	  
	  # Where to look for headers and libraries
	  set(CMAKE_FIND_ROOT_PATH ${TOOLCHAIN_ROOT})
	  
	  # Adjust the search behavior for CMake's find_xxx commands
	  # Search for programs only in the host system paths (not in the toolchain)
	  set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
	  # Search for libraries and headers only in the toolchain paths
	  set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
	  set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
	  set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
	  
	  # Add necessary compiler flags for UCRT environment if needed
	  # Example: set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_UCRT")
	  # Example: set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_UCRT")
	  
	  # It's often good to set the CMAKE_TRY_COMPILE_TARGET_TYPE to STATIC_LIBRARY
	  # when cross-compiling or using a specific toolchain to avoid issues with
	  # executable linking during compiler tests.
	  # set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
	  ```
	- 然后在生成build system的时候使用：
		- ```bash
		  cmake -G "MSYS Makefiles" -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DCMAKE_TOOLCHAIN_FILE=../tool-chains/win/msys2-mingw64-ucrt.cmake ..
		  ```
		- **[[$red]]==注意：==*一定要加上*`-G "MSYS Makefiles"`，否则cmake可能会默认选择vs(msvc)建造工具，无法正确使用我们自行配置的工具链。
		- 如果报错找不到`MSYS Makefiles`，说明make的路径没添加到环境变量，确认一下[第二步](68275f4b-4583-4515-9d4b-e4c557b3f19c)是否完成
		-