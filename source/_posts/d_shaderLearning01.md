---
title: Unity Shader入门(一)
date: 2018-11-09
tag: Unity Shader
---
这是自己开的第一篇技术帖，主要讲述下基本的shader入门知识。预计在这个寒假结束之前写完吧。
<br/>参考书籍：《Unity Shader入门精要》
<br/>Unity版本: Unity 2018.2.4f1
<br/>有人说过，程序员的三大浪漫是：编译原理，操作系统和计算机图形学。诚然，我认为这几门技术都是难度比较大的，或许再加上计算机网络与数据库系统？游戏中的美术是无比重要的，没有好的美术，纵使你的玩法再完美，受众玩家都不会多——毕竟控制台的游戏感官实在要差很多。美术只是贴图，而那些光照，反射与现实下的物理模拟，则是我们程序员的事情，shader便是解决这些问题的工具(当然，美术画出这些真实场景的情况也有——部分精致的动漫中，画师细节把握实在到位，观众往往会发出"啊，这光，啊这水"的惊讶感叹)，但是对实时要求极高的游戏而言，画师的画画速度则是远远不够了。
### 1.渲染流水线
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
### 2.Unity Shader 与 ShaderLab
Unity使用Materials，Shaders和Textures来进行渲染，它们之间联系紧密，最终渲染出美轮美奂的画面。
>* Material: Material通过它包含的贴图，布局与颜色等信息来定义一个表面应该如何被渲染。一个Material根据其使用的shader来使用不同的可选项。
>* Shader：Shader是一种基于光照输入和Material配置来进行数学运算并计算出每个像素点(pixel)渲染颜色的脚本。
>* Texture: Texture是一种位图图像。一个Material包含对textures的引用，所以Material的Shader就可以使用Texture去计算一个物体(GameObject)的表面颜色了。除了游戏对象表面的基本颜色(albedo)外，纹理还可以表示材质表面的许多其他方面，如反射率，粗糙度等等。

一个Material对应于一个Shader，这个Shader决定了这个Material种哪些可用的选项。一个Shader则对应了一个或多个Texture，在Unity的Inspector面板你可以直接拖拽Texture资源给Shader。总体而言，在Unity中，常见的使用流程是：
> 1. 创建一个Material  
> 2. 创建一个Unity Shader，并把它赋给上一步的Material  
> 3. 把材质赋给要渲染的对象(GameObject)

在Unity中，上诉操作都可以在Unity菜单栏中选择Asset->Create->Shader/Material等实现。
<div style="text-align:left"><img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/shaderCapture.png"/></div>

其中有四种模板的Unity Shader供我们选择:
>* Standard Surface Shader：产生一个包含了标准光照模型的表面着色器模板
>* Unlit Shader：产生一个不含光照效果(但含雾效果)的基本的顶点/片元着色器
>* Image Effect Shader：提供一个屏幕后处理效果的基本模板
>* Computer Shader：特殊Shader文件，旨在利用GPU的并行性来计算一些与常规渲染流水线无关的计算

创建好Shader文件后，我们就可以开始我们的Unity Shader编程工作了。在Unity中，所有的Unity Shader都是由ShaderLib来编写的:
>ShaderLab是Unity提供的编写Unity Shader的一种说明性语言。它使用一些嵌套在花括号内部的语义(syntax)来描述一个Unity Shader文件的结构。
一个Unity Shader的基础结构如下所示：

``` C
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
Unity在背后会根据所使用的平台来把这些结构编译为真正的代码和Shader文件，而开发者只需要和Unity Shader打交道即可，这就像我们使用C/C++等编程语言，而编译器会编译为机器语言。在这里，我们使用Unity Shader进行编程，Unity将其翻译为真正的Shader文件。

在没有Unity这类编辑器的情况下，如果我们想要对某个模型设置渲染状态，需要十分复杂的代码，有了解过OpenGL的同学可能会有感受：
``` C++
void Initialization()
{
    //初始化渲染设置
    //从硬盘加载顶点着色器代码
    //从硬盘加载片元着色器代码
    //把顶点着色器加载到GPU
    //把片元着色器加载到GPU
    string vertexShaderCode = LoadShaderFromFile(VertexShader.shader);
    string fragmentShaderCode = LoadShaderFromFile(FragmentShader.shader);
    LoadVertexShaderFromString(vertextShaderCode);
    LoadFragmentShaderFromString(fragmentShaderCode);

    //设置模型顶点坐标
    //加载纹理图片
    //加载预处理的变换矩阵
    SetVertexShaderProperty("vertexPosition",vertices);
    SetVertexShaderProperty("MainTex",someTexture);
    SetVertexShaderProperty("MVP",MVP);

    //关闭混合
    Disable(Blend);
    //设置深度测试
    Enable(ZTest);
    SetZTestFunction(LessOrEqual);

    //其他设置
    ...
}

