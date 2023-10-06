## Ray Tracing: The Next Week 阅读笔记

> 在 "Ray Tracing in One Weekend"篇中，我们搭建了简单的路径追踪渲染器。在这篇中，我们会添加纹理，Volume(如雾)，rectangles，instances，灯光，并支持使用BVH的许多对象。完成上述内容后，我们将会实现一个“真正的”光线追踪器。本篇的难点在于**BVH结构和Perlin Noise的生成**，下面的文档会重点讲解这两个部分。

------



## Chapter 2 运动模糊

在真实的相机中，快门会在很短的时间间隔内保持打开状态，在此期间，相机和外界的物体可能会移动。为了准确地再现这样的相机镜头，我们寻求相机在打开快门时感知到的平均值。

### 1.“时空”上的光线追踪介绍

运动模糊的意思是，现实世界中，相机快门开启的时间间隔内，相机或者物体发生了位移，画面最后呈现出来的像素，是移动过程中像素的平均值。我们可以通过随机一条光线持续的时间，最后计算出像素平均的颜色，这也是[光线追踪](https://www.zhihu.com/search?q=光线追踪&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"44503768"})使用了很多随机性的方法，最后的画面接近真实世界的原因。

这种方法的基础是当快门开机的时间段内，随机时间点生成光线（比如说随机生成x个时间点的光线，并对渲染得到的结果进行平均，就可以得到近似运动模糊后的效果）。

修改之前的ray类，添加一个光线存在时间的变量。

```c++
class ray {
public:
	ray(){}
    //修改构造函数，添加变量tm
	ray(const point3& origin, const vec3& direction) : orig(origin), dir(direction),tm(0) {}
	ray(const point3& origin, const vec3& direction, double time=0.0): orig(origin),dir(direction),tm(time){}
	point3 origin() const { return orig; }
	vec3 direction()const { return dir; }
	double time() const { return tm; } //++
	
	point3 at(double t) const {
		return orig + t * dir;
	}

public:
	point3 orig;
	vec3 dir;
	double tm; //添加一个与时间有关的变量
};
```

------



### 2.管理时间

快门计时有两个方面需要考虑：

- 从一个快门打开到下一个快门打开的时间；
- 每帧快门保持打开的时间。现代电影一般会使用24，30，48，60，120等帧率。

每一帧都可以有自己的快门速度。这个快门速度不必是——通常也不是——整个画面的最长持续时间。你可以让快门每帧打开1/1000秒，或者1/60秒。

就暂时来说，我们先用一个简单的模型。让相机在`time=[0，1]`的区间内随机生成一根光线，并且实现一个可以移动的球（Sphere）类。

#### （1）随机生成一根光线

```c++
//camera.h
//修改camera类的get_ray函数如下：
ray get_ray(double s, double t) const {
    vec3 rd = lens_radius * random_in_unit_disk();
    vec3 offset = u * rd.x() + v * rd.y();
    auto ray_time = random_double();
    return ray(
        origin+offset, //把光线的起点定为圆盘内随机一点,因为圆盘是与u,v所成平面平行的所以偏移后的结果可以用u,v表示
        lower_left_corner + s * horizontal + t * vertical - origin - offset,
        ray_time); //在后面每个像素打出的光线都是随机的时间，平均下来之后就是运动模糊的结果
}
```



#### （2）添加可以移动的球体

修改Sphere类，使其添加关于是否随着时间移动的逻辑：
```c++
class sphere :public hittable {
public:
	sphere(){}
	//stationary Sphere
	sphere(point3 cen,double r,shared_ptr<material> m)
		:center(cen),radius(r),mat_ptr(m),is_moving(false){}
	//Moving Sphere
	sphere(point3 cen1, point3 cen2, double r, shared_ptr<material> m)
		:center(cen1), radius(r), mat_ptr(m), is_moving(true) 
	{
		center_vec = cen2 - cen1;  //center_vec 指的是球心移动的向量
	}
    
	virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec)const override;
public:
	point3 center;
	double radius;
	shared_ptr<material> mat_ptr;  
    //新增的变量的函数
	bool is_moving;
	vec3 center_vec;

	point3 center_cal(double time) const {
		return center + time * center_vec;
	}
};
```

同时，我们也需要修改对应的`hit`函数，因为随着时间的变换球心的位置会发生变化：

```c++
bool sphere::hit(const ray& r, double t_min, double t_max, hit_record& rec) const {
	point3 center1 = is_moving ? center_cal(r.time()) : center; //add

	vec3 oc = r.origin() - center1;
	auto a = r.direction().length_squared();
	auto half_b = dot(oc, r.direction());
	auto c = oc.length_squared() - radius * radius;
	//...与三部曲第一部保持一致
}
```

------



#### （3）interval类的实现

在我们继续之前，我们将实现一个区间类来管理具有最小值和最大值的实值区间。我们以后会经常用到这个类（特别是在比如AABB和BVH Tree相关的代码上）。

```c++
//interval.h，记得把这个头文件的引用放在rtweekend.h文件里
#ifndef INTERVAL_H
#define INTERVAL_H

#include <limits>
const double infinity = std::numeric_limits<double>::infinity();
class interval {
public:
    double min, max;

    interval() : min(+infinity), max(-infinity) {} // Default interval is empty

    interval(double _min, double _max) : min(_min), max(_max) {}

    bool contains(double x) const {
        return min <= x && x <= max;
    }

    bool surrounds(double x) const {
        return min < x&& x < max;
    }

    static const interval empty, universe;
};

const static interval empty(+infinity, -infinity);
const static interval universe(-infinity, +infinity);

#endif
```

有了这个类之后，我们就可以进一步对`hit`函数进行修改了：

```c++
//hittable.h
class hittable {
public:
	virtual bool hit(const ray& r, interval ray_t, hit_record& rec) const = 0;  //修改基类，把中间的min和max用统一interval类对象替代
};  

//hittable_list.h,记得把函数对应的声明也给改了
bool hittable_list::hit(const ray& r, interval ray_t, hit_record& rec) const {
	hit_record temp_rec;
	bool hit_anything = false;
	auto closest_so_far = ray_t.max;

	for (const auto& object: objects) {
		if (object->hit(r, interval(ray_t.min,closest_so_far), temp_rec)) {
			hit_anything = true;
			closest_so_far = temp_rec.t;
			rec = temp_rec;  //修改传入的引用形式的rec,这样就可以把相交点信息传出了
		}
	}
	return hit_anything;
}

//sphere.h，记得把函数对应的声明也给改了
bool sphere::hit(const ray& r, interval ray_t, hit_record& rec) const {
	//...
	//Find the nearest root that lies in the acceptable range.
	auto root = (-half_b - sqrtd) / a; //root求解出来的是最近的hit point对应的t值
	if (!ray_t.surrounds(root)) {
		root = (-half_b + sqrtd) / a;
		if (!ray_t.surrounds(root)) return false;
	}
	//...
}

//main.cpp
//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r, const hittable& world, int depth) {
	//...
	if (world.hit(r, interval(0.0001,infinity), rec)) {  //修改这个函数调用
		//对于不同材质进行判断
		ray scattered;
		color attenuation;
		if (rec.mat_ptr->scatter(r, rec, attenuation, scattered)) {
			return attenuation * ray_color(scattered, world, depth - 1);
		}
	}
	//...
}
```

------

#### （4）对反射光添加时间有关的量

对于反射，镜面反射和折射材质类，都需要修改对应的`scatter`函数，将与时间有关的量同样加入到反射光线中：
```c++
//material.h
//以下均改在材质类的scatter函数里
scattered = ray(rec.p, scatter_direction,r_in.time()); //lambertain类
scattered = ray(rec.p, reflected+fuzz*random_in_unit_sphere(), r_in.time());  //matal类
scattered = ray(rec.p, direction,r_in.time());  //dieletric类
```

------

### 3.综合

以下代码提供一个场景的示例，在t=0的时候每个球心所处的位置是$C_i$，而t=1的时候对应球心的位置是$C_i+vec3(0,randomDouble(0.5),0)$。

```c++
hittable_list random_scene() {
	//...
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
                      
                    //新增的代码++++
					auto center2 = center + vec3(0, random_double(0, 0.5), 0);
					world.add(make_shared<sphere>(center, center2, 0.2, sphere_material));
				}
                //.......
		}
   }
            
//...
//Image
const auto aspect_ratio = 3.0 / 2.0;
int image_width = 400;
int image_height = static_cast<int>(image_width/aspect_ratio);
const int samples_per_pixel = 100;
const int max_depth = 5;  //运算速度有限，这个可以调大
```

最终渲染出的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230920100404243.png" alt="image-20230920100404243" style="zoom:67%;" />

------

## Chapter 3 BVH 数据结构

这一部分的难度还是比较大的，我们需要搭建一个BVH数据结构来做空间加速。有两种常见的加速方法：（1）分割空间，个人理解是类似与KD Tree；（2）分割物体，比如接下来的BVH数据结构。

### 1.核心思想

采用包围盒进行空间加速的核心思想在于，**如果光线不与包围盒相交，那么一定不会和里面的物体相交。**这样就可以先判断光线是否与包围盒相交，如果相交的话再求解子包围盒，以下是一个BVH Tree的结构和基本说明（**更详细的见第2和第3部分**）：

![image-20230920095851370](Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230920095851370.png)

构建BVH结构的步骤可以总结如下：

- （1）找到包围盒；
- （2）递归地将包围盒里的物体分成两个子集，设定终止条件（比如可设定为当前包围盒内三角形数量足够少）；
- （3）分别重新计算每个子集的包围盒；
- （4）在每个叶子节点当中存储物体的信息。

**如何划分节点呢**？

- 每次划分一般选择最长的那一轴划分；
- 选择位置在中位数的三角形作为划分依据（比如可以找**重心的中位数**，这里涉及到找中位数的算法，可以排序但是效率不高，也可以使用**快速选择算法（时间复杂度为O(n)）**。），这样可以使两边的三角形数目差不多。

以下图的BVH Tree为例，可以写出伪代码：

![image-20230920100345817](Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230920100345817.png)

```c++
if(hits purple)
{
    hit0 = hits blue enclosed objects;
    hit1 = hits red enclosed objects;
    if(hit0 or hit1)
        return true and info of closer hit;
}
return false;
```

------



### 2.AABB创建

对于大多数的模型来说，AABB作为包围盒表现都是很不错的。但需要注意的是特殊情况也需要特殊考虑，这个遇到再说。对于光线和AABB求交来说，我们并不需要知道交点信息以及法线方向，只需要能够判断是否相交即可。这里以一组对边为例：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230920101013748.png" alt="image-20230920101013748" style="zoom:67%;" />

已知光线方程可以表示为：$P(t)=A+tb$。

上述方程对于x，y，z轴都是适配的，比如说：$x(t)=A_x+tb_x$。

当光线与$x=x_0$相交时，有$X_0=A_x+t_0b_0$,解得$t_0=\LARGE \frac{x_0-A_x}{b_x}$；同理有$t_1=\LARGE \frac{x_1-A_x}{b_x}$。

根据上面的推导，对于3D的AABB包围盒来说，只需要对三组对边的$t_0$和$t_1$进行求解，然后**依据某些判断条件判断光线和AABB是否相交即可**。简单来说伪代码可以书写如下：

```c++
compute(tx0, tx1);
compute(ty0, ty1);
compute(tz0, tz1); //这些tx1之类的公式参考上面，这六个值是指光线和三组对面相交时计算出的t值
return overlap? ((tx0,tx1), (ty0,ty1), (tz0,tz1))
```

看上去还是比较清晰的，但是**需要考虑到以下的特殊情况**：

- （1）想象一下光线是沿着-x方向传播的，此时(tx0，tx1)可能就需要进行翻转，比如（3，7）翻转为（7，3）；
- （2）由于使用了除法，就需要考虑除0等情况。比如如果光线的原点在某个AABB的面上，就会得到t的结果为NaN。在不同的光线追踪器中有不同的解决方案。
- （3）还有矢量化问题，如SIMD，我们在这里不讨论。Ingo Wald的论文是一个很好的起点，如果你想在矢量化方面做得更快。



对于我们的渲染器来说，尽可能采用简单的方式来实现，讨论如下：

根据前文，解出$t_{x0}=\LARGE \frac{x_0-A_x}{b_x}$；$t_{x1}=\LARGE \frac{x_1-A_x}{b_x}$。此时如果$b_x=0$，则会出现除0的情况，解决方案是取min和max操作，这样可以规避由分母为0带来的问题（当分母为0时，结果为正/负无穷，取min或max操作后依然可以最终得到正确的结果）：

$$
\large \begin{array}{l}
  t_{x 0}=\min \left(\frac{x_{0}-A_{x}}{b_{x}}, \frac{x_{1}-A_{x}}{b_{x}}\right) \\
  t_{x 1}=\max \left(\frac{x_{0}-A_{x}}{b_{x}}, \frac{x_{1}-A_{x}}{b_{x}}\right)
  \end{array}
$$

对于两组包围盒对面来说，假定interval的第一个值小于第二个值，可以写出如下的伪代码：

```c++
bool overlap(d,D,e,E,f,F)
{
    f = max(d,e); //两组对边都进入才算进入
    F = min(D,E); //只要有一组对边离开就算离开
    return (f < F);
}
```

这里还有个问题，就是如果$x_0-A_x$和$b_x$同时为0的时候，还是会出现NaN的情况，解决方案是可以做一个padding包围盒的操作。

```c++
class interval {
public:
    //...
    double size() const {
        return max - min;
    }

    interval expand(double delta) const {
        auto padding = delta / 2;
        return interval(min - padding, max + padding);
    }
	//...
};
```

------

#### AABB 代码

```c++
//AABB.h
#ifndef AABB_H
#define AABB_H
#include "rtweekend.h"

class aabb {
public:
	interval x, y, z;
	aabb(){}  //the default AABB is empty, since intervals are empty by default
	aabb(const interval& ix, const interval& iy, const interval& iz): x(ix),y(iy),z(iz){}
	aabb(const point3& a, const point3& b)
	{
		//传入的两个点是包围盒的“左下角”和“右上角”的值，即两个点代表了三个轴的最大最小值
		//求解出的是三组对立面的x，y，z范围
		x = interval(fmin(a[0], b[0]), fmax(a[0], b[0]));
		y = interval(fmin(a[1], b[1]), fmax(a[1], b[1]));
		z = interval(fmin(a[2], b[2]), fmax(a[2], b[2]));
	}

	const interval& axis(int n) const {
		if (n == 1) return y;
		if (n == 2) return z;
		return x;
	}

	bool hit(const ray& r, interval ray_t) const{
		for (int a = 0; a < 3; a++) {
			//实现上述公式：针对x，y，z轴分别计算t0（tenter）和t1（texit），然后求ray_t.min=max(tenter), ray_t.max=min(exit)
			auto t0 = fmin((axis(a).min - r.origin()[a]) / r.direction()[a],
				(axis(a).max - r.origin()[a]) / r.direction()[a]);
			auto t1 = fmax((axis(a).min - r.origin()[a]) / r.direction()[a],
				(axis(a).max - r.origin()[a]) / r.direction()[a]);
			ray_t.min = fmax(t0, ray_t.min);
			ray_t.max = fmin(t1, ray_t.max);
			if (ray_t.max <= ray_t.min)
				return false;
		}
		return true;
	}
};
#endif
```

------

#### 一个优化之后的Hit函数

皮克斯的Andrew Kensler做了一些实验，并提出了以下版本的代码，在许多编译器上都执行良好：
```c++
//aabb.h
bool hit(const ray& r, interval ray_t) const {
    for (int a = 0; a < 3; a++) {
        auto invD = 1 / r.direction()[a];
        auto orig = r.origin()[a];

        auto t0 = (axis(a).min - orig) * invD;
        auto t1 = (axis(a).max - orig) * invD;

        if (invD < 0)
            std::swap(t0, t1);
        if (t0 > ray_t.min) ray_t.min = t0;  //最终记录的ray_t.min是max
        if (t1 < ray_t.max) ray_t.max = t1;

        if (ray_t.max <= ray_t.min)
            return false;
    }
    return true;
}
```

------



### 3.构建场景物体的包围盒

对于interval类来说，默认是empty的，在书写类的时候考虑到了这种情况，因此如果`hittable_list`里面没有物体也不影响运算。

同样需要考虑的是，有些物体是会运动的，此时AABB会发生变化。

对代码做的改动如下：

```c++
//hittable.h
#include "aabb.h"
class hittable {
public:
	virtual bool hit(const ray& r, interval ray_t, hit_record& rec) const = 0;
	//新增:包围盒计算
	virtual aabb bounding_box() const = 0;
};
```

对于静态的球体，包围盒计算函数是比较简单的。而对于移动的球体来说，可以把t=0和t=1时刻的包围盒都求解出来，然后对两个包围盒再求解出一个包围盒。代码如下：

```c++
class sphere :public hittable {
public:
	sphere(){}
	//stationary Sphere
	sphere(point3 cen,double r,shared_ptr<material> m)
		:center(cen),radius(r),mat_ptr(m),is_moving(false)
	{
         //+++++++++对于静态的球只需要解出包围盒即可++++++++++
		auto rvec = vec3(radius, radius, radius);
		bbox = aabb(center - rvec, center + rvec);
	}
    
	//Moving Sphere
	sphere(point3 cen1, point3 cen2, double r, shared_ptr<material> m)
		:center(cen1), radius(r), mat_ptr(m), is_moving(true) 
	{
         //对于动态的球，可以分别求出两个时刻的包围盒，再对两个包围盒取一次包围盒操作
         auto rvec = vec3(radius, radius, radius);
		aabb box1(cen1 - rvec, cen1 + rvec);
		aabb box2(cen2 - rvec, cen2 + rvec);
		bbox = aabb(box1, box2); 
		center_vec = cen2 - cen1;  //center_vec 指的是球心移动的向量
	}
	virtual bool hit(const ray& r, interval ray_t, hit_record& rec)const override;
	aabb bounding_box() const override { return bbox; }
public:
	point3 center;
	double radius;
	shared_ptr<material> mat_ptr;  //hittable.h里面声明了material,因此不需要引入新的头文件
	bool is_moving;
	vec3 center_vec;
	aabb bbox;

	point3 center_cal(double time) const {
		return center + time * center_vec;
	}
};
```

对应的aabb类里应该添加一个将两个aabb作为参数的构造函数，并为interval类添加一个构造函数：

```c++
class interval{
    //+++  两个包围盒求包围盒的话只需要求解两个包围盒的最小的x，最大的x，其他轴同理
    interval(const interval& a, const interval& b):min(fmin(a.min,b.min)),max(fmax(a.max,b.max)){}
}

class aabb{
    //+++
    aabb(const aabb& box0, const aabb& box1) {
		x = interval(box0.x, box1.x);
		y = interval(box0.y, box1.y);
		z = interval(box0.z, box1.z);
	}
}
```

接下来，就是对hittable_list类提供构建包围盒的函数：

```c++
//hittable_list.h
#include "aabb.h"
class hittable_list :public hittable {
public:
	//...
	void add(shared_ptr<hittable> object) {
		objects.push_back(object);
		bbox = aabb(bbox, object->bounding_box()); //相当于每加入一个新物体就更新一次包围盒
	}
	virtual bool hit(
		const ray& r, interval ray_t, hit_record& rec) const override;
	aabb bounding_box()const override { return bbox; }
public:
	std::vector<shared_ptr<hittable>> objects;
	aabb bbox;
};
```

------



### 4. BVH节点类

对于BVH Tree数据结构而言，有两种构建方式：一是将BVH树与其对应节点拆成两个类来写，而是合并到一个类当中。这里采用第二种方式。注意bvh_node类也需要继承基本的hittable类。可以写出基本的代码结构如下：

```c++
//bvh.h
#ifndef BVH_H
#define BVH_H
#include "rtweekend.h"
#include "hittable.h"
#include "hittable_list.h"

class bvh_node :public hittable {
public:
	bvh_node(const hittable_list& list): bvh_node(list.objects, 0, list.objects.size()){}
	bvh_node(const std::vector<shared_ptr<hittable>>& src_objects, size_t start, size_t end) {
		//To be implemented later
	}

	bool hit(const ray& r, interval ray_t, hit_record& rec) const override {
		if (!box.hit(r, ray_t)) //与包围盒不相交,直接返回false
			return false;
		bool hit_left = left->hit(r, ray_t, rec);
		bool hit_right = right->hit(r, interval(ray_t.min, hit_left ? rec.t : ray_t.max), rec); //如果有hit_left,还要比较一下最近的交点,用rec来记录
		return hit_left || hit_right;
	}

	aabb bounding_box() const override { return bbox; }

private:
	shared_ptr<hittable> left;
	shared_ptr<hittable> right;
	aabb bbox;
};

#endif // !BVH_H
```

------



### 5.创建BVH树

关于BVH数据结构，最难的部分就是如何构建一颗BVH树，这里采用的算法如下：

- （1）随机选择一个轴；
- （2）使用`std::sort`函数对所有的物体进行排序；
- （3）选择排序后中间的物品进行分开，分成左右两棵子树。

如果只剩下了一个物体的话，就把它分别放在两棵subtree当中。这颗树的创建过程放在bvh_node类的构造函数中，如下：
```c++
#include <algorithm>
bvh_node(const std::vector<shared_ptr<hittable>>& src_objects, size_t start, size_t end) {
    auto objects = src_objects;
    int axis = random_int(0, 2); //见rtweekend.h
    auto comparator = (axis == 0) ? box_x_compare
        : (axis == 1) ? box_y_compare
            : box_z_compare;  //这几个compare是类似函数指针,决定comparator是哪个函数

    size_t object_span = end - start; //BVH节点里有几个物体
    if (object_span == 1) {
        left = right = objects[start];
    }
    else if (object_span == 2) {
        if (comparator(objects[start], objects[start + 1])) {
            left = objects[start];
            right = objects[start + 1];
        }
        else {
            left = objects[start + 1];
            right = objects[start];
        }
    }
    else {
        std::sort(objects.begin() + start, objects.begin() + end, comparator);
        auto mid = start + object_span / 2;
        left = make_shared<bvh_node>(objects, start, mid);
        right = make_shared<bvh_node>(objects, mid, end);
    }
    bbox = aabb(left->bounding_box(), right->bounding_box());
}
```

其中random_int函数写在了rtweekend.h头文件里，如下：

```c++
//rtweekend.h
inline int random_int(int min, int max) {
	return static_cast<int>(random_double(min, max + 1));
}
```

比较函数如下所示：

```c++
class bvh_node :public hittable {
//...
private:
    static bool box_compare(
		const shared_ptr<hittable> a, const shared_ptr<hittable> b, int axis_index
	) {
		return a->bounding_box().axis(axis_index).min < b->bounding_box().axis(axis_index).min;
	}
	
	static bool box_x_compare(const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
		return box_compare(a, b, 0);
	}

	static bool box_y_compare(const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
		return box_compare(a, b, 1);
	}

	static bool box_z_compare(const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
		return box_compare(a, b, 2);
	}
}
```

在创建BVH树的时候，只需要增添一行代码即可：

```c++
//main.cpp
#include "bvh.h"
hittable_list random_scene() {
	//...
	auto material3 = make_shared<metal>(color(0.7, 0.6, 0.5), 0.0);
	world.add(make_shared<sphere>(point3(4, 1, 0), 1.0, material3));

	//加入bvh树进行管理
	world = hittable_list(make_shared<bvh_node>(world));

	return world;
}
```

**经过实际测试，添加了BVH Tree空间加速结构的代码执行速度确实会快不少。（其实不确定，毕竟构造BVH Tree本身还要很长时间）**

------



## Chapter 4 纹理映射

很多信息都可以用纹理来表示，比如法线，颜色等。在书写程序中，常见的方法是给定三维物体上的某个点，对应到二维纹理上的某个uv值，来做纹理映射。接下来我们会使用`color value(...)`函数来给定u，v坐标返回对应的采样颜色。同样，我们也会把待着色的点传入该函数，以便于实现一些基于物体位置的纹理映射操作。

### 1.纯色纹理

```c++
#ifndef TEXTURE_H
#define TEXTURE_H

#include "rtweekend.h"

class texture {
public:
	virtual ~texture() = default;
	virtual color value(double u, double v, const point3& p) const = 0;
};

class solid_color :public texture {
public:
	solid_color(color c):color_value(c){}
	solid_color(double red, double green, double blue) :solid_color(color(red, green, blue)){}

	color value(double u, double v, const point3& p) const override {
		return color_value;
	}

private:
	color color_value;
};

#endif
```

------

此时，我们还需要更新`hit_record`结构体，以计算并记录光线与物体交点的uv信息：

```c++
struct hit_record {
	//...
	double t;
	bool front_face;
	double u;
	double v;
    //...
};
```

------

### 2.Solid Textures：A Checker Texture（以棋盘格为例）

> Definition：A solid (or spatial) texture depends only on the position of each point in 3D space. 

基于上述定义，我们先来实现一个基于空间绝对位置的纹理映射。这里采用棋盘格纹理作为一个”checker“纹理，具体代码及相关注解如下：

```c++
//texture.h
class checker_texture :public texture {
public:
	checker_texture(double _scale, shared_ptr<texture>_even, shared_ptr<texture>_odd)
		:inv_scale(1.0 / _scale), even(_even), odd(_odd){}
	checker_texture(double _scale,color c1,color c2)
		:inv_scale(1.0/_scale),
		even(make_shared<solid_color>(c1)),
		odd(make_shared<solid_color>(c2)){}

	color value(double u, double v, const point3& p)const override {
		auto xInteger = static_cast<int>(std::floor(inv_scale * p.x()));
		auto yInteger = static_cast<int>(std::floor(inv_scale * p.y()));
		auto zInteger = static_cast<int>(std::floor(inv_scale * p.z())); //直接对空间坐标向下取整,×inv_scale后可以想到一块区域可能对应一个棋盘格格子颜色

		bool isEven = (xInteger + yInteger + zInteger) % 2 == 0; //棋盘格逻辑

		return isEven ? even->value(u, v, p) : odd->value(u, v, p);
	}
private:
	double inv_scale; // 做tiling需要的
	shared_ptr<texture> even;
	shared_ptr<texture> odd;
};
```

------

接下来，可以在main函数当中为球体添加棋盘格纹理，来查看效果：

```c++
//main.cpp
hittable_list random_scene() {
	hittable_list world;
	auto checker = make_shared<checker_texture>(0.32, color(.2, .3, .1), color(.9, .9, .9));
	//auto ground_material = make_shared<lambertian>(color(0.5, 0.5, 0.5)); //灰色的大地面的材质
	world.add(make_shared<sphere>(point3(0, -1000, 0), 1000, make_shared<lambertian>(checker))); //为地面赋予棋盘格材质

	for (int a = -11; a < 11; a++) {
        //...
    }
}
```

当然，我们需要为lambertian类提供一个以texture作为参数的构造函数，并将albedo变量由color类的对象改为`shared_ptr<texture>`的对象：

```c++
//material.h

//#include "rtweekend.h"
#include "texture.h"
class lambertian :public material {
public:
	lambertian(const color& a) :albedo(make_shared<solid_color>(a)) {}
	lambertian(shared_ptr<texture> a) :albedo(a){}

	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override 
	{
		//...
		scattered = ray(rec.p, scatter_direction,r_in.time());
		attenuation = albedo->value(rec.u,rec.v,rec.p);
		return true;
	}
public:
	shared_ptr<texture>albedo;  //会作为attenuation的量被scatter修改,依据lambert模型可以理解为kd
};
```

最终得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230924095845421.png" alt="image-20230924095845421" style="zoom:67%;" />

------



### 3.球体的纹理坐标

如果看[《Ray Tracing：The Next Week》]([Ray Tracing: The Next Week](https://raytracing.github.io/books/RayTracingTheNextWeek.html#boundingvolumehierarchies/thekeyidea))的4.3节，会发现作者渲染出上下两个球体（使用棋盘格纹理）时出现了一些断掉的线，这是因为在球体上采用绝对位置坐标进行纹理坐标的映射，就会出现断层。解决方案就是接下来会提到的**使用球体的纹理坐标**。

首先，我们来探索三维物体上的相对位置和纹理采样坐标（u，v）之间的关系，这里以球体为例。在《高数下》里我们学过，球面坐标可以用$\theta,\phi$来表示，其中英文定义如下：

> $\theta$ is the angle up from the bottom pole (that is, up from -Y), and $\phi$is the angle around the Y-axis (from -X to +Z to +X to -Z back to -X).

因此，如果我们要把$\theta,\phi$归一化到[0，1]的uv坐标，可以用下式：
$$
u = \frac{\phi}{2\pi} \\
v = \frac{\theta}{\pi}
$$
而$\theta,\phi$对应到空间相对的x，y，z坐标的公式为：
$$
y=-cos(\theta) \\
x=-cos(\phi)sin(\theta) \\
z=sin(\phi)sin(\theta)
$$
此时就可以利用相对的xyz坐标解出对应的$\theta,\phi$，进而求解出uv坐标了。在C++的`<cmath>`头文件中，有`atan2()`函数，其相关介绍如下：[C++：atan2(y, x)函数用法详解_c++ atan2_孙 悟 空的博客-CSDN博客](https://blog.csdn.net/weixin_46098577/article/details/116026828)。

根据上式，可以解出：
$$
\phi=atan2(z,-x)
$$
这里有一个问题：对于`atan2`函数，其在一二象限是正值，结果范围是$[0,\pi]$，而在第三四象限是负值，结果范围是$[-\pi,0]$，这样对应到u坐标上结果是不正确的，不过好在有下述式子：
$$
atan2(a,b)=atan2(-a,-b)+\pi
$$
此时的函数值域范围就变为了$[0,2\pi]$，并且是连续的。可以写出以下映射关系：
$$
\phi=atan2(-z,x)+\pi \\
\theta = arccos(y)
$$
根据上文推导，可以写出下面的函数：

```c++
//sphere.h
class sphere :public hittable {
//...
private:
	static void get_sphere_uv(const point3& p, double& u, double& v) {
		//p:a given point on the sphere of radius one,centered at the origin
		auto theta = acos(-p.y());
		auto phi = atan2(-p.z(), p.x()) + pi;
		u = phi / (2 * pi);
		v = theta / pi;
	}
};
```

在hit函数中修改光线与物体求交后的逻辑，计算出交点的uv坐标：

```c++
//sphere.h
bool sphere::hit(const ray& r, interval ray_t, hit_record& rec) const {
	//...
	vec3 outward_normal = (rec.p - center) / radius;  //相当于归一化了
	rec.set_face_normal(r, outward_normal);

    //++++
	//聪明之处在于,使用outward_normal作为参数,直接就是相对于半径为1的球的球心的坐标,这样求解出的球面坐标就是正确的
	get_sphere_uv(outward_normal, rec.u, rec.v); 
	rec.mat_ptr = mat_ptr;
	return true;
}
```

接下来，我们要加载图像作为纹理，关于缩放纹理也会在下一节中介绍。

------



### 4.图像作为纹理

作者在这里使用了`stb_image.h`文件，链接如下：[stb/stb_image.h at master · nothings/stb (github.com)](https://github.com/nothings/stb/blob/master/stb_image.h)

这个头文件提供将图像资源转为unsigned char数组的方法。首先将这个头文件导入项目工程当中，然后我们创建一个`rtw_image`类用来更方便地加载图像资源：

```c++
//rtw_stb_image.h
#ifndef RTW_STB_IMAGE_H
#define RTW_STB_IMAGE_H
#pragma warning(disable:4996)  //note：需要加这一句，否则VS 会报warnings
// Disable strict warnings for this header from the Microsoft Visual C++ compiler.
#ifdef _MSC_VER
#pragma warning (push, 0)
#endif

#define STB_IMAGE_IMPLEMENTATION
#define STBI_FAILURE_USERMSG
#include "external/stb_image.h"

#include <cstdlib>
#include <iostream>

class rtw_image {
public:
    rtw_image() : data(nullptr) {}

    rtw_image(const char* image_filename) {
        // Loads image data from the specified file. If the RTW_IMAGES environment variable is
        // defined, looks only in that directory for the image file. If the image was not found,
        // searches for the specified image file first from the current directory, then in the
        // images/ subdirectory, then the _parent's_ images/ subdirectory, and then _that_
        // parent, on so on, for six levels up. If the image was not loaded successfully,
        // width() and height() will return 0.

        auto filename = std::string(image_filename);
        auto imagedir = getenv("RTW_IMAGES");

        // Hunt for the image file in some likely locations.
        if (imagedir && load(std::string(imagedir) + "/" + image_filename)) return;
        if (load(filename)) return;
        if (load("images/" + filename)) return;
        if (load("../images/" + filename)) return;
        if (load("../../images/" + filename)) return;
        if (load("../../../images/" + filename)) return;
        if (load("../../../../images/" + filename)) return;
        if (load("../../../../../images/" + filename)) return;
        if (load("../../../../../../images/" + filename)) return;

        std::cerr << "ERROR: Could not load image file '" << image_filename << "'.\n";
    }

    ~rtw_image() { STBI_FREE(data); }

    bool load(const std::string filename) {
        // Loads image data from the given file name. Returns true if the load succeeded.
        auto n = bytes_per_pixel; // Dummy out parameter: original components per pixel
        data = stbi_load(filename.c_str(), &image_width, &image_height, &n, bytes_per_pixel);
        bytes_per_scanline = image_width * bytes_per_pixel;
        return data != nullptr;
    }

    int width()  const { return (data == nullptr) ? 0 : image_width; }
    int height() const { return (data == nullptr) ? 0 : image_height; }

    const unsigned char* pixel_data(int x, int y) const {
        // Return the address of the three bytes of the pixel at x,y (or magenta if no data).
        static unsigned char magenta[] = { 255, 0, 255 };
        if (data == nullptr) return magenta;

        x = clamp(x, 0, image_width);
        y = clamp(y, 0, image_height);

        return data + y * bytes_per_scanline + x * bytes_per_pixel;
    }

private:
    const int bytes_per_pixel = 3;
    unsigned char* data;
    int image_width, image_height;
    int bytes_per_scanline;

    static int clamp(int x, int low, int high) {
        // Return the value clamped to the range [low, high).
        if (x < low) return low;
        if (x < high) return x;
        return high - 1;
    }
};

// Restore MSVC compiler warnings
#ifdef _MSC_VER
#pragma warning (pop)
#endif

#endif
```

此时就可以补充`image_texture`类了，用来读取图像并做纹理映射：

```c++
//iterval类需要补充以下函数：
double clamp(double x) const {
    if (x < min) return min;
    if (x > max) return max;
    return x;
}
//texture.h文件中补充：
class image_texture :public texture {
public:
	image_texture(const char* filename):image(filename){}

	color value(double u, double v, const point3& p) const override {
		//没有纹理信息,返回一个debug的纯色
		if (image.height() <= 0)return color(0, 1, 1);

		//clamp input texture coordinates to [0,1]x[1,0]
		u = interval(0, 1).clamp(u);
		v = 1.0 - interval(0, 1).clamp(v); //Flip V to image coordinates
		auto i = static_cast<int>(u * image.width());
		auto j = static_cast<int>(v * image.height());
		auto pixel = image.pixel_data(i, j);

		auto color_scale = 1.0 / 255.0;
		return color(color_scale * pixel[0], color_scale * pixel[1], color_scale * pixel[2]);
	}

private:
	rtw_image image;
};
```

> 对`image_texture`类的`value`函数进行说明：
>
> 对于计算出的光线与物体交点对应的纹理采样点（u，v），需要知道该纹理对应图片的哪个像素，此时就可以让u，v（$u,v \in[0,1]$）分别乘图像的长和宽再做取整运算，从而得到应该采样哪个像素的颜色。

------

### 5.重写camera类及相关函数

由于后面我们要构建不同的场景，因此需要复用一些代码，这里重写一下camera类，把渲染场景的函数封装进camera类里（注：关于这里的原理不再赘述了，大概思路可以参考光追三部曲第一部的说明文档）：
```c++
#ifndef CAMERA_H
#define CAMERA_H

#include "rtweekend.h"
#include <fstream>
using namespace std;

class camera {
public:
	double aspect_ratio = 1.0; //ratio of image width over height
	int image_width = 100; //rendered image width in pixel count
	int samples_per_pixel = 10; //count of random samples for each pixel
	int max_depth = 10; //maximum number of ray bounces into scene

	double vfov = 90; //vertical view angle(field of view)
	point3 lookfrom = point3(0, 0, -1); //point camera is looking from
	point3 lookat = point3(0, 0, 0);
	vec3 vup = vec3(0, 1, 0);

	double defocus_angle = 0; //variation angle of rays through each pixel
	double focus_dist = 10; //distance from camera lookfrom point to plane of perfect focus

	point3 origin;
	point3 lower_left_corner;
	vec3 horizontal;
	vec3 vertical;
	vec3 u, v, w;
	double lens_radius;
	color background; //光线不与物体相交时返回的背景色.

	void render(const hittable& world) {
		initialize();
		//把原来main函数里打出光线渲染场景的函数放在了这里
		ofstream outfile;
		string filename = "output.ppm";
		outfile.open(filename);
		if (!outfile)
		{
			cerr << "no file exists!" << endl;
		}
		outfile << "P3\n" << image_width << " " << image_height << "\n255\n"; //根据格式,输出nx行和ny列
		for (int j = 0; j < image_height; j++) //因为现在原点本身就在左上角了,因此j从0开始遍历即可
		{
			std::cerr << "\rScanlines remaining: " << (image_height-j) << ' ' << std::flush;  //添加一个进度条
			for (int i = 0; i < image_width; i++)
			{
				color pixel_color(0, 0, 0);
				for (int s = 0; s < samples_per_pixel; s++) {
					ray r = get_ray(i, j);
					pixel_color += ray_color(r, max_depth,world);
				}
				write_color(outfile, pixel_color, samples_per_pixel);
			}
		}
		outfile.close();
		std::cerr << "\nDone.\n";
	}

private:
	int image_height; //rendered image height
	point3 center; //camera center
	point3 pixel00_loc; //location of pixel 0,0
	vec3 pixel_delta_u; //offset to pixel to the right
	vec3 pixel_delta_v; //offset to pixel below
	vec3 defocus_disk_u; //defocus disk horizontal radius
	vec3 defocus_disk_v;

	void initialize() {
		image_height = static_cast<int>(image_width / aspect_ratio);
		image_height = (image_height < 1) ? 1 : image_height; //至少保证图像高为1
		center = lookfrom;

		//determine viewport dimensions
		auto theta = degrees_to_radians(vfov);
		double h = tan(theta / 2);
		auto viewport_height = 2 * h * focus_dist;
		auto viewport_width = viewport_height * (static_cast<double>(image_width) / image_height);

		//calculate the u,v,w unit basis vectors for the camera coordinate frame
		w = unit_vector(lookfrom - lookat);
		u = unit_vector(cross(vup, w));
		v = cross(w, u);

		//calculate the vectors across the horizonal and down the vertical viewport edges
		//注:这里相机坐标和<one weekend>的不同,y方向是向下的
		vec3 viewport_u = viewport_width * u; //vector across viewport horizontal edge
		vec3 viewport_v = viewport_height * (-v); //vector down viewport vertical edge

		//calculate the horizontal and vertical delta vectors to the next pixel.
		pixel_delta_u = viewport_u / image_width;
		pixel_delta_v = viewport_v / image_height;

		//calculate the location of the upper left pixel
		auto viewport_upper_left = center - (focus_dist * w) - viewport_u / 2 - viewport_v / 2;
		pixel00_loc = viewport_upper_left + 0.5 * (pixel_delta_u + pixel_delta_v); //采样位置按照像素正中间算

		//calculate the camera defocus disk basis vectors
		auto defocus_radius = focus_dist * tan(degrees_to_radians(defocus_angle / 2));
		defocus_disk_u = u * defocus_radius;
		defocus_disk_v = v * defocus_radius;
	}

	ray get_ray(int i, int j) const {
		//get a randomly-sampled camera ray for the pixel at location i,j, originating from the camera defocus disk.
		auto pixel_center = pixel00_loc + (i * pixel_delta_u) + (j * pixel_delta_v);
		auto pixel_sample = pixel_center + pixel_sample_square();
		auto ray_origin = (defocus_angle <= 0) ? center : defocus_disk_sample();
		auto ray_direction = pixel_sample - ray_origin; //光线的起点和方向都是圆盘里随机的
		auto ray_time = random_double();
		return ray(ray_origin, ray_direction, ray_time);
	}

	vec3 pixel_sample_square() const {
		// Returns a random point in the square surrounding a pixel at the origin.
		auto px = -0.5 + random_double();
		auto py = -0.5 + random_double();
		return (px * pixel_delta_u) + (py * pixel_delta_v);
	}

	point3 defocus_disk_sample() const {
		// Returns a random point in the camera defocus disk.
		auto p = random_in_unit_disk();
		return center + (p[0] * defocus_disk_u) + (p[1] * defocus_disk_v);
	}

	//根据输入的光线方向决定输出的颜色
	color ray_color(const ray& r, int depth, const hittable& world) const {
		hit_record rec;  //rec会作为引用在函数里被修改

		//限制递归溢出
		if (depth <= 0) return color(0.0, 0.0, 0.0);
		if (!world.hit(r, interval(0.0001, infinity), rec)) return background;
		
		ray scattered;
		color attenuation;
		if (rec.mat_ptr->scatter(r, rec, attenuation, scattered)) {
			return attenuation * ray_color(scattered, depth-1, world);
		}
		return background;
	}
};


#endif // !CAMERA_H
```
此时原来的场景需要封装为对应的函数，把main函数里原来的内容都删掉，改为：

```c++
//main.cpp
void random_scene() {
	hittable_list world;

	auto checker = make_shared<checker_texture>(0.32, color(.2, .3, .1), color(.9, .9, .9));
	world.add(make_shared<sphere>(point3(0, -1000, 0), 1000, make_shared<lambertian>(checker)));

	for (int a = -11; a < 11; a++) {
		for (int b = -11; b < 11; b++) {
			auto choose_mat = random_double();
			point3 center(a + 0.9 * random_double(), 0.2, b + 0.9 * random_double());

			if ((center - point3(4, 0.2, 0)).length() > 0.9) {
				shared_ptr<material> sphere_material;

				if (choose_mat < 0.8) {
					// diffuse
					auto albedo = color::random() * color::random();
					sphere_material = make_shared<lambertian>(albedo);
					auto center2 = center + vec3(0, random_double(0, .5), 0);
					world.add(make_shared<sphere>(center, center2, 0.2, sphere_material));
				}
				else if (choose_mat < 0.95) {
					// metal
					auto albedo = color::random(0.5, 1);
					auto fuzz = random_double(0, 0.5);
					sphere_material = make_shared<metal>(albedo, fuzz);
					world.add(make_shared<sphere>(center, 0.2, sphere_material));
				}
				else {
					// glass
					sphere_material = make_shared<dielectric>(1.5);
					world.add(make_shared<sphere>(center, 0.2, sphere_material));
				}
			}
		}
	}

	auto material1 = make_shared<dielectric>(1.5);
	world.add(make_shared<sphere>(point3(0, 1, 0), 1.0, material1));

	auto material2 = make_shared<lambertian>(color(0.4, 0.2, 0.1));
	world.add(make_shared<sphere>(point3(-4, 1, 0), 1.0, material2));

	auto material3 = make_shared<metal>(color(0.7, 0.6, 0.5), 0.0);
	world.add(make_shared<sphere>(point3(4, 1, 0), 1.0, material3));

	world = hittable_list(make_shared<bvh_node>(world));

	camera cam;

	cam.aspect_ratio = 16.0 / 9.0;
	cam.image_width = 400;
	cam.samples_per_pixel = 100;
	cam.max_depth = 50;
	cam.background = color(0.70, 0.80, 1.00);

	cam.vfov = 20;
	cam.lookfrom = point3(13, 2, 3);
	cam.lookat = point3(0, 0, 0);
	cam.vup = vec3(0, 1, 0);

	cam.defocus_angle = 0.02;
	cam.focus_dist = 10.0;

	cam.render(world);
}

int main()
{
	random_scene();
}
```

点击运行，观察到结果是正确的。

------



### 6.渲染一个有纹理的球体

将一张照片放在文件路径下，在`main.cpp`文件中写一个新函数，用于生成一个球体并将图片作为纹理赋给该球体。代码如下：

```c++
void sphere_genshin() {
	auto genshin_texture = make_shared<image_texture>("genshin.jpg");
	auto genshin_surface = make_shared<lambertian>(genshin_texture);
	auto globe = make_shared<sphere>(point3(0, 0, 0), 2, genshin_surface);

	camera cam;

	cam.aspect_ratio = 16.0 / 9.0;
	cam.image_width = 400;
	cam.samples_per_pixel = 100;
	cam.max_depth = 50;
	cam.background = color(0.70, 0.80, 1.00);

	cam.vfov = 20;
	cam.lookfrom = point3(12, 0, 0);
	cam.lookat = point3(0, 0, 0);
	cam.vup = vec3(0, 1, 0);

	cam.defocus_angle = 0;

	cam.render(hittable_list(globe));
}
```

在main函数当中调用`sphere_genshin()`函数，可以得到下图所示的结果：

![image-20230924153700862](Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230924153700862.png)

有其他感兴趣的图也可以自己导入文件夹当中作为纹理来使用。

------



## Chapter 5 Perlin Noise

> 这一部分作者的行文思路是Value Noise-> Perlin Noise。其中从第5部分开始是与Perlin Noise有关的。关于这部分的理论知识基础可以参考这篇文章：[猴子都能看懂的噪声（noise）专题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/599263679)
>
> 1、关于Value Noise和Perlin Noise的知识点，这篇博客讲的很好：[如何生成一张 Value Noise 算法图片（包括 Perlin Noise） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/52054806)

Perlin Noise的关键：

- （1）它是可重复的：它以一个3D点作为输入，并且总是返回相同的随机数字。附近的点会返回类似的数字。
- （2）它简单而快速，所以它通常是作为一个hack来完成的。

### 1.使用随机数值的Block

我们先将纹理的每个块都赋予一个随机的灰度值，这样就会产生类似于MC游戏里的方块效果（**作者说是一种hashing操作，但是我没太看出来，不过可以看出生成的是随机的噪声**）。

代码实现如下：

```c++
//perlin.h
#ifndef PERLIN_H
#define PERLIN_H

#include "rtweekend.h"

class perlin {
public:
	perlin() {
		ranfloat = new double[point_count];
		for (int i = 0; i < point_count; i++) ranfloat[i] = random_double();
		perm_x = perlin_generate_perm();
		perm_y = perlin_generate_perm();
		perm_z = perlin_generate_perm();
	}
	~perlin() {
		delete[] ranfloat;
		delete[] perm_x;
		delete[] perm_y;
		delete[] perm_z;
	}

	double noise(const point3& p) const {
		auto i = static_cast<int>(4 * p.x()) & 255;
		auto j = static_cast<int>(4 * p.y()) & 255;
		auto k = static_cast<int>(4 * p.z()) & 255;
		return ranfloat[perm_x[i] ^ perm_y[j] ^ perm_z[k]];
	}

private:
	static const int point_count = 256;
	double* ranfloat;
	int* perm_x;
	int* perm_y;
	int* perm_z;

	static int* perlin_generate_perm() {
		auto p = new int[point_count];
		for (int i = 0; i < perlin::point_count; i++) {
			p[i] = i;
		}
		permute(p, point_count);
		return p;
;	}

	//重新打乱数组的顺序
	static void permute(int* p, int n) {
		for (int i = n - 1; i > 0; i--) {
			int target = random_int(0, i);
			int tmp = p[i];
			p[i] = p[target];
			p[target] = tmp;
		}
	}
};
#endif // !PERLIN_H
```

此时添加`noise_texture`类并构建对应的场景，得到的结果如本节开头所示。

```c++
//texture.h
#include "perlin.h"
class noise_texture :public texture {
public:
	noise_texture(){}
	color value(double u, double v, const point3& p) const override {
		return color(1, 1, 1) * noise.noise(p);
	}
private:
	perlin noise;
};

//main.cpp
void two_perlin_spheres() {
    hittable_list world;

    auto pertext = make_shared<noise_texture>();
    world.add(make_shared<sphere>(point3(0,-1000,0), 1000, make_shared<lambertian>(pertext)));
    world.add(make_shared<sphere>(point3(0,2,0), 2, make_shared<lambertian>(pertext)));

    camera cam;

    cam.aspect_ratio      = 16.0 / 9.0;
    cam.image_width       = 400;
    cam.samples_per_pixel = 100;
    cam.max_depth         = 50;
    cam.background = color(0.70, 0.80, 1.00);  //记得加上这句,不然渲染出来的图会是全黑色的(因为递归最后打到background默认就是黑色,所以不加就全是黑色)

    cam.vfov     = 20;
    cam.lookfrom = point3(13,2,3);
    cam.lookat   = point3(0,0,0);
    cam.vup      = vec3(0,1,0);

    cam.defocus_angle = 0;

    cam.render(world);
}
```

最终得到的渲染结果如下图所示:

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230924165141846.png" alt="image-20230924165141846" style="zoom:67%;" />

注：球体上的错误现象也是由于此时纹理采样坐标是由物体空间位置决定。

------



### 2.将结果做平滑操作

这里可以直接使用三线性插值。3D情况下的三线性插值其实是和双线性插值类似的，原理可见：[线性插值与双/三线性插值 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/77496615)。

代码修改如下：

```c++
//perlin.h
class perlin {
public:
	//...
	double noise(const point3& p) const {
		auto u = p.x() - floor(p.x());
		auto v = p.y() - floor(p.y());
		auto w = p.z() - floor(p.z());

		auto i = static_cast<int>(floor(p.x()));
		auto j = static_cast<int>(floor(p.y()));
		auto k = static_cast<int>(floor(p.z()));
		double c[2][2][2];

		for (int di = 0; di < 2; di++) {
			for (int dj = 0; dj < 2; dj++) {
				for (int dk = 0; dk < 2; dk++) {
					c[di][dj][dk] = ranfloat[
						perm_x[(i + di) & 255] ^
							perm_y[(j + dj) & 255] ^
							perm_z[(k + dk) & 255]
					];
				}
			}
		}

		return trilinear_interp(c, u, v, w);
	}

private:
	//...
	static double trilinear_interp(double c[2][2][2], double u, double v, double w) {
		//三维空间上的三线性插值
		auto accum = 0.0;
		for (int i = 0; i < 2; i++) {
			for (int j = 0; j < 2; j++) {
				for (int k = 0; k < 2; k++) {
					accum += (i * u + (1 - i) * (1 - u))
						* (j * v + (1 - j) * (1 - v))
						* (k * w + (1 - k) * (1 - w)) * c[i][j][k];
				}
			}
		}
		return accum;
	}
	//...
};
```

三线性插值之后得到的平滑结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925132024121.png" alt="image-20230925132024121" style="zoom:67%;" />

------



### 3.使用Hermitian 平滑

平滑操作虽然改善了效果，但产生了明显的grid features。有些是由于颜色的线性插值造成的Mach bands效应。一个常见的trick是使用Hermite cubic插值。具体的原理感兴趣可以上网找（后面遇到再说吧）。对代码修改如下：

```c++
//perlin.h
double noise(const point3& p) const {
		auto u = p.x() - floor(p.x());
		auto v = p.y() - floor(p.y());
		auto w = p.z() - floor(p.z());
		//++++++新增逻辑+++++
		u = u * u * (3 - 2 * u);
		v = v * v * (3 - 2 * v);
		w = w * w * (3 - 2 * w);

		auto i = static_cast<int>(floor(p.x()));
		//....
	}
```

得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925133610121.png" alt="image-20230925133610121" style="zoom: 67%;" />

可以看到，平滑的效果确实好了不少。

------



### 4.为noise添加一个scale值

```c++
//texture.h
class noise_texture :public texture {
public:
	noise_texture(){}
	noise_texture(double sc):scale(sc){} //添加一个与scale有关的构造函数
	color value(double u, double v, const point3& p) const override {
		return color(1, 1, 1) * noise.noise(scale * p); //修改逻辑
	}
private:
	perlin noise;
	double scale;
};
```

修改main函数的场景:

```c++
void two_perlin_spheres() {
	hittable_list world;
	auto pertext = make_shared<noise_texture>(4);
	//...
}
```

最后得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925141037952.png" alt="image-20230925141037952" style="zoom:67%;" />

------



### 5.在格子点使用随机向量

这里就是Perlin Noise的核心思想了。关于Perlin Noise的原理可以参考文档：[如何生成一张 Value Noise 算法图片（包括 Perlin Noise） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/52054806)。也就是说我们需要以下步骤：

（1）生成一个格子四个角上的随机向量；

（2）将生成的随机向量与“待求解点与四个角连线的向量”分别做点乘，得到的结果即为Perlin Noise生成的结果。

对代码修改如下：
```c++
//perlin.h
//删除了ranfloat变量，因为现在perlin noise随机生成的是向量
#ifndef PERLIN_H
#define PERLIN_H

#include "rtweekend.h"

class perlin {
public:
	perlin() {
		ranvec = new vec3[point_count];
		for (int i = 0; i < point_count; i++) 
			ranvec[i] = unit_vector(vec3::random(-1,1));
		perm_x = perlin_generate_perm();
		perm_y = perlin_generate_perm();
		perm_z = perlin_generate_perm();
	}
	~perlin() {
		delete[] ranvec;
		delete[] perm_x;
		delete[] perm_y;
		delete[] perm_z;
	}

private:
	static const int point_count = 256;
	vec3* ranvec; //perlin noise:生成的随机向量
	int* perm_x;
	int* perm_y;
	int* perm_z;

};
```

关于noise的生成逻辑也要进行修改：

```c++
//perlin.h
double noise(const point3& p) const {
    //将随机生成的颜色值改为随机生成的向量值
    auto u = p.x() - floor(p.x());
    auto v = p.y() - floor(p.y());
    auto w = p.z() - floor(p.z());

    auto i = static_cast<int>(floor(p.x()));
    auto j = static_cast<int>(floor(p.y()));
    auto k = static_cast<int>(floor(p.z()));
    vec3 c[2][2][2];

    for (int di = 0; di < 2; di++) {
        for (int dj = 0; dj < 2; dj++) {
            for (int dk = 0; dk < 2; dk++) {
                c[di][dj][dk] = ranvec[
                    perm_x[(i + di) & 255] ^
                    perm_y[(j + dj) & 255] ^
                    perm_z[(k + dk) & 255]
                ];
            }
        }
    }

    return perlin_interp(c, u, v, w);  //新的插值函数
}
```

插值函数也需要由原来的变更为`perlin_interp`函数，其会利用向量点乘求解出对应的值：

```c++
static double perlin_interp(vec3 c[2][2][2], double u, double v, double w) {
    auto uu = u * u * (3 - 2 * u);
    auto vv = v * v * (3 - 2 * v);
    auto ww = w * w * (3 - 2 * w);
    auto accum = 0.0;

    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            for (int k = 0; k < 2; k++) {
                vec3 weight_v(u - i, v - j, w - k);
                accum += (i * uu + (1 - i) * (1 - uu))
                    * (j * vv + (1 - j) * (1 - vv))
                    * (k * ww + (1 - k) * (1 - ww)) * dot(c[i][j][k],weight_v);
            }
        }
    }
    return accum;
}
```

**注意，由于Perlin Noise是由点乘产生的，因此可能会产生负值。**这些产生的负值会导致后续的gamma矫正的`sqrt`操作返回`NaN`，因此在noise_texture类的value函数里，我们需要把[-1,1]的范围映射回[0，1]。代码修改如下：

```c++
//texture.h
color value(double u, double v, const point3& p) const override {
    return color(1, 1, 1) * 0.5 * (1.0 + noise.noise(scale * p));  //[-1,1]->[0,1]
}
```

最终得到的渲染结果如下，可以看到已经好很多了：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925150346715.png" alt="image-20230925150346715" style="zoom:67%;" />

------



### 6.添加turbulence

很多时候一个复杂的噪声是由不同频率的噪声累加得到的，这种往往被称为turbulence。一个简单的方法是多次用不同的参数去调用Perlin Noise，并得到累加的结果：

```c++
//perlin.h
class perlin{
public:
    double turb(const point3& p,int depth=7) const {
		auto accum = 0.0;
		auto temp_p = p;
		auto weight = 1.0;
		for (int i = 0; i < depth; i++) {
			accum += weight * noise(temp_p);
			weight *= 0.5;
			temp_p *= 2;
		}
		return fabs(accum);
	}
}
```

此时noise_texture类的value函数也可以进行更改：

```c++
color value(double u, double v, const point3& p) const override {
    auto s = scale * p;
    return color(1, 1, 1) * noise.turb(s);
}
```

渲染得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925153038391.png" alt="image-20230925153038391" style="zoom:67%;" />

------



### 7.调整Phase

一种常见的turbulence噪声的使用场景是用噪声值来做三角函数相位的变换，比如说$sin(x+\phi)$，这里的$\phi$由turbulence噪声生成。这种噪声生成方式可以用来生成具有起伏的条纹。修改noise_texture类的value函数如下：

```c++
//texture.h
color value(double u, double v, const point3& p) const override {
    auto s = scale * p;
    return color(1, 1, 1) * 0.5*(1+sin(s.z()+3*noise.turb(s)));
}
```

这样就会得到一种类似于大理石的材质：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925154207338.png" alt="image-20230925154207338" style="zoom: 67%;" />

------

## Chapter 6 四边形（Quadrilaterals）

在之前的学习中，我们待渲染的一直是球体，这一章我们会讲述如何渲染第二种图元：四边形。

### 1.定义四边形

我们将这种新的图元取名为quad，在技术实现上使用parallelogram（平行四边形）来实现。定义平行四边形需要如下内容：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925154532879.png" alt="image-20230925154532879" style="zoom:67%;" />

（1）Q：左下角；

（2）$\vec{u}$，表示第一条边的方向；$\vec{v}$，表示第二条边的方向。

对于一个完全的平行四边形来说，其AABB理论上应该有一个方向宽度为0，此时在计算AABB与光线求交时会出现数学运算的错误。解决方案是可以做一个宽度为0方向上AABB的padding操作。从BVH空间加速结构中可以看出，对AABB做padding操作不会影响光线本身与quad的求交。为AABB类添加一个pad函数：

```c++
//aabb.h
aabb pad() {
    //return an AABB that has no side narrower than some delta,padding if neccessary
    double delta = 0.0001;
    interval new_x = (x.size() >= delta) ? x : x.expand(delta);
    interval new_y = (y.size() >= delta) ? y : y.expand(delta);
    interval new_z = (z.size() >= delta) ? z : z.expand(delta);
    return aabb(new_x, new_y, new_z);
}
```

> 这里说明一下：由于BVH的节点计算是和里面的物体有关的，在第三章能够保证BVH Tree能够把物体划分好，因为是先划分的左右孩子节点对应物体再去计算整个的AABB，因此即使AABB有一点padding操作，也不影响BVH Tree的划分，以及光线求交的基本逻辑。

此时就可以书写quad类了：
```c++
//quad.h
#ifndef QUAD_H
#define QUAD_H

#include "rtweekend.h"
#include "hittable.h"

class quad :public hittable {
public:
    //注意，在实现的时候u和v并不需要归一化，这是为了方便后续做交点是否在平行四边形内的逻辑判断
	quad(const point3& _Q, const vec3& _u, const vec3& _v, shared_ptr<material> m) :Q(_Q),u(_u),v(_v),mat(m){
		//to be implemented
        set_bounding_box();
	}
	virtual void set_bounding_box() {
		bbox = aabb(Q, Q + u + v).pad();
	}
	aabb bounding_box() const override { return bbox; }
	bool hit(const ray& r, interval ray_t, hit_record& rec)const override {
		return false; //to be implemented
	}

private:
	point3 Q;
	vec3 u, v;
	shared_ptr<material> mat;
	aabb bbox;
};

#endif
```

------



### 2.光线和plane求交

与和球体相交类似，这里我们要补充quad类的hit函数，并在相交的时候记录交点的位置、法线、uv坐标等相关信息。

光线和quad的求交可以分为以下三步：

- （1）找到quad所对应的平面方程；
- （2）求解光线与该平面方程的求交问题；
- （3）判断交点是否在quad内部。

复习games101，判断光线与三角形求交基本上也是这三个步骤。这里我们先来求解光线与一个已知平面相交的问题。

已知平面方程如下:

$Ax+By+Cz=D$

则对于平面上的点$p$,有:

- $p:(p-p')·N=0$  ($p'$是平面上一点,$N$是平面的法线方向,$\vec{N}=(A,B,C)$)



**此时我们联立光线方程与平面方程,求解二者的交点$p$**
$$
\begin{cases}  r(t)=p=o+td(0≤t<∞)\quad① \\   p:(p-p')·N=0\quad ②\end{cases}.
$$
将①代入②,有$(o+td-p')·N=0$,通过点乘的分配律求解出:$\large t=\frac{(p'-o)·N}{d·N}$(需要验证是否满足$0≤t<∞$)

> 注：《The Next Week》的作者最终推导出来的公式是$\large t=\frac{D-o·N}{d·N}$，与我们上面的推导结果有些许不同。这是因为$p'·N$的结果就是D，因为由上述定义，$p’=(x,y,z)$是平面上任意一点，$\vec{N}=(A,B,C)$，此时$p'·N=Ax+By+Cz=D$，所以两个式子实际上是等价的。

注意到如果光线与平面平行的话$d·N=0$，此时一定是不相交的，需要特判一下。

------



### 3.对于给定的Quadrilateral找到对应的Plane

这一步的关键在于给定构成了一个平行四边形的$Q，u，v$参数，如何求解出对应的平面方程。注意到：

- （1）法线方向与u，v方向均垂直，因此`N=unit_vector(u×v)`；
- （2）由于平面上任何点应该符合：$Ax+By+Cz=D$，而Q点本身也在平面上，所以一定有$N·Q=D$。

根据以上知识，就可以解出平面的方程了，对应代码如下：

```c++
class quad :public hittable {
public:
	quad(const point3& _Q, const vec3& _u, const vec3& _v, shared_ptr<material> m) :Q(_Q),u(_u),v(_v),mat(m){
		auto n = cross(u, v);
		normal = unit_vector(n);
		D = dot(normal, Q);
		set_bounding_box();
	}
	virtual void set_bounding_box() {
		bbox = aabb(Q, Q + u + v).pad();
	}
	aabb bounding_box() const override { return bbox; }
	bool hit(const ray& r, interval ray_t, hit_record& rec)const override {
		auto denom = dot(normal, r.direction());
		//no hit if the ray is parallel to the plane
		if (fabs(denom) < 1e-8) {
			return false;
		}
		//return false if the hit point parameter t is outside the ray interval
		auto t = (D - dot(normal, r.origin())) / denom;
		if (!ray_t.contains(t)) return false;

		auto intersection = r.at(t);
		rec.t = t;
		rec.p = intersection;
		rec.mat = mat;
		rec.set_face_normal(r, normal);
		return true;
	}

private:
	point3 Q;
	vec3 u, v;
	shared_ptr<material> mat;
	aabb bbox;
    vec3 normal;
	double D;
};
```

> 注：现在还有两件事情没有做，接下来的两节会进行补充：
>
> - （1）如何判断交点是否在平行四边形内；
> - （2）与平行四边形相交的点的uv坐标如何进行求解；

------



### 4.判断交点是否在平行四边形内（上）

这里的推导如下图所示：

![image-20230925191414886](Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925191414886.png)

根据上述推导，由于w是定值，可以提前计算：

```c++
class quad :public hittable {
public:
	quad(const point3& _Q, const vec3& _u, const vec3& _v, shared_ptr<material> m) :Q(_Q),u(_u),v(_v),mat(m){
		auto n = cross(u, v);
		normal = unit_vector(n);
		D = dot(normal, Q);
		w = n / dot(n, n); //构造函数中求解出w的值
		set_bounding_box();
	}
	//...
private:
	//...
	//+++
	vec3 w;
};
```

------



### 5.判断交点是否在平行四边形内（下）

根据第4部分的描述，判断交点是否在平行四边形内，只需要判断是否满足$0≤\alpha≤1$&&$0≤\beta≤1$即可。

对hit函数进行修改如下：

```c++
//quad.h，quad类
bool hit(const ray& r, interval ray_t, hit_record& rec)const override {
    auto denom = dot(normal, r.direction());
    //no hit if the ray is parallel to the plane
    if (fabs(denom) < 1e-8) {
        return false;
    }
    //return false if the hit point parameter t is outside the ray interval
    auto t = (D - dot(normal, r.origin())) / denom;
    if (!ray_t.contains(t)) return false;

    //determine the hit point lies within the planar shape using its plane coordinates
    auto intersection = r.at(t);
    vec3 planar_hitpt_vector = intersection - Q;
    auto alpha = dot(w, cross(planar_hitpt_vector, v));
    auto beta = dot(w, cross(u, planar_hitpt_vector));
    if (!is_interior(alpha, beta, rec))
        return false;

    //ray hits the 2D shape;set the rest of the hit record and return true
    rec.t = t;
    rec.p = intersection;
    rec.mat = mat;
    rec.set_face_normal(r, normal);
    return true;
}

virtual bool is_interior(double a, double b, hit_record& rec) const {
    if ((a < 0) || (a > 1) || (b < 0) || (b > 1))return false;
    rec.u = a;
    rec.v = b;
    return true;
}
```

此时创建下面的场景并进行测试：

```c++
//main.cpp
#include "quad.h"
void quads() {
	hittable_list world;

	// Materials
	auto left_red = make_shared<lambertian>(color(1.0, 0.2, 0.2));
	auto back_green = make_shared<lambertian>(color(0.2, 1.0, 0.2));
	auto right_blue = make_shared<lambertian>(color(0.2, 0.2, 1.0));
	auto upper_orange = make_shared<lambertian>(color(1.0, 0.5, 0.0));
	auto lower_teal = make_shared<lambertian>(color(0.2, 0.8, 0.8));

	// Quads
	world.add(make_shared<quad>(point3(-3, -2, 5), vec3(0, 0, -4), vec3(0, 4, 0), left_red));
	world.add(make_shared<quad>(point3(-2, -2, 0), vec3(4, 0, 0), vec3(0, 4, 0), back_green));
	world.add(make_shared<quad>(point3(3, -2, 1), vec3(0, 0, 4), vec3(0, 4, 0), right_blue));
	world.add(make_shared<quad>(point3(-2, 3, 1), vec3(4, 0, 0), vec3(0, 0, 4), upper_orange));
	world.add(make_shared<quad>(point3(-2, -3, 5), vec3(4, 0, 0), vec3(0, 0, -4), lower_teal));

	camera cam;

	cam.aspect_ratio = 1.0;
	cam.image_width = 400;
	cam.samples_per_pixel = 100;
	cam.max_depth = 50;

	cam.vfov = 80;
	cam.lookfrom = point3(0, 0, 9);
	cam.lookat = point3(0, 0, 0);
	cam.vup = vec3(0, 1, 0);
	cam.background = color(0.70, 0.80, 1.00);
	cam.defocus_angle = 0;

	cam.render(world);
}
```

得到的结果如下图：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230925193309068.png" alt="image-20230925193309068" style="zoom:67%;" />

------

## Chapter 7 光

在光线追踪中，光源是十分重要的。因此首要步骤是能够让一些规整的几何形体可以代表光源。

### 1.自发光材质

首先，为material基类添加一个`emitted`函数，用来表示自发光属性。由于不发光的物体不需要真正实现`emitted`函数，因此对基类修改如下：

```c++
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
	//新增emitted函数
	virtual color emitted(double u, double v, const point3& p) const {
		return color(0, 0, 0);
	}
};
```

新增一个继承于material类的`diffuse_light`类，用来表示自发光的物体：

```c++
class diffuse_light :public material {
public:
	diffuse_light(shared_ptr<texture> a): emit(a){}
	diffuse_light(color c):emit(make_shared<solid_color>(c)) {}
	//自发光物体设定上不需要反射任何光
	virtual bool scatter(
		const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered) const override 
	{
		return false;
	}

	color emitted(double u, double v, const point3& p) const override {
		return emit->value(u, v, p);
	}

private:
	shared_ptr<texture> emit;
};
```

------



### 2.修改相机的`ray_color`函数

可以想到，如果一个发光物体的光线打到了相机，那么就可以被记录在对应像素上。此时同样可以利用光线追踪的思路从相机出发，如果打到了光源所在的物体，就把颜色加上光源`emitted`函数所表示的颜色。对相机的ray_color函数修改如下：

```c++
//camera.h
//根据输入的光线方向决定输出的颜色
color ray_color(const ray& r, int depth, const hittable& world) const {
    hit_record rec;  //rec会作为引用在函数里被修改

    //限制递归溢出
    if (depth <= 0) return color(0.0, 0.0, 0.0);
    if (!world.hit(r, interval(0.0001, infinity), rec)) return background;

    ray scattered;
    color attenuation;
    //color_from_emission记录光源发光的情况,如果不是自发光的就直接返回0了
    color color_from_emission = rec.mat_ptr->emitted(rec.u, rec.v, rec.p); 

    if (!rec.mat_ptr->scatter(r, rec, attenuation, scattered)) { //返回false的情况，比如打到了自发光的物体
        return color_from_emission;
    }
    color color_from_scatter = attenuation * ray_color(scattered, depth - 1, world); //可以想到，递归的光线如果打到光源也会算进最终颜色的计算里，因此可以实现间接光照的效果。
    return color_from_emission + color_from_scatter;
}
```

------



### 3.将物体转换为光源

我们设置一个quad，并将其赋予`diffuse_light`生成的材质，这样它就拥有了emiited的值，并且scattered函数会永远返回false。在main.cpp文件中加入一个场景函数并显示这个场景：

```c++
void simple_light() {
    hittable_list world;

    auto pertext = make_shared<noise_texture>(4);
    world.add(make_shared<sphere>(point3(0,-1000,0), 1000, make_shared<lambertian>(pertext)));
    world.add(make_shared<sphere>(point3(0,2,0), 2, make_shared<lambertian>(pertext)));

    auto difflight = make_shared<diffuse_light>(color(4,4,4));  //注意到光源的颜色比（1，1，1）更亮，这样就有足够的亮度照亮周围的物体了
    world.add(make_shared<quad>(point3(3,1,-2), vec3(2,0,0), vec3(0,2,0), difflight));

    camera cam;

    cam.aspect_ratio      = 16.0 / 9.0;
    cam.image_width       = 400;
    cam.samples_per_pixel = 100;
    cam.max_depth         = 50;
    cam.background        = color(0,0,0);

    cam.vfov     = 20;
    cam.lookfrom = point3(26,3,6);
    cam.lookat   = point3(0,2,0);
    cam.vup      = vec3(0,1,0);

    cam.defocus_angle = 0;

    cam.render(world);
}
```

最后渲染得到的结果如下：

![image-20230926133359916](Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926133359916.png)

注意到，我们生成的场景很多地方直接变成了全黑色，而不是之前的比较光亮的背景色。这是因为我们把camera的background属性设置成了黑色。

现在，我们可以再加入一个球形的光源，使场景更亮；

```c++
void simple_light(){
    //...
    world.add(make_shared<sphere>(point3(0, 7, 0), 2, difflight));  
    //...
} 
```

此时得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926141739994.png" alt="image-20230926141739994" style="zoom:67%;" />

------



### 4.创建Cornell Box

在main函数当中创建cornell box：

```c++
void cornell_box() {
    hittable_list world;

    auto red   = make_shared<lambertian>(color(.65, .05, .05));
    auto white = make_shared<lambertian>(color(.73, .73, .73));
    auto green = make_shared<lambertian>(color(.12, .45, .15));
    auto light = make_shared<diffuse_light>(color(15, 15, 15));

    world.add(make_shared<quad>(point3(555,0,0), vec3(0,555,0), vec3(0,0,555), green));
    world.add(make_shared<quad>(point3(0,0,0), vec3(0,555,0), vec3(0,0,555), red));
    world.add(make_shared<quad>(point3(343, 554, 332), vec3(-130,0,0), vec3(0,0,-105), light));
    world.add(make_shared<quad>(point3(0,0,0), vec3(555,0,0), vec3(0,0,555), white));
    world.add(make_shared<quad>(point3(555,555,555), vec3(-555,0,0), vec3(0,0,-555), white));
    world.add(make_shared<quad>(point3(0,0,555), vec3(555,0,0), vec3(0,555,0), white));

    camera cam;

    cam.aspect_ratio      = 1.0;
    cam.image_width       = 600;
    cam.samples_per_pixel = 200;
    cam.max_depth         = 50;
    cam.background        = color(0,0,0);

    cam.vfov     = 40;
    cam.lookfrom = point3(278, 278, -800);
    cam.lookat   = point3(278, 278, 0);
    cam.vup      = vec3(0,1,0);

    cam.defocus_angle = 0;

    cam.render(world);
}
```

得到的结果如下：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926142257363.png" alt="image-20230926142257363" style="zoom:67%;" />

注意到，得到的结果非常的noisy，这是因为光源比较小导致的。（我的理解是有些地方其实是打不到光源的，所以最后会被显示为黑色，这样的点混在场景中就会产生很多的噪点。）

做一个实验，同样的场景在实验室的机器上跑，将samples_per_pixel设置为2000，max_depth设置为50，得到的结果如下图：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926153456887.png" alt="image-20230926153456887" style="zoom:67%;" />

能明显感觉到噪点确实少了很多，不过带来的额外时间开销还是很恐怖的。（应该至少渲染了半个小时）

------

## Chapter 8 实例（Instance）

真正的Cornell盒里有两个立方体，并且具有一定的旋转角度。不过首先，我们先用六个quad来合成一个立方体：

```c++
// quad.h
#include "hittable_list.h"
inline shared_ptr<hittable_list> box(const point3& a, const point3& b, shared_ptr<material> mat)
{
    // Returns the 3D box (six sides) that contains the two opposite vertices a & b.

    auto sides = make_shared<hittable_list>();

    // Construct the two opposite vertices with the minimum and maximum coordinates.
    auto min = point3(fmin(a.x(), b.x()), fmin(a.y(), b.y()), fmin(a.z(), b.z()));
    auto max = point3(fmax(a.x(), b.x()), fmax(a.y(), b.y()), fmax(a.z(), b.z()));

    auto dx = vec3(max.x() - min.x(), 0, 0);
    auto dy = vec3(0, max.y() - min.y(), 0);
    auto dz = vec3(0, 0, max.z() - min.z());

    sides->add(make_shared<quad>(point3(min.x(), min.y(), max.z()),  dx,  dy, mat)); // front
    sides->add(make_shared<quad>(point3(max.x(), min.y(), max.z()), -dz,  dy, mat)); // right
    sides->add(make_shared<quad>(point3(max.x(), min.y(), min.z()), -dx,  dy, mat)); // back
    sides->add(make_shared<quad>(point3(min.x(), min.y(), min.z()),  dz,  dy, mat)); // left
    sides->add(make_shared<quad>(point3(min.x(), max.y(), max.z()),  dx, -dz, mat)); // top
    sides->add(make_shared<quad>(point3(min.x(), min.y(), min.z()),  dx,  dz, mat)); // bottom

    return sides;
}
```

接下来，我们可以利用上面的函数生成两个立方体放置于cornell box里面，在main.cpp的cornell_box()函数中补充以下内容：

```c++
void cornell_box(){
    //...
    world.add(box(point3(130, 0, 65), point3(295, 165, 230), white));
    world.add(box(point3(265, 0, 295), point3(430, 330, 460), white));
    //...
}
```

得到的渲染结果如下图所示：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926145235317.png" alt="image-20230926145235317" style="zoom:67%;" />

关于在光线追踪中做物体的平移、旋转，有一个不错的思路。实际上我们并不需要真正对物体进行空间变换。这里以平移为例，如果我们想实现一个物体的平移，只需要把光线沿着相反的方向平移即可。以下图为例：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926144237998.png" alt="image-20230926144237998" style="zoom:50%;" />

想象一个**平移**操作, 设定上我们把位于原点的粉色盒子向右平移两个单位，得到黑色的盒子，有两种策略：

- （1）我们可以将位于原点的粉红色盒子所有的组成部分的的x值+2；
- （2） 把盒子放在那里, 然后在hit函数中, 相对的将射线的原点-2。(这也是我们在ray tracing中惯用的做法)**【注: 射线原点-2计算出hit record后, 得到是左边盒子, 最后还要将计算结果+2, 才能获得正确的射入点(右边盒子)】**

不太好理解的话直接进入下面的代码来看吧。

> 补充：在后面作者提到了一种理解方式，我个人感觉更好理解，直接贴上来光线与改变位置的物体求交的流程：
>
> - （1）将光线从世界空间转移到模型空间；
> - （2）在模型空间里进行物体与光线求交的运算，如果相交计算相关信息；
> - （3）将交点从模型空间重新转移回世界空间。

------



### 1.Instance的平移

不妨来直接看一下代码的实现：

```c++
class translate :public hittable {
public:
    //构造函数里把包围盒也修改一下：
    translate(shared_ptr<hittable> p, const vec3& displacement) :object(p), offset(displacement) {
		bbox = object->bounding_box() + offset;
	}
    
	bool hit(const ray& r, interval ray_t, hit_record& rec) const override {
		//move the ray backwards by the offset
		//因为物体实际上是没平移过的，所以在计算求交的时候要把光线挪回去，但是在真正计算出交点信息之后要加回去，这是为了着色正确
		ray offset_r(r.origin() - offset, r.direction(), r.time());

		//determine where(if any) an intersection occurs along the offset ray
		if (!object->hit(offset_r, ray_t, rec))
			return false;
		//move the intersection point forwards by the offset
		rec.p += offset; //思考一下，rec.normal以及rec.uv是不需要修改的，因为这些参数不会随着平移改变。但对于旋转来说就需要改了。
		return true;
	}
    aabb bounding_box() const override { return bbox; }
private:
	shared_ptr<hittable> object;
	vec3 offset;
    
    aabb bbox;
};
```

对于aabb类，要重载+运算符，以实现aabb和vec3变量的相加：

```c++
//aabb.h，写在类外
aabb operator+(const aabb& bbox, const vec3& offset) {
	return aabb(bbox.x + offset.x(), bbox.y + offset.y(), bbox.z + offset.z());
}

aabb operator+(const vec3& offset, const aabb& bbox) {
	return bbox + offset;
}
```

因为`bbox.x`这种是interval类，所以interval类也要实现重载+运算符：

```c++
//interval.h，写在类外
interval operator+(const interval& ival, double displacement) {
    return interval(ival.min + displacement, ival.max + displacement);
}
interval operator+(double displacement, const interval& ival) {
    return ival + displacement;
}
```

------



### 2.Instance的旋转

这里需要回顾一下Games101所讲的旋转矩阵。再回顾一下我们在第一部分提到的：

> 光线与改变位置的物体求交的流程：
>
> - （1）将光线从世界空间转移到模型空间；
> - （2）在模型空间里进行物体与光线求交的运算，如果相交计算相关信息；
> - （3）将交点从模型空间重新转移回世界空间。

对于一个旋转的物体来说，不仅与光线的交点会变，相交表面的法线也会变。不过幸运的是如果不考虑物体的缩放问题，而只考虑平移和旋转的话，法线的旋转与物体的旋转可以用相同的公式来表示。

在这里我们只实现沿着y轴的旋转：

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926153206137.png" alt="image-20230926153206137" style="zoom: 80%;" />

rotate_y类的实现如下：

```c++
//hittable.h
class rotate_y :public hittable {
public:
	rotate_y(shared_ptr<hittable> p, double angle) :object(p) {
		auto radians = degrees_to_radians(angle);
		sin_theta = sin(radians);
		cos_theta = cos(radians);
		bbox = object->bounding_box();

		point3 min(infinity, infinity, infinity);
		point3 max(-infinity, -infinity, -infinity);

		//包围盒按照正常物体旋转了theta角来算，因此跟文档里的公式是theta与-theta的关系
		//其实这段代码就是找到了旋转之后的包围盒的每个轴的最小min值和最大max值
		for (int i = 0; i < 2; i++) {
			for (int j = 0; j < 2; j++) {
				for (int k = 0; k < 2; k++) {
					auto x = i * bbox.x.max + (1 - i) * bbox.x.min;
					auto y = j * bbox.y.max + (1 - j) * bbox.y.min;
					auto z = k * bbox.z.max + (1 - k) * bbox.z.min;

					auto newx = cos_theta * x + sin_theta * z;
					auto newz = -sin_theta * x + cos_theta * z;

					vec3 tester(newx, y, newz);
					for (int c = 0; c < 3; c++) {
						min[c] = fmin(min[c], tester[c]);
						max[c] = fmax(max[c], tester[c]);
					}
				}
			}
		}
		bbox = aabb(min, max);
	}

	bool hit(const ray& r, interval ray_t, hit_record& rec) const override {
		//change the ray from world space to object space
		auto origin = r.origin();
		auto direction = r.direction();

		origin[0] = cos_theta * r.origin()[0] - sin_theta * r.origin()[2];
		origin[2] = sin_theta * r.origin()[0] + cos_theta * r.origin()[2];
		direction[0] = cos_theta * r.direction()[0] - sin_theta * r.direction()[2];
		direction[2] = sin_theta * r.direction()[0] + cos_theta * r.direction()[2];

		ray rotated_r(origin, direction, r.time());

		//determine where(if any) an intersection occurs in object space
		if (!object->hit(rotated_r, ray_t, rec))
			return false;

		//change the intersection point from object space to world space
		auto p = rec.p;
		p[0] = cos_theta * rec.p[0] + sin_theta * rec.p[2];
		p[2] = -sin_theta * rec.p[0] + cos_theta * rec.p[2];

		//change the normal from object space to world space
		auto normal = rec.normal;
		normal[0] = cos_theta * rec.normal[0] + sin_theta * rec.normal[2];
		normal[2] = -sin_theta * rec.normal[0] + cos_theta * rec.normal[2];

		rec.p = p;
		rec.normal = normal;
		return true;
	}

	aabb bounding_box() const override { return bbox; }

private:
	shared_ptr<hittable> object;
	double sin_theta;
	double cos_theta;
	aabb bbox;
};
```

此时在main.cpp文件的`cornell_box()`函数中，我们可以加入平移与旋转的对应操作：

```c++
void cornell_box(){
    //...
    world.add(make_shared<quad>(point3(0, 0, 555), vec3(555, 0, 0), vec3(0, 555, 0), white));
	
    //这里做了一些修改，改为translate和rotate变换后的立方体
	shared_ptr<hittable> box1 = box(point3(0, 0, 0), point3(165, 330, 165), white);
	box1 = make_shared<rotate_y>(box1, 15);
	box1 = make_shared<translate>(box1, vec3(265, 0, 295));
	world.add(box1);

	shared_ptr<hittable> box2 = box(point3(0, 0, 0), point3(165, 165, 165), white);
	box2 = make_shared<rotate_y>(box2, -18);
	box2 = make_shared<translate>(box2, vec3(130, 0, 65));
	world.add(box2);
	
    //...
	camera cam;
}
```

渲染得到的结果如下图：



------

## Chapter 9 Volumes

一个不错的尝试是在光线追踪器里添加smoke/fog/mist，这些又被叫做*volumes* 或者 *participating media*。也可以为光追器添加次表面散射（subsurface scattering）的相关内容，不过这通常会增加软件架构的混乱，因为体积（volumes）与表面（surfaces）是不同的。一个trick是将volume作为随机表面。作者的原文如下：

> A bunch of smoke can be replaced with a surface that probabilistically might or might not be there at every point in the volume. 

直接看代码可以有更好的理解。

------

### 1.Constant Density Mediums

首先让我们来生成一个固定密度的Volume。光线可以在Volume内部发生散射, 也可以像图中的中间那条射线一样直接穿过去。Volume越薄越透明, 直接穿过去的情况就越有可能会发生。光线在Volume中直线传播所经过的距离也决定了光线采用图中哪种方式通过Volume。

<img src="Ray%20Tracing%20The%20Next%20Week%20%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0.assets/image-20230926164412225.png" alt="image-20230926164412225" style="zoom:67%;" />

当光线穿过Volume时，其可能在任何点发生散射。Volume的密度越大，越有可能发生散射。在任意微小的距离差 ΔL发生散射的概率如下:

$$probability=C⋅ΔL$$

其中 C是Volume的光学密度比例常数。 经过了一系列不同的等式运算, 你将会随机的得到一个光线发生散射的距离值。如果根据这个距离来说, 散射点在Volume外, 那么我们认为没有相交, 不调用`hit`函数。对于一个静态的Volume来说, 我们只需要他的密度C和边界。我会用另一个hittable物体来表示Volume的边界：

（剩下的有时间再看）



## 额外补充：

- Voronoi噪声：[Voronoi 噪音入门（又名Worley/Cell噪音，都是一个意思） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/53660462)
