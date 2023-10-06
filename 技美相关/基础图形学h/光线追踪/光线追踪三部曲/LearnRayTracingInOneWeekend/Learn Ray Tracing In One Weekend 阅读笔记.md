## Learn Ray Tracing In One Weekend 阅读笔记

[Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html#thevec3class)

注：这里代码有一些改动，比如为了避免精度问题用double替代float，以及一些语法可能不太一样，但基本逻辑遵循作者的思路。在文档里只会给出核心的代码，详细的代码可以看Github的链接：

本文档的目录结构如下：

[TOC]



## Chapter 2:输出一张图像

![image-20220304163823263](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304163823263.png)

可以看到ppm的文件格式比较简单，因此在这里作者使用.ppm格式作为输出的格式。注：ppm格式的输出流是从上到下，从左到右。

- 以下的代码仅仅将信息输出到控制台窗口:

```c++
//main.cpp
#include <iostream>

int main()
{
	int nx = 200;
	int ny = 100;
	std::cout << "P3\n" << nx << " " << ny << "\n255\n"; //根据格式,输出图像的长和宽
	//像素从左往右，从上往下打印
    for (int j = ny - 1; j >= 0; j--)
	{
		for (int i = 0; i < nx; i++)
		{
			float r = float(i) / float(nx); 
			float g = float(j) / float(ny);
			float b = 0.2;
			//此时计算出的r,g,b都是[0,1]区间内,因此需要将其转换到
			int ir = static_cast<int>(255.99 * r);
			int ig = static_cast<int>(255.99 * g);
			int ib = static_cast<int>(255.99 * b);
			std::cout << ir << " " << ig << " " << ib << "\n";
		}
	}
}
```

- 将代码进行修改,以使其能够输出到`output.ppm`文件当中,并可以通过ppm查看器来查看输出结果:
  - 文件流的操作可以查看下述链接:[c++如何读取数据和写入.txt文件？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/358411072/answer/944779854)

```c++
//main.cpp
#include <iostream>
#include <fstream>
using namespace std;
int main()
{
	ofstream outfile;
	string filename = "output.ppm";
	outfile.open(filename);
	if (!outfile)
	{
		cerr << "no file exists!" << endl;
	}
	int nx = 200;
	int ny = 100;
	outfile << "P3\n" << nx << " " << ny << "\n255\n"; //根据格式,输出nx行和ny列
	for (int j = ny - 1; j >= 0; j--)
	{
		for (int i = 0; i < nx; i++)
		{
			double r = double(i) / double(nx); 
			double g = double(j) / double(ny);
			double b = 0.2;
			//此时计算出的r,g,b都是[0,1]区间内,因此需要将其转换到[0 255]
			int ir = static_cast<int>(255.99 * r);
			int ig = static_cast<int>(255.99 * g);
			int ib = static_cast<int>(255.99 * b);
			outfile << ir << " " << ig << " " << ib << "\n";
		}
	}
	outfile.close(); 
}
```

此时可以看到我们的输出结果是正确的：(左:输出的效果图,右:文件中输出的内容)

![image-20220304165741283](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304165741283.png)

------



## Chapter3：vec3类的书写

### 1.基本部分

许多系统里面的颜色是用四维向量来存储的(R,G,B,A);

对于我们来说，三维向量就足够了。 我们将使用相同的类vec3用于颜色，位置，方向，偏移，等等。 **三维向量更为简单，而且可以带来代码的简洁性**。

```c++
// vec3.h
#ifndef VEC3_H
#define VEC3_H

#include <cmath>
#include <iostream>

using std::sqrt;

class vec3 {
public:
	double e[3];
public:
	vec3() :e{ 0,0,0 } {}
	vec3(double e0, double e1, double e2) : e{ e0,e1,e2 } {}
	double x() const { return e[0]; }
	double y() const { return e[1]; }
	double z() const { return e[2]; }

	vec3 operator-() const { return vec3(-e[0], -e[1], -e[2]); }
	double operator[](int i) const { return e[i]; }
	double& operator[](int i) { return e[i]; }

	vec3& operator+=(const vec3& v) {
		e[0] += v.e[0], e[1] += v.e[1], e[2] += v.e[2];
		return *this;
	}

	vec3& operator*=(const double t) {
		e[0] *= t, e[1] *= t, e[2] *= t;
		return *this;
	}

	vec3& operator/=(const double t) {
		return *this *= (1 / t);
	}
	double length_squared() const {
		return e[0] * e[0] + e[1] * e[1] + e[2] * e[2];
	}
	double length() const { return sqrt(length_squared()); }
};
//Type aliases for vec3
using point3 = vec3;  //for 3D point
using color = vec3;

#endif // !VEC3_H
```

- 注意，这里使用的是double，这样做可以避免某些浮点数的精度可能造成的问题。

------

### 2.vec3类的其他方法

这里直接给出代码（vec3.h）：

```c++
//vec3 utility functions
inline std::ostream& operator<<(std::ostream& out, const vec3& v) {
	return out << v.e[0] << ' ' << v.e[1] << ' ' << v.e[2];
}

inline vec3 operator+(const vec3& u, const vec3& v) {
	return vec3(u.e[0] + v.e[0], u.e[1] + v.e[1], u.e[2] + v.e[2]);
}

inline vec3 operator-(const vec3& u, const vec3& v) {
	return vec3(u.e[0] - v.e[0], u.e[1] - v.e[1], u.e[2] - v.e[2]);
}

inline vec3 operator*(const vec3& u, const vec3& v) {
	return vec3(u.e[0] * v.e[0], u.e[1] * v.e[1], u.e[2] * v.e[2]);
}

inline vec3 operator*(double t, const vec3& v) {
	return vec3(t * v.e[0], t * v.e[1], t * v.e[2]);
}

inline vec3 operator*(const vec3& v, double t) {
	return t * v;
}

inline vec3 operator/(vec3 v, double t) {
	return (1 / t) * v;
}

inline vec3 unit_vector(vec3 v) {
	return v / v.length();
}

//比较重要的点乘和叉乘操作
inline double dot(const vec3& u, const vec3& v) {
	return u.e[0] * v.e[0] + u.e[1] * v.e[1] + u.e[2] * v.e[2];
}

inline vec3 cross(const vec3& u, const vec3& v) {
	return vec3(
		u.e[1] * v.e[2] - u.e[2] * v.e[1],
		u.e[2] * v.e[0] - u.e[0] * v.e[2],
		u.e[0] * v.e[1] - u.e[1] * v.e[0]);
}
```

------



### 3.封装一个输出颜色函数

```c++
//color.h
#ifndef COLOR_H
#define COLOR_H

#include "vec3.h"
#include <iostream>

void write_color(std::ostream& out, color pixel_color) {
	//write the translated [0,255] value of each color component
	out << static_cast<int>(255.999 * pixel_color.x()) << ' '
		<< static_cast<int>(255.999 * pixel_color.y()) << ' '
		<< static_cast<int>(255.999 * pixel_color.z()) << '\n';
}

#endif // !COLOR_H
```

此时主函数就可以改为：

```c++
#include "vec3.h"
#include "color.h"
//...
ofstream outfile; 
//...
for (int j = ny - 1; j >= 0; j--)
{
    std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;  //添加一个进度条
    for (int i = 0; i < nx; i++)
    {
        color pixel_color(double(i) / (nx - 1), double(j) / (ny - 1), 0.25);
        write_color(outfile, pixel_color);
    }
}
```



------



## Chapter4：光线类，简单的相机，以及背景

所有的光线追踪程序都需要一个`ray`类，用来表示光线。

- 我们将光线定义为$p(t)=A+t*B$

  - p是一个三维坐标的点，沿着光线移动；A是起点，B则是光线的方向；t是一个实数（代码里是double量）

  - 不同的t值可以到达不同的地方，如果t只能取正数那么这就是一条射线：

![image-20220304173948676](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304173948676.png)

```c++
//ray.h
#ifndef RAY_H
#define RAY_H

#include "vec3.h"

class ray {
public:
	ray(){}
	ray(const point3& origin, const vec3& direction) : orig(origin), dir(direction) {}
	point3 origin() const { return orig; }
	vec3 direction()const { return dir; }
	
	point3 at(double t) const {
		return orig + t * dir;
	}

public:
	point3 orig;
	vec3 dir;
};

#endif // !RAY_H
```

书写完光线类之后，我们回到之前的光线追踪部分：

光线追踪器的核心是**从像素发射射线, 并计算这些射线得到的颜色。**这包括如下的步骤: 

1. 计算从相机到像素点的射线，也即光线追踪的光线；
2. 计算光线是否与场景中的物体相交； 
3. 如果有, 计算交点的颜色。在做光线追踪器的初期, 我们先弄个简单摄像机让代码能跑起来。同时也会编写一个简单的ray_color(ray)函数来返回背景颜色值(一个简单的渐变色)。

**关于光线追踪的原理，可以参考之前写的一篇博客：**

[基于Whitted模型的光线追踪 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/406753413)

![image-20220304174858041](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304174858041.png)

- 在本项目当中，为了防止混淆导致不好Debug，我们选择不同长宽的图像

- 暂时来说，认为视角的aspect ratio就是渲染画面的宽高比。定义viewport的高度是2.0，同时定义被投影点与投影平面的距离是焦距，为1.0。我们将相机放置在（0，0，0）点，同时认为**X轴向右，Y轴向上，Z轴正方向指向屏幕外面**（也就是说，这是一个**右手坐标系**）（补充一下，**Unity的模型坐标系和世界坐标系都是左手坐标系**，Z轴正方向是指向屏幕内侧的，见https://blog.csdn.net/aihiao/article/details/80073477）。从左上角开始遍历屏幕空间。

- 此时，如何表示从视角打向成像平面的像素光线的方向？

  - 认为屏幕左下角为`lower_left_corner`,在上图当中其坐标为（-2，-1，-1）；
  - 认为水平方向为`horizontal`,在这里是(4.0,0.0,0.0),而竖直方向为`vertical`,在这里是(0.0,2.0,0.0)，后面将会用比例系数来表示点在屏幕中的位置；
  - 在上图当中，$\vec{direction}=\vec{b}+\vec{a}$
    - $\vec{b}=lowerleftcorner-origin$
    - $\vec{a}=u*\vec{horizontal}+v*\vec{vertical}$ 

  - 代入$\vec{a}和\vec{b}$,得到结果为$\vec{direction}=lowerleftcorner-origin+u*\vec{horizontal}+v*\vec{vertical}$



在接下来的代码中，光线 $\vec{r}$ 会近似穿过像素的中心（**暂时不用考虑抗锯齿因为后面会进行处理**）

```c++
//暂时就先把打出光线这个逻辑写在main函数里
#include "ray.h"

//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r) {
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);  //[-1,1]->[0,1])
	//y方向值越小越接近白色,越大越接近天蓝色
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); //lerp,后面那个是天蓝色
}

int main(){
    //...
    //Image
	const auto aspect_ratio = 16.0 / 9.0;
	int image_width = 400;
	int image_height = static_cast<int>(image_width/aspect_ratio);
	//Camera
	double viewport_height = 2.0;
	double viewport_width = aspect_ratio * viewport_height;
	double focal_length = 1.0;
	
	auto origin = point3(0,0,0);
	auto horizontal = vec3(viewport_width, 0, 0);
	auto vertical = vec3(0, viewport_height, 0);
	auto lower_left_corner = origin - horizontal / 2 - vertical / 2 - vec3(0, 0, focal_length);

	//Render
	outfile << "P3\n" << image_width << " " << image_height << "\n255\n"; //根据格式,输出nx行和ny列
	for (int j = image_height - 1; j >= 0; j--) //这里的i和j循环的逻辑中,原点是左下角,而输入进ppm文件的次序是从左上角开始,因此j是从image_height开始的
	{
		std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;  //添加一个进度条
		for (int i = 0; i < image_width; i++)
		{
			auto u = double(i) / (image_width - 1);
			auto v = double(j) / (image_height - 1);
            //核心的打出光线的逻辑
			ray r(origin, lower_left_corner - origin + u * horizontal + v * vertical);
			color pixel_color = ray_color(r);
			write_color(outfile, pixel_color);
		}
	}
}
```

`ray_color(ray)`函数根据y值将蓝色和白色做了**线性插值的混合**：

- 我们这里把射线做归一化, 以保证y的取值范围(-1.0<y<1.0)。此时用y轴做渐变效果就比较理想，但同时我们也需要将[-1.0，1.0]的范围变到[0.0，1.0]，对应`float t = 0.5*(unit_direction.y() + 1.0);`一句（**这是一个常用的小技巧**）

- 此时t=1.0时就是蓝色, 而t=0.0时就是白色。如果在蓝白之间想要一个混合效果(blend)，就可以采用**线性混合(linear blend)或者说线性插值(liner interpolation)**。或者简称其为lerp。一个lerp一般来说会是下面的形式:

$blended\_value=(1-t)*start\_value+t*end\_value,t\in[0,1]$

最终输出的结果如下：

![image-20220304183023060](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304183023060.png)

------



## Chapter5：添加一个球体

向光线追踪的场景当中添加一个球体，原因是球体与光线的相交比较方便计算。

- 半径为R的球体的方程：

  - $x^2+y^2+z^2=R^2$

  - 也就是说，如果一个坐标点（x，y，z）满足上面的方程，我们就认为这个点在球面上，否则不在球面上；

- 根据上式推广，如果球心在 C  =（cx，cy，cz），则球的方程为：

  - $(x-cx)^2+(y-cy)^2+(z-cz)^2=R^2$

- 再次推广，此时对于球面上的点p，有：
  - $dot(p-C,p-C)=R*R$,其中c为中心点的坐标

- 也就是说，对于光线上的点$p(t)=A+t*B$，有$dot(p(t)-C,p(t)-C)=R*R$，也就是$dot(A+t*B-C,A+t*B-C)=R*R$

**将上式中的点乘展开，进行多项式展开，可以得到化简之后的式子：**

$ t^2\vec{B}·\vec{B}+2*t*\vec{B}·\vec{(A-C)}+\vec{(A-C)}·\vec{(A-C)}-R^2=0$

可以看到，通过这个式子**就可以求解出t的值**，针对不同的t值可以分为如下情况：

![image-20220304193021658](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304193021658.png)

另外，理论上也要考虑摄像机在球体内部的情况，此时可以借用我在之前的博客中的讨论结果：

<img src="https://pic1.zhimg.com/80/v2-22b7e5fc31e3d79e3b936cda8734ea6a_1440w.jpg?source=d16d100b" alt="img" style="zoom: 50%;" />

（注，这里的$1e^{-3}$是因为计算机具有浮点数误差，因此不能简单地认为是0，要引入误差$\epsilon$）



在（0，0，-1）处放置一个小球（注意相机z轴正方向指向屏幕外，因此物体z=-1说明在相机前面，可能能被看到），并用上面的公式来判断是否与其相交,程序改动部分如下：

```c++
//暂时写在main.cpp里

//指定球体,判断光线是否与球体相交
bool hit_sphere(const point3& center, double radius, const ray& r) {
	vec3 oc = r.origin() - center;
	auto a = dot(r.direction(), r.direction());
	auto b = 2.0 * dot(oc, r.direction());
	auto c = dot(oc, oc) - radius * radius;
	auto discriminant = b * b - 4 * a * c;
	return (discriminant > 0);
}

//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r) {
    //新增++++++++++++++++++++++++++
	if (hit_sphere(point3(0, 0, -1), 0.5, r)) {
		return color(0, 1, 0);
	}
    //新增结束+++++++++++++++
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);  //[-1,1]->[0,1])
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); 
}
```

此时输出的运行结果如下：

![image-20230417195306789](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230417195306789.png)

虽然现在缺少光照, 反射, 以及更多的物体, 但是已经开始有一些成果了。**需要注意我们其实求的是直线与球相交的解, t<0的那些情况也计算进去了, 而我们只想要直线中一段射线的解。**（正如刚才所描述的部分）

如果将你的球心设置在(0,0,1), 会得到完全相同的结果（但这是错误的，因为相机是面向z轴负方向的（指向屏幕里面），而z正方向是指向屏幕外面，所以这个球体会在相机后面，但仍会被看到，所以有bug），后续将修复这个bug。

------



## Chapter 6：表面法线以及更多的物体

### 1.基本部分

- 首先，我们获得表面法线来计算着色。面法向应该是一种垂直于交点所在平面的三维向量，并指向外面。在这里 **对其进行归一化，因为这样比较方便着色**（但不是绝对的，看个人喜好）；
  - 对球体来说，光线相交点p处的法线方向为：$\vec{p}-\vec{C}$,示意图如下：
  - ![image-20220304195135884](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304195135884.png)

- 由于现在还没有光源等信息，因此我们首先使用法线方向作为颜色输出的参考，方法与上面类似，将法线N归一化，使其各个分量范围在[-1,1]中，并将其映射到[0,1]作为R,G,B的值输出：
  - **值得考虑的是，如果求解到有两个相交点t的值，我们选择更小的t作为交点**；
  - 代码如下：

```c++
//暂时写在main.cpp当中

//改成了返回double值,因为现在要返回t值了
double hit_sphere(const point3& center, double radius, const ray& r) {
	vec3 oc = r.origin() - center;
	auto a = dot(r.direction(), r.direction());
	auto b = 2.0 * dot(oc, r.direction());
	auto c = dot(oc, oc) - radius * radius;
	auto discriminant = b * b - 4 * a * c;
	if (discriminant < 0) {
		return -1.0; //相当于不做处理
	}
	else {
		return (-b-sqrt(discriminant))/ (2.0 * a);
	}
}

//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r) {
	auto t = hit_sphere(point3(0, 0, -1), 0.5, r);
	if (t > 0.0) {
		vec3 N = unit_vector(r.at(t) - vec3(0, 0, -1));
		return 0.5 * color(N.x() + 1, N.y() + 1, N.z() + 1);  //法线颜色映射,[-1,1]->[0,1]
	}

	vec3 unit_direction = unit_vector(r.direction());
	t = 0.5 * (unit_direction.y() + 1.0);  //[-1,1]->[0,1])
	//y方向值越小越接近白色,越大越接近天蓝色
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); //lerp,后面那个是天蓝色
}
```

此时运行的结果如下：

![image-20220304200248921](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304200248921.png)

------

### 2.简化光线求交代码

在这里简化基于下面两条原则：

- （1）一根向量跟自己点乘，相当于求squared length（上文有相关函数。）
- （2）对于求根公式来说，可以令b=2h，则此时求根公式可以简化如下：

$$
\begin{aligned}
& \frac{-b \pm \sqrt{b^{2}-4 a c}}{2 a} \\
= & \frac{-2 h \pm \sqrt{(2 h)^{2}-4 a c}}{2 a} \\
= & \frac{-2 h \pm 2 \sqrt{h^{2}-a c}}{2 a} \\
= & \frac{-h \pm \sqrt{h^{2}-a c}}{a}
\end{aligned}
$$

简化后的光线求交代码如下：

```c++
//采用了简化后的求根公式，暂时写在main.cpp当中
double hit_sphere(const point3& center, double radius, const ray& r) {
	vec3 oc = r.origin() - center;
	auto a = r.direction().length_squared();
	auto half_b = dot(oc, r.direction());
	auto c = oc.length_squared() - radius * radius;
	auto discriminant = half_b * half_b - a * c;
	if (discriminant < 0) {
		return -1.0; //相当于不做处理
	}
	else {
		return (-half_b-sqrt(discriminant))/a;
	}
}
```

------



### 3.在场景当中渲染更多的球体

比较简单的想法是我们声明一个Sphere的数组，但其实**更好的做法是利用抽象类，任何可能与光线求交的东西实现时都继承这个类, 并且让球以及球列表也都继承这个类，**将这个类命名为`hittable`；

- 这个类会有一个`hit`函数，传入的参数是一个光线，许多光线追踪器为了便利, 加入了一个区间 $tmin<t<tmax$ 来判断相交是否有效。对于一开始的光线来说, 这个t值总是正的(所以tmin=0，tmax=MAXFLOAT), 但加入这部分对代码实现的一些细节有着不错的帮助。
- 是否需要在每次求交的时候都计算法向？
  - 不需要。只需要计算离射线原点最近的那个交点的法向就行了, 而后面的东西会被遮挡。

`hittable`抽象类及其所在文件如下所示：

```c++
//hittable.h
#ifndef HITTABLE_H
#define HITTABLE_H

#include "ray.h"

struct hit_record {
	point3 p;
	vec3 normal;
	double t;
};

class hittable {
public:
	virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec) const = 0;
};
#endif

```

此时的球体类如下(**注意：有些系数2可以消去，所以在这里顺便把系数也消去以简化运算**)：

```c++
//sphere.h
#ifndef SPHERE_H
#define SPHERE_H

#include "hittable.h"
#include "vec3.h"

class sphere :public hittable {
public:
	sphere(){}
	sphere(point3 cen,double r):center(cen),radius(r){}
	virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec)const override;
public:
	point3 center;
	double radius;
};

bool sphere::hit(const ray& r, double t_min, double t_max, hit_record& rec) const {
	vec3 oc = r.origin() - center;
	auto a = r.direction().length_squared();
	auto half_b = dot(oc, r.direction());
	auto c = oc.length_squared() - radius * radius;
	auto discriminant = half_b * half_b - a * c;
	if (discriminant < 0) return false;
	auto sqrtd = sqrt(discriminant);

	//Find the nearest root that lies in the acceptable range.
	auto root = (-half_b - sqrtd) / a; //root求解出来的是最近的hit point对应的t值
	if (root<t_min || root>t_max) {
		root = (-half_b + sqrtd) / a;  //小的解不符合要求,选择大的解再试试
		if (root<t_min || root>t_max) return false;
	}
	rec.t = root;
	rec.p = r.at(rec.t);
	rec.normal = (rec.p - center) / radius;  //相当于归一化了
	return true;
}

#endif // !SPHERE_H
```

**注意这里判断相交时先验证小的解，如果小的解不满足条件，则可能相机在物体内部，这个解就不能用了，所以要用另一个解再判断**

接下来的这段引入下述链接中作者的补充（引入了一些C++11 的新特性）

[Ray Tracing in One Weekend V3.0中文翻译（上） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/128582904)

------



### 4.解决法线的两面问题

- 如果按照我们刚才的代码，如果发射的光线起点在物体内部，则法线方向是指向外侧；如果发射的光线起点在物体外部，则法线方向也是指向外部。
- **我们也可以让法向向量与射线方向大致相反, 即如果射线从外部射向球面, 则法向量朝外, 若射线从内部射向球面, 法向量向着球心。**如下图所示：

<img src="https://pic4.zhimg.com/80/v2-f3be9d1529ba6a7d923f9c4e3533aba3_720w.jpg" alt="img" style="zoom:80%;" />



思考该使用哪种方式至关重要，尤其是对于双面材质来说。例如一张双面打印的A4纸, 或者玻璃球这样的同时具有内表面和外表面的物体。

- （1）如果我们使用第一种方式,即无论如何让法线指向外侧,则可以**利用光线方向与法线方向做点乘(相当于计算cos值的正负)**来判断是内侧射出还是外侧射入：

```c++
if (dot(ray_direction, outward_normal) > 0.0){  //说明和指向外的法线夹角是锐角，此时光线也是指向外的，因此是从里面打出去的
    // ray is inside the sphere
    ...
} 
else {
    // ray is outside the sphere
    ...
}
```

- （2）如果我们永远让法线方向与入射方向相反, 我们就不用去用点乘来判断射入面是内侧还是外侧了, **但相对的, 我们需要用一个变量储存射入面的信息**:

```cpp
bool front_face;  //记录是否是正面，也就是“外侧”的面，法线指向外
if (dot(ray_direction, outward_normal) > 0.0) {
    // ray is inside the sphere
    normal = -outward_normal;
    front_face = false;
}
else {
    // ray is outside the sphere
    normal = outward_normal;
    front_face = true;
}
```

采取哪种策略, 关键在于想把这部分放在着色阶段还是几何求交的阶段。在本项目中，材质数量要多于几何体的种类数量，所以我们将在几何部分先判别射入面是内侧还是外侧，也就是第（2）种策略【注：直接写第（2）种策略就行，正常是要求这个交点是在外还是在内的，方便后续着色】。

在`hit_record`类中引入front_face变量，需要修改的部分如下：

```c++
struct hit_record
{
	float t;
	vec3 p;
	vec3 normal;
	bool front_face; //判断是哪一面,如果是物体内部的则这一项是false

	inline void set_face_normal(const ray& r, const vec3& outward_normal) //新增函数：从里面打出的要反转法线
	{
		front_face = dot(r.direction(), outward_normal) < 0;
		normal = front_face ? outward_normal : -outward_normal;
	}
};
```

而在sphere类的hit函数当中，我们会对相交的是内侧还是外侧进行判断：

```c++
bool sphere::hit(const ray& r, float tmin, float tmax, hit_record& rec)const
{
	//...与之前的一样
	rec.t = root;
	rec.p = r.at(rec.t);
	//原来的：rec.normal = (rec.p - center) / radius;  //相当于归一化了
	//现在:如果是从内侧打的,对法线反转
	vec3 outward_normal = (rec.p - center) / radius;
	rec.set_face_normal(r, outward_normal);
	return true;
}
```

------



### 5.加入存放物体的列表

继续参考上述文章的链接：[Ray Tracing in One Weekend V3.0中文翻译（上） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/128582904)

注：这里要用到一些C++ 11的新特性，会在后面进行补充。

```c++
//hittable_list.h
#ifndef HITTABLE_LIST_H
#define HITTABLE_LIST_H

#include "hittable.h"
#include <vector>
using std::shared_ptr;
using std::make_shared;

class hittable_list :public hittable {
public:
	hittable_list(){}
	hittable_list(shared_ptr<hittable>object) { add(object); }

	void clear() { objects.clear(); }
	void add(shared_ptr<hittable> object) {
		objects.push_back(object);
	}
	virtual bool hit(
		const ray& r, double t_min, double t_max, hit_record& rec) const override;
public:
	std::vector<shared_ptr<hittable>> objects;
};

bool hittable_list::hit(const ray& r, double t_min, double t_max, hit_record& rec) const {
	hit_record temp_rec;
	bool hit_anything = false;
	auto closest_so_far = t_max;

	for (const auto& object: objects) {
		if (object->hit(r, t_min, closest_so_far, temp_rec)) {
			hit_anything = true;
			closest_so_far = temp_rec.t;  //更新t_max的值，因为我们此时只需要记录最近的交点，所以大于更新后的t_max的值就不考虑了。
			rec = temp_rec;  //最终rec里面会记录最近的交点的信息
		}
	}
	return hit_anything;
}

#endif // !HITTABLE_LIST_H
```

`hittable_list`类用了两种C++的特性:`vector`和`shared_ptr`:

- `shared_ptr<type>`是指向一些已分配内存的类型的指针。每当你将它的值赋值给另一个智能指针时, 物体的引用计数器就会+1。当智能指针离开它所在的生存范围(例如代码块或者函数外), 物体的引用计数器就会-1。一旦引用计数器为0, 即没有任何智能指针指向该物体时, 该物体就会被销毁;

- 一般来说, 智能指针首先由一个刚刚新分配好内存的物体来初始化:

  ```cpp
  shared_ptr<double> double_ptr = make_shared<double>(0.37);
  shared_ptr<vec3>   vec3_ptr   = make_shared<vec3>(1.0,2.0,3.0);
  shared_ptr<sphere> sphere_ptr = make_shared<sphere>(vec3(0,0,0), 1.0);
  ```

`make_shared<thing>(thing_constructor_params ...)`为指定的类型分配一段内存, 使用你指定的构造函数与参数来创建这个类, 并返回一个智能指针`shared_ptr<thing>`

使用C++的auto类型关键字, 可以自动推断`make_shared<type>`返回的智能指针类型, 于是我们可以把上面的代码简化为:

```cpp
auto double_ptr = make_shared<double>(0.37);
auto vec3_ptr   = make_shared<vec3>(1.0,2.0,3.0);
auto sphere_ptr = make_shared<sphere>(vec3(0,0,0), 1.0);
```

我们在代码中使用智能指针的目的是为了能让多个几何图元共享一个实例(举个例子, 一堆不同球体使用同一个纹理材质), 并且这样内存管理比起普通的指针更加的简单方便。

std::shared_ptr在头文件`<memory>`中

```cpp
#include<memory>
```

- 补充：关于`const override`:
  - [C++11 之 override - 飞鸢逐浪 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xinxue/p/5471708.html)

------



### 6.定义常用常数和工具头文件

定义一个头文件，用来存放一些我们需要的量和一些数学公式，目前需要的如下：

```c++
//rtweekend.h
#ifndef RTWEEKEND_H
#define RTWEEKEND_H

#include <cmath>
#include <limits>
#include <memory>

//Usings
using std::shared_ptr;
using std::make_shared;
using std::sqrt;

//Constants
const double infinity = std::numeric_limits<double>::infinity();
const double pi = 3.1415926535897932385;

//Utility functions
inline double degrees_to_radians(double degrees) {
	return degrees * pi / 180.0;
}

//Common headers
#include "ray.h"
#include "vec3.h"

#endif // !RTWEEKEND_H
```

此时修改main函数为如下：

```c++
#include <iostream>
#include <fstream>
#include "rtweekend.h"
#include "color.h"
#include "hittable_list.h"
#include "sphere.h"

using namespace std;

//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r, const hittable& world) {
	hit_record rec;
	if (world.hit(r, 0, infinity, rec)) {  //新增逻辑，判断所有hittable的物体的最近相交点
		return 0.5 * (rec.normal + color(1, 1, 1));
	}

	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);  
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); //lerp,后面那个是天蓝色
}


int main()
{
	ofstream outfile;
	string filename = "output.ppm";
	outfile.open(filename);
	if (!outfile)
	{
		cerr << "no file exists!" << endl;
	}
	//Image
	const auto aspect_ratio = 16.0 / 9.0;
	int image_width = 400;
	int image_height = static_cast<int>(image_width/aspect_ratio);
	
	//World，新增对场景物体的描述
	hittable_list world;
	world.add(make_shared<sphere>(point3(0, 0, -1), 0.5));
	world.add(make_shared<sphere>(point3(0, -100.5, -1), 100)); //大的球体,直接当底座了

	//Camera
	double viewport_height = 2.0;
	double viewport_width = aspect_ratio * viewport_height;
	double focal_length = 1.0;
	
	auto origin = point3(0,0,0);
	auto horizontal = vec3(viewport_width, 0, 0);
	auto vertical = vec3(0, viewport_height, 0);
	auto lower_left_corner = origin - horizontal / 2 - vertical / 2 - vec3(0, 0, focal_length);

	//Render
	outfile << "P3\n" << image_width << " " << image_height << "\n255\n"; //根据格式,输出nx行和ny列
	for (int j = image_height - 1; j >= 0; j--)
	{
		std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;  //添加一个进度条
		for (int i = 0; i < image_width; i++)
		{
			auto u = double(i) / (image_width - 1);
			auto v = double(j) / (image_height - 1);
			ray r(origin, lower_left_corner - origin + u * horizontal + v * vertical);
			color pixel_color = ray_color(r,world); //注：这个API的参数有改动
			write_color(outfile, pixel_color);
		}
	}
	outfile.close();
	std::cerr << "\nDone.\n";
}
```

此时实现的效果如下：

![image-20220304224847332](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304224847332.png)

- 彩色的部分是因为此时最近相交点判定在了小球体上,所以按小球体的法线方向上色;
- 绿色的部分是因为最近相交点判定在了大球体上,而大球体半径较大,法线认为沿y方向,所以显示绿色;
- 背景部分由于没有检测相交,因此保持背景色.

从结果来看，可以看到锯齿现象是比较严重的，因此我们接下来会**进行抗锯齿的相关操作。**

------



## Chapter 7：反走样（抗锯齿）

**真实世界中的摄像机拍摄出来的照片是没有像素状的锯齿的。**因为边缘像素是由背景和前景混合而成的。

理论上，我们也可以在程序中**简单的对每个边缘像素多次采样取平均达到类似的效果**。（个人注：这样应该会起到一种“柔和”的效果）但这里**不会使用分层采样**。尽管对某些光线追踪器来说分层采样是很关键的部分, 但是对于我们写的这个小光线追踪器并不会有什么很大的提升。（**todo：目前没太分层采样应该是什么，不过既然这个光追程序没用就先不写了。**进一步的反走样技术可以参考MSAA）

需要设计一个`Camera`类，方便后续能对摄像机有额外的操作。

### 1.一些随机数方法

- 理论上，我们需要随机数生成器**来生成真正的随机数**
  - 默认来说这个函数应该返回0≤r<1的随机实数。注意**这个范围取不到1是很重要的。**

- 有两种方式来实现这种随机效果

  - 1.使用`<cstdlib>`中的rand()函数，这个函数本身会返回一个0到RAND_MAX之间的数

  - ```c++
    //rtweekend.h
    #include <cstdlib>
    ...
    
    inline float random_double() 
    {
        // Returns a random real in [0,1).
        return rand() / (RAND_MAX + 1.0);
    }
    
    inline float random_double(float min, float max) 
    {
        // Returns a random real in [min,max).
        return min + (max-min)*random_double();
    }
    ```

  - 2.传统C++并没有随机数生成器, **但是新版C++中的`<random>`头实现了这个功能**(某些专家觉得这种方法不太完美)。如果你想使用这种方法, 你可以参照下面的代码:

  - ```c++
    //rtweekend.h
    #include <functional>
    #include <random>
    
    inline float random_double() {
        static std::uniform_real_distribution<double> distribution(0.0, 1.0);
        static std::mt19937 generator;
        return distribution(generator);
    }
    ```
    
    在这里我们使用第一种方式，直接调用这个函数就可以生成随机数了。

------



### 2.对像素多次采样

**对于给定的像素, 我们发射多条射线进行多次采样。然后我们对颜色结果求一个平均值:**

![image-202203042231422294](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-202203042231422294.png)

#### （1）创建`Camera`类

```c++
//camera.h
#ifndef CAMERA_H
#define CAMERA_H

#include "rtweekend.h"

class camera {
public:
	camera() {
		double aspect_ratio = 16.0 / 9.0;
		double viewport_height = 2.0;
		double viewport_width = aspect_ratio * viewport_height;
		double focal_length = 1.0;

		origin = point3(0, 0, 0);
		horizontal = vec3(viewport_width, 0.0, 0.0);
		vertical = vec3(0.0, viewport_height, 0.0);
		lower_left_corner = origin - horizontal / 2 - vertical / 2 - vec3(0, 0, focal_length);
	}

	ray get_ray(double u, double v) const {  //打到像素坐标为（u，v）的光线
		return ray(origin, lower_left_corner + u * horizontal + v * vertical - origin);
	}

private:
	point3 origin;
	point3 lower_left_corner;
	vec3 horizontal;
	vec3 vertical;
};

#endif // !CAMERA_H
```

为了完成后面的步骤，需要先在`rtweekend.h`头文件中添加一个clamp函数：
```c++
//rtweekend.h
//用来把x的值限制在[min,max]范围里
inline double clamp(double x, double min, double max) {
	if (x < min) return min;
	if (x > max) return max;
	return x;
}
```



- 重写在color.h文件当中的`write_color()`函数，使其能对多次采样的结果取平均值。我们不会在每次发出射线采样时都计算一个0-1之间的颜色值, 而是一次性把所有的颜色都加在一起, 然后最后只需要简单的除以采样点个数,就可以得到最后的颜色了。

```c++
#include "rtweekend.h"

void write_color(std::ostream& out, color pixel_color, int samples_per_pixel) {
	double r = pixel_color.x();
	double g = pixel_color.y();
	double b = pixel_color.z();

	//Divide the color by the number of samples
	double scale = 1.0 / samples_per_pixel;
	r *= scale, g *= scale, b *= scale;
	//Write the translated [0,255] value of each color component
	out << static_cast<int>(256 * clamp(r, 0.0, 0.999)) << ' '
		<< static_cast<int>(256 * clamp(g, 0.0, 0.999)) << ' '
		<< static_cast<int>(256 * clamp(b, 0.0, 0.999)) << '\n';
}
```



此时main也要被重写：

```c++
int main()
{
	ofstream outfile;
	string filename = "output.ppm";
	outfile.open(filename);
	if (!outfile)
	{
		cerr << "no file exists!" << endl;
	}
	//Image
	const auto aspect_ratio = 16.0 / 9.0;
	int image_width = 400;
	int image_height = static_cast<int>(image_width/aspect_ratio);
	const int samples_per_pixel = 100;

	//World
	hittable_list world;
	world.add(make_shared<sphere>(point3(0, 0, -1), 0.5));
	world.add(make_shared<sphere>(point3(0, -100.5, -1), 100)); //大的球体,直接当底座了

	
	//Camera
	camera cam;

	//Render
	outfile << "P3\n" << image_width << " " << image_height << "\n255\n"; //根据格式,输出nx行和ny列
	for (int j = image_height - 1; j >= 0; j--)
	{
		std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;  //添加一个进度条
		for (int i = 0; i < image_width; i++)
		{
			color pixel_color(0, 0, 0);
			for (int s = 0; s < samples_per_pixel; s++) {  //多次采样的思想
				auto u = double(i + random_double()) / (image_width - 1);
				auto v = double(j + random_double()) / (image_height - 1);
				ray r = cam.get_ray(u, v);
				pixel_color += ray_color(r, world);
			}
			
			write_color(outfile, pixel_color, samples_per_pixel);
		}
	}
	outfile.close();
	std::cerr << "\nDone.\n";
}
```

- 反走样操作前后的对比如下：

![image-20220304235239610](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220304235239610.png)

------

接下来，我们会：

[8. Diffuse Material 漫反射材质]
[9. Metal 金属材质]
[10. Dielectric 绝缘体材质]
[11. Positionable Camera 可自定义位置的摄像机]
[12. Defocus Blur 对焦模糊]
[13. Where Next? 接下来学什么?]

参考链接：

[Ray Tracing in One Weekend V3.0中文翻译（上） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/128582904)

------



## Chapter 8：漫反射材质

从这里开始，我们会逐步实现不同材质的效果，首先在这一节里会实现**漫反射材质。**

在这里我们把材质和物体设计成两个类, 这样就可以将材质赋值给物体类的成员变量,而大多数的渲染器也是这样做的。（另一种是把物体和材质紧密结合）。

漫反射材质**不仅仅接受其周围环境的光线, 还会在散射时将自己的颜色也考虑进去（其实参考Lambert模型即可）。**光线射入漫反射材质后, 其反射方向是随机的。所以如果我们为下面这两个漫发射的球射入三条光线, 光线都会有不同的反射角度:

![image-20220305094551245](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305094551245.png)

相对来说，光线更有可能被吸收而不是反射。表面越暗，就越有可能吸收光线（所以才这么黑）。

------



### 1.简单漫反射材质（Lazy hack，主要看2部分的真正的漫反射模型）

我们使用random算法生成随机的反射方向, 就能让其看上去像一个粗糙不平的漫反射材质。这里我们采用最简单的算法就能得到一个理想的漫反射表面。（**在数学上近似Lambertian，不过是lazy hack，后面会对其进行修复**）

（这一部分简单看看即可）

[Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html#diffusematerials)

- 有两个单位半径圆相切于光线与表面的交点p，这两个球体的球心分别为$p+\vec{N}$和$p-\vec{N}$,$\vec{N}$是表面的法线方向。球心坐标为$p-\vec{N}$的球体被认为在表面内侧，而球心坐标为$p+\vec{N}$的球体则被认为在表面外侧。选择和光线原点位于表面同一侧的那个单位球, 并从球中随机选取一点 s , 向量 （s-p）就是我们要求的反射光线的方向:

![image-20220305100045504](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305100045504.png)

（其实就是在与交点相切的靠外侧的单位球中随机选一个方向作为漫反射方向）

- 此时用到了一个很不错的技巧：**否定法(rejection method)。**首先, 在一个xyz取值范围为-1到+1的单位立方体中选取一个随机点, 如果这个点在球外就重新生成直到该点在球内:

```c++
//在vec3.h的vec3类当中补充以下内容：
inline static vec3 random(){
	return vec3(random_double(), random_double(), random_double());//调用我们在之前的头文件中定义的函数
}
inline static vec3 random(double min, double max){
	return vec3(random_double(min,max), random_double(min, max), random_double(min, max));//调用我们在之前的头文件中定义的函数
}

//在外面补充一个函数
//返回在单位球体里的一个任意点,其与球心的距离可能<1
inline vec3 random_in_unit_sphere() {
	while (true)
	{
		auto p = vec3::random(-1, 1);
		if (p.length_squared() >= 1) continue;//如果向量长度>=1，说明不在单位球体内，重新生成
		return p;
	}
}
```

此时ray_color函数也要进行修改，使用随机漫反射光线生成算法：

```c++
//main.cpp
color ray_color(const ray& r,const hitable& world) //这个函数会根据不同的光路方向返回不同的颜色,从而"渲染"在屏幕上
{
	hit_record rec;
	if (world.hit(r, 0, infinity, rec)) //infinity在头文件rtweekend.h当中有所定义
	{
		point3 target = rec.p + rec.normal + random_in_unit_sphere(); //此时定位位置一定是在与法线相切的单位圆内部,这里的.normal是已经做了归一化的，也就是上面那张图的s点
		return 0.5 * ray_color(ray(rec.p,target-rec.p),world); //这里就是只考虑最近相交点的法线了,递归打出这根随机出来的光线，此处假定有50%的光被吸收，剩下的则从入射点开始取随机方向再次发射一条射线。
	}

	//否则返回背景色
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5*(unit_direction.y() + 1.0);
	return (1.0 - t)*vec3(1.0, 1.0, 1.0) + t * vec3(0.5, 0.7, 1.0);
}
```

#### （1）解决递归问题

从上述ray_color函数中可以看出，这个函数会递归调用自己，那么**可能就会造成栈溢出**，所以我们要限制递归深度来解决无限递归的问题：

```c++
//main.cpp
color ray_color(const ray& r,const hitable& world, int depth) //这个函数会根据不同的光路方向返回不同的颜色,从而"渲染"在屏幕上
{
    hit_record rec;
	if (depth <= 0) return vec3(0.0, 0.0, 0.0); //防止栈溢出
	
	if (world.hit(r, 0, infinity, rec)) //infinity在头文件rtweekend.h当中有所定义
	{
		vec3 target = rec.p + rec.normal + random_in_unit_sphere(); //此时定位位置一定是在与法线相切的单位圆内部
		return 0.5 * ray_color(ray(rec.p,target-rec.p),world,depth-1); 
	}

	//否则返回背景色
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5*(unit_direction.y() + 1.0);
	return (1.0 - t)*vec3(1.0, 1.0, 1.0) + t * vec3(0.5, 0.7, 1.0);
}
```

> 注：这里解释一下递归的问题。每轮递归的时候有这样几种可能：
>
> - 如果判断没有hit到物体，此时`if world.hit`返回false，函数会返回背景色，那么此时在递归回来的时候就相当于每有hit一次就有50%的光被吸收（回忆递归的出栈过程），直到目标点返回颜色。
> - 如果一直有hit到物体，则到最大递归深度的时候返回0.0，即黑色，最后递归回来显示的也会是黑色，这样也比较合理，说明光线一直在弹来弹去，看结果图对应显示黑色的区域就懂了（对应类似shadowing区域）。

对应的，main函数修改如下：

```c++
int main()
{
    //...前面的不变
    const int max_depth = 50;
    //...中间的不变
    for (int j = image_height - 1; j >= 0; j--)
	{
		std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;  //添加一个进度条
		for (int i = 0; i < image_width; i++)
		{
			color pixel_color(0, 0, 0);
			for (int s = 0; s < samples_per_pixel; s++) {
				auto u = double(i + random_double()) / (image_width - 1);
				auto v = double(j + random_double()) / (image_height - 1);
				ray r = cam.get_ray(u, v);
				pixel_color += ray_color(r, world, max_depth); //修改这一行
			}
			
			write_color(outfile, pixel_color, samples_per_pixel);
		}
	}
}
```

此时渲染的结果如下：

![image-20220305102812785](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305102812785.png)

可以看到，漫反射材质大致出来了，但整体比较糊（因为我们的图像大小为200*100，节省性能，而上面的截图进行了放大处理），同时**整体比较暗，所以需要进行一步gamma校正**。

------



#### （2）Gamma校正

在数字图像处理当中，我们知道当整体较暗的时候，可以使用幂次变换将亮度调高（α的值为(0,1)），这里我们取0.5，此时在write_color函数修改如下：

```c++
//color.h
void write_color(std::ostream& out, color pixel_color, int samples_per_pixel) {
	double r = pixel_color.x();
	double g = pixel_color.y();
	double b = pixel_color.z();

	//Divide the color by the number of samples
	double scale = 1.0 / samples_per_pixel;
	//Gamma correct
	r = sqrt(scale * r); //伽马校正，gamma=2.0，相当于提高整体亮度
	g = sqrt(scale * g);
	b = sqrt(scale * b);

	//Write the translated [0,255] value of each color component
	out << static_cast<int>(256 * clamp(r, 0.0, 0.999)) << ' '
		<< static_cast<int>(256 * clamp(g, 0.0, 0.999)) << ' '
		<< static_cast<int>(256 * clamp(b, 0.0, 0.999)) << '\n';
}
```

此时修改之后的效果图如下：

![image-20220305103656392](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305103656392.png)

可以看到，漫反射的效果已经很不错了。

------



##### （3）修正浮点数精度带来的问题

这里还有个不太重要的潜在bug。**有些物体反射的光线会在t=0时再次击中自己。**然而由于精度问题, 这个值可能是t=-0.000001或者是t=0.0000000001或者任意接近0的浮点数。所以我们要忽略掉0附近的一部分范围, 防止物体发出的光线再次与自己相交。【译注: 小心自相交问题】

```c++
//main.cpp
if (world.hit(r, 0.0001, infinity, rec)) //这个是在主程序的ray_color函数当中修改，将刚反射就与自己相交的光线剔除
{
	vec3 target = rec.p + rec.normal + random_in_unit_sphere(); 
	return 0.5 * color(ray(rec.p,target-rec.p),world,depth-1); 
}
```

这样我们就能避免**阴影痤疮(shadow ance)**的产生。（更多与之有关的内容可以查看链接https://digitalrune.github.io/DigitalRune-Documentation/html/3f4d959e-9c98-4a97-8d85-7a73c26145d7.htm）

------



### 2.真正的漫反射模型

- 在我们刚才的模型当中，由于使用否定法在表面法线附近的单位球体内找随机点，因此**找到的方向方向大概率和法线方向接近，而很小的概率会沿光线射入方向返回**。这个分布律的表达式有一个$cos^3(\phi)$的系数, 其中 $\phi$  是反射光线距离法向量的夹角。这样当光线从一个离表面很小的角度射入时, 也会散射到一片很大的区域, 不过对最终颜色值的影响也会更低。

- 然而, 事实上的lambertian的分布律并不是这样的, 它的系数是 $cos(\phi)$。**真正的lambertian散射后的光线距离法向比较近的概率会更高, 但是分布律会更加均衡。这是因为我们选取的是单位球面上的点。**
  - **我们可以通过在单位球内选取一个随机点, 然后将其单位化来获得该点。**【注: 下面的代码却用了极坐标的形式】

上面这两句话比较难以理解，暂时不需要细究。为了便于理解, 简单来说这种方法与上一节的方法都选取了一个随机方向的向量, **不过一种是从单位球体内取的, 其长度是随机的, 另一种是从单位球面上取的, 长度固定为单位向量长度。**为什么要采取单位球面并不是能很直观的一眼看出，但我们还是按照lambertian的分布律，在单位球面上找到一个随机点：

```c++
//vec3.h
//新增函数random_unit_vector(),用极坐标来表示生成点在球体表面上，这个应该是一个球面取随机点的方法，我记得Leetcode有题
//version 1
vec3 random_unit_vector()
{
    auto a = random_double(0, 2*pi);
    auto z = random_double(-1, 1);
    auto r = sqrt(1 - z*z);
    return vec3(r*cos(a), r*sin(a), z);//注意到，vec3三个分量的平方和为1，一定在球面上
}

//注：在新版里这个函数被改成了这样，也就是直接对Lazy hack做法求出的随机方向随机长度的向量进行归一化，也是可以的
//version 2
//返回在单位球体里的一个任意点,其与球心的距离可能<1
inline vec3 random_in_unit_sphere() {
	while (true)
	{
		auto p = vec3::random(-1, 1);
		if (p.length_squared() >= 1) continue;//如果向量长度>=1，说明不在单位球体内，重新生成
		return p;
	}
}

//返回随机方向的归一化的向量
vec3 random_unit_vector() {
	return unit_vector(random_in_unit_sphere());
}
```

![image-20220305111028151](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305111028151.png)

所以，前面的ray_color函数也要有所更改：

```c++
//main.cpp
//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r, const hittable& world, int depth) {
	hit_record rec;  //rec会作为引用在函数里被修改

	//限制递归溢出
	if (depth <= 0) return color(0.0, 0.0, 0.0);

	if (world.hit(r, 0.0001, infinity, rec)) {
		vec3 target = rec.p + rec.normal + random_unit_vector();
		return 0.5 * ray_color(ray(rec.p, target - rec.p),world,depth-1);  //生成的光线的方向为target - rec.p，不需要归一化，
	}

	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);  
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); //lerp,后面那个是天蓝色
}
```

得到的漫反射图如下：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230418203902282.png" alt="image-20230418203902282" style="zoom:67%;" />

和之前的进行对比，会发现：

1.阴影部分少了
2.大球和小球都变亮了

这些变化都是**由散射光线的单位规整化引起的**，现在更少的光线会朝着法线方向散射。对于漫发射的物体来说, 他们会变得更亮。因为更多光线朝着摄像机反射。对于阴影部分来说, 更少的光线朝上反射, 所以小球下方的大球区域会变得更加明亮。（**这里如果理解的不是很透彻的话也不用担心，因为我们后面就会用这个正常的lambert表面模型来做。**）

==**todo：发现计算得到的反射光线并没有进行归一化，这样得到的相交之后的t值计算会不会不准确，这里是否存在问题？**==（应该是没什么问题的，虽然由于方向没有归一化会导致生成的解`t`不同，但在计算交点的时候本身也是用`o+td`算的，所以应该不影响，实测归一化前后也确实看不出差别。）

------



### 3.另一种启发式漫反射模型（不太用看）

另一种具有启发性的方法是, **为远离命中点的所有角度设置统一的散射方向，而不依赖于法线角度。**在使用lambertian漫发射模型前, 早期的光线追踪论文中大部分使用的都是这个方法:

```c++
vec3 random_in_hemisphere(const vec3& normal)
{
    vec3 in_unit_sphere = random_in_unit_sphere();
    if (dot(in_unit_sphere, normal) > 0.0) // In the same hemisphere as the normal
        return in_unit_sphere;
    else
        return -in_unit_sphere;
}
```

同样要修改color函数：

```c++
if (world.hit(r, 0.001, infinity, rec))
{
    vec3 target = rec.p + random_in_hemisphere(rec.normal);
    return 0.5 * ray_color(ray(rec.p, target - rec.p), world, depth-1);
}
```

效果如下：

![image-20220305112015462](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305112015462.png)

就目前而言，我们暂时还是考虑Lambertian模型。（也就是在2部分中介绍的真正的漫反射模型）

这里鼓励使用不同的漫反射模型来进行尝试。

------



## Chapter 9：金属材质

### 1.声明一个材质抽象类

该抽象类应该封装有两个功能：

1.生成散射后的光线(或者说它吸收了入射光线)
2.如果发生散射, 决定光线会衰减多少(attenuate)

```c++
//material.h
#ifndef MATERIAL_H
#define MATERIAL_H

#include "rtweekend.h"
//struct hit_record;  //记录一个神秘的报错,主要是对C++了解的太少,这行取消注释,注释下一行就会报hit_record不完整的错误,不清楚为什么，合理的推测写在了本节的trouble shooting部分里
#include "hittable.h"

class material {
public:
	/*
	* 散射虚函数
    参数：r_in:入射的光线， rec:hit的记录， 
	attenuation: 三维的衰减，scattered:散射后的光线
	*/
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered
	)const = 0;
};

#endif
```

在函数中使用`hit_record`作为传入参数, 就可以不用传入一大堆变量了。

------



### 2.描述光线与物体相交的数据结构

另外，`hit_record`类当中也增加了一个`shared_ptr`智能指针，来指向此时对应的材质。

```c++
//hittable.h
#include "rtweekend.h"  //随着头文件数量的增多，要避免出现循环引用头文件的问题
class material;

struct hit_record {
	point3 p;
	vec3 normal;
	shared_ptr<material> mat_ptr;  //新增：指向材质的指针
	double t;
	bool front_face;

	inline void set_face_normal(const ray& r, const vec3& outward_normal) {
		front_face = dot(r.direction(), outward_normal) < 0;
		normal = front_face ? outward_normal : -outward_normal;
	}
};
```

这里的材质信息会告诉我们**光线与物体表面是如何作用的。**

当光线射入一个表面(比如一个球体), hit_record中的材质指针会被球体的材质指针所赋值, 而球体的材质指针是在main()函数中构造时传入的。当ray_color()函数获取到hit_record时, 他可以找到这个材质的指针, 然后由材质的函数来决定光线是否发生散射, 如何散射。

根据上文的描述，我们必须在球体的构造函数和变量区域中加入材质指针, 以便之后传给`hit_record`。见下面的代码:

```c++
//sphere.h
class sphere :public hittable {
public:
	sphere(){}
	sphere(point3 cen,double r,shared_ptr<material> m)
		:center(cen),radius(r),mat_ptr(m){}  //构造函数新增对材质的赋值
	virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec)const override;
public:
	point3 center;
	double radius;
	shared_ptr<material> mat_ptr;  //hittable.h里面声明了material,因此不需要引入新的头文件
};

bool sphere::hit(const ray& r, float tmin, float tmax, hit_record& rec)const
{
	//和之前一样
	//...
    vec3 outward_normal = (rec.p - center) / radius;  //相当于归一化了
	rec.set_face_normal(r, outward_normal);
	rec.mat_ptr = mat_ptr;  //新增在相交点记录的结构体里加入材质的信息  
}
```

------



### 3.重写漫反射材质类

对于之前的漫反射材质，有两种情况：

- 1.光线发生散射, 每次散射衰减因子R
- 2.不发生散射，但是物体会吸收（1-R）的光线。

也可以理解成这两者的结合，所以此时可以写出Lambertian材质类：

```c++
//mateial.h
class lambertian :public material {
public:
	lambertian(const color& a) :albedo(a) {}

	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override {
		auto scatter_direction = rec.normal + random_unit_vector();
		scattered = ray(rec.p, scatter_direction);
		attenuation = albedo;
		return true;
	}
public:
	color albedo;  //会作为attenuation的量被scatter修改,依据lambert模型可以理解为kd
};
```

注意我们也可以让光线根据一定的概率p发生散射【注: 若判断没有散射, 光线直接消失】, 并使光线的衰减率(代码中的attenuation)为 albedo/p 。随你的喜好来。

注意，**上面的代码有个小问题**。如果生成的 random_unit_vector()正好和rec.normal完全相反，则两者之和会变成0，也就是散射的光线为0。这将导致后面出现糟糕的情况(无穷大和nan)，因此我们需要在传递条件之前拦截它。解决方案是在vec3类当中添加一个对0附近的向量的判断：

```c++
//写在vec3.h文件的vec3类里，新增一个函数
bool near_zero() const {
    //return true if the vector is close to zero in all dimensions
    const double s = 1e-8;
    return (fabs(e[0]) < s) && (fabs(e[1]) < s) && (fabs(e[2]) < s);
}
```

这个时候lambertian类的scatter函数就需要新增对0附近的向量的判断：

```c++
//mateial.h
class lambertian :public material {
public:
	lambertian(const color& a) :albedo(a) {}

	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override 
	{
		auto scatter_direction = rec.normal + random_unit_vector();

		//catch degenerate(恶化的) scatter direction，新增判断
		if (scatter_direction.near_zero()) {
			scatter_direction = rec.normal;
		}
		scattered = ray(rec.p, scatter_direction);
		attenuation = albedo;
		return true;
	}
public:
	color albedo;  //会作为attenuation的量被scatter修改,依据lambert模型可以理解为kd
};
```

#### trouble shooting

> **trouble-shooting**（不保真，todo：后面有新的体会回来改一下）：
>
> 在mateial.h中，出现了如下情况：
>
> ```c++
> //struct hit_record;  //记录一个神秘的报错,主要是对C++了解的太少,这行取消注释,注释下一行就会报hit_record不完整的错误,不清楚为什么
> #include "hittable.h"
> 
> //注意！！！！最终还是采用了struct hit_record;运行之后不报错但是不知道为啥
> ```
>
> 推测原因可能和这个链接有关：[(29条消息) C/C++_的不完整类型详解（参考了各位大佬整理下来，特此鸣谢）_c++ 不允许使用不完整的类型_原石小珂的博客-CSDN博客](https://blog.csdn.net/lovely_ke/article/details/82949556)，在这个头文件中使用了hit_record结构体的字段，但如果只是声明的话无法保证这些字段存在，因此会报不完整的错误。
>
> 其他参考：[(20 封私信 / 46 条消息) 不允许指针指向不完整的类类型是什么意思？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/362445246)
>
> ==**有趣的是，好像运行之后VS就不会报错了，鉴定为神秘的C++，真正的说明看后面把**==
>
> ==声明：这里出现那种一大堆的报错肯定是头文件引用有问题，导致循环引用了，后面注意一下==

------

### 4.镜面反射-金属材质

对于光滑的金属材质来说, 光线是不会像漫反射那样随机散射的, 而是产生类似镜面反射。我们用向量来具体推导出这种反射方向：

<img src="https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305144009752.png" alt="image-20220305144009752" style="zoom:67%;" />

我们想要求出红色的向量（ $\vec{r}$）的公式：

$\vec{r}=\vec{v}+2*\vec{b}$

其中，已知法线方向$\vec{N}$是一个单位向量，所以我们假设向量$\vec{v}$与水平方向的夹角为θ，则$\left|\vec{b}\right |=\left|\vec{v}\right|*sin(\theta)=-\left|\vec{v}\right|*cos(90°+\theta)$

所以$\left|\vec{b}\right |=-\vec{v}·\vec{N}$,又因为向量$\vec{b}$ 的方向与$\vec{N}$保持一致，所以$\vec{b}=(-\vec{v}·\vec{N})*\vec{N}$

可以推导出：

$\vec{r}=\vec{v}+2*\vec{b}=\vec{v}-2*(\vec{v}·\vec{N})*\vec{N}$

此时就可以写出反射的函数：

```c++
//vec3.h
//反射函数,参数分别是从外指向内的向量v,以及表面法线方向n
vec3 reflect(const vec3& v, const vec3& n) {
	return v - 2 * dot(v, n) * n;
}
```

此时，新建一个金属类：

```c++
// material.h
class metal :public material {
public:
	metal(const color& a) : albedo(a) {}
	
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override
	{
		vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
		scattered = ray(rec.p, reflected);
		attenuation = albedo;
		return (dot(scattered.direction(), rec.normal) > 0);  //反射的光线与法线方向不呈现锐角,不合理,舍弃
	}

public:
	color albedo;
};
```

对ray_color函数进行进一步的修改，以使其能够适应不同的材质：

```c++
//main.cpp
color ray_color(const ray& r, const hittable& world, int depth) {
	hit_record rec; 
	if (depth <= 0) return color(0.0, 0.0, 0.0);

	if (world.hit(r, 0.0001, infinity, rec)) {
		//对于不同材质进行判断
		ray scattered;
		color attenuation;
		if (rec.mat_ptr->scatter(r, rec, attenuation, scattered)) {
			return attenuation * ray_color(scattered, world, depth - 1);
		}
	}

	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);  
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0); 
}
```

==**这里面的各种头文件互相引用可能会出问题，有需要手动修改整理一下引用关系。**==

------



### 5.场景修改

现在修改我们的场景，main函数中有关场景的部分修改如下：

```c++
//main
int main()
{
    //...前面的部分都不变
    hitable_list world;
	
	//World
	auto material_ground = make_shared<lambertian>(vec3(0.8, 0.8, 0.0));
	auto material_center = make_shared<lambertian>(vec3(0.7, 0.3, 0.3));
	auto material_left = make_shared<metal>(vec3(0.8, 0.8, 0.8));
	auto material_right = make_shared<metal>(vec3(0.8, 0.6, 0.2));

	world.add(make_shared<sphere>(vec3(0.0, -100.5, -1.0), 100.0, material_ground));
	world.add(make_shared<sphere>(vec3(0.0, 0.0, -1.0), 0.5, material_center));
	world.add(make_shared<sphere>(vec3(-1.0, 0.0, -1.0), 0.5, material_left));
	world.add(make_shared<sphere>(vec3(1.0, 0.0, -1.0), 0.5, material_right));


	//引入一个相机
	camera cam;
    //...后面的部分都不变
}
```



#### ==一个务必要注意的报错信息-缺少类型信息（缺省int），或者报一大堆乱七八糟的错==

https://blog.csdn.net/aa4790139/article/details/8235823

`缺少类型信息-缺省int`，这种错误第一次碰到，可能会报一大堆错误，让人觉得代码出了很多问题而且很难调试（比如会报形参个数不对的错误，但实际形参个数是正确的），**原因是在于头文件之间互相引用**，此时必须要注意**不要让A头文件包含B且B头文件包含A（即使两者都写了`#ifndef`语句也是会报错的！！！（调试五十分钟发现是这个错误））**

**解决方案：直接把需要的类声明进来就行，如下：**

```c++
class material;//注意这里不能包含material.h,因为material.h里面已经包含了这个头文件，会报难以理解和修复的错误
struct hit_record
{
	//...跟之前一样
};
```

最终输出的结果如下：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230418235539149.png" alt="image-20230418235539149" style="zoom:67%;" />

------



### 6.fuzzy效果

我们还可以给反射方向**加入一点点随机性, 只要在算出反射向量后**, 在其终点为球心的球内随机选取一个点作为最终的终点，如下图所示：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419155251708.png" alt="image-20230419155251708" style="zoom:67%;" />

当然上图这个虚线的球越大, 金属看上去就更加模糊(fuzzy, 或者说粗糙)。所以我们这里引入一个变量来表示模糊的程度(fuzziness)(所以当fuzz=0时不会产生模糊)。如果fuzz, 也就是随机球的半径很大, 光线可能会散射到物体内部去。这时候我们可以认为物体吸收了光线。

对metal类的修改如下：

```c++
//material.h
class metal :public material {
public:
	metal(const color& a,double f) : albedo(a),fuzz(f<1? f : 1) {}
	
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override
	{
		vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
		scattered = ray(rec.p, reflected + fuzz * random_in_unit_sphere());  //增加对模糊情况的判断，通过对反射光实现一定范围的偏移得到最终的反射光线
		attenuation = albedo;
		return (dot(scattered.direction(), rec.normal) > 0);  //反射的光线与法线方向不呈现锐角,不合理,舍弃
	}

public:
	color albedo;
	double fuzz; //模糊度
};
```

此时只要在main函数里面对金属材质赋予一个fuzzy值就可以了：

```c++
//main函数
auto material_ground = make_shared<lambertian>(vec3(0.8, 0.8, 0.0));
auto material_center = make_shared<lambertian>(vec3(0.7, 0.3, 0.3));
auto material_left = make_shared<metal>(vec3(0.8, 0.8, 0.8),0.3);  //对金属物体增加模糊系数,这个对应左面那个灰色的球
auto material_right = make_shared<metal>(vec3(0.8, 0.6, 0.2),1.0);  //对应右面那个黄色的球
```

此时输出的结果如下：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419161200738.png" alt="image-20230419161200738" style="zoom: 67%;" />



------



## Chapter 10：绝缘体材质（折射相关）

透明的材料, 例如水, 玻璃, 和钻石都是绝缘体。当光线击中这类材料时, **一条光线会分成两条, 一条发生反射, 一条发生折射。**我们会采取这样的策略: 每次光线与物体相交时, **要么反射要么折射**, 一次只发生一种情况,随机选取。反正最后采样次数多, 会给这些结果取个平均值。

- 补充:一种调试手段:将一条光线画出来,来查看其效果(但这里暂时我没有用这种方式调试)



### 1.折射法则(Snell's Law)

$η⋅sinθ=η^′⋅sinθ^′$

四个变量的解释如下（看左图）:

![image-20230919145938035](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230919145938035.png)

$ \theta$与 $\theta'$是入射光线与折射光线距离法向的夹角, $\eta$ 与 $\eta'$(读作eta和eta prime)是介质的折射率(规定空气为1.0, 玻璃为1.3-1.7,钻石为2.4);

从上面的式子推导出折射光线的方向

$sinθ^′=sin\theta\frac{\eta}{\eta^′}$

下面进行折射向量公式的推导：

![image-20230419210834542](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419210834542.png)

根据上图的推导，可以给出折射的函数（注意这里的推导只基于上图的123步骤即可看懂）：
```c++
//vec3.h
//折射函数,参数分别是从外指向内的入射方向uv,表面法线方向n,以及折射率η1/η2
vec3 refract(const vec3& uv, const vec3& n, double etai_over_etat) {
	auto cos_theta = fmin(dot(-uv, n), 1.0);
	vec3 r_out_parallel = etai_over_etat * (uv + cos_theta * n);
	vec3 r_out_perp = -sqrt(fabs(1.0 - r_out_parallel.length_squared())) * n;
	return r_out_parallel + r_out_perp;
}
```

（注：公式推导并不是十分简单，所以也可以先将其作为接口使用，公式推导有需求再看。参考：[反射向量和折射向量的推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/91129191)）

------



### 2.创建绝缘体材质

书写绝缘体材质的相关代码：

```c++
//material.h
class dielectric :public material {
public:
	dielectric(double index_of_refraction) : ir(index_of_refraction) {}
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override
	{
		attenuation = color(1.0, 1.0, 1.0);
		//下一行代码的说明,因为空气的折射率为1,介质的折射率为ir,因此η1/η2=1/ir
		double refraction_ratio = rec.front_face ? (1.0 / ir) : ir;

		vec3 unit_direction = unit_vector(r_in.direction());
		vec3 refracted = refract(unit_direction, rec.normal, refraction_ratio);
		scattered = ray(rec.p, refracted);
		return true;
	}

public:
	double ir; //Index of Refraction,折射率
};
```

此时修改场景物体如下：

```c++
auto material_ground = make_shared<lambertian>(vec3(0.8, 0.8, 0.0));
//auto material_center = make_shared<lambertian>(vec3(0.7, 0.3, 0.3));
//auto material_left = make_shared<metal>(vec3(0.8, 0.8, 0.8), 0.3);  //对金属物体增加模糊系数
auto material_center = make_shared<dielectric>(1.5);
auto material_left = make_shared<dielectric>(1.5);
auto material_right = make_shared<metal>(vec3(0.8, 0.6, 0.2), 1.0);
```

执行代码，渲染出来的结果如下：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419213506623.png" alt="image-20230419213506623" style="zoom:67%;" />

显然，这个效果是错误的。接下来我们会修复这个问题。

------



### 3.解决全内反射问题

注意到如果$\frac{\eta}{\eta^′}$比较大的话，那么$sinθ^′=sin\theta\ ×  \frac{\eta}{\eta^′}$有可能会出现大于1的情况（当光线从高折射律介质射入低折射率介质时, 比如从钻石类的材质射入到空气中，此时上述的Snell方程可能没有实解（因为看前面推导的公式里根号下面可能出现<0的情况）），此时就无法求解出正确的折射光线了。通过上面的公式发现本质上无解的情况应该是$sin\theta_r>1$，也就是$sin\theta × \frac{\eta}{\eta^′}>1$

**解决方案：当求解出光线无法完成折射的任务时，使其反射**：

```c++
//material.h
if(refraction_ratio * sin_theta > 1.0) {
    // 这种情况下一定是反射
    ...
}
else {
    // 可以是折射（但也可以是反射）
    ...
}
```

**什么情况下光线会从高折射律介质射入低折射率介质？**

常常在实心物体的内部发生, 所以我们称这种情况被称为"全内反射"。比如在潜水的时候, 你发现水与空气的交界处看上去像一面镜子的原因。

可以用三角函数解出$\sin\theta$

$$
\sin \theta=\sqrt{1-\cos ^{2} \theta}
$$
其中的$cos\theta$为$$cos\theta=dot(-i,n)$$,其中i是入射光线，n是法线。

所以，我们在这里继续补充上面的函数：

```c++
//material.h
double cos_theta = fmin(dot(-unit_direction, rec.normal), 1.0);
double sin_theta = sqrt(1.0 - cos_theta*cos_theta);
if(refraction_ratio * sin_theta > 1.0) {
    // Must Reflect
    ...
}
else{
    // Can Refract
    ...
}
```

如果我们想让这个绝缘体能折射就折射，不然才考虑反射，则代码修改如下：

```c++
class dielectric :public material {
public:
	dielectric(double index_of_refraction) : ir(index_of_refraction) {}
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override
	{
		attenuation = color(1.0, 1.0, 1.0);
		//下一行代码的说明,因为空气的折射率为1,介质的折射率为ir,因此η1/η2=1/ir
		double refraction_ratio = rec.front_face ? (1.0 / ir) : ir;

		vec3 unit_direction = unit_vector(r_in.direction());
		
		//新增对全内反射现象的判断
		double cos_theta = fmin(dot(-unit_direction, rec.normal), 1.0);
		double sin_theta = sqrt(1.0 - cos_theta * cos_theta);
		bool cannot_refract = refraction_ratio * sin_theta > 1.0;
		vec3 direction;
		
		if (cannot_refract) {
			direction = reflect(unit_direction, rec.normal); //不能够折射的话直接反射
		}
		else {
			direction = refract(unit_direction, rec.normal, refraction_ratio);  //否则能折射就折射
		}

		scattered = ray(rec.p, direction);
		return true;
	}

public:
	double ir; //Index of Refraction,折射率
};
```

- 这里的光线衰减率为1——就是不衰减, 玻璃表面不吸收光的能量。

在main函数中使用如下参数：

```c++
auto material_ground = make_shared<lambertian>(color(0.8, 0.8, 0.0));
auto material_center = make_shared<lambertian>(color(0.1, 0.2, 0.5));
auto material_left   = make_shared<dielectric>(1.5);
auto material_right  = make_shared<metal>(color(0.8, 0.6, 0.2), 0.0);
```

此时渲染的结果如下：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419215639408.png" alt="image-20230419215639408" style="zoom:80%;" />

------



### 4.Schlick近似方法

现实世界中的玻璃, **发生反射的概率会随着入射角而改变**——从一个很狭窄的角度去看玻璃窗, 它会变成一面镜子。如果要完全精确地描述这件事，对应的公式是十分痛苦的（参考学习链接：https://zhuanlan.zhihu.com/p/372110183），不过有一个数学上近似的等式, 它是由Christophe Schlick提出的。Schlick 反射率函数如下：
$$
F_{\text {Schlick }}\left(n, v, F_{0}\right)=F_{0}+\left(1-F_{0}\right)(1-(n \cdot v))^{5} 
$$
其中的F0与介质的折射率有关，公式计算如下：
$$
F_{0}=\left(\frac{\eta_{1}-\eta_{2}}{\eta_{1}+\eta_{2}}\right)^{2}=\left(\frac{\eta-1}{\eta+1}\right)^{2}
$$
根据上面的公式，可以写出计算schlick近似的函数：

```c++
//写入到material.h的dielectric类里面
private:
	static double reflectance(double cosine, double ref_idx) {
		//use schlick's approximation for reflectance
		auto r0 = (1 - ref_idx) / (1 + ref_idx);
		r0 = r0 * r0;
		return r0 + (1 - r0) * pow((1 - cosine), 5);
	}
```

将这个函数加入绝缘体材质类当中，修改如下：

```c++
//material.h
//前面都不变
vec3 direction;

if (cannot_refract||reflectance(cos_theta,refraction_ratio)>random_double()) {  //不能折射或者随机出的值<反射率，则发生反射
    direction = reflect(unit_direction, rec.normal); 
}
else { //否则发生折射
    direction = refract(unit_direction, rec.normal, refraction_ratio);
}
```

- 这里有个简单又好用的trick, 如果你将球的半径设为负值, 形状看上去并没什么变化, 但是法线会全部翻转到内部（查看代码会发现光线与球体相交的法线计算为:`(rec.p - center) / radius`）。可以用这个特性来做出一个中空玻璃球【相当于打到中间的球的时候，是从玻璃“内部”进入到空气当中，因为法线被翻转了，聪明！】

将主函数的一部分修改如下：

```c++
world.add(make_shared<sphere>(point3( 0.0, -100.5, -1.0), 100.0, material_ground));
world.add(make_shared<sphere>(point3( 0.0,    0.0, -1.0),   0.5, material_center));
world.add(make_shared<sphere>(point3(-1.0,    0.0, -1.0),   0.5, material_left));
world.add(make_shared<sphere>(point3(-1.0,    0.0, -1.0),  -0.4, material_left)); //新增一个半径为负数的球（trick，这样法线就会翻转）
world.add(make_shared<sphere>(point3( 1.0,    0.0, -1.0),   0.5, material_right));
```

效果如下：

![image-20220305174143387](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220305174143387.png)

------



## Chapter 11:改变相机位置，视角等

相机和上面的绝缘体一样，都属于很难调试的。因此一步一步来。

### 1.调整相机视野范围（FOV）

由于图像长宽很可能不一样，因此应该有一个水平方向的FOV和一个竖直方向的FOV，一般来说采用竖直的。作者还会在构造函数当中把传入的FOV的角度改成弧度。

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230919152133676.png" alt="image-20230919152133676" style="zoom: 33%;" />

假设我们从上至下的视野范围角度为θ，相机看向z=-1平面（设定如此，也可以改），那么容易看出：

$h=\tan{\frac{\theta}{2}}$

根据上式，就可以修改相机的构造函数：

```c++
//camera.h
camera(
    double vfov,  //Vertical field-of-view in degrees
    double aspect_ratio) 
{
    //新增对于FOV的处理
    double theta = degrees_to_radians(vfov);
    double h = tan(theta / 2); //见上面的公式

    double viewport_height = 2.0*h;
    double viewport_width = aspect_ratio * viewport_height;
    double focal_length = 1.0;

    origin = point3(0, 0, 0);
    horizontal = vec3(viewport_width, 0.0, 0.0);
    vertical = vec3(0.0, viewport_height, 0.0);
    lower_left_corner = origin - horizontal / 2 - vertical / 2 - vec3(0, 0, focal_length);
}
```

当我们使用一个`cam(90, aspect_ratio)`的摄像机去拍摄下面的球:

```c++
//World
auto R = cos(pi / 4);
hittable_list world;
auto material_left = make_shared<lambertian>(color(0, 0, 1));
auto material_right = make_shared<lambertian>(color(1, 0, 0));

world.add(make_shared<sphere>(point3(-R, 0, -1), R, material_left));
world.add(make_shared<sphere>(point3(R, 0, -1), R, material_right));

//camera
camera cam(90.0, aspect_ratio);
```

结果如下:

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230419222955813.png" alt="image-20230419222955813" style="zoom: 67%;" />

也就是一个广角的镜头.

------



### 2.将相机设置在任意位置

这是在图形学当中一种非常常见的相机模型，如下图：

<img src="https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220310091611892.png" alt="image-20220310091611892" style="zoom:80%;" />

在上图当中，我们假设视点位置为lookfrom，相机看向的位置为lookat，此时就需要一个与观察方向垂直的向量作为up向量，表示相机的旋转角度。上图当中那些红色的向量都可以是相机的up向量。

![image-20220310092006262](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220310092006262.png)

实际上，我们可以指定任何up方向向量，简单地把它投影到这个红色的平面上得到最后相机的up向量。我使用命名为vup向量的通用约定。通过一些叉乘运算，就可以最终得到相机的三组正交基。注意到vup，v，w向量应该处于同一平面，并且相机看向的方向应该是-w向量。（如果你不希望相机扭来扭去，可以定死up方向为（0，1，0），不过为了各种情况的考虑这里就不定死了。）

理想情况下，当我们指定vup的方向和w的方向（w是lookat方向指向lookfrom方向），此时相机的朝向就可以确定了（但这必须要求vup和w方向垂直，否则结果不正确）；

但是实际上，用户输入的值并不一定保证vup和w的方向一定是垂直的，所以在这里我们先求出一个u方向（对应左面的图的u向量，也就是相机空间的x方向，通过vup和w叉乘得到），使其一定与w和vup组成的平面垂直，然后再用w和u做叉乘（相当于相机空间下的z方向与x方向叉乘，得到相机空间下的y方向），最终得到修正后的v方向，也就是相机空间下的y方向。



**总结一下，这里的步骤如下：**

（a）首先，根据lookfrom和lookat计算出w向量，并将其归一化；

（b）根据vup和w的方向，通过叉乘计算出u的方向（正是因为vup不一定与w垂直，所以要做一步中间转换），并将其归一化，这一步对应求解出相机空间的x轴方向；

（c）再根据w和u，通过叉乘求出v的方向，也就是真正的相机y轴指向的方向，而一开始的vup则是用户输入的方便其理解的方向；



结合上面的原理，我们对修改camera类的构造函数：

```c++
//camera.h
//重写其构造函数
public:
	camera(
		point3 lookfrom,
		point3 lookat,
		vec3 vup, //这里的vup不要求那么精确和look at方向垂直.
		double vfov,  //Vertical field-of-view in degrees
		double aspect_ratio) 
	{
		double theta = degrees_to_radians(vfov);
		double h = tan(theta / 2);
	
		double viewport_height = 2.0*h;
		double viewport_width = aspect_ratio * viewport_height;
		
        //根据上面的算法求出相机的三个正交基的方向,注意这里求解lower_left_corner的时候是-w
		auto w = unit_vector(lookfrom - lookat);
		auto u = unit_vector(cross(vup, w));
		auto v = cross(w, u);

		origin = lookfrom;
		horizontal = viewport_width * u;
		vertical = viewport_height * v;
		lower_left_corner = origin - horizontal / 2 - vertical / 2 - w;
	}
```

回顾之前的一张遍历像素的图：

![image-20220310093919133](https://cdn.jsdelivr.net/gh/hhlovesyy/ImgHosting/OldBlogs/CGimage-20220310093919133.png)

在此时需要注意，由于我们的相机发生了旋转，因此u和v的方向不再与世界坐标的x轴和y轴方向平行，而是按照相机空间的u方向和v方向来安排，下述的代码部分就是完成了这件事(可以理解成看向的平面会发生旋转，因此每一条光线追踪的光线方向要根据新的u和v方向来计算)：

```c++
vec3 u, v, w;
w = unit_vector(lookfrom - lookat);
u = unit_vector(cross(vup, w));
v = cross(w, u);
		
origin = lookfrom;
lower_left_corner = origin - viewport_width/2 * u - viewport_height/2 * v - w; 

horizontal = 2 * viewport_width/2*u;
vertical = 2 * viewport_height/2*v;
```

此时回到main函数，构建下面的场景：

```c++
hittable_list world;

auto material_ground = make_shared<lambertian>(color(0.8, 0.8, 0.0));
auto material_center = make_shared<lambertian>(color(0.1, 0.2, 0.5));
auto material_left   = make_shared<dielectric>(1.5);
auto material_right  = make_shared<metal>(color(0.8, 0.6, 0.2), 0.0);

world.add(make_shared<sphere>(point3( 0.0, -100.5, -1.0), 100.0, material_ground));
world.add(make_shared<sphere>(point3( 0.0,    0.0, -1.0),   0.5, material_center));
world.add(make_shared<sphere>(point3(-1.0,    0.0, -1.0),   0.5, material_left));
world.add(make_shared<sphere>(point3(-1.0,    0.0, -1.0), -0.45, material_left));
world.add(make_shared<sphere>(point3( 1.0,    0.0, -1.0),   0.5, material_right));

camera cam(point3(-2,2,1), point3(0,0,-1), vec3(0,1,0), 90, aspect_ratio);
```

此时渲染得到的图为：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420151721627.png" alt="image-20230420151721627" style="zoom:67%;" />

调整相机的FOV值，以实现一种放大（zooming in）的效果。修改相机代码为：

```c++
camera cam(point3(-2,2,1), point3(0,0,-1), vec3(0,1,0), 20, aspect_ratio);
```

效果如下图：

![image-20230420152501945](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420152501945.png)

------

## Chapter12：Defocus Blur（散焦模糊，其实是景深效果）

### 1.一些名词解释

景深指的是相机对焦点前后相对清晰的成像范围，在景深之内的影像比较清楚，在这个范围之前或是之后的影像则比较模糊。虽然透镜只能够将光聚到某一固定的距离，远离此点则会逐渐模糊，但是在某一段特定的距离内，影像模糊的程度是肉眼无法察觉的，这段距离称之为景深。一张复杂的原理图如下图：

![img](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/v2-3be0f23b98c1f8dba9b5635f27b31c7a_r.jpg)

我们称投影点和物体完全聚焦的平面之间的距离为focus distance。请注意，focus distance与focus length是不一样的，后者是投影点和图像平面之间的距离。再看一张图理解两者的区别：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420153441764.png" alt="image-20230420153441764" style="zoom: 80%;" />

> ***Focal length*** is the distance from the focusing plane (where the camera’s sensor is) to the rear nodal point of the lens, when focused at infinity. This is a property of the lens, which determines the angle of view as well as the perspective.
>
> ***Focusing distance*** is the distance from the focusing plane to the subject.

在真实的相机当中，焦距是由镜头和胶片/传感器之间的距离控制的。这就是为什么当你改变对焦时，你会看到镜头相对于相机移动(这也可能发生在你的手机相机上，但传感器会移动)。“光圈”（aperture）是一个孔，用来有效地控制镜头的大小。对于一台真正的相机来说，如果你需要更多的光线，你可以把光圈调大，这样就会产生更多的散焦模糊现象。对于我们的虚拟相机，我们可以有一个完美的传感器，不需要更多的光，所以我们只有在需要散焦模糊时才有一个光圈。

------



### 2.模拟薄透镜

真实的相机的透镜是很复杂的，不过这里我们只是模拟一下传感器，透镜和光圈。图形学中常常用薄透镜来模拟相机的透镜，如下图：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420154122228.png" alt="image-20230420154122228" style="zoom:80%;" />

根据上图，我们需要在计算出光线并得到渲染结果后翻转图像(因为是在胶片上倒过来投影)。在这里我们不需要考虑inside这一侧，否则会很麻烦（因为其实在前面的代码中都是从相机打出光线，直接让成像平面在相机前面）。作者的做法见下图：

<img src="Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420155323282.png" alt="image-20230420155323282" style="zoom:67%;" />

这里感觉作者用了一个trick，直接从lens所在的区域发生出光线，打到聚焦平面上（也就是聚焦平面上所有的点都是完美聚焦，作者说的是`focus_dist` away from lens）即可。

------

### 3.代码部分

正常来说，场景中所有光线都是从`lookfrom`点打出的。为了实现景深效果，从以`lookfrom`点为圆心的圆盘disk上随机选择一个起点发射光线。这个圆盘disk越大，模糊效果就越强。可以认为我们之前构建的相机的的这个disk的半径是0，因此所有光线都从`lookfrom`打出，没有模糊效果。

首先写一个函数，用拒绝法生成一个圆盘里的点：

```c++
//vec3.h
//使用拒绝法,在单位圆盘disk中随机生成一个点
vec3 random_in_unit_disk() {
	while (true) {
		auto p = vec3(random_double(-1, 1), random_double(-1, 1), 0);
		if (p.length_squared() >= 1) continue;
		return p;
	}
}
```

此时修改相机的构造函数为：

```c++
//camera.h
class camera {
public:
	camera(
		point3 lookfrom,
		point3 lookat,
		vec3 vup, //这里的vup不要求那么精确和look at方向垂直.
		double vfov,  //Vertical field-of-view in degrees
		double aspect_ratio,
		double aperture, //光圈大小,越大越模糊
		double focus_dist  //lens到聚焦平面上的距离
	) 
	{
		//新增对于FOV的处理
		double theta = degrees_to_radians(vfov);
		double h = tan(theta / 2);
	
		double viewport_height = 2.0*h;
		double viewport_width = aspect_ratio * viewport_height;
		
		w = unit_vector(lookfrom - lookat);
		u = unit_vector(cross(vup, w));
		v = cross(w, u);

		origin = lookfrom;
		horizontal = focus_dist*viewport_width * u;  //原来相当于focus_dist固定为1，现在可以离远一点的话成像平面相当于得放大u倍（想象一下相似三角形），下同
		vertical = focus_dist*viewport_height * v;
		lower_left_corner = origin - horizontal / 2 - vertical / 2 - focus_dist*w;

		//圆盘disk的半径
		lens_radius = aperture / 2;
	}

	ray get_ray(double s, double t) const {
		vec3 rd = lens_radius * random_in_unit_disk();
		vec3 offset = u * rd.x() + v * rd.y();
		return ray(
			origin+offset, //把光线的起点定为圆盘内随机一点,因为圆盘是与u,v所成平面平行的所以偏移后的结果可以用u,v表示
			lower_left_corner + s * horizontal + t * vertical - origin - offset); //最后减的项由原来的origin变为了origin+offset
	}

private:
	point3 origin;
	point3 lower_left_corner;
	vec3 horizontal;
	vec3 vertical;
	vec3 u, v, w;
	double lens_radius;
};
```

在场景中给相机设定一个很大的光圈值，这样可以很好地观察到模糊效果：

```c++
point3 lookfrom(3,3,2);
point3 lookat(0,0,-1);
vec3 vup(0,1,0);
auto dist_to_focus = (lookfrom-lookat).length();
auto aperture = 2.0;

camera cam(lookfrom, lookat, vup, 20, aspect_ratio, aperture, dist_to_focus);
```

![image-20230420161749676](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420161749676.png)

------



## Chapter13：创建最后的场景

利用前面所学的知识，就可以用来创造一个漂亮的场景了，代码和注释如下：

```c++
//main.cpp
hittable_list random_scene() {
	hittable_list world;
	auto ground_material = make_shared<lambertian>(color(0.5, 0.5, 0.5)); //灰色的大地面的材质
	world.add(make_shared<sphere>(point3(0, -1000, 0), 1000, ground_material));

	for (int a = -11; a < 11; a++) {
		for (int b = -11; b < 11; b++) {
			auto choose_mat = random_double();
			point3 center(a + 0.9 * random_double(), 0.2, b + 0.9 * random_double());
			if ((center - point3(4, 0.2, 0)).length() > 0.9) {
				shared_ptr<material> sphere_material;
				if (choose_mat < 0.8) {
					// diffuse,albedo选择随机的颜色
					auto albedo = color::random() * color::random();
					sphere_material = make_shared<lambertian>(albedo);
					world.add(make_shared<sphere>(center, 0.2, sphere_material));
				}
				else if (choose_mat < 0.95) {
					// metal,其中albedo和模糊系数fuzz都随机
					auto albedo = color::random(0.5, 1);
					auto fuzz = random_double(0, 0.5);
					sphere_material = make_shared<metal>(albedo, fuzz);
					world.add(make_shared<sphere>(center, 0.2, sphere_material));
				}
				else {
					// glass
					//玻璃材质的折射率固定为1.5
					sphere_material = make_shared<dielectric>(1.5);
					world.add(make_shared<sphere>(center, 0.2, sphere_material));
				}
			}
		}
	}
	//对应生成场景里的三个大球,漫反射,metal和glass各一个
	auto material1 = make_shared<dielectric>(1.5);
	world.add(make_shared<sphere>(point3(0, 1, 0), 1.0, material1));

	auto material2 = make_shared<lambertian>(color(0.4, 0.2, 0.1));
	world.add(make_shared<sphere>(point3(-4, 1, 0), 1.0, material2));

	auto material3 = make_shared<metal>(color(0.7, 0.6, 0.5), 0.0);
	world.add(make_shared<sphere>(point3(4, 1, 0), 1.0, material3));

	return world;
}
```

对main函数进行修改，调整相机和渲染参数，进行渲染：

```c++
int main()
{
	ofstream outfile;
	string filename = "output.ppm";
	outfile.open(filename);
	if (!outfile)
	{
		cerr << "no file exists!" << endl;
	}
	//Image
	const auto aspect_ratio = 3.0 / 2.0;
	int image_width = 1200;
	int image_height = static_cast<int>(image_width/aspect_ratio);
	const int samples_per_pixel = 500;
	const int max_depth = 50;

	//World
	auto world = random_scene();

	auto material_ground = make_shared<lambertian>(color(0.8, 0.8, 0.0));
	auto material_center = make_shared<lambertian>(color(0.1, 0.2, 0.5));
	auto material_left = make_shared<dielectric>(1.5);
	auto material_right = make_shared<metal>(color(0.8, 0.6, 0.2), 0.0);

	//camera
	point3 lookfrom(13, 2, 3);
	point3 lookat(0, 0, 0);
	vec3 vup(0, 1, 0);
	auto dist_to_focus = 10.0;
	auto aperture = 0.1;

	camera cam(lookfrom, lookat, vup, 20, aspect_ratio, aperture, dist_to_focus);
	
	//Render
	outfile << "P3\n" << image_width << " " << image_height << "\n255\n"; //根据格式,输出nx行和ny列
	for (int j = image_height - 1; j >= 0; j--)
	{
			//...这里面和之前一样
	}
	outfile.close();
	std::cerr << "\nDone.\n";
}
```

最后跑出来的结果如下：

![image-20230422222039588](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230422222039588.png)

在Linux上跑出这张图的时间大概是7小时（还是蛮久的）。注意到玻璃球都没有阴影，就好像浮起来一样。实际上，在现实生活中，玻璃球也会有这种奇怪的效果，特别是在多云天气下（真的么hh，有时间观察观察）。

这是因为在光线追踪中，玻璃球下方的大球体表面上的某个点，虽然被玻璃球所遮挡，但是仍然可以接收到周围环境的光线，因为黑色的阴影的前提是递归到达depth深度，而此时光线会发生折射后到天空中与背景相交。因此，即使在光线追踪中，玻璃球下方的表面仍然能够获得足够的光线，看起来就像玻璃球漂浮在空中一样。

------



## 其他：如何在linux上运行C++代码？

参考教程：[(30条消息) 在Linux系统下编译并执行C++程序_linux如何编译c++文件_Joyce_Ng的博客-CSDN博客](https://blog.csdn.net/u013793399/article/details/51365311)

（1）首先，把项目的cpp文件都通过xftp拖入到linux服务器里：

![image-20230420163705106](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420163705106.png)

（2）接下来，在XShell中进入到这个文件夹，接着输入下面的指令：

```sh
g++ main.cpp -o ray_tracing
./ray_tracing
```

就可以执行了，总的Shell指令和文件结构如下：

![image-20230420164040838](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420164040838.png)

![image-20230420164132105](Learn%20Ray%20Tracing%20In%20One%20Weekend%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230420164132105.png)



### 如何将输出重定向到文件中?

参考链接:[Linux中记录终端（Terminal）输出到文本文件四种方法_linux 如何把终端打印信息输出到文件里-CSDN博客](https://blog.csdn.net/qq_44681788/article/details/126239092)

类比前面的输出C++程序的包含cerr输出的结果，重定向到目标文件中，可以使用如下指令：
```shell
nohup ./ray_tracing 2>&1 | tee log.txt
```

其中nohup的意思是退出后台也会执行.

捕获进程的指令:`ps -ef | grep ray_tracing`,详见[如何让程序在linux服务器下一直运行（关闭远程连接后仍然继续运行）_51CTO博客_远程关闭服务器Linux](https://blog.51cto.com/u_15088375/3247580)
