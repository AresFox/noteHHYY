## Unity 数字人流程

> 作者：ArexFox，ChrisZhang。
>
> 时间:2023-03-10



## 一、DAZ和Unity相关准备

在这一部分中，将会总结DAZ中的数字人导入Unity前需要做的准备，包括插件下载，捏脸部分以及BS导出等。

### 1.DAZ学习

[Getting Started With Daz3d Part 1 | Creating Your First Character with Daz3d - YouTube](https://www.youtube.com/watch?v=jrZv_Tjpzg0)

一些常规的操作,如何使捏脸效果更加美观等关于DAZ的基本使用后面再进行整理学习,

#### （1）下载的资源放在哪

比如在网上下载了对应的资源包,想要放在正确的路径下,从而让DAZ识别并进而可以导出,做法如下:

(1)首先,F2打开Preferences界面,选择Content,点击下面的Content Directory Manager,这里可以修改和查看文件路径存放在哪里:

![image-20230310161310870](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310161310870.png)

比如我把下载的资源放在`D:\Daz3D\official\installPaths\Applications\Data\DAZ 3D\My DAZ 3D Library\Content`这个`DAZ Studio Formats`相关的路径下面,就可以在DAZ 3D的Content Library面板当中找到资源相关的内容,见上图左侧.



#### （2）DAZ 调整人物

按照下图所示的内容进行搜索可以找到控制整个人的BS,调整Aillee选项卡可以调整整个人的情况.

![image-20230310161349782](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310161349782.png)

下巴开合:

![image-20230310161401060](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310161401060.png)

显示骨骼的设定:

![image-20230310161420635](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310161420635.png)



------

### 2.DAZ to Unity 插件下载

参考视频链接：[Daz Bridges Tutorial - Daz to Unity - YouTube](https://www.youtube.com/watch?v=we8hPrz_KQU)

(1)首先,进入到DAZ Central,搜索并下载DAZ To Unity插件,下载并进行安装,重启DAZ 和DAZ Central;

(2)此时在DAZ当中的File->Send To应该可以找到DAZ to Unity的选项,在首次配置的时候进行如下选择:

![image-20230310160818609](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310160818609.png)

下载完之后,应该是可以在项目工程的目录里找到一个Unitypackage文件,根据当前的管线在Unity当中安装即可。（管线在后续的开发中都选择了HDRP管线，其实类似于URP的也是可以的）

相关的参考流程如下图所示：

![image-20230310160958378](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310160958378.png)

在导入之后，要对HDRP默认管线进行一定配置，包括Shader资产相关的，如下图一样调整即可。

![image-20230310161023181](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310161023181.png)

------



### 3.DAZ 导出相关素材

学习了DAZ的基本使用并完成了上述的插件下载之后，打开待导入的Unity文件，就可以准备导入了。首先我们导入对应的人物，接着导入服装资产和对应的动画资产，具体如下：
#### （1）导入人物

这里我们以Aillee人物素材为例，首先在DAZ当中导入对应的人物（不要穿衣服和头发，也不要动画资产），然后在DAZ中选择File->Send To->Send To Unity，然后做出如下调整：

![image-20230310162818657](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310162818657.png)

对于不同要导出的资产，原则如下：

- 如果要导出人物的话，选择Asset Type为Skeletal Mesh，这样导出的角色就会涵盖DAZ自己的骨骼（比如这里的Aillee是Genesis8类型的骨骼），**关于骨骼的运用将在后续进行整理**；
- 如果要导出动画资产的话，选择类型为Animation，并且要进行Bake操作（后面会说）；
- 如果要导出衣服裤子这种资产，也要选择Asset Type为Skeletal Mesh，因为后面衣服要随着身体一起做骨骼动画，所以也要导出对应的骨骼。
- 衣服，人物都可以选择Export Morphs，点击Choose Morphs可以选择要导出的BS类型。

这里我们还是先导出人物。按照上图所示全部准备完毕后就可以按Accept将人物导出了（此时Unity项目工程需要保持开启状态），此时Unity中呈现如下：

![image-20230310163730857](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310163730857.png)

此时可以注意到，人物的眼球材质是有问题的，这也是接下来要修复的点。

##### （a）修复眼球相关材质

经过观察，人物的眼球和眉毛都不带有透明的感觉，因此合理怀疑是Shader的问题。查询DAZ 导入 Unity的自带Shader可以发现，我们需要把眼球和眉毛对应的Shader修改为DAZ3D->Wet。以下是修改前和修改后的Shader对比：

![image-20230310165158587](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310165158587.png)

观察场景中的人物，发现其有两个孩子结点，分别是身体的以及Eyelashes，对于两个孩子分别有如下几个材质球需要修改：

- 人物身上的EyeMoisture，Cornea材质球；
- 人物的Eyelash结点的Eyelashes和EyeMoisture材质球；

修改之后的效果如下图所示：

![image-20230310165605057](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310165605057.png)

至此，DAZ的人物导入Unity的步骤就完成了，接下来是动画资产的相关部分。



#### （2）导入动画

这里介绍两种动画的导入方式，分别是Mixamo的动画和DAZ自带的动画，在后面的动画系统中会对两者进行对比。

##### （a）Mixamo的动画素材

首先，需要有一个Mixamo支持的骨骼类型的角色（当然也可以是Mixamo自带的角色），**这里注意由于DAZ的骨骼跟Mixamo系统的差异性很大，因此直接的DAZ角色的fbx模型是无法上传到Mixamo的**。这里我们使用的是《原神》里的迪卢克的fbx模型上传到Mixamo上，选择下载的动画配置如下：

![image-20230310171230702](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310171230702.png)

将下载好的fbx动画资产导入到Unity当中，留作后续使用。



##### （b）DAZ自带的动画素材

首先，找到对应DAZ的动画资产，然后在DAZ当中拖入到场景当中，接下来需要在动画轨道里右键选择Bake To Studio Keyframes（这个很重要！！），接下来就可以把动画资产导出了（**关于Export Morphs选项：因为动画资产实际上只动了角色的骨骼（骨骼动画），而BS的改动会在后面的AI驱动模块里，因此这里动画是否导出BS暂时是无所谓的，节省空间还是不导出了。**）：

![image-20230310174300958](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310174300958.png)

这里需要注意的一点是要勾选上Transfer Face Bones，否则的话可能会因为骨骼不匹配导致导入Unity之后的五官出现漂移现象。

**注：如果动画资源在第一帧的时候角色会复原为A字型，不需要担心，后面会在Unity当中修改动画资产。**

导入Unity之后，配置简单的状态机（后面也会介绍如何配置状态机），呈现的效果如下图所示：

![image-20230310174641066](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310174641066.png)

------



#### （3）导入衣服

在DAZ当中为角色穿上衣服，**然后点击衣服的相关资产（注意点击衣服而不是人），**将其Unparent一下，从而跟人物模型分开（否则后续导入的话会把整个人都导入进来），如下图：

![image-20230310175830866](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310175830866.png)

此时衣服就可以和人分离开了，不再作为人物的子结点，此时可以删除Aillee人物结点，然后导出衣服。

**按照前面人物SkeletalMesh的方法导出衣服的SkeletalMesh**，导出成功到Unity之后的显示应该跟人物差不多，如下图所示：

<img src="Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310180442450.png" alt="image-20230310180442450" style="zoom:80%;" />

我们会发现衣服是没有材质的，这就需要手动为其附上材质，进入到衣服的FBX模型当中，选择Materials选项卡->Extract Materials，并将导出的材质球放置在某个指定的文件夹里，然后为衣服赋予材质：

![image-20230310181420513](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310181420513.png)

（**注：这里的材质只是简单的给了相关的贴图，还是有一些能凹的地方，不过笔记主要展示流程就不进阶处理衣服材质了，后面会整理头发材质相关的内容。**）

------



## 二、数字人基础功能建设

### 1.捏脸系统

所谓的捏脸系统，就是可以通过UI对导入的人物的BS进行调整，同时要在执行动画的时候保留人物脸部的BS参数（**这点倒是不需要太过在意，因为骨骼动画并不会改变脸型的相关BS值**）。因此大概的思路就是通过slider来控制BS的值（注意，Unity的BS范围是0-100），并通过代码逻辑来控制。这里可以写一个简易的脚本如下：

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HFacePinching : MonoBehaviour
{
    public Slider[] bsSliders; //绑定Canvas下面的控制BS的Slider
    public GameObject digitalHumanShape;  //绑定导入角色的Shape结点，因为这个结点下有BS可以控制
    SkinnedMeshRenderer skinnedMeshRenderer;

    void Awake()
    {
        skinnedMeshRenderer = digitalHumanShape.gameObject.GetComponent<SkinnedMeshRenderer>();
    }

    private void Start()
    {
        for (int i = 0; i < bsSliders.Length; i++)
        {
            int index = i; //需要声明一个新的变量=i,否则可能会有调用顺序问题,这里不表
            bsSliders[index].onValueChanged.AddListener(delegate { OnBSSliderValueChanged(index); } );
        }
        
    }

    private void OnBSSliderValueChanged(int index)
    {
        //回调函数:根据Slider控制的值调整BS的大小
        skinnedMeshRenderer.SetBlendShapeWeight(index, bsSliders[index].value * 100.0f); //因为Unity里BS的范围是0-100
    }
}

```

新建结点并绑定对应的脚本，并为其赋值如下:

![image-20230310185351229](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310185351229.png)

此时就可以通过Slider来调整人物脸上的BS，在后面的AI模块驱动中也有调试BS相关的功能，到时也会进行总结。



### 2.动画系统

#### （1）骨骼系统

DAZ的骨骼系统和传统的软件和引擎中的骨骼系统有所不同，因此并不能让DAZ的骨骼无缝衔接Unity的骨骼系统。

DAZ的骨骼和Unity的骨骼系统分别如下：

![image-20230310201501704](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310201501704.png)

可以看到，DAZ的骨骼结构比Unity的骨骼结构要复杂很多，在骨骼重定向的时候会出现一定问题，这个问题具体地我们会在[存在的问题和接下来可以提升的点](#（5）可以提升的点和接下来的问题)当中进行介绍，并给出可行的解决方案（具体的方案暂时还没有落实）。

正因为DAZ和Unity二者的骨骼系统之间的差异，因此对于动画系统来说有两种可行的解决方案：

- （1）全部使用DAZ的动画，此时动画资产和角色模型资产的Rig全部设置为Generic，并且动画和人物的骨骼需要保持一致。（从DAZ直接导出的动画是符合这一要求的）；
- （2）使用其他常见动画素材库中的骨骼动画资源（如上文提到的Mixamo），此时Mixamo下载的动画资产和DAZ导入的人物和动画资产全部都要适配于Unity的骨骼系统（也就是把Rig调整为Humanoid），也就是需要进行重定向的操作。

**在接下来的篇幅中，我们会对两种方式进行逐一说明，但在此之前需要对动画资产进行预处理，同时设定对应的状态机。**



#### （2）动画预处理&状态机说明

##### （a）修改序列帧动画

在导入动画的章节中我们提到了一种可能的问题，即DAZ的动画在第一帧人物会变为T字型，此时动画就会有一个突变效果，十分影响观感。同时，有些动画资产会出现穿模的问题，**此时都可以通过对动画资产进行精修的方式来适当地解决该问题**，具体介绍如下：

首先，通过Ctrl+D快捷键将动画资产中的序列帧动画复制出来，并双击打开：

![image-20230310202949087](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310202949087.png)

比如说如果原来的动画第一帧会让角色出现T字型，则可以把动画序列帧的第一帧删掉，就可以让动画恢复正常了。

**关于关键帧的位置和数值修改，更好的方式是在Blender或Maya当中进行，但基本原理是一样的（这里只是方便起见在Unity里面进行微调）**。



##### （b）修改Rig

在前文有提及过：

- 如果要全部使用DAZ的系统，则Rig需要修改为Generic，并采用状态机来更改动画；
- 如果要用Unity的骨骼系统并重定向，则Rig需要修改为Humanoid，最好也用状态机来实现修改动画。

下面我们以Humanoid类型为例，来说明Rig的修改是如何进行的：（Generic情况下不需要重定向骨骼，比较简单，就不整理了）

（i）首先，修改人物，动画等资产的Rig为Humanoid：

<img src="Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310203808042.png" alt="image-20230310203808042" style="zoom:80%;" />

此时正常来说是不会报错的，如果报错的话可能原因有骨骼命名重复，根节点不符合规范等原因。



（ii）点击Configure，就可以对角色的骨骼进行重定向了：

![image-20230310204001440](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310204001440.png)

如上图，需要把DAZ人物（图是原神迪卢克的图，但是原理是一样的）的骨骼重定向到Unity的骨骼，也就是箭头指向的右侧区域。一般来说这一个步骤是可以自动匹配的，但是有的时候需要进行手动调整（比如说Unity默认绑定的骨骼不正确的情况）。同时，动画资产和Mixamo上下载的动画资产也都需要重定向到Unity的Humanoid标准骨骼上。此时不难想到，**由于重定向的存在，人物和动画都适配了同一套骨骼**，因此骨骼动画也是可以正常完成的。



##### （c）状态机说明

这一部分如果对Unity引擎比较熟悉的话不需要过多介绍了，大概就是通过Unity自带的状态机来转换动画（**其实也可以用比较复杂一些的有限状态机来做，但这里不过多展开**）。状态机和代码如下：

<img src="Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310204618309.png" alt="image-20230310204618309" style="zoom:80%;" />

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
> ![image-20230310205404630](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230310205404630.png)
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



------

### 3.换装系统

参考教程：**https://moecia.github.io/2021/05/05/UnityChangeOutfit-copy/**

https://docs.unity3d.com/cn/2020.2/Manual/class-SkinnedMeshRenderer.html

**基本思路是基于骨骼点替换的换装系统**。

> 为了让播放动画的时候，人物服装跟着人物一起在动，即有蒙皮，以下使用4个C#脚本实现**将服装的模型的骨骼点 替换成身体模型的骨骼点**，这样unity在播放它的身体动画的时候，服装也会一起做动画。（此时服装模型的SkeletonMeshRenderer里的骨骼就是身体模型的骨骼，因此可以跟着身体骨骼一起动）。

#### （1）基本换装系统的实现

以下为基本换装系统实现所需的4个C#脚本和对应的解释：

###### SkinnedMeshHelper.cs

这个class主要就是按照上面提到的思路进行骨骼点查找，然后返回一个新的骨骼点array供替换。

```C#
using System.Linq;
using UnityEngine;

/// <summary>
/// 进行骨骼点查找，然后返回一个新的骨骼点array供替换。
/// </summary>
public static class SkinnedMeshHelper
{
    public static Transform[] GetNewBones(SkinnedMeshRenderer root, SkinnedMeshRenderer source)
    {
        //复习lambda表达式 s => s.name   传入s传出s.name
        return root.bones
            .Where(x => source.bones.Select(s => s.name).Contains(x.name)).ToArray();
        //整个函数传出的应该是root.bones的子集,这里的子集就是人物的骨骼和衣服的骨骼的共同骨骼子集
    }
}
```

###### Outfit.cs

在数字人的换装系统中，我们实现了衣服，裤子，鞋，头发四种类型的服装，这里需要对每个由DAZ导入Unity的服装都绑定一个`Outfit.cs`，这个脚本放在服装的prefab上，需要设定好OutfitType（Hair,Cloth,Pant,Shoes）后放在文件目录Resources/Outfit/{OutfitType}/下。文件名和**Id**保持一致。（这样做是方便自动加载服装资源，并通过button实现换装效果，具体可以参考下面的代码）：

```C#
using UnityEngine;
/// <summary>
/// 位置：
/// 这个class放在服装的prefab上，
/// 设定好OutfitType（Hair,Cloth,Pant,Shoes）后放在Resources/Outfit/{OutfitType}/下。
/// 文件名和**Id**保持一致。（1234...）
/// 
/// 代码：
/// 获得这个预制体上的SkinnedMeshRenderer
/// </summary>
public class Outfit : MonoBehaviour
{
    [SerializeField] private OutfitType outfitType;
    private SkinnedMeshRenderer skinnedMeshRenderer;

    public OutfitType OutfitType { get => outfitType; set => outfitType = value; }
    public int Id { get => int.Parse(this.name); } //获取预制体名字 比如1,2,3,4,...
    public SkinnedMeshRenderer SkinnedMeshRenderer
    {
        get
        {
            if (skinnedMeshRenderer == null)
            {
                //获得这个预制体上的SkinnedMeshRenderer
                skinnedMeshRenderer = this.GetComponentInChildren<SkinnedMeshRenderer>();
            }
            return skinnedMeshRenderer;
        }
    }
}

public enum OutfitType
{
    Hair,
    Cloth,
    Pant,
    Shoes
}
```

###### EquipmentManager.cs

这个类用于换装前端逻辑，使用前先在人物身上放4个Slot结点，分别对应头，身，腿和脚。并将这四个Slot结点拖拽到这个脚本上。

Slot结点如下图所示：

![image-20230311000509280](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311000509280.png)

书写脚本如下：

```c#
using UnityEngine;
/// <summary>
/// 这个类用于换装前端逻辑，使用前先在人物身上放4个Slot，分别对应头，身，腿和脚。
/// 
/// 代码：加载资源，从文件夹中读取相应资源，
/// </summary>
public class EquipmentManager : MonoBehaviour
{
    [SerializeField] private Transform hairSlot;
    [SerializeField] private Transform clothSlot;
    [SerializeField] private Transform pantSlot;
    [SerializeField] private Transform shoesSlot;
    [SerializeField] private SkinnedMeshRenderer avatarSkinnedMesh;

    public Transform HairSlot { get => hairSlot; }
    public Transform ClothSlot { get => clothSlot; }
    public Transform PantSlot { get => pantSlot; }
    public Transform ShoesSlot { get => shoesSlot; }

    public int HairId { get; set; }
    public int ClothId { get; set; }
    public int PantId { get; set; }
    public int ShoesId { get; set; }

    public void LoadEquipment()
    {
        ChangeOutfit(OutfitType.Hair, 1); //hair 暂时作为装饰，后面会做头发相关的说明
        ChangeOutfit(OutfitType.Cloth, 1);
        ChangeOutfit(OutfitType.Pant, 1);
        ChangeOutfit(OutfitType.Shoes, 1);
    }

    public void ChangeOutfit(OutfitType outfitType, int outfitId)
    {
        GameObject outfit = null;
        Transform target = null;
        //outfitId = outfitId % 2+1;
        switch (outfitType)
        {
            case OutfitType.Hair:
                outfit = Resources.Load<GameObject>($"Outfit/Hair/{outfitId}"); //通过Resources.Load加载资源
                target = hairSlot;
                if (hairSlot.childCount > 0)  //角色身上已经穿上了对应位置的衣服，则将原来的衣服销毁掉
                {
                    Destroy(hairSlot.GetChild(0).gameObject);
                }
                HairId = outfitId;
                break;
            case OutfitType.Cloth:
                outfit = Resources.Load<GameObject>($"Outfit/Clothes/{outfitId}");
                target = clothSlot;
                if (clothSlot.childCount > 0)
                {
                    Destroy(clothSlot.GetChild(0).gameObject);
                }
                ClothId = outfitId;
                break;
            case OutfitType.Pant:
                outfit = Resources.Load<GameObject>($"Outfit/Pants/{outfitId}");
                target = pantSlot;
                if (pantSlot.childCount > 0)
                {
                    Destroy(pantSlot.GetChild(0).gameObject);
                }
                PantId = outfitId;
                break;
            case OutfitType.Shoes:
                outfit = Resources.Load<GameObject>($"Outfit/Shoes/{outfitId}");
                target = shoesSlot;
                if (shoesSlot.childCount > 0)
                {
                    Destroy(shoesSlot.GetChild(0).gameObject);
                }
                ShoesId = outfitId;
                break;
        }
        
        var outfitObj = Instantiate(outfit, target);
        //得到服装的SkinnedMeshRenderer，每一个导入的服装都需要绑定Outfit脚本，即指定服装类型的脚本
        var smr = outfitObj.GetComponent<Outfit>().SkinnedMeshRenderer;
        //去获取服装骨骼和人物骨骼相匹配的地方
        var bones = SkinnedMeshHelper.GetNewBones(avatarSkinnedMesh, smr);
        //将服装骨骼替换为人物骨骼，核心代码
        smr.bones = bones;
    }
}
```

###### ChangeOutfitController.cs

换装演示场景的UI控制逻辑。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
/// <summary>
/// 换装演示场景的UI控制逻辑。
/// </summary>
public class ChangeOutfitController : MonoBehaviour
{
    [SerializeField] private Button prevHair;
    [SerializeField] private Button nextHair;
    [SerializeField] private Button prevCloth;
    [SerializeField] private Button nextCloth;
    [SerializeField] private Button prevPant;
    [SerializeField] private Button nextPant;
    [SerializeField] private Button prevShoes;
    [SerializeField] private Button nextShoes;

    [SerializeField] private EquipmentManager equipmentMgr;

    private int currClothIndex = 1;
    private int currHairIndex = 1;
    private int currPantIndex = 1;
    private int currShoesIndex = 1;

    private int hairSize = 0;
    private int clothesSize = 0;
    private int pantsSize = 0;
    private int shoesSize = 0;
    // Start is called before the first frame update
    void Start()
    {
        equipmentMgr.LoadEquipment();

        prevHair.onClick.AddListener(() => { ChangeOutfit(OutfitType.Hair, false); });
        nextHair.onClick.AddListener(() => { ChangeOutfit(OutfitType.Hair, true); });
        prevCloth.onClick.AddListener(() => { ChangeOutfit(OutfitType.Cloth, false); });
        nextCloth.onClick.AddListener(() => { ChangeOutfit(OutfitType.Cloth, true); });
        prevPant.onClick.AddListener(() => { ChangeOutfit(OutfitType.Pant, false); });
        nextPant.onClick.AddListener(() => { ChangeOutfit(OutfitType.Pant, true); });
        prevShoes.onClick.AddListener(() => { ChangeOutfit(OutfitType.Shoes, false); });
        nextShoes.onClick.AddListener(() => { ChangeOutfit(OutfitType.Shoes, true); });
        
        string[] dirsHair = System.IO.Directory.GetFileSystemEntries("Assets/Resources/Outfit/Hair"); //获取路径下的所有文件名
        hairSize = dirsHair.Length/2; //之所以要÷2，应该是因为有meta文件
        string[] dirsPants = System.IO.Directory.GetFileSystemEntries("Assets/Resources/Outfit/Pants");
        pantsSize = dirsPants.Length / 2;
        string[] dirsClothes = System.IO.Directory.GetFileSystemEntries("Assets/Resources/Outfit/Clothes");
        clothesSize = dirsClothes.Length / 2;
        string[] dirsShoes = System.IO.Directory.GetFileSystemEntries("Assets/Resources/Outfit/Shoes");
        shoesSize = dirsShoes.Length / 2;
        //Debug.Log($"hairSize++{hairSize}");
    }

    private void ChangeOutfit(OutfitType outfitType, bool isNext)  //如果点击“下一个”，则切换到下一个服装，否则切换到上一个服装
    {
        switch (outfitType)
        {
            case OutfitType.Hair:
                if (isNext)
                {
                    currHairIndex = currHairIndex < hairSize ? ++currHairIndex : 1;
                }
                else
                {
                    currHairIndex = currHairIndex > 1 ? --currHairIndex : hairSize;
                }
                equipmentMgr.ChangeOutfit(outfitType, currHairIndex);
                break;
            case OutfitType.Cloth:
                if (isNext)
                {
                    currClothIndex = currClothIndex < clothesSize ? ++currClothIndex : 1;
                }
                else
                {
                    currClothIndex = currClothIndex > 1 ? --currClothIndex : clothesSize;
                }
                equipmentMgr.ChangeOutfit(outfitType, currClothIndex);
                break;
            case OutfitType.Pant:
                if (isNext)
                {
                    currPantIndex = currPantIndex < pantsSize ? ++currPantIndex : 1;
                }
                else
                {
                    currPantIndex = currPantIndex > 1 ? --currPantIndex : pantsSize;
                }
                equipmentMgr.ChangeOutfit(outfitType, currPantIndex);
                break;
            case OutfitType.Shoes:
                if (isNext)
                {
                    currShoesIndex = currShoesIndex < clothesSize ? ++currShoesIndex : 1;
                }
                else
                {
                    currShoesIndex = currShoesIndex > 1 ? --currShoesIndex : clothesSize;
                }
                equipmentMgr.ChangeOutfit(outfitType, currShoesIndex);
                break;
        }
    }
}
```

总结一下脚本的调用顺序：

> 当按下上一个或者下一个的按钮时，首先调用`ChangeOutfitController`脚本的`ChangeOutfit`函数，在这里会根据计算后的服装索引值调用`EquipmentManager`脚本的`ChangeOutfit`函数，这个函数会通过`Resource.Load`方法将文件夹下的对应预制体生成在前面设置的Slot子节点下，并调用每个服装绑定的`Outfit`脚本，从而获取到对应服装的`SkinnedMeshRenderer`。最后，调用`SkinnedMeshHelper`脚本的方法将对应服装`SkinnedMeshRenderer`中的骨骼替换为角色的骨骼，**这样衣服就可以随着角色的骨骼一起做动画了，这就是服装匹配骨骼动画的一种实现方式。**

通过以上的四个脚本，就可以实现换装的相关逻辑，**且具有一般的可拓展性**：

- 如果要引入新的服装类型，只需要添加一种服装的枚举值，并且添加一个新的Slot结点作为人物的子节点，然后在对应脚本中加入新的类型的逻辑判断即可。



------



## 三、AI相关模块驱动

在这里，一些比较基础的内容就不整理了，包括conda环境配置，以及基础的conda create，conda list等命令。



### 1.Unity与Python之间的交互

在做这个模块之前，注意先阅读总结部分遇到的问题[Python解决输出流信息显示不全的问题](#（1）输出流信息显示不全)



#### （1）yml文件配置Python开发环境

参考教程：[python基础----Conda环境管理、yml依赖安装python环境、pip依赖安装python环境 - 回溯法 - 博客园 (cnblogs.com)](https://www.cnblogs.com/guopinghai/p/11087988.html#:~:text=使用yml文件安装python环境（github上、或环境配置文件，这是使用conda来安装环境的，还有一种方法是使用pip，具体看下面即可） conda env,create -f environment.yml)

以下以更新环境为例，首先进入Anaconda Prompt，然后依次输入如下指令：

```cmake
conda env list
conda activate digitalhuman_train  # 如果还没有环境，就create一个然后activate即可
e:
cd E:\mihoyo\nielian\AIModel\new_AI_20230228\dll_digital\model  # 切换到提供的yml文件所在的路径位置
conda env update -f environment.yml  # 根据更新后的yml文件更新对应的环境，如果第一次安装可以用conda env create -f environment.yml
```

**如果配置失败的话检查一下网络相关的问题（可以用清华源解决或者直接VPN）**，配置成功之后的截图如下：

![image-20230311100656773](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311100656773.png)

此时打开PyCharm，新建或打开一个项目，选择环境为刚才配置好的或更新好的环境（如果yml文件提供了环境，比如上例中的voca（对应yml文件第一行会说：`name voca`），就切换到voca环境），如果在前面配置环境的时候报错了可以尝试直接在Pycharm里面安装缺失的包（或者用pip再次安装）。配置完开发环境之后，理论上就可以运行对应的py文件了。

> **补充：关于Crypto包**：
>
> 如果yml文件当中有Crypto包的话在配置环境的时候大概率会出现问题，由于出现的问题可能有很多种，因此擅用搜索引擎搜索报错关键词是很有必要的。这里列举我们遇到的一个问题：
>
> ![image-20230311101348039](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311101348039.png)
>
> 经过检索和学长的点播，以下是查询到的解决方案：https://www.cnblogs.com/chenjw-note/p/16898540.html
>
> 按照解决方案修改完之后，就可以正常的配置好环境了。
>
> 
>
> **再补充:Crypto包是什么？**
>
> [Python crypto模块实现RSA和AES加密解密 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1794013)
>
> [python 利用Crypto进行AES解密&加密文件 - 简书 (jianshu.com)](https://www.jianshu.com/p/5b38b4187b54)
>
> 
>
> **关于Crypto包在运行时出现的问题：**
>
> ![image-20230311105903465](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311105903465.png)
>
> 参考的解决方案链接如下：https://blog.csdn.net/qq_43186986/article/details/106077440
>
> 根据上面的链接修改完之后，就可以运行对应的py文件了。



#### （2）pyinstaller将Python文件打包成exe可执行文件

Pyinstaller是一个可以将Python程序打包成可执行的exe文件的包,具体可以参考这篇文章：https://blog.csdn.net/ZeroSwift/article/details/114778917。

而pyinstaller的官方文档如下:[Using PyInstaller — PyInstaller 5.8.0 documentation](https://pyinstaller.org/en/stable/usage.html#options)

首先，使用pip在对应环境下安装Pyinstaller包：

```cmake
pip install pyinstaller
# 如果因为网络原因导致安装报错，可以尝试用清华源解决：
# pip install pyinstaller -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装完毕之后，可以通过以下指令检查是否配置好pyinstaller：

```cmake
pip list
```

如果在列表当中找到了pyinstaller，则说明配置成功了。需要说明的是，如果我们要某个环境来使得pyintaller可以根据该环境build出对应的exe文件，**则需要把对应环境的site-packages文件加入到环境变量中**，具体如下：

![image-20230311115957629](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311115957629.png)

（注：以防万一，可以同时把环境变量加入到用户变量和系统变量中）

接下来的步骤是通过pyinstaller把py文件以及对应的依赖包一起打包成exe文件，这样在接下来的步骤中就可以提供给Unity去调用。打包指令如下：

```cmake
pyinstaller -F run.py  # run.py是要打包的文件，理论上会将依赖的包全部打包出来，除了-F还有别的参数，具体可以看上面给出的链接
```

成功打包之后的命令提示如下：

![image-20230311115753824](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311115753824.png)

在成功打包之后，对应的待打包文件的目录下面就会出现这三个文件夹：

![image-20230311095519840](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311095519840.png)

其中，dist文件夹里面有一个exe文件，即我们可以执行的exe文件。

**依然是Crypto造成的问题:**

> 在打包成exe文件之后，运行可能会报没有Crypto的问题（如果程序有用到Crypto的加密的话），此时的解决方案可以参考这篇博客：
>
> [用pyinstaller打包exe注意事项 - 简书 (jianshu.com)](https://www.jianshu.com/p/ea3ceac38ea5)
>
> - 原因主要是pyinstaller库延续开发的人漏了Crypto相关包的hook脚本，**因此需要我们手动在对应文件夹添加这个脚本**，脚本的内容参考上述链接。
>
> 在添加完脚本之后，用下面的指令重新打包出exe文件：
>
> ```cmake
> pyinstaller -F run.py
> ```

在本例中，如果成功build出来了可执行的exe文件，则运行该exe文件的输出结果应该是执行脚本后的结果，如下（**这里输出不全的问题可以通过[Python解决输出流信息显示不全的问题](#（1）输出流信息显示不全)来解决**）：

![image-20230311120226843](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230311120226843.png)

这篇文档后面会介绍如何在Unity当中调用exe文件，从而获取到输出的方法。

------



#### （3）在Unity调用可执行exe文件

现在，我们可以调用的exe文件目录结构如下：

![image-20230312150055093](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312150055093.png)

其中，`test_sentence.wav`是一个音频文件，也就是待呈现BS结果的文件，而model是程序运行所需要的模型文件（会从该文件中读取模型数据）。如果我们把这三个文件移植到别的位置，经过测试只要三者的层级结构不变，点击`run.exe`依旧是可以运行的。因此，我们将这三个文件移植到Unity的项目目录下，比如这里：

![image-20230312151547610](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312151547610.png)

**接下来，要考虑的问题就是如何在Unity当中去调用这个exe文件了。**注意，这里为了方便起见，将待测试的文件名统一为test_sentence，否则的话需要以命令行的方式向Python传入对应的音频文件参数，可以做但是暂时先不弄了。



**在Unity当中调用外部exe文件的方法：Process**

可以参考的文档：[C#执行EXE文件与输出消息的提取操作_C#教程_脚本之家 (jb51.net)](https://www.jb51.net/article/209827.htm)

根据上面的参考，可以写出下面的代码：

```c#
public void RunExeThread()
    {
        //https://blog.csdn.net/u012719718/article/details/53358331
        // https://www.jianshu.com/p/9bf35dbdbf25
        //string _exePathName = @"E:/mihoyo/nielian/AIModel/new_digital/dist/run.exe";
        //string fileDirectory = @"E:/mihoyo/nielian/AIModel/new_digital/dist/";

        string _exePathName = Path.GetFullPath(@"Assets/AIModel/dist/run.exe"); //https://learn.microsoft.com/zh-cn/dotnet/api/system.io.path.getfullpath?view=net-7.0，获取相对路径表示的绝对路径
        string fileDirectory = Path.GetFullPath(@"Assets/AIModel/dist/");
        try
        {
            Process myprocess = new Process();  
            ProcessStartInfo startInfo = new ProcessStartInfo(_exePathName);  
            myprocess.StartInfo = startInfo;
            myprocess.StartInfo.WorkingDirectory = fileDirectory;  //注意要切换到对应的fileDirectory里面
            myprocess.StartInfo.UseShellExecute = false; //不启用Shell启动进程
            myprocess.StartInfo.CreateNoWindow = true;  //设置为true则不创建新窗口,在测试的时候可以创建新窗口(设置为false),但是打包出来应该设置为true
            myprocess.StartInfo.RedirectStandardOutput = true;  //重定向到标准输出  https://www.jb51.net/article/209827.htm
            myprocess.Start();
            string output = myprocess.StandardOutput.ReadToEnd();
            myprocess.StandardOutput.ReadToEnd();
            print(output);
            
            // myprocess.BeginOutputReadLine();
            // myprocess.OutputDataReceived += new DataReceivedEventHandler(ReceiveHandler);
        } catch (Exception ex) {  
            UnityEngine.Debug.Log("出错原因：" + ex.Message);  
        } 
    }
```

制作一个Button，将上面的函数设置为该Button的回调函数，在点击Button之后等待一段时间，待程序执行完毕后输出的结果如下：

![image-20230312154555195](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312154555195.png)

可以看到，程序跑完之后的结果是可以被我们接收到的，而这个字符串可以用来接下来用于解析并呈现BS变换的结果，具体可以参考接下来的文档部分。

**注：虽然Unity现在可以直接获取到exe文件的输出结果**，但**根据实验，最好还是让Python可执行文件exe直接输出一个txt文本文件用于保存运行的结果以方便后面的解析**，这是因为numpy在输出到文件中的时候，会自动把控制台的输出模式转换成方便程序解析的形式，具体可以看下图：

![image-20230312194825693](Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312194825693.png)

在后面的解析中，我们的基准是左侧的这种输出流形式，因为这种比较好解析。



------

**存在的问题：**

如果我们用Button的回调函数来调用上述的代码逻辑，会发现在按下Button之后Unity在程序执行完毕之前就不会再执行其他的游戏逻辑了，**这是由于主程序被阻塞了。**直接用回调函数调用Python程序会造成Unity主线程的阻塞现象。

针对上面的问题，解决方案也比较简单，使用Thread来新开一个线程即可。

```c#
using System.Threading;
public void RunExeFile()
{
	Thread thread = new Thread(new ThreadStart(RunExeThread));  //RunExeThread就是上面写的调用exe文件的函数，需要把RunExeThread函数改为static
     thread.Start();        
}
```

当用户按下Button的时候，系统会新建一个线程并执行相关的逻辑，此时新的线程就不会干扰主线程了，其他逻辑得以正常进行。



**再次升级：子线程执行完通知主线程**：

在实际与用户交互的过程中，我们会希望当exe文件执行完毕之后可以通知用户已完成（而不是直接用print的方式），这里初步计划是在执行完程序之后让一个新的Button显示出来，从而告知用户已经训练完成。**此时的技术点就在于如何让子线程执行完通知主线程？**

> 注意：**在Unity中，子线程是无法调用Unity主线程的API的，因为unity是单线程引擎。**也就是说，如果想在子线程里调用Unity的SetActive是不被允许的，因此**此时的解决方案是传统的Update轮询**。在子线程中修改某个变量的值，并在Unity的Update函数中每隔一段时间（比如5s）轮询一次变量是否被修改，如果已经修改则显示出新的Button，表示exe程序已经执行完成。
>
> （**理论上要考虑多线程上锁的问题，这里经过测试不上锁也是可以跑通的，因此这里就不上锁了。**）
>
> **补充：一开始有尝试使用纯用委托和不用Update轮询的方式来做，参考(https://blog.csdn.net/WuLex/article/details/88619805)，虽然这种方法在Unity里这是不能实现的，但对于传统的C#代码还是具有一定的指导意义。**
>
> **补充2：**还有一种方法，暂时没有尝试：即**利用 C# 的委托功能**，将子线程回调主线程的函数以事件形式注册到委托上，主线程使用一个管理类，按一定频率去查询尚未执行的子线程事件，并依次执行。

这里采用`注意部分`提到的的逻辑，具体的代码如下：

首先，声明一个变量用于监控子线程是否执行完成，并在子线程执行结束后修改该字段的值，在Update中每间隔一段时间（这里设置2s）轮询一次是否运行完成，如果运行完成后主线程修改变量为false并将对应的可交互按钮的可见性设置为true：

```c#
public Button showVoiceResBtn;
private static bool voiceTrainIsDone = false;
public float checkInternal = 2.0f;
private float checkRemainTime;
private static string runResStr;

void Start()
{
    showVoiceResBtn.gameObject.SetActive(false);
    voiceTrainIsDone = false;
    checkRemainTime = checkInternal;
}

private void Update()
{
    //每过两秒检查一下是否已经运行完成，运行完成之后将对应UI显示
    checkRemainTime -= Time.deltaTime;
    if (checkRemainTime <= 0)
    {
        checkRemainTime = checkInternal;
        if (voiceTrainIsDone)
        {
            showVoiceResBtn.gameObject.SetActive(true);
            voiceTrainIsDone = false;
        }
    }
}

static void RunExeThread()  //用于运行exe文件的对应线程，在这里会将字段设置为true
{
    //...
    voiceTrainIsDone = true;
    runResStr = output; //存储输出结果
}

```

这样，我们后续的处理运行结果的模块只需要在字段被置为true之后调用即可。

------



### 2.语音驱动模块

#### （1）AI训练部分

在AI训练的时候，参考的链接如下：

https://github.com/TimoBolkart/voca

> 我们是先提取音频特征，然后通过网络估的bs系数，有一个仓库可以看看，他这个是用的FLAME人脸模型，我们换成了daz那个，这个仓库输出的是所有的顶点位置，就是他用tensorflow1.0写的代码——来自学长

训练之后得到的结果应该是一个二维的numpy数组，其中数组的第一维是视频每一帧的数据，第二维则是每一种BS的值，接下来会在Unity当中使用训练AI得到的BS结果并呈现给用户。**具体的训练过程现在了解的还不是很清晰，后面学完了再来补充。**



#### （2）AI训练结果解析并呈现

在Unity当中调用上面所描述的exe文件，20秒钟的音频文件的处理速度大概是28秒左右，这个速度还算是比较合理的。接下来主要的任务是将得到的BS结果解析出来，并呈现在屏幕上。

##### （a）将输出字符串解析成数组并保存

当程序执行完毕之后，会生成一个txt文件，该文件如下：

<img src="Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312195446434.png" alt="image-20230312195446434" style="zoom: 67%;" />

对这个文件进行解析，将所有的数据存储于二维数组中，数组的第一维是每一帧的数据，数组的第二维是每一帧的每一种BS的对应值，其中第二维的索引分别对应下面的BS值（只截取部分，比如说第一列就对应下面的0这一项）：

```c++
0,Cheeks Balloon-Suck In Left.obj
1,Cheeks Balloon-Suck In Right.obj
2,Eye Slide-side left.obj
3,Eye Slide-side.obj
4,Eyelids Lower Up-Down left.obj
5,Eyelids Lower Up-Down right.obj
6,Eyelids Upper Up-Down left.obj
7,Eyelids Upper Up-Down right.obj
//...
```

解析文件的函数如下：
```c#
/// <summary>
/// 这个二维的List的每一行是一帧的数据,每一列是每一个BS的数据
/// </summary>
public List<List<double>> valueArray = new List<List<double>>();

private int BSSize;  //每一帧包含BS的数量
public int frameSize; //帧数

private void Start()
{
    ReadBSIndexFile();
}

/// <summary>
/// 读取所有的BS名字,确认共有多少个BS
/// </summary>
private void ReadBSIndexFile()
{
    string path = Application.dataPath + "/readFiles/txtFiles/expression_index.txt";
    string[] strs = File.ReadAllLines(path);
    BSSize = strs.Length;
    print(BSSize);
}

public void ReadAIAfterRunTxtFile()  //这个函数会在显示运行完成之后的按钮被按下的时候调用，valueArray则会保存转换后的所有运行数据
{
    valueArray.Clear();
    string path = Application.dataPath + "/AIModel/dist/test_sentence.txt"; //暂时写死了，因为生成的txt文件一定会在这个路径下面
    string[] strs = File.ReadAllLines(path);
    frameSize = strs.Length;
    foreach (string item in strs)
    {
        string tmpItem = item;
        List<double> tmpArray = new List<double>();
        string[] splitValues = tmpItem.Split(' ');
        Assert.AreEqual(splitValues.Length, BSSize);
        foreach (var word in splitValues)
        {
            string tmpWord = word;
            double value = double.Parse(tmpWord);
            tmpArray.Add(value);
        }
        valueArray.Add(tmpArray);
    }
    print("=========txt文本文件读取完毕======");
    print("该文本文件一共包含" + frameSize.ToString() + "帧的内容");
    print("共计要处理的BS数量为:" + BSSize.ToString());
}
```

调用`ReadAIAfterRunTxtFile()`函数结束后，`valueArray`数组当中就会存储所有帧的所有BS数据。



##### （b）根据数组的值将BS的值呈现出来

首先，当点击显示结果的按钮的时候，应该要获取到待播放音频的所有BS数据，以及音频的相关信息，因此当按下按钮之后的回调函数书写如下：

```c#
public void PlayAIRunResult()  //按下按钮的回调函数
{
    fileInfo.ReadAIAfterRunTxtFile();
    if(music.isPlaying) music.Stop();
    music.clip = AIsourceClip;  //利用audioSource切换audioClip为测试的audioClip

    frameSize = fileInfo.frameSize;
    BSValues = fileInfo.valueArray;  //此时就拿到了对应的所有数据
    playIndex = 0;  //这个用来控制应该播到第几帧
    music.Play();
    startPlay = true;
}
```

此时我们就可以在FixedUpdate函数里去控制模型的BS的值从而实现说话效果了。

```c#
void FixedUpdate()
{
    if (startPlay)
    {
        bsChange.ChangeBSgroups(BSValues[playIndex]);  //核心代码，提取出每一帧的各个BS值并在Unity中呈现
        playIndex++;
        if (playIndex == frameSize)
        {
            startPlay = false;
            Debug.Log("end animation");
        }
    }
}
```



**根据数组的值呈现BS的值的代码逻辑：**



**还有眨眼的逻辑要写**

**剩下的这些后面再补上去吧,今天先到这里**



#### （3）值得优化的点





## 四、总结

### 1.数字人工作流

在现在的工作流下，假设现在我们用DAZ软件制作数字人并准备在Unity里实现动画系统，换装系统和AI语音驱动模块的话，只需要做下面几件事：

#### （1）非AI模块



#### （2）AI模块

假设现在我们已经有了一个可以跑的`run.py`文件。首先，用pyinstaller将python程序打包成exe文件，然后把包含python可执行exe文件的dist文件夹放入`UnityAssets/AIModel`目录下（后面修改exe文件之后也是同理）。

把要测试的wav音频也放入到`UnityAssets/AIModel/dist`文件里，运行Unity程序，点击“测试驱动Python”，待运行完成后会出现“显示训练结果！”的按钮，点击即可查看结果。



### 2.遇到的问题和解决方案

#### （1）输出流信息显示不全

在调用完Python打包出来的exe文件并将对应的结果写入txt文件之后，发现txt文件里内容如下：

<img src="Unity%20%E6%95%B0%E5%AD%97%E4%BA%BA%E6%B5%81%E7%A8%8B.assets/image-20230312192820261.png" alt="image-20230312192820261" style="zoom:67%;" />

也就是说，由于Unity这边会获取exe文件执行得到的输出流，而由于Python输出的时候是带有省略号...的（因为字符串过长），所以自然在读取输出流的时候把省略号读进去了，因此无法获得所有的数据。解决方案有两种：

- （**推荐**）使得exe可执行文件在执行之后直接把运行结果写入到文件当中，这样在读取的时候就不需要让缓冲区被大量的字符所占领，减少缓冲区的压力。

- 如果需要在执行exe文件的时候直接拿到结果，则可以用如下方式强制让Python输出全部的numpy数组信息，然后再存储到文件当中：

  ```python
  import numpy as np
  np.set_printoptions(threshold=np.inf)
  ```

两种方法都可以解决最终文件只有省略号的问题。解决之后的文件内容如下：