//每一帧进行渲染
void OnRendering()
{
    DrawCall();
    //其他渲染设置
    ...
}
```
上诉是渲染的流程，一般都用C/C++实现，而真正的shader文件则像下面样子:

顶点着色器 VertexShader.shader
``` C++
//输入:顶点位置，纹理，MVP变换矩阵
in float3 vertexPosition;
in sample2D MainTex;
in Matrix4x4 MVP;

//输出:顶点经过MVP变换后的位置
out float4 position;

void main()
{
    position = MVP * vertexPosition;
}
```
片元着色器 FragmentShader.shader
``` C++
//输入:VertexShader输出的position,光栅化程序插值后的position
in float4 position;

//输出:片元颜色值
out float4 fragColor;

void main()
{
    //片元设置为白色
    fragColor = float4(1.0,1.0,1.0,1.0);
}
```
这仅仅是超级简化版本了，除去许多冗长的函数名(函数功能还是很清晰的)之外，执行顺序，渲染数目的增多会让过程变得更加负责。幸运的是，使用Unity shader，我们便可以十分简单的实现上述的所有步骤了——这也就是抽象的好处："计算机中的任何问题都可以增加一层抽象来解决"。

从设计上而言，ShaderLab类似于CgFX和Direct3D Effects(.FX)语言。接下来，我们简要学习一下Unity Shader的基本结构，写出个"Hello Shader"来吧。
### 3.Unity Shader Structure
#### 3.1 Shader Name
就和所有的编程一样，首先我们要为Unity Shader文件中定义的Shader取一个名字：
```
Shader "Customer/MyShader"
{
    ...
}
```
这样，当Material需要使用该shader时，就可以在下拉列表内找到。
#### 3.2 Properties
Properties语义块包含了许多的属性(property)
```
Properties
{
    Name ("display name", PropertyType) = DefaultValue
    Name ("display name", PropertyType) = DefaultValue
    //更多属性
}
```
声明这些属性就像声明一个Public的变量一样，我们可以在材质面板中很方便地调整各种材质属性等。
<br/>Properties 中字段含义:
* Name 是我们在shader中要访问该变量时使用的名字，约定俗成一般以下划线开始
* display name 是出现在材质面板上的名字
* PropertyType 是属性的类型
* DefaultValue 是属性默认值

Properties所支持的PropertyType:

|属性类型|默认值的定义语法|例子
|:-|:-|:-|
|Int|number|_Int("Int",Int)=2
|Float|number|_Float("Float",Float)=1.5
|Range(min,max)|number|_Range("Range",Range(0.0,5.0))=3.0
|Color|(number,number,number,number)|_Color("Color",Color)=(1,1,1,1)
|Vector|(number,number,number,number)|_Vector("Vector",Vector)=(2,3,6,1)
|2D|"defaulttexture"{}|_2d("2D",2D)=""{}
|Cube|"defaulttexture"{}|_Cube("Cube",Cube)="white"{}
|3D|"defaulttexture"{}|_3D("3D",3D)="black"{}
<br/>对于Int,Float,Range这些属性，其默认值就是一个单独的数字
<br/>对于Color,Vector这些属性，默认值是一个四维向量
<br/>对于2D,Cube,3D这些属性，由一个字符串加{}组成，前面字符串要么为空，要么是内置的纹理名称，如"whit","black","gray"或"bump"这些，{}中指定一些纹理属性
<br/>最后我们可以得到一个超级简单的"Hello Shader"的例子：
``` C
Shader "Custom/NewSurfaceShader" {
    Properties {
        _Int("Int",Int) = 2
        _Float("Float",Float) = 1.5
        _Range("Range",Range(0.0,5.0)) = 1.5
        _Color("Color", Color) = (1,1,1,1)
        _Vector("Vector",Vector) = (2,3,6,1)
        _2D("2D",2D)=""{}
        _Cube("Cuben",Cube)="White"{}
        _3D("3D",3D)="black"{}
    }
    FallBack "Diffuse"
}
```
<br/>在Unity编辑器中如下图所示:
<div><img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Material_Shader_Show.png"/></div>

此外一些其他类型的变量若是想显示出来，Unity允许我们重载默认的材质编辑面板，流程与Customer Editor那套自定义Inspector的方式类似，只不过现在是Shader材质面板了，有兴趣了解的同学可以参考官方[Custome Shader GUI](http://docs.unity3d.com/Manual/SL-CustomShaderGUI)。
#### 3.3 SubShader
>每一个Unity Shader文件可以包含多个SubShader语义块，但至少要有一个。当Unity需要运行这个Unity Shader时，会依次扫描所有的SubShader，然后选择第一个可以在目标平台运行的SubShader。如果都不支持的话，会使用FallBack语义所指定的Unity Shader。

之所这么设计的原因，是因为用户的显卡能力不同，为提供更加流畅与精美的画面，对高性能显卡的用户就是用较高计算复杂度的着色器，低性能的就使用较低计算复杂度的着色器。

SubShader语义块形式如下：
``` C
SubShader {
    //可选
    [Tags]

    //可选
    [RenderSetup]

    Pass{}
    //other passes
}
```
字段描述：
* Pass: 每个Pass定义了一次完整的渲染流程,但如果Pass数量过多会造成渲染性能下降
* RenderSetup: 渲染状态设置指令，可以设置显卡的各种状态，例如是否开启混合/深度测试等，设置之后会作用于所有的Pass块，常见RenderSetup设置见下表。
* Tags: 标签(tags)时一个键值对，它的键和值都是字符串类型。常见Tags支持类型见下表。

常见的RenderSetup设置选项：

|状态名称|设置指令|解释
|:---|:---|:---
|Cull|Cull Back/Font/Off|设置剔除模式:剔除背面/正面/关闭剔除
|ZTest|ZTest Less Greater/LEqual/GEqual/Equal/NotEqual/Always|设置深度剔除时使用的函数
|ZWrite|ZWrite On/Off|开启/关闭深度写入
|Blend|Blend SrcFactor DstFactor|开启并设置混合模式

>当在SubShader中设置了上述渲染状态时，将会应用到所有的Pass上去。

Tags支持类型

|标签类别|说明|例子
|:---|:---|:---
|Queue|控制渲染顺序，可以指定该物体属于哪个渲染队列，通过这种方式可以保证所有透明物体都可以在不透明物体之后被渲染|Tags{"Queue"="Transparent"}
|RenderType|对着色器进行分类，例如这是个不透明的着色器，或是一个透明的着色器，可以用于着色器替换功能(Shader Replacement)|Tags{"RenderType"="Opaque"}
|DisableBatching|是否对该SubShader使用批处理|Tags{"DisableBatching"="True"}
|ForceNoShadowCasting|控制使用该SubShader的物体是否会投射阴影|Tags{"ForceNoShadowCasting"="True"}
|IgnoreProjector|控制使用该SubShader的物体是否会受到Projector的影响，通常用于半透明物体|Tags{"Ignore'Projector"="True"}
|CanUseSpriteAtlas|当该SubShader是用于精灵(sprite)时，将该标签设为"False"|Tags{"CanUseSpriteAtlas"="False"}
|PreviewType|指明材质面板将如何预览该材质，默认情况下，材质将显示为一个球形，我们可以通过把该标签设置为"Plane"，"SkyBox"来改变预览模型|Tags{"PreviewType"="Plane"}

上诉Tags仅可以在SubShader中声明，而不可以在Pass中声明。Pass块中也有标签可以定义，但它们不同于SubShader的标签类型。
>Pass语义块

``` C
Pass {
    [Name]
    [Tags]
    [RenderSetup]
    //other Code
}
```
>Name "MyPassName"

通过这个名称，我们可以使用ShaderLab的UsePass命令来直接使用其他Unity Shader中的Pass。如：
>UsePass "MyShader/MYPASSNAME"

这样可以提高代码复用率，但需要注意的是，Unity内部会把所有Pass名字转换为大写字母表示，因此使用UsePass命令时，必须使用大写形式的名字。

其次，Pass也可以设置渲染状态(RenderSetup)与标签(Tags)，渲染状态可以使用SubShader中支持的渲染状态，还可以使用固定管线的着色器命令。

Pass同样可以设置标签，但其标签不同于SubShader标签，Pass中支持的标签如下：

|标签类型 | 说明 | 例子 |
|:--- | :---- | :-------- |
|LightMode | 定义该Pass在Unity渲染流水线中的角色 | Tags{"LightMode"="ForwardBase"} |
|RequireOptions | 用于指定当满足某些条件时才渲染该Pass | Tags{"RequireOptions"="SoftVegetation"} |
#### 3.4 Fallback
紧跟在Unity Shader后面的是一个Fallback语句，它是最后一道保险，即如果所有的SubShader都不能用的话，就使用这个吧。语义如下：
``` C
Fallback "name"
// or
Fallback off
```
综上所述，我们可以通过一个字符串来指明这个"最后的Shader"，当然你也可以"宁缺毋滥"——不能按我的意思渲染的话，那就不要管它了。但是啊，FallBack还会影响阴影的投射，在渲染阴影纹理时，Unity会在每个Unity Shader中寻找一个阴影投影的Pass，通常情况下我们不需要自己去实现一个这样的Pass，这是因为Fallback使用的内置Shader中往往包含了一个这样的通用Pass，因此为每个Unity Shader设置好Fallback还是十分重要的。

### 4.CG/HLSL
目前，我们基本理解了一个Unity Shader的结构了，Properties里面定义属性，在SubShader中写真正的着色器代码。

好了，还记得Unity提供给我们的四种Unity Shader模板么？实际上那便对应了Unity Shader的两个不同类型：表面着色器(Surface Shader)与顶点/片元着色器(Vertex/Fragment Shader)

我们来看一个非常简单的表面着色器代码(Surface Shader)
``` C
Shader "Custom/Simple Surface Shader"{
    SubShader{
        Tags{"RenderType"="Opaque"}
        CGPROGRAM
        #Pragma surface surf Lambert
        struct Input{
            float4 color : COLOR;
        };
        void surf (Input IN,inout surfaceOutput o){
            o.Albedo = 1;
        }
        ENDCG
    }
    Fallback "Diffuse"
}
```
再来看一个非常简单的顶点/片元着色器(Vertex/Fragment Shader)
``` C
Shader "Custom/Simple VertexFragment Shader"{
    SubShader{
        Pass{
            CGProGRAM
            
            #pragma vertex vert
            #pragma fragment frag

            float4 vert(float4 v : POSITION) : SV_POSITION{
                return mul (UNITY_MATRIX_MVP, v);
            }

            fixed4 frag() : SV_Target{
                return fixed4(1.0,0.0,0.0,1.0);
            }

            ENDCG
        }
    }
}
```
这就是Unity Shader的两种不同形式
* 表面着色器(Surface Shader)是Unity自己创造的一种着色器代码类型，其所需要的代码量很少，Unity在背后做了很多工作，但渲染的代价比较大。其本质还是顶点/片元着色器，实际上就是顶点/片元着色器更高一层的抽象。它存在的价值在于为开发者处理了很多很多的光照细节，使得开发者不需要操心这些事情。
  
>表面着色器被定义在**SubShader语义块(而非Pass语义块)**中的CGPROGRAM与ENDCG之间，原因是表面着色器不关心开发者用多少个Pass，每个Pass如何渲染等问题，Unity会在背后做好这些事情。开发者所需要做的，就只是简单地告诉Surface Shader用"这个纹理"去填充颜色，用"这个法线纹理"去填充法线，使用光照模型等。具体细节不用关系。

* 顶点/片元着色器(Vertex/Fragment Shader)则更加复杂，灵活性也更高。

>和表面着色器类似，顶点/片元着色器地代码也需要定义在CGPROGRAM与ENDCG之间，但不同地是，顶点/片元着色器是写在**Pass语义块内(而非SubShader内)**。因此，我们需要自己定义每个Pass需要使用地Shader代码，可以控制渲染的实现细节。

那么CGPROGRAM与ENDCG之间是什么呢？首先我们要了解一下当下主流的几种Shader Language(SL):
>目前主流的Shader Language有3种语言
* GLSL：基于OpenGL的OpenGL Shading Language，简称GLSL
* HLSL: 基于DirectX的High Level Shading Language,简称HLSL
* Cg语言: NVIDIA公司的C for Graphic，简称Cg语言。

因此，我们需要把CG/HLSL的语言嵌套在ShaderLab语言之中，值得注意的是，此处的CG/HLSL是Unity经过封装后的，但其语法和标准的CG/HLSL语法几乎一模一样，只有部分原生的函数和用法没有支持，感觉和当时的JavaScript和UnityScript感觉很类似。而且由于CG与DX9风格的HLSL从写法上而言几乎是同一种语言，因此在Unity里CG与HLSL是等价的。同样的，ShaderLab也支持内嵌GLSL的！
``` C
CGPROGRAM
    // CG/HLSL代码
    // 支持平台：PC，Xbox360等等
