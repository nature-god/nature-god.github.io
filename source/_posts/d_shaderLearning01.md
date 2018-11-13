---
title: Unity Shader入门(一)
date: 2018-11-09
tag: Unity Shader
---
这是自己开的第一篇技术帖，主要讲述下基本的shader入门知识。预计在这个寒假结束之前写完吧。
<br/>参考书籍：《Unity Shader入门精要》
<br/>Unity版本: Unity 2018.2.4f1
<br/>有人说过，程序员的三大浪漫是：编译原理，操作系统和计算机图形学。诚然，我认为这几门技术都是难度比较大的，或许再加上计算机网络与数据库系统？游戏中的美术是无比重要的，没有好的美术，纵使你的玩法再完美，受众玩家都不会多——毕竟控制台的游戏感官实在要差很多。美术只是贴图，而那些光照，反射与现实下的物理模拟，则是我们程序员的事情，shader便是解决这些问题的工具(当然，美术画出这些真实场景的情况也有——部分精致的动漫中，画师细节把握实在到位，观众往往会发出"啊，这光，啊这水"的惊讶感叹)，但是对实时要求极高的游戏而言，画师的画画速度则是远远不够了。
## 1.渲染流水线
渲染，顾名思义，将东西(我们定义好的信息)画(渲染)在纸(屏幕)上，这一部分要用到很多的数学计算，由CPU和GPU共同完成。
《Render-Time Rendering》中将渲染流程分为三个阶段：应用阶段(Application Stage),几何阶段(Geometry Stage)，光栅化阶段(Rasterizer Stage)。
简单而言:
>* 应用阶段：准备好要场景数据，包括摄像机位置，视锥体，光源等信息。就像是画画，先想一下自己想要画什么东西。(大脑(CPU)负责)
>* 几何阶段：应用阶段告诉了我们场景中有啥，而这些基本图形是点，线，三角形这些东西构成，几何阶段便是画出这些东西，但是没有颜色和其他信息，就像是一幅画的线稿一样。(速写笔(GPU)负责))
>* 光栅化阶段：根据上个几何阶段传递过来的数据，来产生屏幕上的像素，并最终渲染出最终的图像，就像画画，线稿画好了，根据我们最开始设定好的颜色，考虑下光照之类的信息，用我们的画笔给这副画上好色。(画笔(GPU)负责))

此外，我们场景中的东西可能很多，但能看到的只有摄像机视野范围那一块，因此，运用"裁剪"技术剔除掉那些不在视野范围内的物体。此外，我们如何把一个现实中的物体画到纸上呢？我们都知道透视几何画3D物体，像我们学过的立体几何那些。因此计算机渲染也需要这么个操作，将世界坐标映射到屏幕之上：
>屏幕映射(Screen Mapping)的任务是把每个物体的x和y坐标转换到屏幕坐标系(Screen Coordinate)下。屏幕坐标系是一个二维坐标系，与显示画面的分辨率有很大关系。物体的Z坐标不做处理，其代表深度值。

映射就涉及到坐标系的转换了，这部分是基本的数学操作。此外，开发者直接访问GPU是很麻烦的事情，因此使用一个中间的图像编程接口是十分有必要的，由于使用的屏幕坐标系可能不同，造成这部分的数学操作也不一样：
>* OpenGL
>* DirectX

## 2.Unity的 Material，Shader 和 Texture
Unity使用Materials，Shaders和Textures来进行渲染，它们之间联系紧密，最终渲染出美轮美奂的画面。
>* Material: Material通过它包含的贴图，布局与颜色等信息来定义一个表面应该如何被渲染。一个Material根据其使用的shader来使用不同的可选项。
>* Shader：Shader是一种基于光照输入和Material配置来进行数学运算并计算出每个像素点(pixel)渲染颜色的脚本。
>* Texture: Texture是一种位图图像。一个Material包含对textures的引用，所以Material的Shader就可以使用Texture去计算一个物体(GameObject)的表面颜色了。除了游戏对象表面的基本颜色(albedo)外，纹理还可以表示材质表面的许多其他方面，如反射率，粗糙度等等。

一个Material对应于一个Shader，这个Shader决定了这个Material种哪些可用的选项。一个Shader则对应了一个或多个Texture，在Unity的Inspector面板你可以直接拖拽Texture资源给Shader。总体而言，在Unity中，常见的使用流程是：
> 1 创建一个Material  
> 2 创建一个Unity Shader，并把它赋给上一步的Material  
> 3 把材质赋给要渲染的对象(GameObject)

在Unity中，上诉操作都可以在Unity菜单栏中选择Asset->Create->Shader/Material等实现。
<div align="left"><img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/shaderCapture.png"/></div>
其中有四种模板的Unity Shader供我们选择:

>* Standard Surface Shader：产生一个包含了标准光照模型的表面着色器模板
>* Unlit Shader：产生一个不含光照效果(但含雾效果)的基本的顶点/片元着色器
>* Image Effect Shader：提供一个屏幕后处理效果的基本模板
>* Computer Shader：特殊Shader文件，旨在利用GPU的并行性来计算一些与常规渲染流水线无关的计算

创建好Shader文件后，我们就可以开始我们的Unity Shader编程工作了。在Unity中，所有的Unity Shader都是由ShaderLib来编写的:

>ShaderLab是Unity提供的编写Unity Shader的一种说明性语言。它使用一些嵌套在花括号内部的语义(syntax)来描述一个Unity Shader文件的结构。

一个Unity Shader的基础结构如下所示：
```
Shader "ShaderName"{
    Properties {
        //Shader Properties
    }
    SubShader {
        //Subshader A
    }
    SubShader {
        //Subshader B
    }
}
```
Unity在背后会根据所使用的平台来把这些结构编译为真正的代码和Shader文件。

...待续

