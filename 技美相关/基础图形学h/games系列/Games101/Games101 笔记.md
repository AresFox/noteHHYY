# Games101 笔记

在这门课程当中,我们采用的坐标系是右手坐标系.

填坑:罗德里格斯公式,shading2后半到光线追踪的内容.

------



## Lecture 02 线性代数

- 基本符号:$\widehat{a}$表示与$\vec{a}$同方向的单位向量.
- 在图形学中,我们习惯于将向量竖着写,比如:$\vec{a}=\left(\begin{matrix} x_a \\ x_b\end{matrix}\right)$



### 1.点乘的作用

- 判断某个向量与已知向量的接近程度,比如判断同侧还是反侧(比如判断是否在NPC的视角范围内,就可以使用点乘判断,根据最后的值还可以得知二者间有多接近)
- 计算投影的长度
- 对于两个单位向量a,b,点乘结果是两者的夹角cos值.



### 2.叉乘的作用

- 判断在左/右
- 判断在里/外

![image-20220720221610782](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220720221610782-16583265717631-16583265742693.png)

> - (1)左侧的图体现了判断b向量在a向量左侧的方法,只需要用向量a叉乘向量b,发现结果方向是正方向,说明b在a左侧;
>
> - (2)右侧的图则可以判断P点是否在三角形ABC内,如果:
>
> `cross(AB,AP),cross(BC,BP),cross(CA,CP)`的结果是同一方向的,则说明P点在三角形ABC的内部.(其实这一条可以看作是上一条的扩展,因为在逆时针情况下,相当于AP在AB左侧,BP在BC左侧,CP在CA左侧,顺时针情况下也是类似的.**请注意这个知识点是光栅化的基础**)

补充:叉乘可以用矩阵来表示,对应的矩阵如下:
$$
\large\vec{a}×\vec{b}=A^*b=\left(\begin{matrix} 0 & -z_a & y_a \\ z_a&0&-x_a \\ -y_a&x_a&0 \end{matrix}\right)\left(\begin{matrix}x_b\\y_b\\z_b\end{matrix}\right)
$$

### 3.换坐标系投影

$\vec{p}=(\vec{p} \cdot \vec{u}) \vec{u}+(\vec{p} \cdot \vec{v}) \vec{v}+(\vec{p} \cdot \vec{w}) \vec{w}$

其中,各个给出的向量都是单位向量,并且$\vec{u},\vec{v},\vec{w}$构成一组坐标系的基.

------



## Lecture 03 04 变换

在学习或复习这一节的知识点前, 最好先对矩阵乘法的**实际含义**有一个比较清晰的认识,推荐看MIT线代前几节或者3B1B的线性代数系列.



### 1.考点自测

- 平移矩阵,旋转矩阵(包括二维旋转矩阵和三维绕着某一个坐标轴的旋转矩阵),缩放矩阵,错切矩阵(shear,注意矩阵推导),不做刻意说明的话认为旋转方向指的是逆时针方向

- 什么是线性变换?哪些变换是线性变换?(lecture 03 25:16秒左右)

- 旋转矩阵的特点:正交矩阵,所以有$R(-\theta)=R(\theta)^{-1}=R(\theta)^T$

**补充:**

**(a)线性变换的基本概念(根据3B1B):**

- (1)直线在变换后依然保持直线,不会发生弯曲;
- (2)原点保持不变

如果只能满足第一条而没有满足第二条,则这种变换叫做**仿射变换**.

**(b)仿射变换矩阵的特点:**

- (1)最后一行前几列为0,最后一列为1;
- (2)与平移有关的部分会体现在最后一列.

(其实上面的规律用矩阵乘法的本质也可以很好地理解,**所以矩阵乘法的本质一定要好好了解,可以降维打击.**)



### 2.齐次坐标

- 对于点来说,其齐次坐标的最后一个维度w=1;
- 对于向量来说,w=0.

注意,对于任意$[x,y,z,w]^T(w≠0)$,其实际上代表的点是$[x/w,y/w,z/w,1](w≠0)$

引入齐次坐标可以很好地解决平移问题.



### 3.逆变换以及变换合成

- 复合矩阵变换的顺序:缩放->旋转->平移(先做线性变换,再做非线性变换)

这里可以以下面一幅图为例,来说明复合变换的顺序:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220721220330550.png" alt="image-20220721220330550" style="zoom:67%;" />

可以看到,假如是先平移再旋转,由于旋转默认绕着原点旋转,因此先平移再旋转就会产生错误的结果.(感觉一种比较好理解的方式是通过3B1B的矩阵乘法变换的本质来看,平移肯定是要放到最后的(想象一下坐标系的旋转,缩放等动画))



### 4.如何计算绕任意点c或任意轴旋转?

- A'=**T(c)R(α)T(-c)**A,先将这个任意点移动到原点,然后再进行旋转,最后再平移回去. 

- 绕任意轴旋转,参考欧拉角

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220721221736429-16584130588713.png" alt="image-20220721221736429" style="zoom:80%;" />

- 罗德里格斯旋转方程以及证明过程(有时间复习):

