---
title: Unity Shader入门(一)
date: 2018-11-09
tag: Unity Shader
---
这是自己开的第一篇技术帖，主要讲述下基本的shader入门知识。预计在这个寒假结束之前写完吧。
## 1.Unity的 Material，Shader 和 Texture
Unity使用Materials，Shaders和Textures来进行渲染，它们之间联系紧密，最终渲染出美轮美奂的画面。
>Material: Material通过它包含的贴图，布局与颜色等信息来定义一个表面应该如何被渲染。一个Material根据其使用的shader来使用不同的可选项。

>Shader：Shader是一种基于光照输入和Material配置来进行数学运算并计算出每个像素点(pixel)渲染颜色的脚本。

>Texture: Texture是一种位图图像。一个Material包含对textures的引用，所以Material的Shader就可以使用Texture去计算一个物体(GameObject)的表面颜色了。除了游戏对象表面的基本颜色(albedo)外，纹理还可以表示材质表面的许多其他方面，如反射率，粗糙度等等。

一个Material对应于一个Shader，这个Shader决定了这个Material种哪些可用的选项。一个Shader则对应了一个或多个Texture，在Unity的Inspector面板你可以直接拖拽Texture资源给Shader。


