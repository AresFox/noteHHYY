## 毕设 捏人 bs+动作



### 1.捏脸系统

所谓的捏脸系统，就是可以通过UI对导入的人物的BS进行调整，同时要在执行动画的时候保留人物脸部的BS参数（**这点倒是不需要太过在意，因为骨骼动画并不会改变脸型的相关BS值**）。因此大概的思路就是通过slider来控制BS的值（注意，Unity的BS范围是0-100），并通过代码逻辑来控制。这里可以写一个简易的脚本如下：

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class yFaceBSTest : MonoBehaviour
{
    public Slider[] bsSliders; //绑定Canvas下面的控制BS的Slider
    public GameObject digitalHumanShape;  //绑定导入角色的Shape结点，因为这个结点下有BS可以控制
    SkinnedMeshRenderer skinnedMeshRenderer;
    void Awake()
    {
        skinnedMeshRenderer = digitalHumanShape.gameObject.GetComponent<SkinnedMeshRenderer>();
    }
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 0; i < bsSliders.Length; i++)
        {
            int index = i; //需要声明一个新的变量=i,否则可能会有调用顺序问题,这里不表
            bsSliders[index].value = 0.5f;
            bsSliders[index].onValueChanged.AddListener(delegate { OnBSSliderValueChanged(index); } );
        }
    }

    private void OnBSSliderValueChanged(int index)
    {
        //-100->100
        //0->1
        //回调函数:根据Slider控制的值调整BS的大小
        skinnedMeshRenderer.SetBlendShapeWeight(index, bsSliders[index].value *200f-100f); //因为Unity里BS的范围是0-100
    }
    // Update is called once per frame
    void Update()
    {
        
    }
}

