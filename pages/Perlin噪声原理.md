- [[理解插值 ]]
- # 基础
	- 接受一个三维坐标(x,y,z)，输出一个``0-1``的double值
	- **第一步**：**只取xyz的小数部分**，得到一个在此晶格(体积为1)的点的坐标
		- 为方便描述，接下来的部分在二维中展示
		- ![image.png](../assets/image_1687511932288_0.png)
			- 此图中淡蓝色点即代表输入值在二维单位正方形中的空间坐标
	- **第二步**：给四个顶点(三维中应该是八个顶点)各自生成一个**伪随机梯度向量**
		-
- # 配置噪声
	- ## 频率(frequency)
		- 在x上乘以频率
		- ```c++
		  float frequency = 0.5;
		  float freqNoise = ValueNoise1D.eval(x * frequency);
		  ```
	- ## 幅度(amplitude)
		- 在函数得最终结果上乘以幅度
		- ```c++
		  float amplitude = 0.5;
		  float freqNoise = ValueNoise1D.eval(x) * amplitude;
		  ```
	- ## 相位(phase)
		- 就是x的偏移
		- ```c++
		  float offset = frameNumber;
		  float freqNoise = ValueNoise1D.eval(x + offset);
		  ```
	- ## 符号
		- 可以通过2*noise -1的计算将范围在[0,1]之间的噪声映射到区间[-1,1]
		- ```c++
		  float signedNoise = 2 * ValueNoise1D.eval(x) - 1;
		  ```