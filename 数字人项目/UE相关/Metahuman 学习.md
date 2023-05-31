## Metahuman 学习

相关网站：[新版MetaHuman带来更便捷的共享和DNA校准 - Unreal Engine](https://www.unrealengine.com/zh-CN/blog/new-metahuman-release-brings-easier-sharing-and-dna-calibration)



## 一、准备工作

1. 先将Epic Games的账号与Github账号相关联，确保可以访问对应资源。参考链接：[GitHub上的虚幻引擎 - Unreal Engine](https://www.unrealengine.com/zh-CN/ue-on-github)
2. 下载Maya 2022，以及准备Python 3 的开发环境。
3. 



------



## 二、相关文档阅读

对应的仓库链接为：[EpicGames/MetaHuman-DNA-Calibration (github.com)](https://github.com/EpicGames/MetaHuman-DNA-Calibration)

------

### 1.DNA Calibration（DNA 校准库）

​		MetaHuman DNA校准是一套用于处理MetaHuman DNA文件的工具，捆绑在一个单独的包中。DNA是Metahuman身份的重要组成部分。DNA文件是用MetaHuman Creator创建的，可以在UE5中用Quixel Bridge和Bifrost下载。

​		上述的仓库链接分享的代码可以帮助用户定制DNA文件，这样就可以更好地将自己创造的角色整合到游戏当中。

------

#### （1）仓库结构

https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/docs/repository_organization.md

仓库包含两个独立的组件:

- dnacalib c++库：用于操作DNA文件
- dna_viewer python代码：用于在Autodesk Maya中可视化DNA

相关的文件结构如下：

- [dnacalib](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib)：DNACalib的源代码
- [dna_viewer](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dna_viewer)：DNA Viewer的源代码
- [examples](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/examples)：一些Python脚本，用来展示dna_viewer和DNACalib的Python wrapper的基本用法
- [lib](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/lib)：DNACalib, PyDNACalib和PyDNA的预构建二进制文件（针对Linux和Windows操作系统）。此外， a Maya plugin for RL4 is also available（**todo：具体还不知道是什么，后面补充**）.
- [data](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/data)：需要的DNA和Maya的场景。包含示例DNA文件。官方提供了两个Metahuman人类DNA文件(Ada和Taro，对应第一个预设)。

​		此外，data文件夹中添加了gui和analog_gui Maya场景，用于Maya场景组装。此外，`additional_assemble_script.py`用于组织场景中的对象和connect controls。理想的设置是这样的（**todo：暂时看不懂，后面补充**）:

![image](Metahuman%20%E5%AD%A6%E4%B9%A0.assets/aas.png)

- [docs](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/docs)：相关文档

具体的细节在后面会继续介绍。

------



### 2.DNACalib

参考：[MetaHuman-DNA-Calibration/dnacalib.md at main · EpicGames/MetaHuman-DNA-Calibration (github.com)](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/docs/dnacalib.md#python)

这个库用于对DNA文件执行修改。它是用c++编写的，还有一个Python包装器。[SWIG](https://www.swig.org/)库用于为Python生成绑定。DNACalib可以在命令行中使用，也可以在Maya中使用。仓库**提供了Windows和Linux的二进制文件**。如果您正在使用不同的体系结构和平台，则必须构建DNACalib。

> 补充SWIG：简单包装界面产生器（英语：Simplified Wrapper and Interface Generator, SWIG）是一个**开源软件工具**，用来**将C语言或C++写的计算机程序或函式库，连接脚本语言**，例如Lua, Perl, PHP, Python, R, Ruby, Tcl, 和其它语言，例如C#, Java, JavaScript, Go, D, OCaml, Octave, Scilab以及Scheme. 也可以输出成XML格式。

#### （1）DNACalib文件结构

- [`DNACalib`](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib/DNACalib) ：包括DNACalib的C++源代码和依赖项。有一个用于读取和写入DNA文件的库，以及一些其他实用程序库。
- [`PyDNACalib`](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib/PyDNACalib)： 包括生成针对DNACalib的Python包装器（Python wrapper）的源代码。
- [`PyDNA`](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib/PyDNA) ：包含为DNA库生成Python包装器的源代码，该库位于包含c++源代码的DNACalib文件夹下。
- [`SPyUS`](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib/SPyUS) - 包含PyDNACalib和PyDNA使用的一些常见SWIG接口文件。
- [`CMakeModulesExtra`](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/dnacalib/CMakeModulesExtra) - 包含一些在整个项目中使用的常见CMake函数，包括c++和Python包装器。

#### （2）

------



### 3.DNA文件

[MetaHuman-DNA-Calibration/dna.md at main · EpicGames/MetaHuman-DNA-Calibration · GitHub](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/main/docs/dna.md)

根据运行环境需求，这里需要Python3.7或者Python3.9版本，

> **补充示例：将DNA文件转为可以阅读的json文件**
>
> 这里暂时每太看懂怎么把对应的文件夹放置在环境变量里，因此采用的运行策略如下：
>
> - （1）lib/Maya2022/linux下的文件拷贝粘贴在examples文件夹里，确保运行的环境为Python 3.7；
> - （2）在examples文件夹下面输入命令行：`python dna_binary_to_json_demo.py`，即可运行并生成转化为json格式的DNA文件，方便阅读。
>   - 转换后的文件会出现在整个项目下的output文件夹里。

接下来对DNA里的内容进行解析：