ENDCG

GLSLPROGRAM
    // GLSL代码
    // 支持平台：Mac OS X，OpenGL ES2.0，Linux
ENDGLSL
```
OK，现在基本的Unity Shader知识储备就差不多了，欢迎进入Shader的世界~
### 5.几大SL的爱恨情仇
摘抄自《GPU Programming And Cg Language Primer 1rd Edition》 中文名《GPU编程与CG语言之阳春白雪下里巴人》
>Shader language目前有3种主流语言：基于OpenGL的GLSL（OpenGL Shading Language，也称为GLslang），基于Direct3D的HLSL（High Level Shading Language），还有NVIDIA公司的Cg （C for Graphic）语言。
>GLSL与HLSL分别提基于OpenGL和Direct3D的接口，两者不能混用，事实上OpenGL和Direct3D一直都是冤家对头，曹操和刘备还有一段和平共处的甜美时光，但OpenGL和Direct3D各自的东家则从来都是争斗不休。争斗良久，既然没有分出胜负，那么必然是两败俱伤的局面。

>首先ATI系列显卡对OpenGL扩展支持不够，例如我在使用OSG（Open Scene Graphic）开源图形引擎时，由于该引擎完全基于OpenGL，导致其上编写的3D仿真程序在较老的显卡上常常出现纹理无法显示的问题。其次GLSL 的语法体系自成一家，而HLSL和Cg语言的语法基本相同，这就意味着，只要学习HLSL和Cg中的任何一种，就等同于学习了两种语言。不过OpenGL 毕竟图形API的曾经领袖，通常介绍OpenGL都会附加上一句“事实上的工业标准”，所以在其长期发展中积累下的用户群庞大，这些用户当然会选择 GLSL学习。此外，GLSL继承了OpenGL的良好移植性，一度在unix等操作系统上独领风骚（已是曾经的往事）。

>微软的HLSL移植性较差，在windows平台上可谓一家独大，可一出自己的院子（还好院子够大），就是落地凤凰不如鸡。这一点在很大程度上限制了 HLSL的推广和发展。目前HLSL多半都是用于游戏领域。我可以负责任的断言，在Shader language领域，HLSL可以凭借微软的老本成为割据一方的诸侯，但，决不可能成为君临天下的霸主。这和微软现在的局面很像，就是一个被带刺鲜花簇拥着的大财主，富贵已极，寸步难行。

>上面两个大佬打的很热烈，在这种情况下可以用一句俗话来形容，“鹬蚌相争，渔翁得利”。NVIDIA是现在当之无愧的显卡之王（尤其在AMD兼并ATI之后），是GPU编程理论的奠基者，GeForce系列显卡早已深入人心，它推出的Cg语言已经取得了巨大的成功，生生形成了三足鼎立之势。NVIDIA公司深通广告之道，目前最流行的GPU编程精粹一书就出自该公司，书中不但介绍了大量的GPU前沿知识，最重要的是大部分都用Cg语言实现。凭借该系列的书籍，NVIDIA不光确定了在青年学子间的学术地位，而且成功的推广了Cg语言。

### 6.Unity与Cg/HLSL
>Unity官方手册上讲Shader程序嵌入的小片段是用Cg/HLSL编写的，从“CGPROGRAM”开始，到“CGEND”结束。所以，Unity官方主要是用Cg/HLSL编写Shader程序片段。Unity官方手册也说明对于Cg/HLSL程序进行扩展也可以使用GLSL，不过Unity官方建议使用原生的GLSL进行编写和测试。如果不使用原生GLSL，你就需要知道你的平台必须是Mac OS X、OpenGL ES 2.0以上的移动设备或者是Linux。在一般情况下Unity会把Cg/HLSL交叉编译成优化过的GLSL。因此我们有多种选择，我们既可以考虑使用Cg/HLSL，也可以使用GLSL。不过由于Cg/HLSL更好的跨平台性，更倾向于使用Cg/HLSL编写Shader程序。

