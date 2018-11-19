---
title: Unity Shader入门(二)
date: 2018-11-15
tag: Unity Shader
---
在[Unity Shader入门(一)](https://nature-god.github.io/2018/11/09/d_shaderLearning01/)里面，我们了解了Unity Shader的一些基本概念与结构，也就是说，我们学会了1+1=2了，接下来就让我们去解非齐次线性方程组吧...好吧开玩笑的，就和大家经常调侃的，平时上课学会受力分析，期末让你造大桥，平时上课学会增删改查，期末让你设计数据库的吐槽一般，学习曲线不能太过陡峭。本来打算这章来说一下Unity Shader学习中所必须的线性代数知识，后来想了一下，学会线代并不就是能写shader了，还是要和shader结合起来看，记忆更深，本节会简要介绍一下shader学习中必备的数学概念，具体的计算公式等会结合Unity Shader例子进行详解。

说实在的，做一个程序员并不需要太多专业数学知识——因为很多时候我们调用标准库的一些函数就足够日常的开发使用了，当然这是很低级的程序员，俗称"搬运工"。而算法工程师对数学就要敏感的多，但有一部分也不会用到太多高等数学的东西，多是数据结构加一些优化分析。但是如果公司业务有着特殊需求——比如Adobe公司，它的PhotoShop需要对图片进行大量处理，再比如AutoDesk公司，它们的Maya建模，涉及到各个坐标系之间的变换，UV映射等更加复杂的数学运算等，又或者是时下超火的机器学习，计算机视觉这块，各种卷积微分多次偏导，这些"很难"的项目往往就是需要比较高的数学水平才能解决了。

>计算机图形学之所以深奥难懂，很大原因在于它是建立在虚拟世界上的模型。

说实在的，凡是与高等数学，线代集合概率论这些东西相关了的科目，都有点难，所以说数学系是万金油的说法也就此而来——数学水平高，啥都能干。
### 1.笛卡尔坐标系
#### 1.1二维笛卡尔坐标系
>传说笛卡尔坐标系来源于笛卡尔对天花板上一只苍蝇的运动轨迹的观察，笛卡尔发现可以使用苍蝇离不同墙面之间的距离来描述它的当前位置。

只要是接受过九年义务教育的人，就使用过笛卡尔坐标系。一个二维的笛卡尔坐标系包含了两部分的信息：
* 一个特殊的位置，即原点，它是整个坐标系的中心
* 两条互相垂直的矢量，即X轴与Y轴，也称为该坐标的基矢量。
  
<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Blog_Shader02/dikaer.png"/>

原点位置是任意的，互相垂直的矢量也是任意的，因此不同的设定就会有不同的坐标表示：

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Blog_Shader02/DiffCoordinate.png"/>

由于一个原点在左上，一个原点在右上，所以坐标变换计算时就可能有不同的结果。
#### 1.2三维笛卡尔坐标系
>人类生活在三维世界里，所以人类对低维度的空间(一维与二维)是比较容易理解的，对于同等维度(三维)理解起来就稍困难一些，而对于更高维度(四维等)的理解难度就更大了。

三维笛卡尔坐标系中，我们需要定义3个坐标轴和一个原点：
* 一个特殊的位置，即原点，它是整个坐标系的中心
* 三条互相垂直的矢量，也称为正交基。如果它们长度为1，则称之为标准正交基(orthonormal basis)

与二维笛卡尔坐标系类似，三维笛卡尔坐标系中坐标轴的方向也不是固定的，但是二维的笛卡尔坐标，你总是可以通过一系列操作：选择，翻转等将一个二维笛卡尔坐标变成一个x轴水平朝右，y轴竖直向上的坐标系。因此理论上所有二维笛卡尔坐标系都是等价的。

但是三维笛卡尔坐标系不行，在某些情况下，你只能重叠x,y轴，但是z轴始终相反——由此区分出来两种坐标系：左手坐标系(Left-handed coordinate space)和右手坐标系(right-handed coordinate space)。
<img src="LeftAndRight.png">
对于开发者而言，使用左手坐标系还是右手坐标系都可以，此间并无优劣之分。其只是在映射到视觉时会有差别。
#### 1.3Unity中的坐标系
在Unity中，模型空间与世界空间(简单而言就是局部坐标系与世界坐标系，原点位置不同罢了)，Unity使用的左手坐标系，即右方，上方，前方对应于x，y，z轴的正方向。但对于观察空间而言(就是摄像机为原点的坐标系)，Unity使用的时右手坐标系。即右方，上方，后方对应于x，y，z轴的正方向(面朝电脑)。因此这意味，摄像机的正前方是z轴的负方向，即z轴坐标的减少意味着场景深度的增加
### 2.点和矢量
矢量(vector，也就是向量)，此处需要的知识点包括：
* 矢量的定义：n维空间中包含了模(magnitude)与方向(direction)的有向线段
* 矢量的加减法：三角形定则
* 矢量的模
* 矢量与标量的乘积与除法
* 单位矢量
* 零矢量
* 矢量的点积
* 矢量的叉积
基本知识点差不多这些了，多是初高中就学过的内容，若是有些许忘记，建议适当查下资料就够了。这些概念都还没还给老师的话，就可以继续看下下一个知识点。
### 3.矩阵
>矩阵，英文名matrix。matrix除去矩阵含义外，还有母体的意思，大名鼎鼎的《黑客帝国》，英文名就叫《The Matrix》

矩阵的定义不用多说，用个大括号扩起来一些数字那就是了。矩阵麻烦的是和向量结合...矢量和矩阵都是一个数组，因此矢量可以看作n x 1的列矩阵或是1 x n的行矩阵了。为何要这么做呢，因为矩阵有很多好的操作与性质，很方便计算机处理。

矩阵的一些基本概念：
* 矩阵和标量的乘法
* 矩阵和矩阵的乘法
* 特殊矩阵：对角矩阵，方块矩阵(方阵)，单位矩阵，转置矩阵，逆矩阵，正交矩阵
这些基本概念了解之后就行了，忘记了的多baidu或是google一下。
### 4.Hello Unity Shader
#### 4.1一个最简单的顶点/片元着色器
在前面我们已经看到了Unity Shader的基本结构：
```C
Shader "MyShaderName"{
    Properties{
        //属性
    }
    SubShader{
        //针对显卡A的SubShader
        
        Pass{
            //设置渲染状态与标签
            //开始CG代码片段
            CGPROGRAM
            //该代码片段的编译指令
            #pragma vertex vert
            #pragma fragment frag
            //CG代码
            ENDCG
            //其他设置
        }
        //其他的Pass
    }
    SubShader{
        //针对显卡B的SubShader
    }
    //默认回掉的Unity Shader
    Fallback "VertexLit"
}
```
OK，现在我们进入Unity中进行实战吧！我的Unity版本：Unity 2018.2.4f1，基本各版本问题不大，但如果遇到问题，请检查下是否和Unity版本相关。

1. 新建一个场景，并去掉天空盒(skybox)：删除场景中的direct light，然后选择坐上tool bar里面:Window->Rendering->LightingSetting，将Skybox material设置为none:

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Blog_Shader02/unityscreen.png"/>

2. 新建一个Unity Shader(Standrad Surface Shader)，命名为HelloShader。
3. 新建一个Material，命名为HelloMaterial，并把第二步中新建的Unity Shader赋给它。
4. 新建一个球体，第三步中的Material赋给它。
5. 在HelloShader中，删除所有默认初始代码，写入以下代码
```C
Shader "HelloShader"{
	SubShader{
		Pass{
			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			float4 vert(float4 v : POSITION) : SV_POSITION{
				return UnityObjectToClipPos (v);
			}

			fixed4 frag() : SV_Target {
				return fixed4(1.0,1.0,1.0,1.0);
			}
			ENDCG
		}
	}
}
```

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Blog_Shader02/ShaderAndMaterial.png"/>

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Blog_Shader02/HelloShader.png"/>

Ok，现在我们来详细对其进行剖析：
>Shader "HelloShader"

此句十分简单，即是对我们自定义的Shader取名，支持"xx/xx/xx"的形式，良好的命名习惯有利于我们快速找到目标Shader。
接下来是一些基本的结构，包括SubShader, Pass块等，此处代码无Properties。然后就是最为关键的CG代码块了。
>#pragma vertex vert
>#pragma fragment frag

它们告诉Unity，哪个函数包含了顶点着色器代码，哪个函数包含了片元着色器代码。通用形式如下：
>pragma vertex name
>pragma fragment name

name就是我们所指定的函数名，但这两个函数的名字不一定是vert和frag，但一般多用这两个，因为十分直观。
```C
float4 vert(float4 v : POSITION) : SV_POSITION{
    return mul (UNITY_MATRIX_MVP, v);    
}
```
这就是vert函数了，即我们所使用的顶点着色器代码，它是逐顶点执行的。vert函数的输入v包含了这个顶点的位置——这是通过POSITION语义指定的。
>语义，是两个处理阶段（顶点程序、片段程序）之间的输入 / 输出数据和寄存器之间的桥梁，同时语义通常也表示数据的含义，如 POSITION 一般表示参数种存放的数据是顶点位置。

vert函数的返回值是一个float4类型的变量，它是该顶点在裁剪空间中的位置。POSITION和SV_POSITION都是CG/HLSL中的语义(se'mantics)，这些语义将会告诉系统用户哪些需要输入值，以及用户的输出是什么。如此处POSITION将告诉Unity，把模型的顶点坐标填充到输入参数v中，SV_POSITION将告诉Unity，顶点着色器的输出是输出是裁剪空间中的顶点坐标。