绕着过原点的旋转轴n旋转角度α的旋转矩阵如下:
$$
R(n,\alpha)=cos(\alpha)I+(1-cos(\alpha))nn^T+sin(\alpha)\left(\begin{matrix} 0 & -n_z & n_y \\ n_z&0&-n_x \\ -n_y&n_x&0 \end{matrix}\right)
$$
推导过程见games101补充材料,或者以下博客:[(17条消息) 罗德里格斯（Rodrigues）旋转公式推导_CA727的博客-CSDN博客_罗德里格斯旋转公式](https://blog.csdn.net/cfan927/article/details/104342592),或者参考后面的四元数的对应博客，里面都有讲解。

说明:罗德里格斯旋转方程适用于绕着过原点的轴进行旋转,如果这个旋转轴不经过原点,则通过类似于**T(c)R(α)T(-c)**的操作移动到原点再进行旋转,最后再移回来.



### 5.关于四元数(等待补充)

四元数是为了解决欧拉角难以插值计算结果,以及产生的万向锁等问题.

强烈推荐的博客如下:[quaternion.pdf (krasjet.github.io)](https://krasjet.github.io/quaternion/quaternion.pdf)。由于篇幅关系在这里不系统地总结了，详见文件夹下的对应笔记（罗德里格斯公式推导也可以在这里看）。(**20230304补充:四元数的公式和基本定理的代码实现可以参考根目录下的`Quaternion.cpp`和`Quaternion.h`文件**)

#### 补充:关于万向锁

[bonus_gimbal_lock.pdf (krasjet.github.io)](https://krasjet.github.io/quaternion/bonus_gimbal_lock.pdf)

关于Unity欧拉角和四元数之间的关系,可以参考这个问题下的回答:(https://www.zhihu.com/question/321381903)



### 6.视图变换-投影变换

模型-视图-投影变换,MVP矩阵,类比于现实世界,M相当于拍照找好位置,V相当于摆好相机,而P则是拍摄投影过程

- 定义相机位置:

> $pos=\vec{e},gaze(lookat)=\vec{g},up\_direct=\vec{t}$

- 为了方便起见,不妨认为相机永远在原点,up指向y方向,lookat指向**-z**方向

![image-20220723112909855](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723112909855-16585469510601.png)

关于这个旋转变换矩阵,**有一个技巧是求出它的逆矩阵,然后直接转置就是正确的矩阵了.	**
                        <img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723113339586-16585472206743-16585472218345.png" alt="image-20220723113339586" style="zoom:67%;" />

截止到相机摆好位置,是模型视图矩阵(model-view),接下来的透视投影和正交投影这种是投影的过程.



#### (1)正交投影矩阵

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723113853700-16585475344327.png" alt="image-20220723113853700" style="zoom: 80%;" />

注意:z**轴定义的范围是[f,n]**,这是因为相机是朝向-z方向的,所以更远处的平面z的值应该更小一些,所以f<=n

像OpenGL的话这一部分采用的是左手系(后面介绍图形API的文档中会做总结)

所以,矩阵变换是先平移到原点,再进行缩放,Mortho的公式如下:
$$
\large M_{ortho}=\left[\begin{matrix} \frac{2}{r-l} & 0 & 0&0 \\ 0&\frac{2}{t-b}&0&0 \\ 0&0&\frac{2}{n-f}&0 \\0&0&0&1 \end{matrix}\right] \left[\begin{matrix} 1 & 0 & 0 &-\frac{l+r}{2} \\ 0&1&0&-\frac{t+b}{2} \\ 0&0&1&-\frac{n+f}{2} \\0&0&0&1 \end{matrix}\right]
$$

#### (2)透视投影矩阵

推导思路如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723115137030-16585482977179.png" alt="image-20220723115137030" style="zoom:80%;" />

- 近平面所有点坐标不变;
- 远平面中心点坐标不变;
- 远平面的z坐标不变.

基于以上三个思路,可以推导矩阵如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723121343850-165854962574511.png" alt="image-20220723121343850" style="zoom: 67%;" />

思考:对于透视投影中间的某个点,在"挤压"为正交投影的过程当中,z=(n+f)/2的点如何变化?更靠近远平面还是近平面?

> 我们可以代入向量$[x,y,(n+f)/2,1]^T$,并且用矩阵和其相乘,计算结果(**请注意,这里的计算结果需要归一化,即将w分量归为1**),并用不等式化简,可以得到数值变小了,会更靠近f这个面.
>
> 更普遍性的,设某点的z坐标为z0，设z0变化后z坐标为z0’，则(z0’-z0)是关于z0的二次函数（$\large z_0'-z_0=\frac{(n+f)z_0-nf}{z}-z_0$在z0=n或f时都为0，其余都小于0,当然是同乘z之后是二次函数），所以都是变远的。(推导的时候不要忘记要归一化,同乘z之后再用二次函数性质证明,证明过程可以看games101手写笔记的对应部分)

一些透视投影的参数如下:

aspect ratio,FOV

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723194443883-165857668601513.png" alt="image-20220723194443883" style="zoom: 67%;" />

根据上图,有如下公式(我们认为x,y方向上是关于坐标轴对称的):
$$
\tan{(fovY/2)}=t/|n| \\
aspect=r/t
$$
做了投影之后,范围会被规范化到$[-1,1]^3$,所以接下来要转换为屏幕坐标,这里我们认为像素的坐标是以左下角为定位点的,中心应该是(x+0.5,y+0.5),矩阵如下(复习前面的,先缩放再平移):
$$
M_{viewport}=\left[\begin{matrix} \frac{width}{2} & 0 & 0&\frac{width}{2} \\ 0&\frac{height}{2}&0&\frac{height}{2} \\ 0&0&1&0 \\0&0&0&1 \end{matrix}\right]
$$

------



## lecture 05 06 光栅化

- 显卡里的内存被称为显存
- LCD:液晶显示器(通过液晶的排布影响光的极化,也就是偏振方向)

### 1.光栅化基础

- 光栅化的图形基本单位是三角形,优势如下:
  - 三角形一定是平面的
  - 内外部定义清晰(可以直接用叉乘来定义)
  - 三角形内部可以用重心来进行插值渐变

#### (1)采样

采样的实质就是离散化的过程,比如光栅化这里就是利用像素重心对屏幕空间进行采样,也就是判断像素中心是否在三角形内(**注意,边界上的点在这门课里不做特殊规定,自己处理**):

伪代码如下:

```c++
bool inside(t,x,y){
    if point(x,y) in triangle t //具体的实现方式应该是用叉乘来实现
        return true;
    else return false;
} 
for (x=0;x<xmax;x++)
     for(y=0;y<ymax;y++)
         image[x][y]=inside(tri,x+0.5,y+0.5); //注意取得是像素重心
```

优化方案:

- 采用包围盒机制(AABB),只对包围盒内的三角形进行判断;
- 可以每一行找到最左和最右的点,比较适用于那种旋转的三角形,其AABB比较大.

仅采用上面的判断方案会导致锯齿(走样,aliasing)现象的产生,这是因为采样率没有很高.

------

实际的液晶显示方式:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723205635107-165858099643115.png" alt="image-20220723205635107" style="zoom: 67%;" />

实际的显示方式还是比较复杂的,比如Galaxy里面绿色的像素点会更多一些,这是因为人眼对绿色更为敏感,同时可以看到没有很严格的像素块定义.而下面那幅图则体现了彩色打印机的减色模型.**但在本门课程中我们还是认为像素内是均匀的**。

------



#### (2)反走样技术

采样会带来的问题:①锯齿  ②摩尔纹  ③车轮效应(时间上,类似于快速行驶的汽车轮子看上去是反着转的)

问题的来源:采样频率跟不上信号变换的速率

解决办法:

- (1)对三角形先进行一步滤波操作,使边缘模糊,然后再进行采样(**反过来先采样再模糊是错误的,原因和频域有关**);

##### (i)频域

傅里叶变换:空域->频域,回忆傅里叶变换公式以及欧拉公式.走样现象的来源如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723211037671-165858183841417.png" alt="image-20220723211037671" style="zoom:67%;" />

越靠下边的图,采样频率虽然不变（横轴方向的每一条竖线对应采样一次）,但是信号频率越来越高,会出现采样频率低于信号变换频率的现象,这样的话可能会出现同一采样方法采样两个函数会得到相同的结果,这就无法区分了，并且随着信号频率的增加越来越难以复原信号原来的样子，造成走样。

复习图像处理课程,高频表示边缘,因此高通滤波提取边缘,低通滤波做模糊,还可以只留下一段的频率其他的滤掉.

复习卷积操作,空域(games101里称之为时域)上的卷积操作等于频域上的乘积操作,**反过来也是成立的,空域上的乘积等于频域上的卷积**

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723211800060-165858228164619.png" alt="image-20220723211800060" style="zoom:67%;" />

空域上的卷积核越大,频域上滤掉的高频信号就越多,留下的信号越低频。

- 从频域上理解采样:采样就是重复频率/频域上的内容

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723213336603-165858321887121.png" alt="image-20220723213336603" style="zoom: 50%;" />

上面的图可以看到,空域上的采样其实就对应到频域上的将频谱复制粘贴的过程,并且空域上的采样频率越低,频域上的频谱就会越密集,当密集到一定程度上就会出现混叠现象(见上图最后sparse sampling),这就是走样的根源。

------



##### (ii)降低走样现象

根据上面所学,降低采样带来的问题可以从以下两个方面入手:

- (1)增加采样率,但有的时候会受到物理的限制;
- (2)先模糊去掉高频信号再进行采样(**反走样**)。
  - 如下图，去掉高频信号后的结果见"Then sparse sampling"部分，此时低一些的采样率就不会发生混叠了。


<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723214309697-165858379165523.png" alt="image-20220723214309697" style="zoom:67%;" />

在实际应用中,我们会根据三角形在像素中所占的比例的大小来决定像素的值(一般是直接平均求值,如下:)

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723214632267-165858399397625.png" alt="image-20220723214632267" style="zoom:67%;" />

------



##### (iii)超采样技术(MSAA)

原理:认为一个像素被分为很多个小像素,每个像素采样N*N次,根据采样点有几个在三角形里面来决定最后的颜色.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220723220417852-165858505957627.png" alt="image-20220723220417852" style="zoom:67%;" />

右图是模糊操作的结果,再下一步再去进行采样(其实这里的采样就很简单了,一个格子的颜色一致,中间点的颜色就是本身的颜色,依此来采样),MSAA本身是一个模糊的操作,并不能提高采样率，实际采样的时候依然是一个像素格填充一种颜色。

MSAA的问题在于会增大很多计算量,以下介绍一些工业界比较实用的反走样技术:

- 在工业界不会严格的一个像素分四个格子这样,而是会使用一些特殊的不规则的形状,或者是利用一些采样点的复用,来实现反走样技术.
- FXAA(fast approximate AA)
  - 图像的后期处理,得到锯齿图之后通过某种方式把锯齿弄掉.本质上是找到锯齿图的边界,然后把这些边界换成没有锯齿的边界,和采样无关.
- TAA(temporal AA)
  - 找上一帧的信息,比如用像素中的一个点来判断是否在三角形内,然后下一帧额外用像素中的另一个点来判断,同时上一帧的感知结果会对这一帧有一定作用,相当于把MSAA的样本点分布在时间上.(**注意,以上针对静止物体,对于移动物体来说详见games202**)

------



##### (iv)超分辨率

- 另一个概念:超分辨率(super resolution)
  - 比如把512 * 512的图拉成1024 * 1024的,如果不做处理肯定全是锯齿,所以如果有一张高分辨率的图但是采样率不高,则可以使用超分辨率的方式来完成.
  - DLSS(deep learning super sampling):利用深度学习来猜测,补上细节.
  - 超分辨率与抗锯齿虽然不是一种东西,在他们的本质是类似的.

------



#### (3)深度缓冲机制

- 复习:画家算法以及其带来的弊端;
- Z-buffer的机制如下(该算法广泛的应用于GPU当中):

```c++
initial depth buffer to ∞
During rasterization:
for (each triangle T)
    for(each sample(x,y,z) in T){
        if (z < zbuffer[x,y]){ //closest sample so far
            framebuffer[x,y] = rgb;//update color
            zbuffer[x,y] = z; //update depth
        }
        else
           ;
    }
        
//复杂度: O(n),n个三角形,三角形里面的像素数量是常数级的
```

**注:在这里我们认为z的值一直是正的,也就是z越小表示越近,越大表示越远。**而且如果一个像素有多个采样点的话，Z-buffer要考虑采样点，而不是仅考虑像素.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724113727645-16586338486751.png" alt="image-20220724113727645" style="zoom:67%;" />

比如上图,深度图黑色的地方表示离我们更近,深度值更小,反映在像素灰度上就更黑一些。

------



## lecture 07 08 着色

### 1.着色

- 在本门课中,定义如下:

> The process of applying a material to an object.

一些概念如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724115226079-16586347477623.png" alt="image-20220724115226079" style="zoom: 67%;" />

注意,**这里面给出的向量为了方便起见定义为单位向量.**

目前的着色我们并不考虑阴影的实现,进阶部分详见光线追踪部分.



#### (1)兰伯特模型

$$
L_d=k_d(I/r^2)max(0,n·l)
$$

从上式可以看出,漫反射项和观察的方向并没有关系,而是和光照方向与法线夹角有关.$k_d$越大,整体的效果越亮.



#### (2)Blinn-Phong

经验性观察发现,光的反射方向和视角方向越接近,镜面反射的结果应该越亮.但Bliin-Phong这里使用半程向量来进行求解.
$$
h=bisector(v,l)=\frac{v+l}{||v+l||} \\
L_s=k_s(I/r^2)max(0,cos\alpha)^p=k_s(I/r^2)max(0,n·h)^p
$$
随着高光系数的增加,高光的效果越集中.具体地可以看下面的图：

![image-20230313160404219](Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230313160404219.png)

如果不用半程向量来做,而还是用v和反射光r的夹角来做,就是普通的phong模型.

补充一下普通的phong模型里反射光$\vec{r}=(2\vec{l}·\vec{n})\vec{n}-\vec{l}$



#### (3)环境光

$$
L_a=k_aI_a
$$

是一个常数,相当于整体提升一个亮度,防止纯黑.



- 所以,总的光照模型公式如下:

$$
L=L_a+L_d+L_s=k_aI_a+k_d(I/r^2)max(0,n·l)+k_s(I/r^2)max(0,n·h)^p
$$

------



### 2.着色频率

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724121507037-16586361080517.png" alt="image-20220724121507037" style="zoom:67%;" />

图1:flat shading:一个面着一种色

图2:shade each vertex(Gourand shading):每个顶点有一个法线向量,计算顶点的颜色,三角形内部的颜色通过插值的方式得到.

图3:Phong Shading:对每个像素插值出法线方向,然后对每个像素进行着色.

注意:当模型足够复杂的时候,分的三角形数量很多,此时三种着色方式的结果相差不大就可以采用更加简单的着色方式。



### 3.逐顶点法线求解

顶点法线的求解方法如下：

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724165025288-16586526265419.png" alt="image-20220724165025288"  />

其中,第一种方式是构造顶点在圆上,接下来根据圆的法线性质求解;第二种方式则是把顶点所关联的三角形按面积进行加权平均,得到这个点的法线方向。



### 4.逐像素法线求解

- 用重心坐标进行插值（关于重心坐标插值的部分在后面会进行介绍）,并且不要忘记插值得到的法线也要归一化。



### 5.渲染管线介绍

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724165420797-165865286171511.png" alt="image-20220724165420797"  />

以下的五张图和对应在渲染管线中的阶段如下：

![image-20230528155333356](Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230528155333356.png)

- (1)顶点处理:MVP矩阵
- (2)光栅化
- (3)fragment processing:这里面有深度缓冲的机制Zbuffer
- (4)shading的过程:在vertex processing或者fragment processing步骤中(前面是gourand着色,后面是像素着色,**渲染管线中这两个由shader控制**)

- (5)纹理映射(texure mapping):和上面涉及到的部分类似.



### 6.shader

shader是每个顶点/像素都会执行一次,并不需要写for循环之类的.分别vertex shader或者fragment shader.

比如下面的GLSL程序:

```glsl
uniform sampler2D myTexture; //program parameter，全局变量
uniform vec3 lightDir;
varying vec2 uv; //per fragment value(interp. by rasterizer)
varying vec3 norm;

void diffuseShader()
{
    vec3 kd;
  	kd=texture2d(myTexture,uv);
    kd*=clamp(dot(-lightDir,norm),0.0,1.0);
    gl_FragColor=vec4(kd,1.0);
}
```

shader的学习网站:shader toy

其他的着色器：几何着色器，Compute Shader

------



## Lecture 09 纹理映射

- 任何一个三维物体的表面都可以认为是二维的.
- 纹理映射就是把一张二维的图映射到三维的物体上.
- 在二维的纹理上会定义一个坐标系,叫做U-V坐标系(通常来说U,V规定在0-1之间)。关于某个点映射到哪个纹理坐标，可以不细研究，因为通常是由美术工作人员直接UV展开并贴图的。
- 纹理应该争取设计的可以无缝衔接,有一种算法叫做Wang Tiling,后面会补充.
  - Wang Tiling可以参考的链接：[Wang tile(王浩瓷砖)与Corner tile生成纹理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/111809836)


### 1.三角形重心及作用

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220725225219940-16587607410341.png" alt="image-20220725225219940" style="zoom:80%;" />

对于三角形所在平面上的一点,满足:
$$
(x,y)=\alpha A+\beta B+\gamma C \\
\alpha+\beta+\gamma=1
$$

**在三角形内部的点符合性质：$\alpha,\beta,\gamma$都是非负的。**

比如对上图来说，A点的重心坐标就是(1,0,0)。

另外,$\alpha,\beta,\gamma$的值可以通过面积来进行求解,如下图：

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220725225704846-16587610271403.png" alt="image-20220725225704846" style="zoom:80%;" />

三角形的重心坐标$(\frac{1}{3}.\frac{1}{3},\frac{1}{3})$，将三角形的面积均等的分成了三份。

**可以利用三角形的重心进行插值操作，插值一些比如法线、颜色、深度等信息。**

重心坐标另外的计算方式：

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230529140426131.png" alt="image-20230529140426131" style="zoom: 67%;" />

重心坐标可以用来插值的属性：纹理坐标，颜色，深度，法线，材质属性等。

- 重心坐标的问题：在投影下无法保证重心坐标不变。
  - 如果在三维空间当中进行插值，就直接在三维中重心插值，而不能在投影之后再进行重心插值。比如说对深度插值的时候，理论上要找到三角形的三个顶点在三维空间中的坐标，在三维空间中进行插值（将投影后的结果转换回到三维空间中，只需要做投影的逆变换即可），再对应回二维空间即可。
  - 光栅化的时候如果做重心插值，那么其实需要将二维上的点映射到三维坐标系的三角形中，具体过程先不研究了。

至此,可以写出纹理映射的框架:

```c++
for each rasterized screen sample(x,y): //通常像素中心
	(u,v)=evaluate texture coordinate at (x,y); //计算该点映射出的u,v坐标,用重心坐标插值
     texcolor=texture.sample(u,v); //找到纹理图上的对应点
	set sample_s color to texcolor; //usually the diffuse albedo Kd
```



### 2.纹理放大

想象一下我们把一张小的纹理图贴在一面墙上,此时插值得到的点会去取纹理坐标中的非整数值,相当于纹理被拉大了,就会出现纹理分辨率不够的情况.此时对于非整数值有几种解决方案:

- 如果找到附近的整数点的值作为结果,就叫最近邻插值,这时多个pixel会采用一个texel（A pixel on a texture）的值作为结果;
- 双线性插值



#### (1)双线性插值

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230529144602638.png" alt="image-20230529144602638" style="zoom:67%;" />

定义lerp函数:

$lerp(x,v_0,v_1)=v_0+x(v_1-v_0)$

所以,红点处的值$f(x,y)$求解如下（相当于水平竖直两步插值）:
$$
u_0=lerp(s,u_{00},u_{10})\\
u_1=lerp(s,u_{01},u_{11})\\
f(x,y)=lerp(t,u_0,u_1)
$$



#### (2)Bicubic

取邻近的周围16个像素做水平和竖直的插值,每次用四个做三次方的插值,而不是线性插值.运算量更大但是效果更好一些。(有时间再补充)



### 3.纹理太大了引发的问题

会出现锯齿和摩尔纹，也就是走样了，并且越远处问题越大。这是因为近处的一个像素相对覆盖的纹理区域较小，而远处的一个像素覆盖的纹理区域则较大.如果像素覆盖区域较大(见下图右侧),则查询中心的纹理的时候,会导致结果错误(不能简单地用像素中心点的值替代整个覆盖区域的平均值).

注：下图当中的每个灰色区域是指一个像素覆盖的纹理区域

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727211215077-16589275364781.png" alt="image-20220727211215077" style="zoom:80%;" />

- 解决方案:可以用超采样，但是相对额外的计算量会很大。（比如说一个像素采样512个点，找到纹理上对应的值，然后求平均）

------



### 4.针对纹理太大——Mipmap

> 核心思路：不采样，立刻得到某个区域内的像素的平均值，转换为一种范围查询的问题。
>
> 因为根据上图，比较远的一个像素点会覆盖纹理上的一大片区域。如果能够直接求出覆盖区域的像素颜色的平均值，作为该像素纹理采样的结果，就可以避免用纹理中心进行采样造成的错误效果或者用超采样带来过多的性能开销。
>
> **MipMap就可以用来做范围查询。**
>
> Mipmap快,但是是一种近似,并且只能计算方形区域的值.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727212129538-16589280911893.png" alt="image-20220727212129538" style="zoom:80%;" />

整体分辨率依次缩小为原来的1/4,直到最后只剩一个像素点,mipmap可以提前根据纹理图生成。mipmap在图像处理界被称为图像金字塔。

额外的存储量为三分之一(每张图分辨率整体变成原来的四分之一,等比数列求和):

$1+1/4+1/16+1/64+...-1=4/3-1=1/3$

#### (1)如何计算纹理映射过去的范围?

![image-20220727213059442](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727213059442-16589286606405.png)

如上图，其实就是看原来的一个像素与临近像素的距离在纹理坐标系下是什么表示形式即可（也可以直接判断一个像素四个角在纹理坐标系下的围住区域）。然后近似用正方形来表示所围住的区域，正方形边长L计算如上，近似如下图：

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727213329635-16589288112597.png" alt="image-20220727213329635" style="zoom:80%;" />

巧妙之处在于,如果我们发现计算出的变成L=4,那么就可以确定这个区域在金字塔第二层时会变成单像素,所以得到层数$D=log_2L$。也就是说可以计算出当前像素框住的纹理区域在第几层会变成一个像素，直接查询对应层Mipmap的对应像素即可。

- 注:如果计算出的层数并不是整数,**则可以考虑进行插值处理**.在一个场景中,近相机的位置用mipmap层数较低的去进行查询,而远离相机的则使用高层数mipmap来查询.

- 插值:Trilinear interpolation:在两层mipmap中分别做双线性插值,然后再在两层之间进行一次插值,这种叫做三线性插值.(其实三线性插值所需要的额外性能也不是很多,所以被广泛使用)

#### (2)Mipmap的局限性

- 只能做方形区域的查询,对于非方形区域会比较麻烦.
- 会导致很远处的纹理出现了糊掉了的现象（模糊过分了）,这也是因为mipmap有做一些近似处理,糊掉的效果如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727215218210-16589299418929.png" alt="image-20220727215218210" style="zoom: 80%;" />

解决方案是采用各向异性过滤.

------



### 5.各向异性过滤

除了Mipmap的每次长宽分辨率各缩小一倍,各向异性过滤补齐了一些长方形的区域带来的像素采样问题(代价:总的开销会变成原来的三倍).在打游戏的时候如果看到2x基本说明方向上压缩一次,4x就是方向上压缩两次。这种开销随着nx的增大最终是收敛的，所以如果显存高的话完全可以把这一项调到最高。

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727215547982-165893015192411.png" alt="image-20220727215547982" style="zoom:80%;" />

不过,对于斜着的长方形采样区域,各向异性过滤依然做的不够好.



### 6.EWA过滤

简单提一下,对于任何一个不规则形状,EWA会用不同的椭圆形来覆盖需要被查询的区域.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727215933436-165893037537813.png" alt="image-20220727215933436" style="zoom:80%;" />

------



### 7.纹理应用

- 可以制作各种贴图
- environment map环境贴图:假设光源只记录方向信息,认为其无限远.
  - Spherical environment map:将不同的环境光记录在球面上,需要的时候进行展开.这样的问题是展开的时候会出现扭曲现象(想象展开地球仪,南北极出现扭曲)
    - 解决方案:Cube Map

#### (1)纹理应用1——cube map

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727221301353-165893118262715.png" alt="image-20220727221301353" style="zoom: 67%;" />

这样就不会出现spherical environment map带来的扭曲问题,cube map展开后有六个面,当然也需要对不同方向来的光(这里的光不止包括直射的光)通过算法转换到cube map上的对应点方便采样。



#### (2)纹理应用2——凹凸贴图

通过凹凸贴图,**本质上是对法线进行"扰动"**。

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727221842986-165893152503517.png" alt="image-20220727221842986" style="zoom:80%;" />

推导过程如下:

首先,我们以二维为例,后面会扩展到三维:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727222405974-165893184767519.png" alt="image-20220727222405974" style="zoom:67%;" />

假设在**局部坐标系当中**原来法线的方向是(0,1)，那么可以先求出变化后点的切线,根据上图有：$dp=c*[h(p+1)-h(p)]$ )(c是常数bump scale)，这时的切线方向为（1，dp），而法线与切线垂直，因此求出法线为$n(p)=(-dp,1).normalized()$。

类比到三维,**在局部坐标系当中的法线变化为:**

origin normal n(p)=(0,0,1)
$$
dp/du=c1*[h(u+1)-h(u)]\\
dp/dv=c2*[h(v+1)-h(v)]\\
n'=(-dp/du,-dp/dv,1).normalize()
$$
上式中的n'其实就是（1，0，dp/du）和（0，1，dp/dv）的叉乘结果归一化的解。因为法线与两条切线分别垂直，所以求出u和v方向上的切线就可以确定该点的切平面，叉乘可以得到法线。

补充:**位移贴图displacement mapping**，也是用纹理定义变化,位移贴图真的会对模型的顶点进行移动.对于凹凸贴图来说没有自阴影现象,并且纹理交界处会穿帮,而位移贴图则解决了这个问题,但是要求模型要足够细致,三角形足够多。

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220727223500230-165893250168221.png" alt="image-20220727223500230" style="zoom:80%;" />

额外资料：dynamic tessellation，可以查阅与DirectX相关的文档，应用这种技术可以一开始不提供非常精细的模型，根据需要再进行细分。



#### (3)其他应用

- 生成噪声,比如perlin noise
- ambient occlusion,AO贴图
- 体渲染技术(后面会介绍)

------



## lecture 10 11几何(1)

### 1.几何的表示方式

#### （1）隐式

隐式：不告诉点具体在哪，而是告诉点之间的关系。举例：x+y+z=1；推广的情况需要满足：f（x，y，z）= 0。

- 缺点：比较难根据表达式看出具体的形状，同时复杂形体也比较难表示，比如一个奶牛；
- 优点：比较好判断一个点是否在面上，还是物体内外；隐式表面也比较好计算和光线求交；比较容易控制拓扑结构的改变（比如流体）

其他的隐式表示方法：

- （a）Constructive Solid Geometry（CSG），通过一系列基本几何体的布尔运算来表示。

- （b）距离函数（Distance Functions）：不直接描述表面，而是描述任何一个点到表面的最近距离，这个距离有正有负，比如在物体内部这个距离就是负的。此时表面就是距离函数f=0的点构成的集合。

  > 举个距离函数的例子：
  >
  > ![image-20230530185322041](Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230530185322041.png)
  >
  > 用距离函数来做Blend操作，可以得到类似如下的融合效果：
  >
  > ![image-20230530185417640](Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230530185417640.png)
  >
  >  再比如，shadertoy当中的那个蜗牛的形状就是用距离函数来做的。

- （c）水平集（Level Set Methods）：与距离函数类似，只不过用一些近似的格子来辅助寻找f=0的点，如下图所示：

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230530190022973.png" alt="image-20230530190022973" style="zoom:80%;" />

这种思想延伸到三维空间上，就比如CT，用三维的纹理表示人体各部分的密度，让密度函数f（x）等于指定值就可以提取出人体对应的组织，方便查看状态（比如骨头，肌肉......）

- （d）分形：自相似，类似计算机中的递归思想。



#### （2）显式

显式：比如直接给出所有点的表示，或者是用参数映射（parameter mapping）的方法，如下图：

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230530165057623.png" alt="image-20230530165057623" style="zoom:80%;" />

举一个参数映射的例子：$f(u, v)=((2+\cos u) \cos v,(2+\cos u) \sin v, \sin u)$,此时比较容易重建出形状，但判断一个点是否在表面上/内外就会比较麻烦。

常见显式的表示方式：

- 点云：通常三维空间扫描得到的就是点云，点云经常会被转换到多边形面；
- 多边形面：如三角形面，四边形面；
- 贝塞尔曲线，B样条曲线等



### 2.贝塞尔曲线

用一系列的控制点来定义某条曲线。比如以下述四个控制点为例：

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230531110527007.png" alt="image-20230531110527007" style="zoom:67%;" />

起始P0处的切线方向为P0P1方向，终点P3处的切线方向为P2P3方向。曲线必须经过起止点，但并不要求经过中间的控制点。

#### （1）绘制贝塞尔曲线的算法——de Casteljau Algorithm

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230531111450233.png" alt="image-20230531111450233" style="zoom:67%;" />

从上式可以看出，表达式应该是一个类似于多项式展开的形式，推广到一般情况，有：

![image-20230531112359561](Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230531112359561.png)

贝塞尔曲线的性质：

- 贝塞尔曲线一定会经过控制点的起点和终点；

- 对于Cubic case（四个控制点的）：b'（0）=3（b1-b0），b'（1）=3（b3-b2）

- 对于**仿射变换**而言，先对控制点进行仿射变换，再基于控制点绘制出贝塞尔曲线，则绘制出的曲线与仿射变换前是一致的。注意针对投影这种变换是没有这个性质的。

- **凸包性质**，形成的贝塞尔曲线一定在几个控制点组成的凸包内。

  

#### （2）Piecewise（逐段的）贝塞尔曲线

常见的逐段控制一般是cube Bezier，即每四个控制点为一段。相关的DEMO可以查看这个网站：[Bezier Curve Edit (hws.edu)](https://math.hws.edu/eck/cs424/notes2013/canvas/bezier.html)

<img src="Games101%20%E7%AC%94%E8%AE%B0.assets/image-20230531141343781.png" alt="image-20230531141343781" style="zoom:67%;" />

如上图，1234这四个控制点组成一段cube贝塞尔曲线，如果想让两段曲线平滑过度的话需要保证两侧的切线方向正好相反且控制点间距相等，例如上图34和54方向相反且间距应相等。

**C0连续与C1连续**





## lecture 12 几何(2)





## lecture 13 14 光线追踪

### 1.为什么使用光线追踪?

- 光栅化与光线追踪(resterization&ray tracing):
  - 光栅化:做不到全局效果,**速度快,但是渲染质量比较低**;
  - 光线追踪:准确,但是比较慢.

- **光栅化是离线渲染,光线追踪是实时渲染**



### 2.光线追踪算法

#### (1) 思路

- (1)光沿直线传播(实际不够准确)
- (2)光线间不相互发生碰撞(实际不够准确)
- (3)光路可逆性(**从光源->摄像机,但是在计算的时候射线从摄像机出发**)



#### (2) Ray Casting

源自于[Appel 1968 - Ray casting].

**该种做法是基于以下思路的**:

① 通过一个像素打出一条光线;

②通过碰撞点与光源连线,来确定是否会产生阴影.

![image-20220724170713139](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724170713139-165865363451513.png)



#### (3)Recursive Ray Tracing(Whitted-Style)

- 注意,在这种方式当中使用了递归的思想,使用递归的光线继续进行求交运算,找到交点与光线相连判断颜色(根据利用Phong等模型)

![image-20220724170725231](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724170725231-165865364647015.png)



### 3.光线求交方程

#### (1)光线与球面相交

- (1)光线方程

$r(t)=o+td(0≤t<∞)\quad①$

- (2)球面方程

$p:(p-c)^2-R^2=0\quad ②$

将①②方程联立求解,解得:$(o+td-c)^2-R^2=0$,**通过求解二次方程,来解出t的值,通过t的值来判定是否相交以及相交的交点数**.



#### (2) **隐式平面如何求交?**

联立方程组:
$$
\begin{cases}
  r(t)=p=o+td(0≤t<∞)\quad① \\ 
  p:f(p)=0\quad ②
\end{cases}.
$$
也就是说,解方程$f(o+td)=0$即可.



#### (3)光线与三角形求交

##### (i)复习:三角形的重心

通过重心的插值,可以得到一种平滑过渡的效果.

![image-20220724171008429](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724171008429-165865380992817.png)

##### (ii)算法

判断光线是否与三角形相交,可以按照下面的步骤(**注:该算法需要进行优化**):

- 1.判断光线是否与三角形所在平面相交;
- 2.判断交点是否在三角形的内部(**使用重心**)

已知平面方程如下:

$ax+by+cz+d=0$

则对于平面上的点$p$,有:

- $p:(p-p')·N=0$  ($p'$是平面上一点,$N$是平面的法线方向,$\vec{N}=(a,b,c)$)



**此时我们联立光线方程与平面方程,求解二者的交点$p$**
$$
\begin{cases}  r(t)=p=o+td(0≤t<∞)\quad① \\   p:(p-p')·N=0\quad ②\end{cases}.
$$
将①代入②,有$(o+td-p')·N=0$,通过点乘的分配律求解出:$\large t=\frac{(p'-o)·N}{d·N}$(需要验证是否满足$0≤t<∞$)

接下来,只需要判断$o+td$这个点是否在三角形内部即可(使用重心坐标来判断)



##### (iii)补充:`Möller Trumbore Algorithm`

- 这是一种简化算法,可以立刻求解判断光线求交

该算法的推导可以参考下列博客:

[(8条消息) Möller-Trumbore算法-射线三角形相交算法_zhanxi1992的博客-CSDN博客](https://blog.csdn.net/zhanxi1992/article/details/109903792)

推导过程如下:

> 利用重心坐标一步求解交点是否在三角形内,我们需要求解如下方程:
>
> $\vec{O}+t\vec{D}=(1-b1-b2)\vec{P_0}+b_1\vec{P_1}+b_2\vec{P_2}$
>
> (以下证明为了方便,不再标注向量符号)
>
> 将括号展开,移项可得:
>
> $O-P_0=(P_1-P_0)b_1+(P_2-P_0)b_2-tD\quad ①$
>
> 通过观察可知,$O-P_0,(P_1-P_0),(P_2-P_0)$都是已知量,因此可以将其用变量来表示:
> $$
> E_1=P_1-P_0 \\
> E_2=P_2-P_0 \\
> S=O-P_0
> $$
>
> 此时①式可化为:
>
> $S=E_1b_1+E_2b_2-tD$
>
> 将其转为矩阵乘法的表示形式,有:
> $$
> \begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}
> \begin{bmatrix}
> t\\b_1\\b_2
> \end{bmatrix}=S
> $$
> 这个方程形如$Ax=c$,因此可以用**克拉默法则**求解出$t$的值(注意到此时的)$$\begin{bmatrix}-D&E_1&E_2\end{bmatrix}$$其实是一个$3×3$的矩阵(因为每一项是一个列向量),这里我们考虑$det\begin{bmatrix}-D&E_1&E_2\end{bmatrix}≠0$的情况.所以有:
>
> ​																						$$ \large{t=det\frac{\begin{bmatrix}S&E_1&E_2\end{bmatrix}}{\begin{bmatrix}-					D&E_1&E_2\end{bmatrix}}} \quad ②$$
>
> 关于克拉默法则的更进一步介绍,可以参考[克拉默法则 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/161429987)
>
> 注意到:
> $$
> det\begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}=det
> \begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}^T
> $$
> **此时由向量混合积可以得出:**
>
> 分母部分:
> $$
> det\begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}=det
> \begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}^T=-D·(E_1×E_2)=E_1·(D×E_2)
> $$
> 令$S_2=D×E_2$
>
> 所以就可以推导出:
> $$
> det\begin{bmatrix}
> -D&E_1&E_2
> \end{bmatrix}=E_1·S_1 \quad ③
> $$
> 对于分子部分也是同理,令$S_2=S×E$
>
> 可以用类似的推导方式推导出:
> $$
> det\begin{bmatrix}
> S&E_1&E_2
> \end{bmatrix}=S_2·E_2 \quad ④
> $$
> 将③④代入②,解得:
>
> ​																					$t=\LARGE\frac{S_2·E_2}{E_1·S_1}$
>
> 同样,也是可以解出$b_1,b_2$的:
>
> ​																					$b_1=\LARGE\frac{S_1·S}{E_1·S_1}$
>
> ​																					$b_2=\LARGE\frac{S_2·D}{E_1·S_1}$
>
> 所以,我们只需要判断是否满足$t≥0,b_1≥0,b_2≥0,(1-b_1-b_2)≥0$即可判断光线是否与三角形相交.