```



Unity里BS的范围是-100->100
slider 0-1

新建结点并绑定对应的脚本，并为其赋值如下:

![image-20230310185351229](毕设 捏人 bs+动作.assets/image-20230310185351229.png)

此时就可以通过Slider来调整人物脸上的BS，在后面的AI模块驱动中也有调试BS相关的功能，到时也会进行总结。



捏脸系统不仅可以适用于用户自定义，我们也利用这个系统去进行AI的语音驱动模块，实现了输入语音，输出对应嘴型。



smpl 也可以实现捏人+动作



还有捏身体



捏脸 还需要是如果闭眼，眉毛要一起闭





### 2.动画系统

#### （1）骨骼系统

DAZ的骨骼系统和传统的软件和引擎中的骨骼系统有所不同，因此并不能让DAZ的骨骼无缝衔接Unity的骨骼系统。

DAZ的骨骼和Unity的骨骼系统分别如下：

![image-20230310201501704](毕设 捏人 bs+动作.assets/image-20230310201501704.png)

可以看到，DAZ的骨骼结构比Unity的骨骼结构要复杂很多，在骨骼重定向的时候会出现一定问题，这个问题具体地我们会在[存在的问题和接下来可以提升的点](#（5）可以提升的点和接下来的问题)当中进行介绍，并给出可行的解决方案（具体的方案暂时还没有落实）。

正因为DAZ和Unity二者的骨骼系统之间的差异，因此对于动画系统来说有两种可行的解决方案：

- （1）全部使用DAZ的动画，此时动画资产和角色模型资产的Rig全部设置为Generic，并且动画和人物的骨骼需要保持一致。（从DAZ直接导出的动画是符合这一要求的）；
- （2）使用其他常见动画素材库中的骨骼动画资源（如上文提到的Mixamo），此时Mixamo下载的动画资产和DAZ导入的人物和动画资产全部都要适配于Unity的骨骼系统（也就是把Rig调整为Humanoid），也就是需要进行重定向的操作。

**在接下来的篇幅中，我们会对两种方式进行逐一说明，但在此之前需要对动画资产进行预处理，同时设定对应的状态机。**



#### （2）动画预处理&状态机说明

##### （a）修改序列帧动画

在导入动画的章节中我们提到了一种可能的问题，即DAZ的动画在第一帧人物会变为T字型，此时动画就会有一个突变效果，十分影响观感。同时，有些动画资产会出现穿模的问题，**此时都可以通过对动画资产进行精修的方式来适当地解决该问题**，具体介绍如下：

首先，通过Ctrl+D快捷键将动画资产中的序列帧动画复制出来，并双击打开：

![image-20230310202949087](毕设 捏人 bs+动作.assets/image-20230310202949087.png)

比如说如果原来的动画第一帧会让角色出现T字型，则可以把动画序列帧的第一帧删掉，就可以让动画恢复正常了。

**关于关键帧的位置和数值修改，更好的方式是在Blender或Maya当中进行，但基本原理是一样的（这里只是方便起见在Unity里面进行微调）**。



##### （b）修改Rig

在前文有提及过：

- 如果要全部使用DAZ的系统，则Rig需要修改为Generic，并采用状态机来更改动画；
- 如果要用Unity的骨骼系统并重定向，则Rig需要修改为Humanoid，最好也用状态机来实现修改动画。

下面我们以Humanoid类型为例，来说明Rig的修改是如何进行的：（Generic情况下不需要重定向骨骼，比较简单，就不整理了）

（i）首先，修改人物，动画等资产的Rig为Humanoid：

<img src="毕设 捏人 bs+动作.assets/image-20230310203808042.png" alt="image-20230310203808042" style="zoom:80%;" />

此时正常来说是不会报错的，如果报错的话可能原因有骨骼命名重复，根节点不符合规范等原因。



（ii）点击Configure，就可以对角色的骨骼进行重定向了：

![image-20230310204001440](毕设 捏人 bs+动作.assets/image-20230310204001440.png)

如上图，需要把DAZ人物（图是原神迪卢克的图，但是原理是一样的）的骨骼重定向到Unity的骨骼，也就是箭头指向的右侧区域。一般来说这一个步骤是可以自动匹配的，但是有的时候需要进行手动调整（比如说Unity默认绑定的骨骼不正确的情况）。同时，动画资产和Mixamo上下载的动画资产也都需要重定向到Unity的Humanoid标准骨骼上。此时不难想到，**由于重定向的存在，人物和动画都适配了同一套骨骼**，因此骨骼动画也是可以正常完成的。



##### （c）状态机说明

这一部分如果对Unity引擎比较熟悉的话不需要过多介绍了，大概就是通过Unity自带的状态机来转换动画（**其实也可以用比较复杂一些的有限状态机来做，但这里不过多展开**）。状态机和代码如下：

<img src="毕设 捏人 bs+动作.assets/image-20230310204618309.png" alt="image-20230310204618309" style="zoom:80%;" />

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class changeAnim : MonoBehaviour
{
    private Animator player;
    private void Start()
    {
        player = gameObject.GetComponent<Animator>();
    }
    void OnGUI()
    {
        if (GUI.Button(new Rect(1500, 100, 100, 30), "Apos"))
        {
            ChangeAnim(2);
        }

        if (GUI.Button(new Rect(1500, 140, 100, 30), "defaultIdle"))
        {
            ChangeAnim(0);
        }
    }
    void ChangeAnim(int num)  //核心是这个，非常常规
    {
        player.SetInteger("animNum", num);
    }
}


```





#### （3）DAZ动画适配（推荐）

**这里比较推荐这种DAZ动画直接使用的方法，原因在（5）部分会进行详细描述**。直接把DAZ的动画资产的Rig修改为Generic，然后Ctrl+D复制出来并在状态机中赋值即可，**该状态机可以复用给任何满足DAZ Genesis 8骨骼结构的数字人。**

**可能存在的问题**

- 面部表情出现错乱，比如眼球不动或者是位置不对，解决方案有：
  - 删除序列帧动画中不相关的骨骼（不推荐，容易出问题）；
  - 在DAZ动画导出的时候记得勾选“Transfer Face Bones“；



#### （4）Mixamo动画适配

