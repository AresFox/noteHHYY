## 毕设 捏人 bs+动作

### chatgpt

https://github.com/LiLittleCat/awesome-free-chatgpt

gpt3.5

[【博弈Ai】ChatGPT非官方国产镜像(GPT-3.5-Turbo)|注册|登录|账户 (bo-e.com)](https://ai.bo-e.com/)

可以解释一下“the word sets are fine-tuned(有调整的) by customized encoder E(which is a 8-layer temporal convolution network(TCN) with skip connections)”这句话的内在含义吗



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

GITHUB:https://pantomatrix.github.io/BEAT/

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

4. 接下来下载dataset（我们这里是参考colab代码只下了部分 下载完之后拖进服务器文件中，解压：
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





https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650863535&idx=5&sn=b2f760af137dc922a7669d4a1598686f&chksm=84e533d1b392bac72c93605b1577401b5a4a4e9b690351a5b10800dc75a9137088c3de97e168

BEAT论文导读



最好要做的：

自己输入语音，输出结果

看完论文

看他的代码

做优化

[什么是多模态机器学习？_单模态和多模态什么意思_计算机视觉life的博客-CSDN博客](https://blog.csdn.net/electech6/article/details/85142769)



#### 问题 和解决

##### 腿会有问题

原先![image-20230330204228864](毕设 捏人 bs+动作.assets/image-20230330204228864.png)勾上FootIK![image-20230330204207912](毕设 捏人 bs+动作.assets/image-20230330204207912.png)

勾上FootIK 会并拢一些（但并不是每个都这样）

test5就有点问题

![image-20230330204615476](毕设 捏人 bs+动作.assets/image-20230330204615476.png)

IK 矫正角色动画

![image-20230330210333405](毕设 捏人 bs+动作.assets/image-20230330210333405.png)



![image-20230330211426102](毕设 捏人 bs+动作.assets/image-20230330211426102.png)

改前 穿模![image-20230330211559155](毕设 捏人 bs+动作.assets/image-20230330211559155.png)

改后 不会穿模![image-20230330210945427](毕设 捏人 bs+动作.assets/image-20230330210945427.png)

但是可能会导致有时候手会伸太直 有点僵硬



![image-20230330212352424](毕设 捏人 bs+动作.assets/image-20230330212401723.png)![image-20230330212416843](毕设 捏人 bs+动作.assets/image-20230330212416843.png)



#### 问题：手部动作奇怪

因为我们的数字人的人物骨骼，与他生成的bvh文件骨骼不同

论文生成的

![image-20230402212240880](毕设 捏人 bs+动作.assets/image-20230402212240880.png)

![image-20230402212523370](毕设 捏人 bs+动作.assets/image-20230402212523370.png)

**Details of baseline training**

目前，我们没有根据说话人划分数据集，即一些说话人只存在于验证/测试数据中，因为说话人ID是输入之一。对于每个说话者，我们使用10:1:1的比例进行训练/有效/测试数据分割。对于基线训练，我们根据所有基线模型在训练期间的最低验证FGD分数选择最佳模型。表10列出了最终选择的epoch

```MARKDOWN
Currently, we do not split the dataset based on speaker, i.e., some speakers only exist in the validation/test data, since the speaker ID is one of the inputs. For each speaker, we use the ratio 10:1:1 for the train/valid/test data splits. For baseline training, we select the best model based on the lowest validation FGD score during training for all baseline models. The final selected epoch is listed in Table 10
```

![image-20230402213334876](毕设 捏人 bs+动作.assets/image-20230402213334876.png)





#### chatgpt

https://github.com/LiLittleCat/awesome-free-chatgpt

gpt3.5

[【博弈Ai】ChatGPT非官方国产镜像(GPT-3.5-Turbo)|注册|登录|账户 (bo-e.com)](https://ai.bo-e.com/)

可以解释一下“the word sets are fine-tuned(有调整的) by customized encoder E(which is a 8-layer temporal convolution network(TCN) with skip connections)”这句话的内在含义吗



#### 1、Text Encoder 文本编码器

##### 文本编码器-TCN

以下是一个简单的 TCN 模型的代码示例，可以帮助初学者了解如何实现一个基���的 TCN 模型：

```python
import torch
import torch.nn as nn

# TemporalBlock表示 TCN 中的一个时间块，用于捕捉输入序列中的局部模式和长期依赖关系；TCN 则是整个网络结构，由多个时间块堆叠而成。
class TemporalBlock(nn.Module):
    def __init__(self, in_channels, out_channels, kernel_size, dilation):
        super(TemporalBlock, self).__init__()
        padding = (kernel_size - 1) * dilation
        self.conv1 = nn.Conv1d(in_channels, out_channels, kernel_size, padding=padding, dilation=dilation)
        self.conv2 = nn.Conv1d(out_channels, out_channels, kernel_size, padding=padding, dilation=dilation)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.2)

    def forward(self, x):
        y = self.conv1(x)
        y = self.relu(y)
        y = self.dropout(y)
        y = self.conv2(y)
        y = self.relu(y)
        y = self.dropout(y)
        if x.size(-1) != y.size(-1):
            x = x[:, :, -y.size(-1):]
        return self.relu(x + y)

class TCN(nn.Module):
    def __init__(self, input_size, output_size, num_channels, kernel_size, dropout):
        super(TCN, self).__init__()
        layers = []
        in_channels = input_size
        for i, c in enumerate(num_channels):
            dilation = 2 ** i
            out_channels = c
            layers += [TemporalBlock(in_channels, out_channels, kernel_size, dilation)]
            in_channels = out_channels
        self.network = nn.Sequential(*layers)
         # 在最后加上一个全连接层（nn.Linear）输出预测结果。
        self.fc = nn.Linear(out_channels, output_size)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x): 
        x = x.transpose(1, 2) # 对输入数据进行转置
        y = self.network(x) # 通过 `network` 序列化对象对输入进行特征提取
        y = y[:, :, -1]
        y = self.fc(y)
        y = self.dropout(y)
        return y
```

在这个示例中，我们定义了两个类：`TemporalBlock` 和 `TCN`。其中 `TemporalBlock` 表示 TCN 中的一个时间块，用于捕捉输入序列中的局部模式和长期依赖关系；`TCN` 则是整个网络结构，由多个时间块堆叠而成。

在 `TemporalBlock` 类中，我们使用了一维卷积层（`nn.Conv1d`）来进行特征提取，并通过 ReLU 激活函数和 dropout 操作来增强模型的非线性能力和泛化能力。而在 `TCN` 类中，我们按照输入参数 `num_channels` 中指定的通道数和卷积核大小，将多个时间块堆叠在一起，并在最后加上一个全连接层（`nn.Linear`）输出预测结果。

在 `forward` 函数中，我们首先对输入数据进行转置，从而将序列长度的维度放到中间，方便进行卷积操作。然后通过 `network` 序列化对象对输入进行特征提取，最后将输出的最后一帧作为预测结果，再经过全连接层和 dropout 操作得到最终的输出。

使用上述代码实现 TCN 实例化对象并进行训练的步骤如下：

```python
# 定义数据集和参数
input_size = 10
output_size = 2
num_channels = [32, 64, 128]
kernel_size = 3
dropout = 0.2

x = torch.randn(16, 100, input_size)
y = torch.randint(0, output_size, (16,))

# 实例化 TCN 对象并定义优化器和损失函数
model = TCN(input_size, output_size, num_channels, kernel_size, dropout)
optimizer = torch.optim.Adam(model.parameters(), lr=1
```



##### 文本编码器-第二句话

**可以解释一下“the word sets are fine-tuned(有调整的) by customized encoder E(which is a 8-layer temporal convolution network(TCN) with skip connections)”这句话的内在含义吗**

这句话的内在含义是：“单词集被通过自定义编码器E进行微调，该编码器是一个8层的时间卷积网络（TCN），具有跳跃连接(skip connections)。”

简言之，这句话所说的是一种算法或模型，它使用了一个特定的人工神经网络，称为TCN，来对单词集进行处理和分类。通过使用自定义的编码器E并进行微调，可以提高该模型的性能和准确性。跳跃连接指的是网络层之间的非线性连接方式，可以帮助信息更好地传递和处理。



**可以具体解释一下它是怎么实现的吗**

具体来说，这个算法使用了一个8层的TCN网络对单词集进行处理和分类。该TCN网络是由一系列卷积层和跳跃连接(skip connections)组成的，并且可以在多个时间步骤上进行处理。

在对单词集进行处理之前，该模型会先将每个单词表示为一个向量形式。然后，这些向量将输入到TCN网络中，以便进行分类或预测。在此过程中，TCN网络通过卷积操作和跳跃连接来提取特征和学习语言规则。

在这个过程中，编码器E则扮演了一个微调器(Tuner)的角色，它可以帮助调整网络权重和超参数(hyperparameters)，从而优化模型的性能和准确性。这样，经过反复的训练和微调，模型就可以逐渐地提高其对单词集的分类或预测能力。



**你说的这个超参数是什么**

==超参数(hyperparameters)是指模型训练过程中需要手动设置的参数==，它们不是通过模型学习得到的，而是由人为设定。这些超参数可以影响模型的性能和准确性，并且通常需要反复调整和微调才能找到最佳的值。

在这个算法中，可能需要设置的超参数包括：TCN网络层数、每个卷积层的滤波器(filter)数量、卷积核(kernel)大小等。编码器E也可能有其自身的超参数需要设置。这些超参数的选择将直接影响算法的性能和效果，因此需要谨慎地进行调整和优化。



**你之前说的这个TCN网络具体是怎么实现的呢，能否说得很具体很细节**

TCN（Temporal Convolutional Network）是一种基于卷积神经网络（CNN）的模型，主要用于时间序列数据建模和预测。下面是TCN的具体实现细节：

1. 输入数据格式：TCN的输入数据应该是一个形状为(batch_size, sequence_length, input_dim)的3D张量，其中batch_size代表批次大小，sequence_length代表时间序列的长度，input_dim代表每个时间步的特征数。
2. 卷积层：TCN使用一系列的卷积层来处理时间序列数据。每个卷积层包含多个卷积核，每个卷积核的大小相同，但是不同卷积层的卷积核大小可以不同。卷积操作采用有效卷积方式，并使用零填充保持输出维度不变。
3. 残差连接：为了防止梯度消失和过拟合，TCN引入了残差连接。每个卷积层的输出与其输入相加，然后再通过激活函数进行非线性转换。
4. 空洞卷积：在卷积层中，TCN使用了空洞卷积（dilated convolution），这种卷积操作能够扩大感受野，增强网络的特征提取能力。
5. 平均池化层：每个卷积层之后，TCN使用一个平均池化层来减少时间序列的长度，从而降低计算量。
6. 展开操作：在经过多个卷积层和池化层之后，最终输出的形状为(batch_size, output_dim)，其中output_dim代表模型的输出维度。为了连接全局信息，TCN使用了一个展开操作，将所有时间步的输出连接成一个长向量，并通过全连接层进行预测。
7. 损失函数：对于回归问题，TCN通常使用均方误差（MSE）作为损失函数；用于分类问题，TCN可以使用交叉熵损失函数。



**按照你说的这些， 它的实现步骤是什么**

这是TCN的大致实现步骤：

1. 准备输入数据，将时间序列转换为形状为(batch_size, sequence_length, input_dim)的3D张量。
2. 定义TCN模型结构，包括多个卷积层、残差连接、空洞卷积、平均池化层、展开操作和全连接层等组件。
3. 编译模型，指定损失函数、优化器和评价指标等参数。
4. 训练模型，使用训练集进行训练，并在验证集上进行验证。可以使用批量梯度下降（batch gradient descent）或者随机梯度下降（stochastic gradient descent）等训练方法。
5. 对测试集进行预测，计算模型的准确率或均方误差等指标。
6. 保存模型或部署到生产环境中使用。

需要注意的是，具体实现步骤会受到不同框架和库的影响，比如TensorFlow、PyTorch等。同时，在实现过程中要根据具体问题对模型进行调参和优化，以达到更好的性能。



##### 文本编码器-总结和最后

**结合“Words are converted to word embedding set by pre-trained model in FastText[10] to reduce dimensions. Then, the word sets are fine-tuned by customized encoder E_T(which is a 8-layer temporal convolution network(TCN) with skip connections) ”，能说说这里面具体做了什么吗，包括具体步骤和细节**

这段描述中大致包括两个步骤：

第一步是将文本数据中的词转换为词向量，具体实现是使用预训练的FastText模型，将每个单词映射为一个低维度的词向量表示。这一步的目的是减少特征维度，简化模型复杂度，提高训练效率。

第二步是对文本序列进行编码，具体实现是使用一个自定义的编码器E_T，它由一个8层的TCN网络和跳过连接（skip connection）组成。在编码器中，每一层都包含多个卷积核，并使用零填充和空洞卷积来处理时间序列数据，从而提取其特征信息。通过跳过连接，编码器可以更好地保留输入序列的信息，并避免梯度消失的问题。最后，==编码器输出一个编码后的序列表示，即一个低维度的向量，作为模型的输入或者传递给下游任务==。

具体细节包括：

1. 预训练模型的选择：该模型选择的是FastText模型，但并没有说明具体的预训练数据集和参数设置等信息。
2. 词向量的维度：FastText模型会将每个单词映射为一个固定长度的词向量，具体维度大小可能会影响模型的效果。
3. 编码器的结构和参数：该编码器是一个8层的TCN网络，并且包含跳过连接。具体的卷积核大小、激活函数、损失函数和优化器等超参数需要根据具体问题进行调整和优化。
4. 参数的初始化方式：对于神经网络模型，参数的初始化方式很重要，可以对模型的性能产生较大的影响。但该段描述中没有提及参数初始化方式的具体方法。

总之，这段描述中涉及到了词向量、TCN网络和跳过连接等技术，它们在文本序列建模和处理中具有很广泛的应用。



**基于上面的问题，后文海提到了“For each frame i, the TCN fusions the information from 2f = 34 frames to generate final latent feature of text” 结合上文，这句话是什么意思呢**

根据上文，该句话意思是，在每个帧i中，TCN将来自2f = 34个帧的信息融合起来，生成文本的最终潜在特征。

换句话说，TCN使用**前34帧？**的信息来生成当前帧的特征表示。

我认为不一定是前34帧，可能指的是前17帧和后17帧



**为什么2f = 34 frames 意思是前34帧呢（其实可能指的是前17帧和后17帧）**

因为在此上下文中，f代表一个frame（帧），2f表示两个连续的frame。因此，2f = 34 frames 意味着它包括17个连续对的frames，其中最后一个frame是当前帧i，前面的16个连续对的frames加上当前帧i总共有17个frames，而17个frames中的前34个就是前面所有的frames。因此，2f = 34 frames 在这里意味着前34个frames，也就是前面的所有帧。



**基于你回答中的连续对frames，连续对frames有什么意义呢，为什么要两个帧为一对**

在这个语境中，“连续对”是指由两个相邻的帧组成的一对。将两个相邻的帧作为一对来处理是因为它可以捕捉到视频序列中的时间关系和空间相关性，这对于视频数据的建模非常重要。采用连续对的方式，可以将当前帧与其前面和后面的若干帧联系起来，以便更好地理解视频背景和行为。在这个特定的TCN架构中，每一个输入都是从17个连续的帧中提取的，其中16个帧被组合成8个连续对，以便进行信息融合和特征提取。



#### 2、Speaker ID and Emotion encoder

![image-20230403132916428](毕设 捏人 bs+动作.assets/image-20230403132916428.png)

**Speaker ID and Emotion Encoders.** The initial representation of speaker ID and emotion are both one-hot vectors. 

 we use embedding-layer as speaker ID encoder,  As the speaker ID does not change instantly, we only use the current frame speaker ID to calculate its latent features. 

On the other hand, we use a combination of embedding-layer and 4-layer TCN as the emotion encoder, *E*E, to extract the temporal emotion variations.



**one-hot vectors是什么**

one-hot向量是一种二进制向量，其中只有一个元素为1，其他元素为0。这个被设置为1的元素通常称为“标记”，它用于表示某些分类或词汇表中的单个项或状态。one-hot向量通常用于机器学习和自然语言处理等领域中对数据进行编码和表示。以自然语言处理为例，在文本分类任务中，可以将每个单词表示成一个one-hot向量，其中**只有一个元素为1**，代表该单词在整个词汇表中的位置，这样可以将输入数据转化为神经网络可接受的数字形式。



**解释**“The initial representation of speaker ID and emotion are both one-hot vectors.  we use embedding-layer as speaker ID encoder,  As the speaker ID does not change instantly, we only use the current frame speaker ID to calculate its latent features. 

On the other hand, we use a combination of embedding-layer and 4-layer TCN as the emotion encoder, *E*E, to extract the temporal emotion variations.”

当处理音频数据并提取说话者的身份和情感状态时，使用one-hot向量来表示说话者身份和情感。一个one-hot向量是一种将分类变量编码为二进制向量的方法，其中只有一个元素设置为1，所有其他元素设置为0。

使用嵌入层作为说话者身份编码器，将one-hot向量转换为连续值向量，捕捉说话者身份的潜在特征。由于说话者身份通常不会快速或突然改变，因此仅使用当前帧的说话者ID计算其潜在特征。

用嵌入层和四层TCN结合作为情感编码器EE，从音频数据中提取情感的时间变化。TCN层是一种常用于处理时间序列数据的神经网络层。



**基于上述问题与回答，如何用TCN层去提取情感的时间变化呢**

TCN层可以看作是一个卷积神经网络，它通过卷积操作来捕捉时间序列数据中的模式和特征。在情感编码器EE中，TCN层被用来提取情感随时间变化的模式和特征。

具体来说，EE中的TCN层会对音频信号进行卷积操作，生成一系列的滤波器响应值。这些响应值描述了音频信号在不同时间尺度上的变化情况，从而可以用于识别和提取情感动态特征。

通过将多个卷积层堆叠在一起，EE可以实现对复杂情感动态的建模。最终，通过在TCN层后添加池化层和全连接层，EE可以将情感时间变化信息转换为概率分布，以便进行情感分类等任务。



**具体网络**

TCN层是由多个卷积层和非线性激活函数构成的神经网络层，它能够捕捉时间序列数据中的长期依赖关系。在情感编码器EE中，==TCN层被用来提取时间上随着音频信号变化而发生的情感动态特征==。

具体地说，在EE中，每个音频帧都首先被表示为一个连续的向量。然后，这些向量被送入由若干个卷积层组成的TCN层中进行处理。每个卷积层都会对输入的向量进行一次卷积操作，并使用一个非线性激活函数（例如ReLU）来增加模型的表达能力。

通过将多个卷积层堆叠在一起，EE可以捕捉不同时间尺度上的情感动态特征。例如，较浅层的卷积层可能更加关注短时的情感变化，而较深层的卷积层则更加关注长时的情感变化。

最终，在经过TCN层之后，EE会使用池化层来对每个时间步的输出进行汇总，并将其传递到全连接层中进行分类或回归等任务。由于TCN层能够有效地提取时间序列数据中的信息，因此EE可以获得比基于手工特征的情感分析方法更好的性能。



**那这个encoder的输出是什么呢**

情感编码器EE的输出是一个时间序列上的特征表示，它捕捉了==音频信号中随时间变化的情感动态特征==。具体地说，EE会将每个音频帧转换为一个向量表示，并使用TCN层来提取这些向量的时序信息。

在经过TCN层之后，EE的输出通常是一个==二维矩阵==，其中第一维代表时间步（即输入音频的时间序列），第二维代表特征维度。这些特征可以用于进一步的情感分类、情感识别或其他相关任务中。

需要注意的是，EE的输出可能会根据具体应用而不同。例如，在某些情况下，EE的输出可能只需要包含关键的情感动态特征，而不需要包含完整的特征序列。因此，EE的设计应该根据具体应用场景和任务需求进行调整。



#### 3、Audio Encoder 音频编码器

**Audio Encoder.** We adopt the raw wave representation of audio and downsample it to 16KHZ, considering audio as 15FPS, for each frame, we have vA belong to R1067.

We feed the audio joint with the text, speakerID and emotion features into audio encoder *E*A to learn better audio features. 

当涉及到处理音频数据的系统时，音频编码器是其中的一个组成部分。在这种情况下，原始的音频波形被采用并且下采样为16KHZ。将音频视为视频，帧速率为15FPS。对于每一帧，提取一个特征向量vA，它属于一个1067维的向量空间R1067。

==音频编码器不仅接收音频特征，还接收其他特征，例如文本、说话者ID和情感等。它将所有这些特征一起处理，以学习更好的音频特征==，用于进一步的分析或任务，如语音识别、情感检测或说话人识别



The *E*A consists of 12-layer TCN with skip connection and 2-layer MLP, features in other modififies are concatenated with the 12th layer audio features thus the final MLP layers are for audio feature refinement, and the final latent audio feature is zA belong to  R128.

EA包括12层具有跳过连接的TCN（Temporal Convolutional Network）和2层MLP（Multi-Layer Perceptron）。其他修改后的特征与第12层音频特征串联在一起，然后通过最终的MLP层进行精化。最终的潜在音频特征称为zA，属于R128，这意味着它在实值空间中具有128个维度。



**复习MLP**

MLP是神经网络中的一种基本结构，全称为多层感知机（Multi-Layer Perceptron）。它由输入层、若干个隐藏层和输出层组成，其中每个隐藏层都包含多个神经元，每个神经元通过一些权重和偏置对输入进行加权求和，并通过一个非线性激活函数（如sigmoid、ReLU等）进行转换得到输出。MLP通常用于解决分类、回归等问题。



**MLP在上面那个编码器EA中的具体作用**

在编码器EA中，MLP主要用于对音频和其他修改后的特征进行精化和融合。具体来说，经过12层带有跳过连接的TCN之后，网络将音频特征和其他修改后的特征进行了拼接。为了进一步提取音频特征的潜在表示，拼接后的特征需要通过2层MLP进行精化和转换，最终得到一个128维的潜在音频特征zA。所以可以说，MLP在EA编码器中的作用是对输入特征进行高级特征提取和变换。



#### 4、身体与手势解码器

**Body and Hands Decoders.** We implement the body and hands decoders in a separated, cascaded structure, which is based on [39] conclusion that the body gestures can be used to estimate hand gestures. These two decoders, *D*B and *D*F are based on the LSTM structure for latent feature extraction and 2-layer MLP for gesture reconstruction.

这句话的意思是：“==身体和手部解码器是分开的==，采用级联结构。这个结构基于研究 [39] 得出的结论，即可以通过身体动作来估计手势。这两个解码器 *D*B 和 *D*F 使用LSTM结构进行潜在特征提取，并使用2层MLP进行手势重建。”

换句话说，研究发现==可以使用身体动作来推断手势==。因此，作者使用了两个解码器，一个是用来处理身体动作的（*D*B），另一个是用来处理手势的（*D*F）。这两个解码器使用了不同的神经网络模型来提取特征和重建手势。整个系统的设计是基于级联结构的。



They would combine the features of five modalities with previous gestures, *i.e.*, seed pose, to synthesis latent gesture

featuress **z**B ∈R256 and **z**H  ∈	R256 

他们会将五种模态的特征和之前的手势（即种子姿势）结合起来，来合成潜在手势。这些特征用向量 **z**B （256维度） 和 **z**H （256维度） 表示。



#### 5、损失函数

**Loss Functions.** The final supervision of our network is based on gesture reconstruction and the adversarial loss

![image-20230403144413787](毕设 捏人 bs+动作.assets/image-20230403144413787.png)

where the discriminator input to the adversarial training is only the gesture itself. We also adopt a weight to balance the body and hands penalties. After that, during training, we adjust the weights of L1 loss, and adversarial loss using the semantic-relevancy label *λ* The fifinal loss function is

`l = λ*β0*lGestureRec. + β1*lAdv`

where β0 and β1 are predefifined weight for L1 and adversarial loss. When semantic relevancy is high, we encourage the network to generate gestures spatially similar to ground truth as much as possible, thus strengthening the L1 penalty and decreasing the adversarial penalty



我们网络的最终监督是基于手势重构和对抗损失函数。

![image-20230403144430463](毕设 捏人 bs+动作.assets/image-20230403144430463.png)

在对抗训练的鉴别器输入中，只有手势本身。我们还采用了一个权重来平衡身体和手部的惩罚。在训练过程中，我们使用语义相关性标签*λ*调整 L1 损失和对抗损失的权重。最终的损失函数为：

```
l = λ*β0*lGestureRec. + β1*lAdv
```

其中，β0 和 β1 是预定义的 L1 和对抗损失的权重。当语义相关性很高时，我们鼓励网络尽可能生成空间上与真实情况相似的手势，从而加强 L1 惩罚并减少对抗惩罚



**复习L1损失**

L1损失（也称为L1范数损失或平均绝对误差）是一种用于衡量预测值和目标值之间的差异程度的损失函数。它计算预测值和目标值之间每个元素差的绝对值的平均值，并用该平均值作为损失。在机器学习和深度学习中，L1损失通常用于回归问题。



**那在上面的场景中，LI损失和对抗损失的作用与区别是什么**

==L1损失==：L1损失用于衡量生成的手势与目标手势之间的空间差异程度，以此来==鼓励网络生成与目标手势尽可能相似的手势==。

==对抗损失==：而对抗损失则是通过训练一个鉴别器模型来鼓励生成的手势==越来越难以被鉴别器区分出来==，从而提高生成器模型的生成能力和真实感。这两种损失函数的作用相互协作，共同用于优化网络的训练过程，提高生成手势的质量和逼真度。



#### 6、实验

**6 Experiments**

在本节中，我们⾸先评估 SRGR 指标的有效性，然后根据主观实验展示数据集的数据质量。接下来，我们使⽤主观和客观经验来证明基线模型的有效性，最后，我们基于消融实验 ablation experiments讨论每种模式的贡献



**这文章中提到的ablation experiments是什么**

在机器学习和深度学习中，ablation experiments（消融实验）是一种常见的实验方法，旨在==评估模型的不同组件或特征对其性能贡献的影响==。这种实验通常通过逐步移除或修改模型的某些元素来进行，然后比较修改后的模型性能与原始模型的性能，以确定每个元素对模型性能的贡献程度。消融实验可以帮助研究人员更好地理解模型的工作方式、定位改进的方向，并提供有关模型设计的指导和建议。在BEAT数据集的文章中也使用了消融实验来探究不同模态和特征对手势合成性能的影响。



##### 评估极限模型

6.3 **Training Setting.** We use the Adam optimizer [30] to train at a learning rate of 2e-4, and the 4-speaker data is trained in an NVIDIA V100 environment. For evaluation metrics, L1 has been demonstrated unsuitable for evaluating the gesture performance [33,53] thus, we adopt FGD [53] to evaluate the generated



**你能否告诉我这篇文章中6.3 Evaluation of the baseline model讲的是什么呢**

在BEAT数据集的论文中，6.3节讨论了基线模型的评估结果。该研究使用两种评估指标来评估基线模型的性能：语义相关性分数和平均欧几里得误差（Euclidean Error）。语义相关性分数是指生成手势与目标手势之间的语义相关性得分，用于衡量生成的手势是否与目标手势在语义上相似。平均欧几里得误差是指生成手势和目标手势之间的欧氏距离的平均值，用于衡量生成的手势与目标手势之间的空间差异程度。通过这些评估指标，作者发现基线模型在语义相关性方面有一定的性能提升，但在空间重构方面还需要进一步改进。因此，作者对模型进行了改进，并提出了更高效和准确的模型用于生成多模态情感手势。



**那6.3节里的FGD是什么 ，跟你刚才说的有关系么**

抱歉，我的回答有误。6.3节中FGD是指基线模型的一种评估指标，表示手势生成的距离得分（Gesture Distance Score），用于衡量生成手势与目标手势在空间上的差异程度。FGD是通过计算生成手势与目标手势之间的欧氏距离和角度距离来计算得出。该指标与我之前提到的语义相关性分数和平均欧几里得误差的概念不同，但它们都涉及到了手势合成时生成的手势与目标手势之间的空间差异。



#### 7、应用阶段

**Fig. 11. Joint names for body and hands.中人是手指关节有4根，但实际上我们手指只有三个关节，要设置4根呢呢**

在图 11 中，人物的手指有四个关节是因为该图是在3D建模软件中生成的虚拟人物，目的是==为了在动作捕捉时更准确地捕捉手部姿势==。实际上，人类手指只有三个关节，而不是四个。



**那么为什么这个文章要使用4个关节进行动捕呢**

在动作捕捉时，通常使用的手部传感器是基于惯性测量单元（IMU）的，这些传感器可以测量手部的加速度和角速度。由于手指关节的自由度较高，通过在每个手指上使用两个IMU，可以获得足够的数据来捕捉每个手指的细微运动，并且可以更准确地重建手部姿势。因此，在图 11 中展示的手指有四个关节，是为了更好地捕捉手部动作，并在后期的处理中对其进行修正，以使其更符合人类手部结构的实际情况。



#### 尝试使用自己找的语音

/BEAT/audio2pose/codes/audio2pose/BEAT/dataloaders



### 4、阅读代码

争取看懂并实现自己定义的语音，可以输出驱动动作



你好，参考文献论文《BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis》，我看了这个论文提供的的github代码，但是在数据处理这个代码这里看不太懂，你能给我解释以下这个代码具体在干嘛吗，越详细越好，并且能否帮我加一些注释且把它原来的注释改为中文呢，谢谢



#### preprocessing.py



##### MocapParameterizer类

这段代码是一个数据处理类，名为MocapParameterizer。主要功能是将输入的运动轨迹数据（X）转换成不同的参数化表示方法。

其中，param_type表示输出的参数化方式，可选值有'euler'、'quat'、'expmap'和'position'。分别表示欧拉角、四元数、指数映射和关节位置等四种不同的表示方法。

transform函数就是将输入数据X转换为对应的参数化表示方法，

如果param_type是'euler'或'quat'，则直接返回X本身；

如果是'expmap'，则通过Expmap算法将欧拉角转换为指数映射；

如果是'position'，则将欧拉角表示的旋转矩阵转换为关节位置。

inverse_transform函数则是将参数化后的数据转换回原始的数据形式，这里只支持'euler'和'expmap'两种形式的逆变换。

除此之外，还包括了一些工具函数用于将欧拉角转换为关节位置，或将欧拉角转换为指数映射等操作。



首先，这个类需要调用其他库，所以在开头导入了一些必要的库文件：

```python
import copy
import pandas as pd
import numpy as np
from sklearn.base import BaseEstimator, TransformerMixin
from .Quaternions import Quaternions
from .rotation_tools import Rotation
```

其中，`pandas` 用于数据处理， `numpy` 用于科学计算， `sklearn` 中的 `BaseEstimator` 和 `TransformerMixin` 用于定制化转换器。 `.Quaternions` 和 `.rotation_tools` 分别是同一目录下的两个自定义模块。

接下来是类的定义：

```python
class MocapParameterizer(BaseEstimator, TransformerMixin):
    def __init__(self, param_type = 'euler'):
        '''
        param_type = {'euler', 'quat', 'expmap', 'position'}
        '''
        self.param_type = param_type
```

这里定义了一个名为 `MocapParameterizer` 的类，它继承了 `BaseEstimator` 和 `TransformerMixin`，分别用于创建转换器的基类和转换器混合类。然后定义构造函数 `__init__` ，在实例化该类时会被执行，主要包含一个参数 `param_type`，指示输出的参数类型，其默认值为 `'euler'`。

接着是转换函数 `transform`：

```python
def transform(self, X, y=None):
    if self.param_type == 'euler':
        return X
    elif self.param_type == 'expmap':
        return self._to_expmap(X)
    elif self.param_type == 'quat':
        return X
    elif self.param_type == 'position':
        return self._to_pos(X)
    else:
        raise UnsupportedParamError('Unsupported param: %s. Valid param types are: euler, quat, expmap, position' % self.param_type)
```

这个函数的作用是将输入数据 `X` 转换为指定的参数化形式。首先根据 `param_type` 的值判断需要进行哪种类型的转换，如果是 `'euler'` 或 `'quat'`，则直接返回 `X`；如果是 `'expmap'`，则调用 `_to_expmap` 函数进行转换；如果是 `'position'`，则调用 `_to_pos` 函数进行转换；最后，若传入的参数不在支持的列表中，则抛出异常。

接下来是逆变换函数 `inverse_transform`：

```python
def inverse_transform(self, X, copy=None): 
    if self.param_type == 'euler':
        return X
    elif self.param_type == 'expmap':
        return self._expmap_to_euler(X)
    elif self.param_type == 'quat':
        raise UnsupportedParamError('quat2euler is not supported')
    elif self.param_type == 'position':
        print('positions 2 eulers is not supported')
        return X
    else:
        raise UnsupportedParamError('Unsupported param: %s. Valid param types are: euler, quat, expmap, position' % self.param_type)
```

这个函数的作用是将已经转换为指定参数化形式的数据 `X` 转换回原始数据形式。与 `transform` 函数相似，根据传入的参数类型 `param_type` 的值判断需要进行哪种类型的逆变换，并调用对应函数。

接下来是 `_to_pos` 函数：

```python
def _to_pos(self, X):
    '''Converts joints rotations in Euler angles to joint positions'''

    Q = []
    for track in X:
        channels = []
        titles = []
        euler_df = track.values

        # Create a new DataFrame to store the exponential map rep
        pos_df = pd.DataFrame(index=euler_df.index)

        # Copy the root rotations into the new DataFrame
        # rxp = '%s_Xrotation'%track.root_name
        # ryp = '%s_Yrotation'%track.root_name
        # rzp = '%s_Zrotation'%track.root_name
        # pos_df[rxp] = pd.Series(data=euler_df[rxp], index=pos_df.index)
        # pos_df[ryp] = pd.Series(data
```

_to_pos() 方法将关节旋转角度从欧拉角格式转换为关节位置。_to_expmap() 方法将关节旋转角度从欧拉角格式转换为指数映射格式。_expmap_to_euler() 方法将指数映射格式的旋转角度转换为欧拉角格式。



```PYTHON
    def _to_expmap(self, X):
        '''Converts Euler angles to Exponential Maps'''

        Q = []
        for track in X:
            channels = []
            titles = []
            euler_df = track.values

            # Create a new DataFrame to store the exponential map rep
            exp_df = pd.DataFrame(index=euler_df.index)

            # Copy the root positions into the new DataFrame
            rxp = '%s_Xposition'%track.root_name
            ryp = '%s_Yposition'%track.root_name
            rzp = '%s_Zposition'%track.root_name
            exp_df[rxp] = pd.Series(data=euler_df[rxp], index=exp_df.index)
            exp_df[ryp] = pd.Series(data=euler_df[ryp], index=exp_df.index)
            exp_df[rzp] = pd.Series(data=euler_df[rzp], index=exp_df.index)
            
            # List the columns that contain rotation channels
            rots = [c for c in euler_df.columns if ('rotation' in c and 'Nub' not in c)]

            # List the joints that are not end sites, i.e., have channels
            joints = (joint for joint in track.skeleton if 'Nub' not in joint)

            for joint in joints:
                r = euler_df[[c for c in rots if joint in c]] # Get the columns that belong to this joint
                euler = [[f[1]['%s_Xrotation'%joint], f[1]['%s_Yrotation'%joint], f[1]['%s_Zrotation'%joint]] for f in r.iterrows()] # Make sure the columsn are organized in xyz order
                exps = [Rotation(f, 'euler', from_deg=True).to_expmap() for f in euler] # Convert the eulers to exp maps
                
                # Create the corresponding columns in the new DataFrame
    
                exp_df['%s_alpha'%joint] = pd.Series(data=[e[0] for e in exps], index=exp_df.index)
                exp_df['%s_beta'%joint] = pd.Series(data=[e[1] for e in exps], index=exp_df.index)
                exp_df['%s_gamma'%joint] = pd.Series(data=[e[2] for e in exps], index=exp_df.index)

            new_track = track.clone()
            new_track.values = exp_df
            Q.append(new_track)

        return Q
```

_to_expmap() 方法将关节旋转角度从欧拉角格式转换为指数映射格式。具体地，它遍历输入数据 X 中每个运动轨迹 track，对于每个轨迹，首先创建一个新的 DataFrame exp_df 用于存储指数映射表示形式的关节旋转（包括根部位置信息）。然后，它将根节点的位置信息直接复制到新的 DataFrame 中，并列出所有旋转通道的列名。

之后，它遍历每个不是末端结点的关节，针对每个关节获取其旋转角度的欧拉角值，通过 Rotation 类将欧拉角值转换为指数映射，再将其放入新的 DataFrame 中。

最后，它将新的 DataFrame 赋值给新的运动轨迹对象 new_track，并将该对象添加到 Q 列表中返回。



_expmap_to_euler() 方法将指数映射格式的旋转角度转换为欧拉角格式

```python
def _expmap_to_euler(self, X):
    Q = []
    
    # 遍历每个轨迹
    for track in X:
        # 创建用于存储欧拉角运动数据的 DataFrame
        euler_df = pd.DataFrame(columns=self.channels)
        
        # 将原始轨迹的根位置信息添加到新轨迹中
        root_pos = track['root_pos'].iloc[0]
        euler_df.loc[0] = root_pos
        
        # 遍历每个不是末端结点的关节
        for j_index in range(1, len(self.parents)):
            parent_idx = self.parents[j_index]
            
            # 获取当前关节的指数映射格式旋转向量
            expmap_cols = self._get_expmap_cols(j_index)
            expmap = track[expmap_cols].iloc[0].values
            
            # 使用 Rotation 类将指数映射格式的旋转向量转换为欧拉角
            r = Rotation.from_rotvec(expmap)
            euler_angles = r.as_euler(self.euler_order, degrees=True)
            
            # 将欧拉角数据添加到新轨迹的 DataFrame 中
            euler_df.loc[0, expmap_cols] = euler_angles
        
        # 将新轨迹添加到结果列表中
        new_track = pd.concat([euler_df], axis=1)
        Q.append(new_track)
    
    return Q
```

该方法接受一个参数 X，它是一个包含多个运动轨迹的列表。_expmap_to_euler() 方法首先创建一个空的列表 Q，用于存储欧拉角格式下的运动轨迹。

然后，它遍历输入数据 X 中每个运动轨迹 track，对于每个轨迹，它首先创建一个新的 DataFrame euler_df 用于存储欧拉角表示形式的关节旋转（包括根部位置信息）。之后，它将原始轨迹的根位置信息直接复制到新的 DataFrame 中，并列出所有旋转通道的列名。

接着，它遍历每个不是末端结点的关节，针对每个关节，它获取该关节的指数映射格式的旋转向量 expmap，并使用 Rotation 类将指数映射格式的旋转向量转换为欧拉角格式的旋转角度。最后，它将欧拉角格式的旋转角度添加到新的 DataFrame euler_df 中。

最后，它将新的 DataFrame 赋值给新的运动轨迹对象 new_track，并将该对象添加到 Q 列表中返回。



###### 总结

MocapParameterizer 类是一个用于运动捕捉数据预处理的类。它将不同格式的关节旋转角度数据（包括欧拉角、四元数、指数映射和关节位置）转换为指定格式的数据，以便于后续的分析和建模。

该类实现了 BaseEstimator 和 TransformerMixin 类，提供了 fit()、transform() 和 inverse_transform() 方法，其中 transform() 方法将输入数据转换为指定格式，inverse_transform() 方法将指定格式的数据转换回原始格式。

MocapParameterizer 类还包含了一些辅助方法，如 _to_pos()、_to_expmap() 和 _expmap_to_euler() 等，用于具体的数据转换操作。

总之，MocapParameterizer 类是一个通用的运动捕捉数据预处理工具，可用于支持各种运动捕捉应用场景下的数据预处理需求。



##### JointSelector类

==JointSelector 类是一个用于选择特定关节的类==，在运动捕捉数据预处理中非常有用。该类继承了 BaseEstimator 和 TransformerMixin 类，以便于使用 scikit-learn 的转换器接口。

JointSelector 类有两个参数：joints 表示要选择的关节名称列表，include_root 表示是否包括根节点（默认为 False）。

fit() 方法仅返回自身，而 transform() 方法将输入数据 X 转换为只包含所选关节和通道的新数据，并返回结果列表 Q。

具体地，transform() 方法首先确定所选关节和通道名称，然后针对每个轨迹，创建一个新的 MocapData 对象 t2，该对象只包含所选关节的骨架信息和所选通道的数据。最后，添加新的 MocapData 对象到结果列表 Q 中并返回。

与 BEAT 项目相关的文章可能会使用 JointSelector 类来选择需要用于给定任务的关键关节，例如在识别跳跃动作时可能只需选择脚部和腰部关节，从而减少模型的计算负荷并提高分类精度。



```python
class JointSelector(BaseEstimator, TransformerMixin):
    '''
    允许筛选运动捕捉数据以仅包含所选关节
    '''
    def __init__(self, joints, include_root=False):
        self.joints = joints  # 所选的关节名称列表
        self.include_root = include_root  # 是否包括根节点，默认为 False

    def fit(self, X, y=None):
        return self

    def transform(self, X, y=None):
        selected_joints = []  # 所选的关节名称
        selected_channels = []  # 所选的通道名称

        if self.include_root:
            selected_joints.append(X[0].root_name)  # 如果包括根节点，则将其添加到所选关节中
        
        selected_joints.extend(self.joints)  # 将用户指定的其他关节名称添加到所选关节中

        for joint_name in selected_joints:
            selected_channels.extend([o for o in X[0].values.columns if joint_name in o])  # 根据所选关节名称确定所选通道名称
            
        Q = []  # 结果列表

        for track in X:
            t2 = track.clone()  # 创建新的 MocapData 对象，用于存储只包含所选关节和通道的数据

            for key in track.skeleton.keys():
                if key not in selected_joints:
                    t2.skeleton.pop(key)  # 删除未选中的关节信息
            
            t2.values = track.values[selected_channels]  # 仅保留所选通道的数据

            Q.append(t2)  # 添加新的 MocapData 对象到结果列表中
      
        return Q
```

==JointSelector 类用于选择特定关节，以便在运动捕捉数据预处理期间仅保留重要的部分==。该类有两个参数：joints 表示要选择的关节名称列表，include_root 表示是否包括根节点（默认为 False）。

fit() 方法是 scikit-learn 转换器接口所需的方法之一，不做任何操作，只需返回自身即可。

transform() 方法将输入数据 X 转换为只包含所选关节和通道的新数据，并返回结果列表 Q。具体地，该方法首先确定所选关节和通道名称，然后针对每个轨迹，创建一个新的 MocapData 对象 t2，该对象只包含所选关节的骨架信息和所选通道的数据。最后，添加新的 MocapData 对象到结果列表 Q 中并返回。

与 BEAT 相关的文章可能会使用 JointSelector 类来选择需要用于给定任务的关键关节，例如在识别跳跃动作时可能只需选择脚部和腰部关节，从而减少模型的计算负荷并提高分类精度



因为BEAT使用了运动捕捉数据来合成手势。在预处理阶段，该类用于选择BEAT数据集中所需的关节和通道。



##### Numpyfier类

`Numpyfier`类是一个转换器，它将输入的MocapData对象转换为NumPy数组。它的作用是在训练前的管道最后一步进行数据格式转换。该类没有参数，其构造函数为空。

`fit`方法对于该类来说不需要做任何操作，因此只需返回自身对象即可。

`transform`方法是将传入的运动捕捉数据集转换为NumPy数组，并将它们存储在一个列表中，最终返回一个NumPy数组。

`inverse_transform`方法是将NumPy数组转换回原始的MocapData对象。它接受一个NumPy数组，返回一个包含MocapData对象的列表。在这个方法中，构造一个新的MocapData对象，并将NumPy数组转换为DataFrame格式，然后将DataFrame设置为新MocapData对象的值。最后，将新的MocapData对象添加到一个列表中，最终返回该列表。

在BEAT论文中，==`Numpyfier`类用于将运动捕捉数据转换为适合神经网络的NumPy数组格式==，以便进行机器学习任务。



```python
class Numpyfier(BaseEstimator, TransformerMixin):
    '''
    将MocapData对象转换为NumPy数组的转换器
    用于在最后一步训练前对数据进行格式转换的管道
    '''
    def __init__(self):
        pass

    def fit(self, X, y=None):
        # 记录原始的MocapData对象
        self.org_mocap_ = X[0].clone()
        # 清空原始MocapData对象的值
        self.org_mocap_.values.drop(self.org_mocap_.values.index, inplace=True)

        return self

    def transform(self, X, y=None):
        # 存储转换后的NumPy数组的列表
        Q = []
        
        for track in X:
            # 把每个MocapData对象的值转换为NumPy数组，并存储到Q列表
            Q.append(track.values.values)

        # 将Q列表转换为NumPy数组并返回
        return np.array(Q)

    def inverse_transform(self, X, copy=None):
        # 存储转换回来的MocapData对象的列表
        Q = []

        for track in X:
            # 复制原始的MocapData对象
            new_mocap = self.org_mocap_.clone()
            # 构造时间索引
            time_index = pd.to_timedelta([f for f in range(track.shape[0])], unit='s')
            
            # 将NumPy数组转换为DataFrame格式
            new_df =  pd.DataFrame(data=track, index=time_index, columns=self.org_mocap_.values.columns)
            
            # 将DataFrame设置为新MocapData对象的值
            new_mocap.values = new_df
            
            # 将新的MocapData对象添加到Q列表
            Q.append(new_mocap)

        # 返回转换回来的MocapData对象的列表
        return Q
```

`Numpyfier`类是一个转换器，它将输入的`MocapData`对象转换为NumPy数组。它的作用是在训练前的管道最后一步进行数据格式转换。该类没有参数，其构造函数为空。

`fit`方法对于该类来说不需要做任何操作，因此只需返回自身对象即可。

`transform`方法是将传入的运动捕捉数据集转换为NumPy数组，并将它们存储在一个列表中，最终返回一个NumPy数组。

`inverse_transform`方法是将NumPy数组转换回原始的`MocapData`对象。它接受一个NumPy数组，返回一个包含`MocapData`对象的列表。在这个方法中，构造一个新的`MocapData`对象，并将NumPy数组转换为DataFrame格式，然后将DataFrame设置为新`MocapData`对象的值。最后，将新的`MocapData`对象添加到一个列表中，最终返回该列表。

在BEAT论文中，`Numpyfier`类用于将运动捕捉数据转换为适合神经网络的NumPy数组格式，以便进行机器学习任务。



##### RootTransformer类

该代码实现了一个名为RootTransformer的类，用于将骨骼动作数据进行预处理。具体而言，它支持两种不同的转换方法：1）abdolute_translation_deltas和2）pos_rot_deltas。

在第一种情况下，该类计算了每个帧之间==根节点位置==的绝对平移量，并将其存储在新的列中。在第二种情况下，该类计算了每个帧之间根节点的位置和旋转角度之间的差值，并将其存储在新的列中。这些变量也是被删除的。

该类还实现了逆变换版本，可以将经过上述变换的数据恢复为原始格式。逆变换方法与正变换方法相反，通过将差异累积到先前的点来重新生成平移和旋转变量。

在BEAT论文中，该类用于将MoCap数据集中的骨骼动作数据转换处理后的特征，以供后续分类和生成任务使用。

```PYTHON
import pandas as pd
from sklearn.base import BaseEstimator, TransformerMixin

class RootTransformer(BaseEstimator, TransformerMixin):
    def __init__(self, method):
        """
        参数：
            method: str
                转换方法
                可接受的值有：
                abdolute_translation_deltas: 计算每个帧之间根节点位置的绝对平移量
                pos_rot_deltas: 计算每个帧之间根节点的位置和旋转角度之间的差值
        """
        self.method = method
    
    def fit(self, X, y=None):
        return self
    
    def transform(self, X, y=None):
        """
        按指定的转换方法处理传入的骨骼动作记录，并返回处理后的结果。

        参数：
            X: list of Skeleton objects
                待处理的骨骼动作数据
        返回值：
            Q: list of Skeleton objects
                处理后的骨骼动作数据
        """
        Q = []

        for track in X:
            if self.method == 'abdolute_translation_deltas':
                # 计算每个帧之间根节点位置的绝对平移量
                new_df = track.values.copy()
                
                # 获取各列名称
                xpcol = '%s_Xposition'%track.root_name
                ypcol = '%s_Yposition'%track.root_name
                zpcol = '%s_Zposition'%track.root_name
                dxpcol = '%s_dXposition'%track.root_name
                dzpcol = '%s_dZposition'%track.root_name
                
                # 计算绝对平移量，并将其存储在新的列中
                dx = track.values[xpcol].diff()
                dz = track.values[zpcol].diff()    
                dx[0] = 0
                dz[0] = 0
                new_df.drop([xpcol, zpcol], axis=1, inplace=True)
                new_df[dxpcol] = dx
                new_df[dzpcol] = dz
                
                # 构造新的 Skeleton 对象，存储处理后的数据
                new_track = track.clone()
                new_track.values = new_df
            
            elif self.method == 'pos_rot_deltas':
                # 计算每个帧之间根节点的位置和旋转角度之间的差值
                new_track = track.clone()

                # Absolute columns
                xp_col = '%s_Xposition'%track.root_name
                yp_col = '%s_Yposition'%track.root_name
                zp_col = '%s_Zposition'%track.root_name

                xr_col = '%s_Xrotation'%track.root_name
                yr_col = '%s_Yrotation'%track.root_name
                zr_col = '%s_Zrotation'%track.root_name

                # Delta columns
                dxp_col = '%s_dXposition'%track.root_name
                dzp_col = '%s_dZposition'%track.root_name

                dxr_col = '%s_dXrotation'%track.root_name
                dyr_col = '%s_dYrotation'%track.root_name
                dzr_col = '%s_dZrotation'%track.root_name


                new_df = track.values.copy()

                #计算各种差值，并将其存储在新的列中
                root_pos_x_diff = pd.Series(data=track.values[xp_col].diff(), index=new_df.index)
                root_pos_z_diff = pd.Series(data=track.values[zp_col].diff(), index=new_df.index)

                root_rot_y_diff = pd.Series(data=track.values[yr_col].diff(), index=new_df.index)
                root_rot_x_diff = pd.Series(data=track.values[xr_col].diff(), index=new_df.index)
                root_rot_z_diff = pd.Series(data=track.values[zr_col].diff(), index=new_df.index)


                root_pos_x_diff[0] = 0
                root_pos_z_diff[0] = 0

                root_rot_y_diff[0] = 0
                root_rot_x_diff[0] = 0
                root_rot_z_diff[0] = 0

                new_df.drop([xr_col, yr_col, zr_col, xp_col, zp_col], axis=1, inplace=True)

                new_df[dxp_col] = root_pos_x_diff
                new_df[dzp_col] = root_pos_z

```



###### RootCentricPositionNormalizer

`RootCentricPositionNormalizer` 类用于将骨骼动作序列的根节点（root）位置调整为原点，以便进行后续处理。具体来说，该类的 `transform` 方法将传入的骨骼动作序列中每个关节的位置都减去根节点在三维空间中的位置，从而使得根节点的位置为原点(0, 0, 0)。该类的 `inverse_transform` 方法则实现了相反的操作，即将处理过的骨骼动作序列恢复为原始格式。

代码中的注释解释了每个步骤的具体含义：

- `fit`: 模型拟合方法，不做任何操作，直接返回 `self`。
- `transform`: 对输入的骨骼动作序列进行变换，并返回变换后的结果。
  - 遍历输入的骨骼动作序列，处理每个轨迹。
  - 获取各个关键帧序列的信息，计算它们相对于根节点在三个方向上的偏移。
  - 创建新的 DataFrame 存储调整后的关键帧序列。
  - 遍历所有的关键帧，计算它们相对于根节点的偏移量，并将这些值存储到新的 DataFrame 中。
  - 保持根节点位置不变。
  - 构造新的 Skeleton 对象，将处理后的数据存储其中。
  - 将所有 Skeleton 对象存储到 Q 中并返回。
- `inverse_transform`: 将处理过的骨骼动作序列恢复为原始格式。
  - 遍历输入的骨骼动作序列，处理每个轨迹。
  - 获取各个关键帧序列的信息，计算它们相对于根节点在三个方向上的偏移。
  - 创建新的 DataFrame 存储调整后的关键帧序列。
  - 遍历所有的关键帧，计算它们相对于根节点的绝对位置，并将这些值存储到新的 DataFrame 中。
  - 构造新的 Skeleton 对象，将处理后的数据存储其中。
  - 将所有 Skeleton 对象存储到 Q 中并返回。

在 BEAT 论文中，`RootCentricPositionNormalizer` 类主要用于在步态识别任务中将骨骼动作序列的根节点位置标准化。具体来说，该类可用于在多种不同的设备和环境下收集到的骨骼动作序列中，消除由于人物身高、摄像机角度等因素导致的差异，从而使得模型能更好地捕捉人类行走时的特征。



##### Flattener

`Flattener` 类用于将多个骨骼动作序列拼接成一个单一的序列。具体来说，该类的 `transform` 方法将传入的骨骼动作序列组成的列表按轨迹拼接成一个单一的序列，并返回拼接后的结果。

代码中的注释解释了每个步骤的具体含义：

- `fit`: 模型拟合方法，不做任何操作，直接返回 `self`。
- `transform`: 对输入的骨骼动作序列进行变换，并返回变换后的结果。
  - 将输入的骨骼动作序列组成的列表按轨迹拼接成一个单一的序列。
  - 返回拼接后的序列。

在 BEAT 论文中，`Flattener` 类主要用于将多个骨骼动作序列组成的数据集转换为单个序列，以便对其进行分析和处理。例如，在步态识别任务中，可以使用该类将多个患者的骨骼动作数据拼接成一个单一的序列，从而构建更大规模的训练集，并提高模型的泛化能力。

```python
class Flattener(BaseEstimator, TransformerMixin):
    def __init__(self):
        pass

    def fit(self, X, y=None):
        return self

    def transform(self, X, y=None):
        return np.concatenate(X, axis=0)

```



##### ConstantsRemover

这个类是用于从MoCap数据中移除常量维度的预处理步骤。具体来说，它会找到标准差小于一定阈值的维度，并将这些维度视为常量维度。它会记录常量维度及其对应的常量值，并在后续的转换步骤中删除这些维度。

该类的主要方法是`fit`、`transform`和`inverse_transform`。`fit`方法用于计算常量维度及其对应的常量值。`transform`方法用于删除常量维度。`inverse_transform`方法则是用于恢复被删除的常量维度。

在BEAT论文中，ConstantsRemover类被用于预处理MoCap数据，以便可靠地生成语音合成时的手势。

```python
class ConstantsRemover(BaseEstimator, TransformerMixin):
    '''
    For now it just looks at the first track
    '''

    def __init__(self, eps = 10e-10):
        self.eps = eps
        

    def fit(self, X, y=None):
        # 计算第一个轨迹（MoCap数据）中的标准差
        stds = X[0].values.std()
        # 获取所有维度的名称
        cols = X[0].values.columns.values
        # 找到标准差小于阈值(eps)的维度，并记录它们的名称
        self.const_dims_ = [c for c in cols if (stds[c] < self.eps).any()]
        # 记录常量维度及其对应的常量值
        self.const_values_ = {c:X[0].values[c].values[0] for c in cols if (stds[c] < self.eps).any()}
        return self

    def transform(self, X, y=None):
        Q = []
        # 对于每个轨迹（MoCap数据），删除常量维度并将结果添加到新列表Q中
        for track in X:
            t2 = track.clone()
            t2.values = track.values[track.values.columns.difference(self.const_dims_)]
            Q.append(t2)
        
        return Q
    
    def inverse_transform(self, X, copy=None):
        Q = []
        # 对于每个轨迹（MoCap数据），恢复被删除的常量维度并将结果添加到新列表Q中
        for track in X:
            t2 = track.clone()
            for d in self.const_dims_:
                t2.values[d] = self.const_values_[d]
            Q.append(t2)

        return Q
```

ConstantsRemover类的作用是从数据中移除常量特征，即在整个特征集上取相同值的特征。这些特征对于机器学习算法没有任何信息价值，因为它们的值始终不变，无法帮助模型做出有意义的预测。

移除常量特征还有其他好处，包括：

1. 减少了内存占用和计算成本；
2. 引入噪音会导致模型过拟合，而移除常量特征可以降低过拟合的风险；
3. 在特征选择过程中可以减少冗余的信息，提高模型的可解释性。

因此，在进行数据预处理时，移除常量特征是一个很常见的步骤。



##### ListStandardScaler

ListStandardScaler是一个自定义的数据预处理类，用于将输入的列表或DataFrame对象中的数据进行标准化（Standardization）处理。下面是对每个方法的解释：

1. `__init__(self, is_DataFrame=False)`：构造函数，创建一个ListStandardScaler对象，并设置其是否需要将输入数据转换为DataFrame格式。
2. `fit(self, X, y=None)`：计算输入数据的平均值和标准差，并返回一个拟合好的ListStandardScaler对象。
3. `transform(self, X, y=None)`：对输入的数据进行标准化处理，并返回已经标准化后的数据。
4. `inverse_transform(self, X, copy=None)`：将标准化后的数据还原为未标准化的数据。

在BEAT论文中，ListStandardScaler被用于对动作序列数据进行预处理。作者首先使用MocapParameterizer类将原始的动作序列转换为根相对位置和角度信息，再使用RootCentricPositionNormalizer类和JointSelector类对其进行进一步处理，以便选取感兴趣的关节信息并将其转化为沿根轴向的旋转信息。然后，作者使用ListStandardScaler对这些旋转信息进行标准化处理，以便提高模型的训练效果。最后，作者使用RootTransformer类将标准化后的数据转换为机器学习模型所需的输入格式。

总之，ListStandardScaler是BEAT论文中用于对动作序列数据进行标准化处理的一个预处理类。它的作用是将输入的数据进行标准化处理，以便提高机器学习模型的训练效果。



##### DownSampler

DownSampler是一个自定义的数据预处理类，用于对输入的动作序列数据进行下采样操作。下面是对每个方法的解释：

1. `__init__(self, rate)`：构造函数，创建一个DownSampler对象，并设置其下采样的速率（即保留原始观测值的百分比）。
2. `fit(self, X, y=None)`：不执行任何操作，只返回已经拟合好的DownSampler对象。
3. `transform(self, X, y=None)`：对输入的动作序列数据进行下采样操作，并返回下采样后的数据。
4. `inverse_transform(self, X, copy=None)`：不执行任何操作，直接将输入数据返回。

在BEAT论文中，DownSampler被用于对动作序列数据进行降采样（Downsampling）。作者通过对原始数据进行下采样操作以减少数据集大小并提高模型训练效率。具体来说，作者将输入的动作序列按照设定的速率进行下采样，并返回下采样后的数据。最终，下采样后的数据将作为机器学习模型的输入进行训练和测试。

因此，DownSampler是BEAT论文中用于对动作序列数据进行下采样处理的一个预处理类，它的作用是将输入的数据进行下采样操作，以便提高模型的训练效率。

```PYTHON
class DownSampler(BaseEstimator, TransformerMixin):
    def __init__(self, rate):
        """
        构造函数，用于创建一个DownSampler对象并设置其下采样速率。
        
        参数：
        rate：int，下采样速率，即保留原始观测值的百分比。
        """
        self.rate = rate
        
    
    def fit(self, X, y=None):  
        """
        不执行任何操作，只返回已经拟合好的DownSampler对象。
        
        参数：
        X：array-like，输入的动作序列数据。
        y：None，不需要传入参数y。
        
        返回：
        self：返回已经拟合好的DownSampler对象本身。
        """
        return self
    
    def transform(self, X, y=None):
        """
        对输入的动作序列数据进行下采样，并返回下采样后的数据。
        
        参数：
        X：array-like，输入的动作序列数据。
        y：None，不需要传入参数y。
        
        返回：
        Q：list，下采样后的数据列表。
        """

        Q = []
        
        for track in X:
            # 下面这行代码将原始数据按照设定的速率进行下采样
            new_track = track[0:-1:self.rate]
            Q.append(new_track)
        
        return Q
        
    def inverse_transform(self, X, copy=None):
        """
        不执行任何操作，直接将输入数据返回。
        
        参数：
        X：array-like，输入的动作序列数据。
        copy：None，不需要传入copy参数。
        
        返回：
        X：array-like，输入的动作序列数据。
        """
        return X

```

` new_track = track[0:-1:self.rate]`这句话原理:

这句代码的作用是对`track`数组进行下采样，生成一个新的、下采样后的数组`new_track`。具体来说，它使用了Python的切片（Slicing）操作，其中`[0:-1:self.rate]`表示从第一个元素开始，按照步长为`self.rate`取出一部分元素，直到倒数第二个元素结束。

例如，如果`track`数组长度为10，`self.rate`为2，则`new_track`数组将包含原始数据中的1、3、5、7四个元素。如果`track`数组长度不能被`self.rate`整除，那么最后一个元素不会被包含在`new_track`数组中。因此，这行代码的实际效果就是对输入的动作序列数据进行下采样，并返回下采样后的数据。



#### preprocessing.ipynb



##### 1

这段代码是用于BEAT数据集的预处理，包括将原始数据转换为numpy格式并进行缩放、计算均值和标准差，并将所有数据保存到缓存中以便后续使用。下面是每个注释的解释：

```python
# calculate mean and build cache for data.计算数据的均值并构建缓存。
target_fps = 15
ori_list = joint_list["beat_joints"]  # 原始骨骼点的列表
target_list = joint_list["beat_141"]  # 目标骨骼点的列表
ori_data_path = "/home/ma-user/work/datasets/beat_rawdata_english/"  # 原始数据的路径
ori_data_path_npy = "/home/ma-user/work/datasets/beat_english_npy/"  # 原始数据转换为numpy格式后的路径
ori_data_path_ann = "/home/ma-user/work/datasets/beat_annotations_english/"  # 原始注释数据的路径
cache_path = f"/home/ma-user/work/datasets/beat_cache/beat_4english_{target_fps}_141/"  # 缓存数据的路径
reduce_factor_json = int(60/target_fps)  # json格式文件数据缩放因子
reduce_factor_bvh = int(120/target_fps)  # bvh格式文件数据缩放因子
print(f"target_fps: {target_fps}, reduce json {reduce_factor_json}, reduce bvh {reduce_factor_bvh}")
speakers = sorted(os.listdir(ori_data_path),key=str,)  # 所有说话者的列表

npy_s_v = []  # 用于存储均值和标准差的空列表
npy_s_k = []
json_s_v = []
bvh_s_v = []

load_type = "train"  # 数据集类型为“训练集”

if not os.path.exists(f"{cache_path}"): 
    os.mkdir(cache_path)  # 如果缓存路径不存在，则创建相应的目录
if not os.path.exists(f"{cache_path}{load_type}/"): 
    os.mkdir(f"{cache_path}{load_type}/")
    os.mkdir(f"{cache_path}{load_type}/wave16k/")
    os.mkdir(f"{cache_path}{load_type}/bvh_rot/")
    os.mkdir(f"{cache_path}{load_type}/bvh_full/")
    os.mkdir(f"{cache_path}{load_type}/bvh_rot_vis/")
    os.mkdir(f"{cache_path}{load_type}/facial52/")
    os.mkdir(f"{cache_path}{load_type}/text/")
    os.mkdir(f"{cache_path}{load_type}/emo/")
    os.mkdir(f"{cache_path}{load_type}/sem/")     

for speaker in [2,4,6,8]:  # 对于四个说话者，其中[2,4,6,8]代表它们的编号。如果想要处理所有说话者，将这行代码替换为for speaker in range(1,32)
    all_data = os.listdir(ori_data_path_npy+str(speaker))  # 获取当前说话者的所有数据列表
    npy_all = []
    json_all = []
    bvh_all = []   
    for ii, file in tqdm(enumerate(all_data)):  # 对每个文件进行循环操作
        file = file[:-4]
        shutil.copy(f"{ori_data_path_npy}/{file.split('_')[0]}/{file}.npy", f"{cache_path}{load_type}/wave16k/{file}.npy")
        try:shutil.copy(f"{ori_data_path}/{file.split('_')[0]}/{file}.TextGrid", f"{cache_path}{load_type}/text/{file}.TextGrid")
        except: print(f"{file}.TextGrid")
        try: shutil.copy(f"{ori_data_path_ann}/{file.split('_')[0]}/{file}.txt", f"{cache_path}{load_type}/sem/{file}.txt")
        except: print(f"{file}.txt")
        try: shutil.copy(f"{ori_data_path_ann}/{file.split('_')[0]}/{file}.csv", f"{cache_path}{load_type}/emo/{file}.csv")
        except: print(f"{file}.csv")
        npy_all.extend(list(np.load(f"{ori_data_path_npy}/{file.split('_')[0]}/{file}.npy")))

        # 加载json格式的文件
        with open(f"{ori_data_path}/{file.split('_')[0]}/{file}.json", "r", encoding='utf-8') as json_file_raw:
            json_file = json.load(json_file_raw)
            with open(f"{cache_path}{load_type}/facial52

```



##### 2

这段代码的功能是为BEAT数据集计算全局音频、面部表情和身体姿态的平均值和标准差，并保存为.npy文件。它遍历了列表中的每个说话者，加载预先生成的.npy文件（在上面的代码中创建），并计算出所有说话者的平均值和标准差，最后将其保存。下面是这段代码中每个注释的解释：

```python
# calculate global mean and std for average all speakers 计算所有说话者的平均值和STD

# 用于存储来自三种模态的所有说话者的平均值和标准差。
npy_s_v = []
json_s_v = []
bvh_s_v = []

# 存储数据的路径。
npy_path = f"{cache_path}{load_type}/wave16k/"
bvh_path = f"{cache_path}{load_type}/bvh_rot/"
json_path = f"{cache_path}{load_type}/facial52/" 
        
# 循环遍历四个说话者。
for i in [2,4,6,8]:
    # 加载.npy文件，其中包含第i个说话者的平均值和标准差，并将其添加到对应列表中。
    npy_s_v.append([np.load(f"{npy_path}npy_mean_{i}.npy"), np.load(f"{npy_path}npy_std_{i}.npy")])
    json_s_v.append([np.load(f"{json_path}json_mean_{i}.npy"), np.load(f"{json_path}json_std_{i}.npy")])
    bvh_s_v.append([np.load(f"{bvh_path}bvh_mean_{i}.npy"), np.load(f"{bvh_path}bvh_std_{i}.npy")])

# 设置总长度变量为 0。
all_length = 0

# 新的均值和标准差。
new_m = np.zeros_like(npy_s_v[0][0])
new_s = np.zeros_like(npy_s_v[0][0])

# 遍历所有说话者。
for i, (m, s) in enumerate(npy_s_v):
    # 计算总长度。
    all_length += npy_s_k[i][0]
    # 加权和，用于计算加权平均值和标准差。
    new_m += npy_s_k[i][0] * m
    new_s += (m**2+s**2) * npy_s_k[i][0]

# 计算加权平均值。
new_m /= all_length

# 根据方差的定义，减掉平方均值。
new_s -= (new_m**2)

# 对方差进行开根操作，即得到标准差。
new_s = math.sqrt(new_s)

# 打印新的均值和标准差。
print(f" audio mean: {new_m}, std: {new_s}") 

# 保存音频模态的平均值和标准差。
np.save(f"{npy_path}npy_mean.npy", new_m)
np.save(f"{npy_path}/npy_std.npy", new_s)  

# 新的均值和标准差。
new_m = np.zeros_like(json_s_v[0][0])
new_s = np.zeros_like(json_s_v[0][0])

# 设置总长度变量为 0。
all_length = 0

# 遍历所有说话者。
for i, (m, s) in enumerate(json_s_v):
    # 计算总长度。
    all_length += npy_s_k[i][0]
    # 加权和，用于计算加权平均值和标准差。
    new_m += npy_s_k[i][0] * m
    new_s += (m**2+s**2) * npy_s_k[i][0]

# 计算加权平均值。
new_m /= all_length

# 根据方差的定义，减掉平方均值。
new_s -= (new_m**2)

# 对方差进行开根操作，即得到标准差。
new_s = np.sqrt(new_s)

# 打印新的均值和标准差。
print(f" json mean: {new_m}, std: {new_s}") 

# 保存面部表情模态的平均值和标准差。
np.save(f"{json_path}json_mean.npy", new_m)
np.save(f"{json_path}/json_std.npy", new_s)

# 新的均值和标准差。
new_m = np.zeros_like(bvh_s_v[0][0])
new_s = np.zeros_like(bvh_s_v[0][0])

# 设置总长度变量

```



##### 3

这段代码定义了一个名为`split_rule_english`的字典，其中包含BEAT数据集中两组演讲者的训练、验证和测试集。字典的键指定哪些演讲者属于哪个组。

对于每个组，“train”，“val”和“test”键包含字符串列表，表示要在每个拆分中使用的视频序列。每个字符串由三个用下划线隔开的数字组成：第一个数字指定演讲者，第二个数字指定会话，第三个数字指定会话中的视频序列。

该代码似乎是BEAT数据集的数据预处理步骤的一部分，在此步骤中，根据指定的拆分将数据集分成训练、验证和测试集。该代码看起来不涉及到定义任何类。

需要注意的是，该代码只是整个BEAT数据集创建过程的一小部分，可能无法提供足够的上下文来了解它如何适应更大的框架。

```python
# 定义一个字典 split_rule_english，包含 BEAT 数据集中两组演讲者的训练、验证和测试集
split_rule_english = {
    # 第一组：4小时的演讲者 x 10个演讲者
    "1, 2, 3, 4, 6, 7, 8, 9, 11, 21":{
        # 训练集: 188 分钟每个序列 (48+40+100=188)
        "train": [
            "0_9_9", "0_10_10", "0_11_11", "0_12_12", "0_13_13", "0_14_14", "0_15_15", "0_16_16",
            "0_17_17", "0_18_18", "0_19_19", "0_20_20", "0_21_21", "0_22_22", "0_23_23", "0_24_24",
            "0_25_25", "0_26_26", "0_27_27", "0_28_28", "0_29_29", "0_30_30", "0_31_31", "0_32_32",
            "0_33_33", "0_34_34", "0_35_35", "0_36_36", "0_37_37", "0_38_38", "0_39_39", "0_40_40",
            "0_41_41", "0_42_42", "0_43_43", "0_44_44", "0_45_45", "0_46_46", "0_47_47", "0_48_48",
            "0_49_49", "0_50_50", "0_51_51", "0_52_52", "0_53_53", "0_54_54", "0_55_55", "0_56_56",
            
            "0_66_66", "0_67_67", "0_68_68", "0_69_69", "0_70_70", "0_71_71",
            "0_74_74", "0_75_75", "0_76_76", "0_77_77", "0_78_78", "0_79_79",
            "0_82_82", "0_83_83", "0_84_84", "0_85_85",
            "0_88_88", "0_89_89", "0_90_90", "0_91_91", "0_92_92", "0_93_93",
            "0_96_96", "0_97_97", "0_98_98", "0_99_99", "0_100_100", "0_101_101",
            "0_104_104", "0_105_105", "0_106_106", "0_107_107", "0_108_108", "0_109_109",
            "0_112_112", "0_113_113", "0_114_114", "0_115_115", "0_116_116", "0_117_117",
            
            "1_2_2", "1_3_3", "1_4_4", "1_5_5", "1_6_6", "1_7_7", "1_8_8", "1_9_9", "1_10_10", "1_11_11",
        ],
        # 验证集: 25 分钟每个序列 (8+7+10=25)
        "val": [
            "0_57_57", "0_58_58", "0_59_59", "0_60_60", "0_61_61", "0_62_62", "0_63_63", "0_64_64",
            "0_72_72", "0_80_80", "0_86_86", "0_94_94", "0_102_102", "0_110_110", "0_118_118",
            "1_12_12",
        ],
        # 测试
 "test": [
           "0_1_1", "0_2_2", "0_3_3", "0_4_4", "0_5_5", "0_6_6", "0_7_7", "0_8_8", \
           "0_65_65", "0_73_73", "0_81_81", "0_87_87", "0_95_95", "0_103_103", "0_111_111", \
           "1_1_1",
        ],
    },
    
    # 1h speakers x 20
    "5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30":{
        # 8+7+20=35mins each
        "train": [
            "0_9_9", "0_10_10", "0_11_11", "0_12_12", "0_13_13", "0_14_14", "0_15_15", "0_16_16", \
            "0_66_66", "0_74_74", "0_82_82", "0_88_88", "0_96_96", "0_104_104", "0_112_112", "0_118_118", \
            "1_2_2", "1_3_3", 
            "1_0_0", "1_4_4", # for speaker 29 only
        ],
        # 4+3.5+5 = 12.5mins each
        # 0_65_a and 0_65_b denote the frist and second half of sequence 0_65_65
        "val": [
            "0_5_5", "0_6_6", "0_7_7", "0_8_8",  \
            "0_65_b", "0_73_b", "0_81_b", "0_87_b", "0_95_b", "0_103_b", "0_111_b", \
            "1_1_b",
        ],
        # 4+3.5+5 = 12.5mins each
        "test": [
           "0_1_1", "0_2_2", "0_3_3", "0_4_4", \
           "0_65_a", "0_73_a", "0_81_a", "0_87_a", "0_95_a", "0_103_a", "0_111_a", \
           "1_1_a",
        ],
    },
}




```



##### 4

这个代码定义了一个`cut_sequence`函数，这个函数用于将视频序列按照时间分割成两部分并保存到不同的文件中。具体来说：

- `source_path`参数为要切割的视频序列的路径；
- `save_path_a`和`save_path_b`参数是两个目标文件的路径，分别代表第一部分和第二部分的文件；
- `file_id`参数指定了当前视频序列的编号；
- `fps`参数表示视频的帧率；
- `sr`参数表示音频的采样率；
- `tmp`参数是一个临时目录的路径，用于存储原始文件。

该函数支持不同类型的输入文件，包括`.npy`（numpy数组）、`.bvh`（3D运动捕捉数据）和`.json`（带有关键点注释的JSON文件），对于每种输入格式都有相应的处理流程。如果输入文件与这些格式不匹配，则会将其复制到目标文件中而不进行任何处理。

该函数在论文《BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis》的应用是将BEAT数据集的视频序列划分成两个时间段，以便进行连续的情感手势合成。

```python
def cut_sequence(source_path, save_path_a, save_path_b, file_id, fps = 15, sr = 16000, tmp="/home/ma-user/work/datasets/beat_tmp/"):
    if not os.path.exists(tmp): os.mkdir(tmp)
    cut_point = 30 if file_id.split("_")[0] == "0" else 300 #in seconds
    if source_path.endswith(".npy"):
        data = np.load(source_path)
        data_a = data[:sr*cut_point]
        data_b = data[sr*cut_point:]
        np.save(save_path_a, data_a)
        np.save(save_path_b, data_b)
        
    elif source_path.endswith(".bvh"):
        copy_lines = 431 if "full" in source_path or "vis" in source_path else 0
        with open(source_path, "r") as data:
            with open(save_path_a, "w") as data_a:
                with open(save_path_b, "w") as data_b:
                    for i, line_data in enumerate(data.readlines()):
                        if i < copy_lines:
                            data_a.write(line_data)
                            data_b.write(line_data)
                        elif i < cut_point * fps:
                            data_a.write(line_data)
                        else:
                            data_b.write(line_data)
    
    elif source_path.endswith(".json"):
        with open(source_path, "r", encoding='utf-8') as data:
            json_file = json.load(data)
            with open(save_path_a, "w") as data_a:
                with open(save_path_b, "w") as data_b:
                    new_frames_a = []
                    new_frames_b = []
                    for json_data in json_file["frames"]:
                        if json_data["time"] < cut_point:
                            new_frames_a.append(json_data)
                        else:
                            new_frame = json_data.copy()
                            new_frame["time"]-=cut_point
                            new_frames_b.append(new_frame)
                    json_new_a = {"names":json_file["names"], "frames": new_frames_a}
                    json_new_b = {"names":json_file["names"], "frames": new_frames_b}
                    json.dump(json_new_a, data_a)
                    json.dump(json_new_b, data_b) 
        
    else:
        # processing in the dataloader
        shutil.copy(source_path, save_path_a)
        shutil.copy(source_path, save_path_b)
    try:
        shutil.move(source_path, tmp)
    except:
        print(source_path)
```



##### 5

这段代码的作用是将BEAT数据集中的视频序列按照指定比例（在`split_rule_english`字典中定义）进行划分，并将其保存到训练、验证和测试集文件夹中。具体来说：

- `speaker_names`列表包含BEAT数据集中所有演讲者的姓名；
- `default_path`变量存储了BEAT数据集的默认路径；
- `four_hour_speakers`和`one_hour_speakers`分别是四小时和一小时语音样本的演讲者编号列表；
- `folders`变量包含默认路径下所有的文件夹；
- `endwith`列表包含每个文件夹里第500个文件的扩展名，用于确定文件类型；
- 在循环中，根据指定的演讲者ID和`split_rule_english`字典，在验证集或测试集中选择要使用的文件。对于每个选定的文件，如果其名称以`_b`结尾，则使用`cut_sequence`函数将其拆分为两个不同的部分并保存到不同的文件中；否则，将它们移动到目标文件夹中。

在BEAT论文中，这段代码用于将数据集中的视频序列按照指定比例随机分成训练、验证和测试三个集合，以便进行情感手势合成的实验。

```python
#spilt data
#存储BEAT数据集中所有演讲者的姓名
speaker_names = [
    "wayne", "scott", "solomon", "lawrence", "stewart", "carla", "sophie", "catherine", "miranda", "kieks", \
    "nidal", "zhao", "lu", "zhang", "carlos", "jorge", "itoi", "daiki", "jaime", "li", \
    "ayana", "luqi", "hailing", "kexin", "goto", "reamey", "yingqing", "tiffnay", "hanieh", "katya",
]

#BEAT数据集默认路径
default_path = "/home/ma-user/work/datasets/beat_cache/beat_4english_15_141/train"

#四小时语音样本的演讲者编号列表
four_hour_speakers = "1, 2, 3, 4, 6, 7, 8, 9, 11, 21".split(", ")
#一小时语音样本的演讲者编号列表
one_hour_speakers = "5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30".split(", ")

# 获取默认路径下的所有文件夹
folders = os.listdir(default_path)

# 创建验证集和测试集文件夹
if not os.path.exists(default_path.replace("train", "val")): os.mkdir(default_path.replace("train", "val"))
if not os.path.exists(default_path.replace("train", "test")): os.mkdir(default_path.replace("train", "test"))

# 获取每个文件夹中第500个文件的扩展名，用于确定文件类型
endwith = []
for folder in folders:
    if not os.path.exists(default_path.replace("train", "val")+"/"+folder): os.mkdir(default_path.replace("train", "val")+"/"+folder)
    if not os.path.exists(default_path.replace("train", "test")+"/"+folder): os.mkdir(default_path.replace("train", "test")+"/"+folder)
    endwith.append(os.listdir(default_path+"/"+folder)[500].split(".")[-1])
    
# 针对特定演讲者ID，将数据分成验证集和测试集
for speaker_id in tqdm([2,4,6,8]):
    # 选择要在验证集中使用的文件列表
    val = split_rule_english["5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30"]["val"] if str(speaker_id) in one_hour_speakers else split_rule_english["1, 2, 3, 4, 6, 7, 8, 9, 11, 21"]["val"]
    # 选择要在测试集中使用的文件列表
    test = split_rule_english["5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30"]["test"] if str(speaker_id) in one_hour_speakers else split_rule_english["1, 2, 3, 4, 6, 7, 8, 9, 11, 21"]["test"]
    
    # 针对选定的每个文件，在目标文件夹中创建两个新文件并将其拆分为两个部分
    for file_id in val:
        for ide, folder in enumerate(folders):
            if "b" in file_id:
                cut_sequence(
                    source_path=f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id.split('_')[0]}_{file_id.split('_')[1]}_{file_id.split('_')[1]}.{endwith[ide]}",
                    save_path_a=f"{default_path.replace('train', 'test')}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id.split('_')[0]}

```



















下面提到的DownSampler与上面那个RootCentricPositionNormalizer,MocapParameterizer类和JointSelector类，Numpyfier类,RootTransformer在同一个预处理文件中，能否结合上面这些类具体告诉我其作用与意思，包括具体每一句注释等，且由于这个代码是论文《BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis》所提出的对应代码，可以说说这个类在BEAT论文的应用

```python

```

谢谢





下面提到的代码是接在上面那个代码后面的，也是在一个预处理文件中的一部分，能否结告诉我其作用与意思，并且在代码中给他每一句加注释等，且由于这个代码是论文《BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis》所提出的对应代码，可以说说这个类在BEAT论文的应用吗

```python
# spilt data
speaker_names = [
    "wayne", "scott", "solomon", "lawrence", "stewart", "carla", "sophie", "catherine", "miranda", "kieks", \
    "nidal", "zhao", "lu", "zhang", "carlos", "jorge", "itoi", "daiki", "jaime", "li", \
    "ayana", "luqi", "hailing", "kexin", "goto", "reamey", "yingqing", "tiffnay", "hanieh", "katya",
]
default_path = "/home/ma-user/work/datasets/beat_cache/beat_4english_15_141/train"
four_hour_speakers = "1, 2, 3, 4, 6, 7, 8, 9, 11, 21".split(", ")
one_hour_speakers = "5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30".split(", ")
folders = os.listdir(default_path)
if not os.path.exists(default_path.replace("train", "val")): os.mkdir(default_path.replace("train", "val"))
if not os.path.exists(default_path.replace("train", "test")): os.mkdir(default_path.replace("train", "test"))
endwith = []
for folder in folders:
    if not os.path.exists(default_path.replace("train", "val")+"/"+folder): os.mkdir(default_path.replace("train", "val")+"/"+folder)
    if not os.path.exists(default_path.replace("train", "test")+"/"+folder): os.mkdir(default_path.replace("train", "test")+"/"+folder)
    endwith.append(os.listdir(default_path+"/"+folder)[500].split(".")[-1])
    
for speaker_id in tqdm([2,4,6,8]):
    val = split_rule_english["5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30"]["val"] if str(speaker_id) in one_hour_speakers else split_rule_english["1, 2, 3, 4, 6, 7, 8, 9, 11, 21"]["val"]
    test = split_rule_english["5, 10, 12, 13, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 25, 26, 27, 28, 29, 30"]["test"] if str(speaker_id) in one_hour_speakers else split_rule_english["1, 2, 3, 4, 6, 7, 8, 9, 11, 21"]["test"]
    for file_id in val:
        for ide, folder in enumerate(folders):
            if "b" in file_id:
                cut_sequence(
                    source_path=f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id.split('_')[0]}_{file_id.split('_')[1]}_{file_id.split('_')[1]}.{endwith[ide]}",
                    save_path_a=f"{default_path.replace('train', 'test')}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id.split('_')[0]}_{file_id.split('_')[1]}_a.{endwith[ide]}",
                    save_path_b=f"{default_path.replace('train', 'val')}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id.split('_')[0]}_{file_id.split('_')[1]}_b.{endwith[ide]}",
                    file_id = file_id,
                        )
            else:
                #pass
                try:
                    shutil.move(f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id}.{endwith[ide]}", f"{default_path.replace('train', 'val')}/{folder}/")
                except:
                    print(f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id}.{endwith[ide]}")
    for file_id in test:
        for ide, folder in enumerate(folders):
            if "a" in file_id:
                pass
            else:
                #pass
                try:
                    shutil.move(f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id}.{endwith[ide]}", f"{default_path.replace('train', 'test')}/{folder}/")
                except:
                    print(f"{default_path}/{folder}/{speaker_id}_{speaker_names[speaker_id-1]}_{file_id}.{endwith[ide]}")

```





### 5、毕业论文

#### 时间安排

4/6 列提纲 给老师看

4/7写目录之类的



#### 提纲



题目：

“基于数字人的多模态单向交互技术——语音驱动动作和捏人技术研究与应用”

面向数字人应用的捏人系统与语音驱动手势动作





#### 参考文章

别人的论文

**已下载**

**基于语音驱动的表情动画设计与实现_郭梦婷** 

艺术学位硕士论文，但是是工程类论文，可以参考其中一些文案

**基于多模态智能交互的虚拟数字人_李晓明**

公司做的完善的数字人系统

**面向数字人应用的人脸表示与动画驱动_陈柯宇**

很牛的硕士论文 一堆神经网络

**虚拟数字人手势交互设计_王晓慧**

理论研究









能给我列一个本科毕业论文的提纲吗，这是我的论文大致内容：

捏人模块包括实现捏脸、捏表情和捏体型，语音驱动动作模块包括实现输入语音文件，输出语音对应的姿态动画，即通过语音驱动动作。动画模块包括动画模块实现各种资产适应数字人动作并结合捏人系统。其中语音驱动动作模块参考论文《BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis》







本科毕业论文提纲：

I. 绪论

- 研究背景和意义
- 国内外研究现状及不足之处
- 研究目的和意义
- 论文结构

II. 相关技术与理论

- 数字人相关技术
- 捏人模块实现技术
- 语音驱动动作模块实现技术
- 动画模块实现技术
- BEAT数据集介绍

III. 捏人模块的设计与实现

- 捏脸功能实现
- 捏表情功能实现
- 捏体型功能实现

IV. 语音驱动动作模块的设计与实现

- 输入语音文件处理
- 姿态动画生成算法
- 语音驱动动作实现细节

V. 动画模块的设计与实现

- 资产适应数字人动作
- 捏人系统与动画模块的结合

VI. 实验设计与结果分析

- 设计实验方案
- 数据采集与处理
- 结果展示与分析

VII. 总结与展望

- 研究工作总结
- 不足之处与改进方向
- 发展前景与应用价值

参考文献
