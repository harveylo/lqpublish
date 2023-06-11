- # 输出一张图像(image)
	- ## PPM图像格式
		- ppm(ppm pixmap format)图象格式应该是最简单明了的图像格式，其格式如下：
			- ```text
			  P3           
			  # "P3" means this is a RGB color image in ASCII 
			  # "3 2" is the width and height of the image in pixels
			  # "255" is the maximum value for each color
			  # This, up through the "255" line below are the header.
			  # Everything after that is the image data: RGB triplets.
			  # In order: red, green, blue, yellow, white, and black.
			  3 2         
			  255       
			  255 0   0 0   255 0   0 0 255
			  255 255 0 255 255 255 0 0 0
			  ```
			- 以上内容对应的图像为
				- ![undefined](https://upload.wikimedia.org/wikipedia/commons/5/57/Tiny6pixel.png)
			- 如果是二进制而非ASCII编码的PPM，开头的魔数应该是ASCII码对应的**``P6``**
	- ## 输出第一张图像
		- ```c++
		  #include <iostream>
		  
		  int main(int argc, char const *argv[])
		  {
		      // image size
		      const int image_width = 256;
		      const int image_height = 256;
		  
		      // Render
		      std::cout<<"P3\n" << image_width <<' '<<image_height<<"\n255\n";
		      for(int i = image_height-1;i>=0;i--){
		          for(int j = 0;j<image_width;j++){
		              double r = (double)i/(image_height-1);
		              double g = (double)j/(image_width-1);
		              double b = 0.25;
		              
		              int rp = (int)(r*255);
		              int gp = (int)(g*255);
		              int bp = (int)(b*255);
		              std::cout<<rp<<' '<<gp<<' '<<bp<<'\n';
		          }
		      }
		      return 0;
		  }
		  ```
		- 以上代码会输出一张256*256的渐变图像
		- ![image.png](../assets/image_1686378609113_0.png)
	- ## 增添进度指示(Progress Indicator)
		- 通过向标准错误输出输出剩余扫描行并flush可以实时显式进度
		- 使用`\r`可以将输出光标移动回行首，**做出更新当前行而不是另外输出一行的效果**
		- ```c++
		  #include <iostream>
		  #include <unistd.h>
		  
		  int main(int argc, char const *argv[])
		  {
		      // image size
		      const int image_width = 256;
		      const int image_height = 256;
		  
		      // Render
		      std::cout<<"P3\n" << image_width <<' '<<image_height<<"\n255\n";
		      for(int i = image_height-1;i>=0;i--){
		          std::cerr<<"\rScanlines remaining: "<<"\e[44m"<<i<<"\e[0m "<<std::flush;
		          for(int j = 0;j<image_width;j++){
		              double r = (double)i/(image_height-1);
		              double g = (double)j/(image_width-1);
		              double b = 0.25;
		              
		              int rp = (int)(r*255);
		              int gp = (int)(g*255);
		              int bp = (int)(b*255);
		              std::cout<<rp<<' '<<gp<<' '<<bp<<'\n';
		          }
		      }
		      std::cerr<<"\n\e[42m Done.\e[0m\n";
		      return 0;
		  }
		  ```
- # vec3 类
	- 用于存放集向量和颜色的类
	- 在很多系统中，这样的向量都是四维的：
		- 对于几何来说，3维坐标加上**齐次坐标(homogeneous coordinate)**
		- 对于颜色来说，三维RGB值加上alpha透明通道
- # 光线(ray)，简单相机和背景
	- ## ray class
		- 所有的光线追踪器(ray tracer)都会有一个光线类
		- 一条光线可以看作一个函数：$\mathbf{P}(t) = \mathbf{A}+t\mathbf{b}$
			- $\mathbf{A}$ 是一个光线源
			- $\mathbf{b}$ 是光线方向
			- $t$ 是光线参数，为一个实数
			- ![](https://raytracing.github.io/images/fig-1.02-lerp.jpg){:height 118, :width 359}
	- ## 将光线添加到场景中
		- ### [[思考：viewport如何控制视野大小以及其与输出图片大小的关系]]
		- 一个光线追踪器负责将光线**发送过每一个像素**，并且计算从光线的角度看到的颜色，通过三步完成
			- 计算**从人眼到像素**的光线
			- 判明有哪些物体和光线**相交**(intersect)
			- 计算相交点的**颜色**
		- ![](https://raytracing.github.io/images/fig-1.03-cam-geom.jpg)
		- ``ray_color``函数根据光线的y分量进行**线性融合(Linear Blend)**，也称**lerp**，其一般形式为
			- $\text{blendedValue}=(1−t)⋅\text{startValue}+t⋅\text{endValue}$
- # 添加一个球体
	- 一个球体的表达就是一个中心$\mathbf{C}$和一个半径$r$
	- 一个点$\mathbf{P}$是否在球体上，可以表达为：$(\mathbf{(P-C)\cdot (P-C)}=r^2$
	- 而一条光线$\mathbf{P}(t)$是否击中一个球体的某一点，则可以描述为存在一个$t$使得
		- $(\mathbf{P}(t)-\mathbf{C})\cdot (\mathbf{P}(t)-\mathbf{C})=r^2$
	- 可展开为：
		- $(\mathbf{A}+t\mathbf{b}-\mathbf{C})\cdot (\mathbf{A}+t\mathbf{b}-\mathbf{C})=r^2$
	- 进一步展开为：
		- $t^2\mathbf{b\cdot b}+2t\mathbf{b}\cdot\mathbf{(A-C)+(A-C)\cdot(A-C)}-r^2 = 0$
	- 求解$t$即可，可能有三种情况
		- ![](https://raytracing.github.io/images/fig-1.04-ray-sphere.jpg){:height 171, :width 242}
		-
- # 表面法向量(Surface Normals)和多物体
	- 对于一个球体来说一个朝外的法向量的方向是表面的一个点减去中心点
	- ![](https://raytracing.github.io/images/fig-1.05-sphere-normal.jpg)
	- ## 简化光线-球体接触计算代码
		- ```c++
		  double hit_sphere(const point3& center, double radius, const ray& r) {
		      vec3 oc = r.origin() - center;
		      auto a = dot(r.direction(), r.direction());
		      auto b = 2.0 * dot(oc, r.direction());
		      auto c = dot(oc, oc) - radius*radius;
		      auto discriminant = b*b - 4*a*c;
		  
		      if (discriminant < 0) {
		          return -1.0;
		      } else {
		          return (-b - sqrt(discriminant) ) / (2.0*a);
		      }
		  }
		  ```
		- 一个向量点乘自己等于其长度的平方
		- $b$含有一个2的因素，因此：
			- ![image.png](../assets/image_1686489930625_0.png){:height 181, :width 155}
			- 可以简化计算
	- ## 对可击中物体(Hittable Objects)的抽象
		- 将所有可以被光线击中的物体抽象为一个类
		- 此类会有一个``hit``函数，此函数接受一个光线作为参数，计算$t$
		- 大多数光线追踪器在计算$t$时都会有一个有效区间$t_{min}, t_{max}$只有当$t$在区间内才算是击中
	- ## 正面(Front Face)和背面(Back Face)
		- 若法线方向总是从内到外(击中点-中心)，那么通过将光线和法线的方向进行比较即可获知光线是从内部还是从外部击中了物体
			- 如果光线和(外部)法线的方向一致，说明是从内部击中了物体
			- 如果光线和(外部)法线的方向相反，说明是从外部击中了物体
		- 而**两个向量可以通过点乘的正负[[$red]]==来判断方向是否一致==**
			- **<0**说明方向相反
			- **>0**说明方向相同
		- 确定光线是从内击中物体还是从外击中物体实际上有两种实现方式，一种是几何时确定，一种是上色时确定
		- 还可以通过确定法线是否总是向外来确定