将角色和所有的动画资产的Rig全部更换为Humanoid之后，同样绑定在状态机上即可（**原理在[修改Rig](#（b）修改Rig)的部分已经说过了**）。



#### （5）存在的问题和接下来可以提升的点

**关于DAZ和Unity的骨骼究竟哪里不适配**

> 如果我们仔细观察重定向之后的结果（如下图），就会发现如果重定向造成的问题：
>
> ![image-20230310205404630](毕设 捏人 bs+动作.assets/image-20230310205404630.png)
>
> 可以看到，DAZ的骨骼的四肢（上图是腿，胳膊也是一样）都比Unity自带的骨骼要多上一节，这个通过②和③所框起来的区域也可以很好地看出来。**因此在重定向的时候，无论怎么重定向都会有一根骨骼没有用上**。这样的话自然适配于Unity的骨骼系统的动画应用在DAZ的骨骼上就会显得很奇怪。**应当可行的解决方案有如下几种：**
>
> - （1）直接使用DAZ系统自己的动画资产（**目前是这么做的，效果还不错**）
>   - 优点：难度较低，同时动画的适配性也不错；
>   - 缺点：比较依赖于DAZ的动画资产，如果要自己做动画的话也基本离不开DAZ这个软件；
> - （2）重新为角色制作一套更合适的骨骼，并且适配于Unity的骨骼系统：
>   - 优点：一劳永逸，并且并不依赖DAZ,因为支持Unity标准Avatar格式的动画资产相对会更多一些；
>   - 缺点：**重新制作骨骼需要技术栈的学习，有一定学习和制作成本**；
> - （3）直接凹某个动画资产的关键帧数值。这种方法基本上只能饮鸩止渴，因此并不推荐。





记录现能用的动画：

采用daz系统：

idle

![image-20230323201228782](毕设 捏人 bs+动作.assets/image-20230323201228782.png)

Apos？

![image-20230323201248901](毕设 捏人 bs+动作.assets/image-20230323201248901.png)

dance

![image-20230323203149431](毕设 捏人 bs+动作.assets/image-20230323203149431.png)



状态机

![image-20230323201517453](毕设 捏人 bs+动作.assets/image-20230323201517453.png)



使用角色

![image-20230323201833668](毕设 捏人 bs+动作.assets/image-20230323201833668.png)





要写一下bs的原理 以及 你动作的原理 lbs



Linear Blending Skinning (LBS) 指的是通过操纵骨骼来使得 mesh 发生形变，LBS 定义骨骼点（joint）和 mesh 的 vertices 之间的关系是线性关系，一个 vertice 的位置会受到其他 joint 共同的加权影响，具体操作是会先计算 joint 的位置，然后根据 joint 的位置，求其他 vertices 的位置，所以可以通过操纵骨骼来改变整个 mesh。



人体bs

![image-20230323214416162](毕设 捏人 bs+动作.assets/image-20230323214416162.png)





变形有两种方式

一种是bs

一种是动骨骼

人脸的大都采用bs

身体部位既可以是bs，也可以是骨骼。



![image-20230323225012053](毕设 捏人 bs+动作.assets/image-20230323225012053.png)

若想获取所有并放到屏幕上



### 3. 语音驱动动作

介绍文章：

https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650863535&idx=5&sn=b2f760af137dc922a7669d4a1598686f&chksm=84e533d1b392bac72c93605b1577401b5a4a4e9b690351a5b10800dc75a9137088c3de97e168



BEAT 采用了 ViCon，16 个摄像头的动作捕捉系统来记录演讲和对话数据，最终所有数据以 120FPS**, 记载关节点旋转角的表示形式的 bvh 文件发布**。对于面部数据，BEAT 采用 Iphone12Pro 录制谈话人的 52 维面部 blendsshape 权重，并不包括每个人的头部模型，推荐使用 Iphone 的中性脸做可视化。BEAT 采用 16KHZ 音频数据，并通过语音识别算法生成文本伪标签，并依此生成具有时间标注的 TextGrid 数据。



Paper (ECCV 2022): [https://www.ecva.net/papers/eccv_2022...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbHRMNXo3SHB5X3ptRE10cEpFZWRZYmVjSjBFd3xBQ3Jtc0trNzBUMVh5bFppM0c5X0M3Q1VhUnc1dnE4bElFazhCTi1QRFlhQnNtUHUtczFPaTZKUFlpeGtrbXlsQmhxS1BENF9VVk1lbFpLbUVXTWdsdFdpNEdXclo4OFZ5WUV0LTBORVJhcjFRa091bTlOdEVOcw&q=https%3A%2F%2Fwww.ecva.net%2Fpapers%2Feccv_2022%2Fpapers_ECCV%2Fpapers%2F136670605.pdf&v=F6nXVTUY0KQ) Project Page: [https://pantomatrix.github.io/BEAT/](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbnZ4cm4yTGppZUZEWGVWNHB3RFdLaGNJVzZXd3xBQ3Jtc0tuR1N5SFI5TEpkT2VWQ011TGc0elRoSUlTbDVpbkdSR1FnTHhUQnZDZ2h4UVotWl9kWXR1RDQzQnp6WjZKZXIzYzNNdEhmSERPMEZGUmh0a0RQWTFRVVYxVjVERVl3VkxLbzFfZ0tJb3VhZi1fbmJnZw&q=https%3A%2F%2Fpantomatrix.github.io%2FBEAT%2F&v=F6nXVTUY0KQ)



http://lo-th.github.io/olympe/BVH_player.html



##### 连接实验室电脑

软件：xftp xshell

创建环境BEAT

![image-20230325222657278](毕设 捏人 bs+动作.assets/image-20230325222657278.png)

github：

https://github.com/PantoMatrix/BEAT

根据说明建立文件夹

![image-20230325222323223](毕设 捏人 bs+动作.assets/image-20230325222323223.png)



##### 具体步骤

参考步骤：

1. colab的 https://colab.research.google.com/drive/1bB3LqAzceNTW2urXeMpOTPoJYTRoKlzB?usp=sharing#scrollTo=hReOJTyJy2sb
2. 官方github的

![image-20230325223400523](毕设 捏人 bs+动作.assets/image-20230325223400523.png)



----



1. download the scripts to `codes/audio2pose/`

2. run `pip install -r requirements.txt` in the path `./codes/audio2pose/`

   https://colab.research.google.com/drive/1bB3LqAzceNTW2urXeMpOTPoJYTRoKlzB?usp=sharing#scrollTo=ONixJiMgwyTH

3. 

改为清华镜像

![image-20230325222908551](毕设 捏人 bs+动作.assets/image-20230325222908551.png)

pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

再下依赖包

pip install -r requirements.txt

pip install configargparse

4.   接下来下载dataset（我们这里是参考colab代码只下了部分 下载完之后拖进服务器文件中，解压：
 cd datasets/beat_cache/ && unzip beat_4english_15_141.zip

5. 执行test.py

   此处我们遇到了问题，

   1）pip install protobuf==3.20.*

   2）文件没找到 ![image-20230325223716659](毕设 捏人 bs+动作.assets/image-20230325223716659.png)

   所以我们把datasets放在了别的他应该在的地方

   再次执行

   ![image-20230325223752540](毕设 捏人 bs+动作.assets/image-20230325223752540.png)

   python test.py -c ./configs/camn_beat_4english_15_141.yaml --new_cache False

6. 结果：成功跑出代码结果

   ![image-20230325223828422](毕设 捏人 bs+动作.assets/image-20230325223828422.png)

![image-20230325223845047](毕设 捏人 bs+动作.assets/image-20230325223845047.png)

![image-20230325223855287](毕设 捏人 bs+动作.assets/image-20230325223855287.png)

![image-20230325223909176](毕设 捏人 bs+动作.assets/image-20230325223909176.png)

![image-20230325223918238](毕设 捏人 bs+动作.assets/image-20230325223918238.png)

![image-20230325223928025](毕设 捏人 bs+动作.assets/image-20230325223928025.png)

![image-20230325223936767](毕设 捏人 bs+动作.assets/image-20230325223936767.png)

![image-20230325223944623](毕设 捏人 bs+动作.assets/image-20230325223944623.png)

。。。。。。。

![image-20230325224005878](毕设 捏人 bs+动作.assets/image-20230325224005878.png)



##### 效果

代码：跑成功

![image-20230325224110775](毕设 捏人 bs+动作.assets/image-20230325224110775.png)

结果：

![image-20230325224216361](毕设 捏人 bs+动作.assets/image-20230325224216361.png)

![image-20230325224159481](毕设 捏人 bs+动作.assets/image-20230325224159481.png)



然后我们放进一些在线bvh解释器中

http://lo-th.github.io/olympe/BVH_player.html

![BEAT语音驱动动作](毕设 捏人 bs+动作.assets/BEAT语音驱动动作.gif)

bvh效果很不错



使用在线的bvh转maya

![image-20230325224654321](毕设 捏人 bs+动作.assets/image-20230325224654321.png)

但是解析的不是很好，会颤抖，但是大致动作都是有的

unity要求角色都有跟骨骼hip，所以导入unity前 需要将maya中预处理

因此需要在maya中去做  我是将左右腿放到spine中，这样不太好其实，最好加一个hip骨骼

（ctrl+ctrl+p）变为父子关系



unity中也是 有点点问题







E:\本科\vcg实验室\毕设相关\bvh\9999

E:\本科\vcg实验室\毕设相关\bvh\9999\res_2_scott_0_1_1.bvh





E:\本科\vcg实验室\毕设相关\bvh\res_2_scott_0_1_1.bvh

![image-20230326112452454](毕设 捏人 bs+动作.assets/image-20230326112452454.png)







Assets\BVHParser\Resources\Bonemaps2.txt

Assets\BVHParser\Resources\bvh\res_2_scott_0_1_1.bvh

![image-20230326120324203](毕设 捏人 bs+动作.assets/image-20230326120324203.png)

![image-20230326120332843](毕设 捏人 bs+动作.assets/image-20230326120332843.png)







转换器

手可能会颤抖

<img src="毕设 捏人 bs+动作.assets/image-20230326134637875.png" alt="image-20230326134637875" style="zoom:50%;" />





https://miconv.com/convert-bvh-to-fbx/

也会颤抖 感觉跟上面是一个算法

<img src="毕设 捏人 bs+动作.assets/image-20230326134755393.png" alt="image-20230326134755393" style="zoom:50%;" />







问题1:

自动张嘴

![image-20230326150821914](毕设 捏人 bs+动作.assets/image-20230326150821914.png)

是因为lowerjaw张开了



问题2：手指有问题

对从BEAT转译的骨骼动画，进行重定向

![image-20230326151110990](毕设 捏人 bs+动作.assets/image-20230326151110990.png)

可以看到这里的很多地方一看就是有问题的





问题3：会颤抖





参考：

https://blog.csdn.net/zb1165048017/article/details/112394097



bvh左边

![image-20230326154602423](毕设 捏人 bs+动作.assets/image-20230326154602423.png)



使用blender3.3.1  导入bvh

然后导出fbx

导入unity即可

！！！！！震惊了



![image-20230326222806049](毕设 捏人 bs+动作.assets/image-20230326222806049.png)

![image-20230326222820955](毕设 捏人 bs+动作.assets/image-20230326222820955.png)

https://zh.wikipedia.org/wiki/%E6%97%8B%E8%BD%AC%E7%9F%A9%E9%98%B5

![image-20230326222841829](毕设 捏人 bs+动作.assets/image-20230326222841829.png)







暂时

下巴会一直开着

原因是下巴的骨骼绑错了

正确的是belowjaw，而不是lowerjaw

![image-20230327115842704](毕设 捏人 bs+动作.assets/image-20230327115842704.png)



BEAT官方数据集

https://drive.google.com/drive/folders/1CVyJOp3G_A9l1N_CsKdHgXQfB4pXhG8c

bvh时间：两者相乘

![image-20230327122004819](毕设 捏人 bs+动作.assets/image-20230327122004819.png)

然后我们放进一些在线bvh解释器中

http://lo-th.github.io/olympe/BVH_player.html





记录一下原本的对应

![image-20230327164533453](毕设 捏人 bs+动作.assets/image-20230327164533453.png)



那个语音的 不能太长 不然无法解析

https://vocalremover.org/zh/cutter

可以用上面这个剪辑





#### 一些操作



目前动作系统需要使用generic

通过rig改



语音驱动动作需要使用humannoid

通过rig改

改完要重定向下巴![image-20230328212508432](毕设 捏人 bs+动作.assets/image-20230328212508432.png)



现在测试用例是245可用
