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
  
<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/dikaer.png"/>

原点位置是任意的，互相垂直的矢量也是任意的，因此不同的设定就会有不同的坐标表示：

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/DiffCoordinate.png"/>

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

To Be Continued