于是,就得到了PPT当中的公式:

![image-20220724171034279](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724171034279-165865383685719.png)



### 4.光线-表面相交的空间加速

#### simple intersection

- 找到最近的相交点(比如说求解出来的合理的最小的t值)

接下来为了增加这种加速结构的普适性,我们用objects代替triangles(但这里的objects只是一个概念,不是真正的完整的物体/模型,比如后面会说包围盒中有一个object,理解意思即可)



#### (1) Bounding Volumes(包围盒)

- 物体完整放置于包围盒当中.
- 如果光线不与包围盒相交,则一定不会与其中的object相交;
  - 换句话说,先检查是否与BVol(即包围盒)相交,如果相交再检查是否与里面的物体相交.

##### (i) AABB:轴对齐包围盒

以下例子以2D的AABB为例,实际上三维情况下也是类似的.

对于一条光线而言,我们不妨将其先拓展为直线,那么其一定会与正方形的两组对边各有两个交点(同样,暂时不予考虑平行这种特殊情况),如下左侧和中间两张图.

![image-20220724171053484](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724171053484-165865385897621.png)

**注意到以下结论**:

- **光线进入box,只有两组边都进去了才算进去**
- **光线离开box,只要一组出来就算出来**

对于每一组对边,记录$t_{min}和t_{max}$,则有:

$\large t_{enter}=max(t_{min}),t_{exit}=min(t_{max})$



##### 最终结论

同时,我们还必须要考虑$t$的限制范围

- 一种特殊情况在于如果$t_{exit}>=0$同时$t_{enter}<0$,**那么可以想象到这种情况应该是光源在盒子的里面,显然这种情况也是相交的.**

最终得出结论,**即光线和AABB相交,当且仅当**:

$ \large t_{enter}<t_{exit}\&\&t_{exit}>=0$



##### 为什么采用AABB?

- 因为判断光线与AABB求交是比较简单的,公式如下:

![image-20220724171116049](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724171116049-165865387739023.png)

上图中那个公式的推导如下:

![image-20220724171126190](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724171126190-165865388875625.png)

类比之前的与普通平面相交:$\large t=\frac{(p'-o)·N}{d·N}$,这种与AABB相交的情况下$t$的计算公式更为简单.



## lecture 15 辐射度量学+全局光照

辐射度量学:Radiometry

学习目的:将光的属性精确的表现出来

学习内容:依然认为光是沿着直线传播的,依然是基于几何学.

Radiant flux,intensity,irradiance,radiance



### 1.radiant energy and flux

- radiant energy:电磁辐射的能量,单位是焦耳,比如光源辐射出来的肯定是能量,就可以用radiant energy

​								$Q[J=Joule]$

- randiant flux(power):单位时间的能量

$$
\Phi \equiv \frac{dQ}{dt}[W=Watt][lm=lumen]^*
$$

功率,单位是瓦特,在图形学里往往我们考虑的是单位时间(照的时间越长物体越热),其实还有另一个单位是流明(表示光的亮度)

flux的另一种理解: #photons flowing through a sensor in unit time.(#photons表示光子数量)



### 2.其他物理量

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724172908366-16586549495571.png" alt="image-20220724172908366" style="zoom:80%;" />

- radiant intensity:后面可能简称intensity,光源会往各个方向辐射能量,定义一个与方向有关的量叫做radiant intensity.

- 物体表面接收多少光的能量,叫做irradiance
- 光线传播中如何度量能量,叫做radiance



### 3.radiant intensity

- power **per unit solid angle** emitted by a point light source

$$
I(\omega)\equiv \frac{d\Phi}{d\omega}[\frac{W}{sr}][\frac{lm}{sr}=cd=candela]
$$

The candela is one of the seven SI base units. dω是单位立体角.

#### (1)立体角的定义

对于二维来说,我们定义弧度.而扩展到三维来说,就利用立体角来定义.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724173847848-16586555292383.png" alt="image-20220724173847848" style="zoom:80%;" />

立体角:面积A/半径的平方.(注意这个面要正对着球心)

具体推导如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724174241486-16586557623415.png" alt="image-20220724174241486" style="zoom:80%;" />

重点是要看到$dω=sin\theta d\theta d\phi$计算方式,这里可以回顾一下高数下的内容.(理解上是θ和Φ各变化一点点,立体角发生的变化)

对于整个球来说;
$$
\Omega= \int_{s^2}d\omega=\int_{0}^{2\pi}\int_{0}^\pi sin\theta d\theta d\phi = 4\pi
$$
在辐射度量学中,我们直接用ω来表示空间中的方向(ω可以由θ和Φ来进行定义,unit length)

因为有:
$$
I(\omega)\equiv \frac{d\Phi}{d\omega}[\frac{W}{sr}][\frac{lm}{sr}=cd=candela]
$$
所以想要求出单位时间的能量Φ,我们就可以对各个立体角方向的$I$进行积分求解,这就可以得到一个点光源均匀的往四周辐射能量,对应的intensity计算公式如下:
$$
\Phi=\int_{S^2}I d\omega=4\pi I \\
I=\frac{\Phi}{4\pi}
$$


### 4.irradiance

- power per unit area

$$
E(x)\equiv \frac{d\Phi(x)}{dA}[\frac{W}{m^2}][\frac{lm}{s^2}=lux]
$$

与intensity做对比,intensity是对应立体角上,而irradiance则是对应单位面积上.**这个单位面积是指和入射光线垂直的面积,如下:**

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724183200533-16586587217807.png" alt="image-20220724183200533" style="zoom: 67%;" />

用上述公式来理解光到球壳上某一点的能量值:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724183454267-16586588952999.png" alt="image-20220724183454267" style="zoom:67%;" />

其实,intensity并没有发生衰减,而是irradiance在发生衰减.



### 5.radiance(重要)

- 为了描述光在传播过程中的属性.power per unit solid angle, per projected unit area.

![image-20220724183711660](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724183711660-165865903251211.png)
$$
L(p,\omega)\equiv \frac{d^2\Phi(p,\omega)}{d\omega dA cos\theta}\\ 单位[\frac{W}{sr\ m^2}][\frac{cd}{m^2}=\frac{lm}{sr\ m^2}=nit]
$$
(这里的cosθ则是参与计算了与光垂直的面的面积,之前有说)

因此,radiance的定义可以这么理解:

- irradiance per solid angle
  - it is the light arriving at the surface along a given ray(point on surface and incident direction)
  - ![image-20220724191311989](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724191311989-165866119340513.png)
- intensity per projected unit area
  - for an area light it is the light emitted along a given ray(point on surface and exit direction)

按第一种理解方式,就是$\large L(p,\omega)=\frac{dE(p)}{d\omega cos \theta}$

按第二种理解方式,就是$\large L(p,\omega)=\frac{dI(p,\omega)}{dA cos \theta}$



#### irradiance&radiance

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724191819358-165866150034915.png" alt="image-20220724191819358" style="zoom:80%;" />

其实就是radiance在irradiance的基础上加了一个方向性.

根据前面的可以求解积分:
$$
∵ L(p,\omega)=\frac{dE(p)}{d\omega cos \theta}\\
∴dE(p,\omega)=L_i(p,\omega)cos\theta d\omega \\
∴E(p)=\int_{H^2}L_i(p,\omega)cos\theta d\omega
$$
所以,p点收到的所有能量就是从各个方向进来的能量求和.



### 6.BRDF(双向反射分布函数)

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724202513399-165866551474417.png" alt="image-20220724202513399" style="zoom:80%;" />

Differential irradiance incoming:
$$
dE(\omega_i)=L(\omega_i)cos\theta_id\omega_i
$$
differential radiance exiting(due to $dE(\omega_i)$): $dL_r(\omega_r)$

BRDF定义如下(出射的radiance比上入射的irradiance):

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724203023726-165866582467219.png" alt="image-20220724203023726" style="zoom:80%;" />
$$
f_r(w_i\rightarrow w_r)=\frac{dL_r(\omega_r)}{dE(\omega_i)}=\frac{dL_r(\omega_r)}{L_i(\omega_i)cos\theta_id\omega_i}[\frac{1}{sr}]
$$
BRDF描述了光线是如何与物体作用的,可以用BRDF项来定义不同的材质.

#### (1)反射方程

现在,BRDF说明了沿着某一个方向出射的情况,那么对所有方向的入射的irradiance进行积分(将每个入射方向对出射方向的贡献加起来)**可以得到反射方程如下**:
$$
L_r(p,\omega_r)=\int_{H^2}f_r(w_i\rightarrow w_r){L_i(p,\omega_i)cos\theta_id\omega_i}
$$


#### (2)渲染方程

对于上式,有一点我们没有考虑,那就是没有考虑非光源的光线到达所在点的情况.所以上式的$L_i$项其实是一种递归的定义方式.

暂时先不考虑递归,求出通用化的渲染方程:

渲染方程需要加入物体的emission自发光属性,即可写出:
$$
L_0(p,\omega_0)=L_e(p,\omega_0)+\int_{\Omega+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)d\omega_i
$$
(**注意:我们假设所有的方向都朝外,$\Omega+,H^2$**的意思都是半球)

关于渲染方程的解法,在后面会进行介绍.

如果我们没有考虑到其他物体反射的光打在待求解的点上,则可以将其他反射光的物体也统一理解成"光源",这样渲染方程就可以进行化简:
$$
l(u)=e(u)+\int l(v)K(u,v)dv
$$
$l(v)$就是统一成从其他的点来的光线(包括直接和间接入射光),K(u,v)叫做`kernel of equation light transport operator`如果再次简写,可以写为:
$$
L=E+KL
$$
这是一种递归的定义,可以这样求解:
$$
\begin{align*}
&L=E+KL \\
&IL-KL=E \\
&(I-K)L=E \\
&L=(I-K)^{-1}E \\
&这种算子有类似于泰勒展开的形式,故有:\\
&L=(I-K)^{-1}E =(I+K+K^2+K^3+...)E \\
&L=E+KE+K^2E+K^3E+...
\end{align*}
$$
从最后一个式子中,就可以看出光源经过一次弹射的结果(直接光照),两次弹射的结果(间接光照)

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724210838853-165866812004421.png" alt="image-20220724210838853" style="zoom: 67%;" />

把这些全都加在一起,**这就是全局光照.**

随着弹射次数的计算不断增加,最终的结果会倾向于收敛到某一个值(不会无限变亮),会在路径追踪中进行解释(**路径追踪就是解渲染方程的一种方式**).



### 7.概率论基础

PDF:概率分布函数,比如X的概率密度函数符合X ~ p(x),则有:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724213324467-165866960554023.png" alt="image-20220724213324467" style="zoom:80%;" />
$$
p(x)\ge0 \ and \int p(x)dx=1 \\
E[X]=\int xp(x)dx
$$
E(x)是说x的期望.

另外一个性质是,如果X~p(x),且Y=f(X),则有:
$$
E[Y]=E[f(X)]=\int f(x)p(x)dx
$$


## lecture 16 蒙特卡洛路径追踪

### 1.蒙特卡洛积分

对于一个复杂的函数的定积分,希望用近似的方式求得定积分的结果.**蒙特卡洛采用的是一种随机采样的方法**.随机采样到x得到的f(x)是多少,就计算这个矩形的面积作为积分的结果,多次采样求出平均值就是蒙特卡洛方法.例如,我们要求解如下积分:
$$
\int_{a}^{b}f(x)dx
$$
令采样点X符合某一概率密度函数$X_i\sim p(x)$,蒙特卡洛近似结果为:
$$
F_N=\frac{1}{N} \sum_{i=1}^N \frac{f(X_i)}{p(X_i)}
$$
比如说,如果X在定义域内均匀采样,那么$X_i\sim p(x)=\frac{1}{b-a}$,此时:
$$
F_N=\frac{b-a}{N} \sum_{i=1}^N {f(X_i)}
$$
有几点说明:

- 采样点越多,结果越近似;
- 如果对x进行积分,就要对x进行采样



### 2.路径追踪

对比Whitted光线追踪,这种光线追踪的特点如下:

- always perform specular reflections/refractions
- stop bouncing at diffuse surfaces

这种光线追踪无法很好地解决磨砂类的材质,这就需要路径追踪.

正确的渲染方程如下:
$$
L_0(p,\omega_0)=L_e(p,\omega_0)+\int_{\Omega+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)d\omega_i
$$

- (1)这是一个积分形式;
  - 解决方案:蒙特卡洛积分
- (2)这是一个递归的过程.



### 3.蒙特卡洛解积分

暂时我们先考虑上面的渲染方程的Li项为直接光源照射过来的项,且暂时忽略自发光,则解法如下:
$$
L_0(p,\omega_0)=\int_{\Omega+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)d\omega_i
$$
且有蒙特卡洛方程:
$$
F_N=\frac{1}{N} \sum_{i=1}^N \frac{f(X_i)}{p(X_i)} \quad  X_k \sim pdf(x)
$$
核心思路在于随机采样,并计算对应被积分函数的值,然后利用蒙特卡洛方程来求解.

这里的f(x)就是$L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)$,$pdf(x)\sim 1/2\pi$(采用对半球均匀采样),所以:
$$
L_0(p,\omega_0)=\int_{\Omega+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)d\omega_i \\
\approx \frac{1}{N}\sum_{i=1}^N\frac{L_i(p,\omega_i)f_r(p,\omega_i,\omega_0)(n·\omega_i)}{pdf(\omega_i)}
$$
伪代码如下:

```c++
shade(p,w0)
    Randomly choose N directions wi_pdf
    L0=0.0
    for each wi
        Trace a ray r(p,wi)
        if ray r hit the light
            L0+=(1/N)*L_i*f_r*cosine/pdf(wi)
    return L0
```

#### (1)引入间接光照

在上述算法中加入对间接光照的描述:

![image-20220724222058194](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724222058194-165867245952525.png)

```c++
shade(p,w0)
    Randomly choose N directions wi_pdf
    L0=0.0
    for each wi
        Trace a ray r(p,wi)
        if ray r hit the light
            L0+=(1/N)*L_i*f_r*cosine/pdf(wi)
        else if ray r hit an object at q //引入间接光照
            L0+=(1/N)*shade(q,-wi)*f_r*cosine/pdf(wi)
    return L0
```

以上算法存在问题:

- (1)光线数量会发生爆炸,因为假设每次发射100根光线发射到物体上,又会每个产生100根光线,这就爆炸了.

  - 只有光线数N=1才不会爆炸,只采用一根光线,**这就叫做路径追踪,算法改进如下:**(N不等于1叫做分布式光线追踪)

  ```c++
  shade(p,w0)
      Randomly choose ONE directions wi_pdf
      L0=0.0
      for each wi
          Trace a ray r(p,wi)
          if ray r hit the light
              L0+=L_i*f_r*cosine/pdf(wi)
          else if ray r hit an object at q //引入间接光照
              L0+=shade(q,-wi)*f_r*cosine/pdf(wi)
      return L0
  ```

  - 只是这样还是会有很多噪声之类的,可以从一个像素打出若干个方向的光线,从而求path,再把结果求平均值,增加一个函数如下:

  ```c++
  ray_generation(camPos,pixel)
      Uniformly choose N sample positions within the pixel
      pixel_radiance=0.0
      for each sample in the pixel
          Shoot a ray r(camPos,cam_to_sample)
          if ray r hit the scene at p
              pixel_radiance+=1/N*shade(p,sample_to_cam)
      return pixel_radiance
  ```

  

- (2)这个递归算法没有结束条件.

  - 注意,我们不能仅仅简单的限制递归次数,这样的效果不好,**采用的方式是俄罗斯轮盘赌(RR)**

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724223351425-165867323257227.png" alt="image-20220724223351425" style="zoom:80%;" />

可以发现,最终的期望是一样的,这就是RR的妙处,因此代码修改如下:

```c++
shade(p,w0)
    //新增:RR
    Maually specify a probability P_RR
    Randomly select ksi in a uniform dist. in [0,1] //生成一个随机数
    if (ksi>P_RR) return 0.0
    Randomly choose ONE directions wi_pdf
    L0=0.0
    for each wi
        Trace a ray r(p,wi)
        if ray r hit the light
            L0+=L_i*f_r*cosine/pdf(wi)/P_RR
        else if ray r hit an object at q //引入间接光照
            L0+=shade(q,-wi)*f_r*cosine/pdf(wi)/P_RR
    return L0
```

以上就是正确的路径追踪算法了,但不够高效,原因如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724225014512-165867421561029.png" alt="image-20220724225014512" style="zoom:80%;" />

如果想让采样都不浪费,可以直接在光源上采样,但是有一步推导:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220724225156648-165867431765231.png" alt="image-20220724225156648" style="zoom:80%;" />

采样定义在光源上,但渲染方程是对dw进行积分,所以要进行转换,把渲染方程改为与x相关.有如下关系:
$$
dw=\frac{dAcos \theta'}{||x'-x||^2}
$$
这时就可以改写刚才的积分公式:
$$
L_0(x,\omega_0)=\int_{\Omega+}L_i(x,\omega_i)f_r(x,\omega_i,\omega_0)cos\theta d\omega_i \\
=\int_A L_i(x,\omega_i)f_r(x,\omega_i,\omega_o)\frac{dAcos\theta cos \theta'}{||x'-x||^2}dA
$$
此时pdf=1/A,并且上述算法可以进行改进,如果是来自于光源的光线我们直接对光源进行采样(不需使用RR),否则再用原来的方式,并且我们需要考虑当前点与光源的连线之间是否存在遮挡物:

```python
shade(p,w0)
#contribution from the light source.
L_dir=0.0
Uniformly sample the light at x1 (pdf_light=1/A)
Shoot a ray from p to x1
if the ray is not blocked in the middle
	L_dir = L_i*f_r*cosθ*cosθ1/|x1-p|^2/pdf_light

#Contribution from other reflectors.
L_indir=0.0
Test Russian Roulette with probability P_RR

Uniformly sample the hemosphere toward wi(pdf_hemi=1/2pi)
Trace a ray r(p,wi)
if ray r hit a non-emitting object at q
	L_indir=shade(q,-wi)*f_r*cosθ/pdf_hemi/P_RR

return L_dir+L_indir
```



### 4.几点补充说明:

- (1)点光源比较麻烦,不做过多讨论,这里建议把点光源替换为小面积的面光源;
- (2)其他的一些工业界使用的方法:
  - (Unidirectional & bidirectional) path tracing
  - photon mapping
  - metrophlis light transport
  - VCM/UPBP

后面会进行补充,还有一些本课程没有涉及的问题以后的学习中也会进行补充的.



## lecture 17 Materials and appearances

研究光照与不同的材质之间的作用方式.在渲染方程当中,BRDF决定了材质的表现方式.
### 1.漫反射材质
在这里我们认为入射的irradiance和出射的radiance的总量是相等的(也就是能量守恒,没有吸收多余的能量),同时近似认为来自各个方向的irrandiace是平均的,并且反射出去到各个方向的radiance也是平均的.那么就可以认为反射光强和入射光强相等,即对渲染方程来说,$L_o(\omega_o)=L_i(\omega_i)$,可以推导:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814171639660.png" alt="image-20220814171639660" style="zoom:80%;" />

从以上公式中可以看出漫反射的BRDF的计算方式是$\large f_r=\frac{ρ}{\pi}$,其中ρ又被叫做albedo,可以是RGB颜色等.



### 2.镜面反射材质
复习一下镜面反射的方程:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814172531306.png" alt="image-20220814172531306" style="zoom:67%;" />

右侧的这幅图中,定义Φ是方向角,定义θ为任何方向和法线方向的夹角(称为仰角),所以任何角度可以用θ和Φ来表示(类似于之前的方向角说明)

注:镜面反射材质的BRDF函数比较复杂,有时间再进行学习补充.



### 3.折射材质

复习:Snell's law

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814173456308.png" alt="image-20220814173456308" style="zoom:67%;" />

从上面的式子可以看出,如果入射介质的折射率大于折射介质的折射率,有可能会出现没有折射的现象(全反射),比如说snell's window现象.所以在做光线追踪的时候要注意带来的全反射的问题。
> snell's window现象的一个示例是如果我们从水底往各个方向看的话,只能看到一个锥形的发光区域,这就是有全反射的发生,示意图如下:
>
> ![image-20220814174123377](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814174123377.png)

对折射来说,不应该叫做BRDF,而应该叫做BTDF.BTDF公式也比较复杂,暂时不做追述,可以统一将BRDF和BTDF统称为BSDF.



#### (1)菲涅尔项 Fresnel Reflection / Term

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814174602816.png" alt="image-20220814174602816" style="zoom:67%;" />

可以看到,由于光路可逆,如果我们把观察方向发出的光当成是入射光,那么有多少光被反射有多少光被折射是与入射光角度有关的.**这就是菲涅尔项.**下图是关于绝缘体和导体的相关性质:

![image-20220814175638845](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814175638845.png)

直观来看,对于绝缘体来说,随着入射方向与法线夹角的不断增加,越来越多的反射会发生,而对应折射的比例则会变小.(图里的S与P是与光的极化有关的量,这里不需要过多考虑),这里举个生活中的例子就是如果坐在车的后排,观察车窗可以看到窗外的景色,这里更多地发生了折射;而如果观察前排的车窗则会发现看到的更多是车里的景象,这就是因为观察方向夹角与法线夹角过大,反射占据的比例比较高.

对于导体来说,对各个观察方向,其反射率都是比较高的.值得一提的是导体的折射率是负数,这里就不做展开了.



#### (2)如何计算菲涅尔项?

如果想要非常精确地计算上图中曲线的方程,则公式会非常复杂,带来较高的计算量.一个近似的方式是拟合曲线的方程,这就是Schlick's approximation:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814180327422.png" alt="image-20220814180327422" style="zoom:67%;" />



### 4. 微表面材质(基于物理)

这里假设了两个表面,分别是MacroScale和MicroScale,定义如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814210757472.png" alt="image-20220814210757472" style="zoom:67%;" />

比如远处看的话看到的是材质和外观,而近处看到的则是几何.微表面模型建立了材质和几何之间的关系.

可以看到,如果微表面的法线方向分布在一个比较集中的区域,那么可以用来表示一种glossy的材质;否则如果微表面法线方向包含的范围比较大的话,就更靠近漫反射的材质.所以,**材质的粗糙程度就可以通过微表面的法线分布情况来体现.**注意,在实践过程中我们都认为微表面是"镜子".

#### (1)微表面模型的BRDF

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220814211649950.png" alt="image-20220814211649950" style="zoom: 67%;" />

首先,要考虑菲涅尔项F(i,h),因为不同的入射光方向会决定不同的反射程度.同时,也要考虑模型表面的法线分布D(h).

> 注意法线分布的D(h)又一次用到了半程向量,可以根据上图来更好地理解.由于我们认为微表面是镜子,那么只有当半程向量h与法线方向完全重合地时候,才能做到入射方向反射到正确的反射方向.所以,对D(h)进行查询就可以找到有多少微表面的法线是沿着正确的方向去的,这就是D(h)项.

同时需要注意,由于微表面互相之间可能会存在一定的遮挡关系,因此需要一个修正项G(称为shadow masking)来进行修正.直观感受越斜的光(学术上叫glazing angle),造成这种自遮挡的现象就越明显.

总的公式如下(f(i,o)就是指的微表面的BRDF):
$$
f(i,o)=\frac{F(i,h)G(i,o,h)D(h)}{4(n,i)(n,o)}
$$


微表面模型是现代PBR材质的基础.



### 5.Isotrophic/Anisotrophic Materials

各向同性/各向异性材质.(这里可以顺便复习一下各向异性过滤,思想是类似的.)

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829203738767.png" alt="image-20220829203738767" style="zoom:67%;" />

各项异性:不满足在方向角上旋转之后,BRDF不变.

![image-20220829205353334](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829205353334.png)



### 6.BRDF的性质

(1)非负性
$$
f_r(\omega_i->\omega_r)\ge 0
$$
(2)线性性质,也就是多个BRDF块可以叠加在一起,比如漫反射+镜面反射+折射:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829205812884.png" alt="image-20220829205812884" style="zoom:80%;" />

(3)BRDF具有可逆性.

交换入射方向与出射方向,得到的BRDF结果不变;

(4)能量守恒,反映出的式子如下:
$$
\forall \omega_r \int_{H^2}f_r(\omega_i->\omega_r)cos\theta_id\omega_i \le 1
$$
(5)对于各向同性的材质,有如下的性质:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829210322179.png" alt="image-20220829210322179" style="zoom:80%;" />

上述的性质给我们的启发在于测量BRDF的时候,四维的参数可以在一定程度上降为三维,并且由于对称性还可以砍掉一半的数据计算量,如果再利用一些trick的话还可以进一步减少计算量.



### 7.BRDF的测量

一个相关的数据库:MERL BRDF Database



## lecture 18 更高级的渲染话题

### 一.关于路径追踪

这一节更多是一些稍微进阶一些的话题,后面会深入进行介绍,这里先采用导图的方式进行概述:

#### 1.无偏/有偏的概念

![image-20220829223225588](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223225588.png)

#### 2.无偏的光线传播——BDPT

![image-20220829223320644](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223320644.png)

#### 3.无偏的光线传播——MLT

![image-20220829223402246](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223402246.png)

#### 4.有偏的光线传播——光子映射

![image-20220829223539673](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223539673.png)

#### 5.有偏的光线传播——VCM

![image-20220829223614901](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223614901.png)

#### 6.实时辐射度方法

![image-20220829223708418](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220829223708418.png)



### 二.Advanced Apperence Modeling

推荐数学公式编辑网站:

[Welcome To Mathcha](https://www.mathcha.io/editor)

#### 1.非表面模型

- 散射介质Participating media:比如云,雾等(其实类似巧克力这种也算是散射材质)
- 毛发(BCSDF)

##### (1)散射介质Participating media

![image-20220903101355640](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101355640.png)

可以看到,此时光在传播过程中出现了吸收和散射现象,那么如何散射呢?

定义一个函数Phase Function,这个函数会决定散射的性质是如何的,比如下图(类比BRDF决定如何反射,Phase Function决定如何散射):

![image-20220903101419852](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101419852.png)

##### 如何渲染?

![image-20220903101455215](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101455215.png)

每一段可以"走多远"是由介质的吸收能力所决定的,而下一次发射的方向则是由刚才的Phase Function来决定.这里由别的方程来实现(不同于渲染方程),具体的后面再学吧.



##### (2)毛发渲染

此时要考虑到的是光和毛发丝线的作用,而不能仅仅考虑表面相关.

###### Kajiya-Kay Model

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101521451.png" alt="image-20220903101521451" style="zoom:67%;" />

下方的圆柱就是模拟的头发,右侧的圆锥是模拟出来的散射出的圆锥,同时也会有一些光线会散射到四面八方去.



###### Marschner Model

(1)人的头发渲染

考虑到光线打到头发的时候,一部分会反射掉,但还有一部分会穿过头发,发生类似于折射的现象,定义反射为R,穿透为T,则会产生多种情况的光线,比如R,TT(两次穿透),TRT(穿透进入头发内部,发生一次反射,再穿透出去),如下图:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101556214.png" alt="image-20220903101556214" style="zoom:80%;" />

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903101610950.png" alt="image-20220903101610950" style="zoom:67%;" />

对于非常多的头发而言,光会经过多次的散射过程,所以对于头发来说渲染的计算量很大,比较困难.



(2)动物的毛发渲染

如果直接把人的毛发渲染作用到动物身上,会得到错误的结果.这里就要用生物学上的构造来进行解释了:

![image-20220901104054705](E:/%E7%B1%B3%E5%93%88%E6%B8%B8/%E5%B7%A5%E4%BD%9C%E6%80%BB%E7%BB%93/games/Games101%20%E5%85%B6%E4%BD%99%E5%92%8C%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9.assets/image-20220901104054705.png)

其中cuticle是外表皮,medulla是生物学当中的髓质,其中光线进入髓质当中会发生类似散射的现象,反射到四面八方.人和动物的区别在于,动物毛发内髓质区域非常大,因此如果不考虑的话就会得到错误的结果.因此,在Marschner Model的基础上,又有人提出了双层模型Double Cylinder Model,用于精准的表现出髓质的特点:

![image-20220901104456418](E:/%E7%B1%B3%E5%93%88%E6%B8%B8/%E5%B7%A5%E4%BD%9C%E6%80%BB%E7%BB%93/games/Games101%20%E5%85%B6%E4%BD%99%E5%92%8C%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9.assets/image-20220901104456418.png)

这里考虑到髓质可能会对光路造成影响,因此额外增加了TTs和TRTs的分量,此时就可以用上图左侧的五个分量来表述所有的结果.



##### (3)Granular Material 颗粒材质

用来描述这种一粒一粒的模型,比如沙子和盐等,这种材质的计算量是相当大的.



#### 2.表面模型

有一种类似于玉石(Jade)之类的材质,叫做Translucent Material,光从某个点传播进入材质之后,可以从另一个方向打出来.其他比如水母也是类似的材质.**将这种反射现象称为次表面散射Subsurface Scattering**.次表面散射可以理解为是对BRDF概念的一种延伸,BRDF是作用于一个点的,而BSSRDF则是考虑到光可能从与入射点不同的地方反射出去:

![image-20220901105754139](E:/%E7%B1%B3%E5%93%88%E6%B8%B8/%E5%B7%A5%E4%BD%9C%E6%80%BB%E7%BB%93/games/Games101%20%E5%85%B6%E4%BD%99%E5%92%8C%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9.assets/image-20220901105754139.png)

相比BRDF而言,BSSRDF不仅要考虑当前点,还要考虑附近的其他点会不会对光的传播有影响,所以积分的话既要对方向积分又要对面积进行积分.

##### (1)次表面散射近似-Dipole Approximation

这里的近似是类比一种现象,比如我们打开手机的手电筒盖在手掌上,观察到的次表面散射现象就好像是手掌中有一个光源,因此可以用这种机制来近似表示次表面散射,如下图(为了考虑物理上的真实性,需要有一个外界的virtual source):

![image-20220901110623601](E:/%E7%B1%B3%E5%93%88%E6%B8%B8/%E5%B7%A5%E4%BD%9C%E6%80%BB%E7%BB%93/games/Games101%20%E5%85%B6%E4%BD%99%E5%92%8C%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9.assets/image-20220901110623601.png)

可以考虑做一个BSSRDF的材质编辑器(虽然现在应该已经有了)



##### (2)布料cloth

布料是一系列纤维缠绕而成的,具有不同的层级.纤维fibers缠绕到一起形成股Ply,股再缠绕到一起形成Yarn.接下来在编织到一起.

![image-20220901111340178](E:/%E7%B1%B3%E5%93%88%E6%B8%B8/%E5%B7%A5%E4%BD%9C%E6%80%BB%E7%BB%93/games/Games101%20%E5%85%B6%E4%BD%99%E5%92%8C%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9.assets/image-20220901111340178.png)

这个时候渲染就要考虑编织方式是什么样的,通过不同的编织图案来求解,有三种方案:

- (1)采用BRDF模型.但这样是有问题的,因为比如天鹅绒这种表面是由很多细小的"绒毛"组成,此时直接按照表面来计算就不太合理.
- (2)可以把织物认为是空间中分布的体积,可以将其分为细小的格子,进而可以按照介质的方式来计算.
- (3)当然还有一个比较暴力的做法,既然布料是由纤维编织而成的,可以把所有的纤维单独渲染出来

由于织物渲染的复杂性,直到现在以上的三种方法在应用角度上都有被应用.



##### (3)表面细节

有的时候我们做的表面过于完美了,反而与实际不符,真实的表面应该有一些如划痕之类的细节,这是需要考虑的.同时这种微型的细节甚至还要考虑光的衍射现象等,具体的原理在这里就不总结了,可以看原视频或者后续的补充.



#### 3.程序化生成的外观 Procedural Appearance

常见地使用一些noise纹理来实现,比如模拟一些木头材质等.道理在于随用随取,需要的话来采样即可.比如perlin noise,常用于生成地形,水面等.专业做程序化的软件其中之一是Houdini,后面一定要学一下.



## lecture 19 相机的相关参数

对于图像的生成而言,有合成图像和捕捉图像,也就是:

Imaging = Synthesis + Capture

###  1.相机的各种基本知识点

- 原理:小孔成像(对应针孔相机,针孔相机不会拍出来虚像,都是实的),透镜
- 组件:快门,传感器(记录irradiance),
- 补充:在做光线追踪的时候,我们使用的就是针孔相机的相关模型,所以不会有景深效果;
- 影响相机拍照的因素:光圈大小(aperture size,由F-Stop控制),快门,ISO增益(感光度,相当于后期处理,具体实现有多种方式)

![image-20220903105954025](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903105954025.png)



#### (1)FOV

对应相机的名词,广角相机意味着更大的FOV(下图暂时不考虑透镜):

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903103847505.png" alt="image-20220903103847505" style="zoom:80%;" />

通常在市面上的相机,定义35mm格式的胶片为基准,此时就直接定义焦距来确定FOV,焦距越短视野范围角越大.



#### (2)曝光

$$
H=T×E
$$

T:time,E:irradiance,T是由快门来决定的.E则是由光圈大小(lens aperture)和焦距所决定的.

#### (3)ISO

- ISO是直接的线性作用效果,比如ISO 200只需要ISO 100一半的光量就可以实现效果.

#### (4)光圈F-Number/F-Stop

写法:FN或者F/N,N就叫f-number,

**非正式理解**:如果光圈直径是d,那么f number(N) = 1/d

**正式理解:焦距/光圈的直径 N=f/d**

通常相机有的光圈:1.4,2,2.8,4.0等



#### (5)快门

- 快门速度比较慢会造成运动模糊,不过运动模糊也不全是坏事
- 快门带来的问题:由于快门打开是有一个过程的,所以如果物体运动的速度比快门打开的速度还要快的话,就会出现扭曲的现象(rolling shutter比如下图当中的螺旋桨,可能会出现图像中的不同位置记录的是不同时间点采样的结果):

![image-20220903120054485](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903120054485.png)

- F-Stop和Shutter Speed

![image-20220903120433076](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903120433076.png)

以上表格的数据产生的曝光度是等同的,不过要注意修改F-Stop还会带来景深的相关变化.



### 2.相机镜头近似

目前相机基本都使用透镜组来进行成像,并且还会有各式各样的透镜,在这里我们考虑理想化的薄的透镜,也就是平行光打进来聚焦到焦点.并且我们认为这个透镜可以修改焦距(现实生活中用透镜组来实现).

理想化透镜有如下特点:

- 平行光穿透透镜,会打到焦点上
- 穿过透镜中心的点,光路方向不会发生改变.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903121514698.png" alt="image-20220903121514698" style="zoom: 67%;" />

z0是物距,zi是像距,f是焦距,所以如果焦距不变的话改变物距,像距就会发生改变.公式推导过程如下:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903122243302.png" alt="image-20220903122243302" style="zoom:67%;" />

将两个式子的左侧都简化为h0/hi的式子,然后联立两个方程,即可得到上面的透镜方程.



#### (1)景深原理

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903122711862.png" alt="image-20220903122711862" style="zoom: 80%;" />

如上图所示,本来如果物体放在focal plane,会正好成像到sensor plane上,此时拍摄的结果是清晰的;但如果物体不在Focal Plane上,则根据上面的透镜方程,会成像到非sensor plane上,如上图成像到Image平面上,此时光并不会停止,而是会继续发射到传感器上,所以会造成"模糊"现象.此时打在传感器上的成像区域就叫做COC:

- Circle of Confusion(COC)

$$
\large \frac{C}{A}=\frac{d'}{z_i}=\frac{|z_s-z_i|}{z_i}
$$

A指的是透镜的尺寸,C指的就是COC.将上式进行转化,可得:
$$
C=A\frac{|z_s-z_i|}{z_i} =\frac{f}{N}\frac{|z_s-z_i|}{z_i},其中N是f-number,N=f/A
$$


#### (2) 光线追踪,棱镜模型设计方案和原理

![image-20220903173933465](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903173933465.png)

在设计的时候可以:

- 确定sensor的大小
- 透镜的属性,包括焦距f和光圈的大小,不需要考虑相机位置这种
- 物体离透镜的距离Z0

根据以上的信息,可以得出zi的值,也就是像距,这个时候的ray tracing计算如下:

- 对于sensor上的点x',在薄的透镜上选择一个点x'',此时在subject plane上对应的点x'''就一定是x'与透镜中心连线与subject plane的交点(见图中比较细的那条虚线),此时只需要记录x'''->x''的光照结果,并记录到x'上即可.



#### (3)景深的概念

大小光圈影响到的是模糊的范围,景深就是认为COC的值足够小的区域:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903175136880.png" alt="image-20220903175136880" style="zoom:67%;" />



### 3.光场(Light Field/Lumigraph)

#### (1)基本思想

> 如果从人眼出发,将看到的内容全部"存"在一张幕布上,并放置在人眼前合适的位置,就可以起到让人觉得跟真实观察到的一样的效果,类似于碟中谍4中的情节,或者是虚拟现实的原理.

- 全光函数(plenoptic function)
  - 概念:是人的视觉能看到的所有东西.

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903193653244.png" alt="image-20220903193653244" style="zoom:80%;" />

θ,Φ指的是对应的角度(球面的极坐标形式),λ是波长(对应光的各种颜色),t是时间,VX,VY,VZ则是不同的视角看到的内容.以上七个变量可以定义我们观察到的世界.

- 光场的概念

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903194134429.png" alt="image-20220903194134429" style="zoom:50%;" />

一个物体被观察到的所有情况,是从任意方向的光打到物体上的点的集合.由光路的可逆性,我们用光场记录从物体所有点发出的,向所有方向传播的光.**光场是全光函数的一部分,记录方向和位置**(一共记录四维,分别是物体包围盒(图中那个立方体)表面的两个参数表示位置,用θ.Φ表示方向).此时观察物体的时候就可以根据光路方向,找到光场中存储的值,直接获得结果:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903194758279.png" alt="image-20220903194758279" style="zoom:50%;" />

进一步地,其实可以用两个平面上的点进行采样,因为两点就可以确定一条直线,所以采样两个平面上的点(s,t)和(u,v)进行组合,就可以代表各个方向的光,如下图:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903195227020.png" alt="image-20220903195227020" style="zoom:67%;" />



#### (2)光场相机

这里图比较多,有需求的话看一下Games101对应Lecture 20的前半部分即可,也可以参考下面的链接:

[Mars说光场（1）— 光场技术综述 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/47492390)

[(13 条消息) 光场相机是如何实现的？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/20511442/answer/26251790)



## lecture 20 颜色和感知

### 1.基本概念

- (1)光谱

- (2)SPD谱功率密度

  - 用来描述光在不同波长上分布的量的多少,如下图:
  - <img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903201935753.png" alt="image-20220903201935753" style="zoom:67%;" />
  - SPD的性质:线性性质,不同光的SPD具有可叠加性

- (3)颜色感知原理——回顾人的眼睛结构，视杆细胞（感知强度）和视锥细胞（感知颜色，数量相对较少，内部分为三种视锥细胞S,M,L）

  <img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903202941960.png" alt="image-20220903202941960" style="zoom: 67%;" />

  所以,其实人眼感知到的就是S,M,L计算出的结果,这些由光的分量波长和对应的响应函数纵坐标的值相乘积分得到.

- (4)同色异谱(Metamerism)

  - 根据上面的公式,可能存在一种情况就是波长不相同,但是计算出来的感知结果相同,这就是同色异谱.
  - metamers:负责进行同色异谱的调节
  - 启发:可以用不同的颜色混合,来实现某种颜色

- (5)加色系统:R,G,B

- (6)有的时候不同颜色混合成某个颜色的时候,配方系数不一定全是正的(实际上是待匹配颜色加上某种颜色,相当于配方出的颜色减去某种颜色,如下图:)

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903203852064.png" alt="image-20220903203852064" style="zoom:80%;" />



### 2.颜色空间

#### (1)sRGB

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903204252205.png" alt="image-20220903204252205" style="zoom:80%;" />

其中,右侧的图对应单波长光谱在不同波长下对应的R,G,B分量,左侧则是给定混合光谱之后,要对混合光里的不同波长光进行累积求解,这就是积分的思想.

- RGB系统能表示出的色域范围是有限的



#### (2)CIE XYZ系统

是一套人造的颜色匹配系统.Y分量可以一定程度上表示亮度,将XYZ分量分别归一化,可以在二维空间中表示:

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903204839660.png" alt="image-20220903204839660" style="zoom:80%;" />

用上面的表示方式之后,x+y+z = 1,所以其实只用记录x,y就可以相应求出z.在修改值的时候可以固定Y(也就是固定亮度),修改X和Z的值,不过在显示的时候显示的是x,y的对应值,右图中**颜色空间所能显示的所有颜色就叫做色域**.有如下结论:

- 白色是最不纯的颜色,对应(1/3,1/3),纯色都在图的边界上
- 不同颜色空间色域范围不同,其中XYZ模型范围要广于sRGB模型.



#### (3)减色系统CMYK

CMY三种颜色可以调出各种颜色,K的作用是为了使用角度,因为印刷上大部分的内容用黑色打印即可.所以直接把K作为模型的一部分.



#### (4)其他颜色空间

- HSV: 在生活中的应用是各种颜色拾取器.由色调,饱和度,亮度组成.
- CIELAB 空间(AKA`L*a*b*` ):
  - <img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220903205928306.png" alt="image-20220903205928306" style="zoom:80%;" />
  - 认为一对轴两端都是互补色,互补色本身是基于人脑的一个概念

本节未提及的内容: HDR,以及gamma校正.



## lecture 21 22 动画/模拟

### 1.关键帧动画

关键帧本身就是插值技术的一种体现,并且希望可以有东西来控制插值的效果,这就是样条的重要性.

![image-20220904115706807](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220904115706807.png)



### 2.物理模拟

复习牛顿定律:$F=ma$

物理模拟的基本思想在于正确建模,将实际情景通过方程体现出来.



#### (1)Mass Spring System(质点弹簧系统)

典型例子:可以用多段弹簧来模拟绳子.

- 基本单元:一个弹簧左右连接着两个质点

<img src="E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220904121319075.png" alt="image-20220904121319075" style="zoom:80%;" />

其中Rs叫做弹性系数,$\frac{b-a}{||b-a||}$是将方向进行归一化,l是弹簧松弛的长度.

上面这个弹簧系统带来的问题是:其会永远地震荡下去,因为我们没有考虑类似于摩擦力的项.

> 补充:在物理仿真中,我们通常用$\dot{x}$表示x的一阶导数,用$\ddot{x}$表示x的二阶导数

如果x是我们感兴趣的位置,那么$\dot{x} = v,\ddot{x}=a$

为刚才的模型增加能量损失的部分,如下($\dot{b}$表示b的速度方向):

![image-20220904125924997](E:/prepareForWorkFinal/Conputer_Graphics/Games101%20%E7%AC%94%E8%AE%B0.assets/image-20220904125924997.png)

应用以上模型还会带来问题,这个模型会让物体停下来.比如一根弹簧拴着两个质点向下掉落,该模型会使得物体效果不正确.**该模型无法表示弹簧内部的能量损耗**.



剩下的内容直接看PPT吧,就不整理了.
