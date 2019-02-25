---
title: Qt5 QtCreator QGLViewer
date: 2019-2-25
tag: Qt 
---

 

毕设设计可能要做一个与雷达可视化相关的项目，学长给了我一份之前的Qt的项目来看看，因为以前一直只听说过Qt，其怎么怎么样，怎么怎么好用，但实际上却从来都并未使用过。因此正好开坑记录一下Qt的学习。

### Qt

Qt，发音同“cute”，是一个跨平台的C++应用程序开发框架。广泛用于开发GUI程序，这种情况下又被称为部件工具箱(widget toolkits)。同时它也可以用于开发非GUI程序，比如控制台工具和服务器。Qt使用于OPIE，Skype，VLC media player，Adobe Photoshop Elements，VirtualBox与Mathematica以及被Autodesk，欧洲空间局，梦工厂，Google，HP，KDE，卢卡斯，西门子公司等所使用。

它是Digia公司的产品，使用标准C++和特殊的代码生成扩展(被称为元对象编译器)以及一些宏。通过语言绑定，其他的编程语言也可以使用Qt。

Qt是自由开放源码的软件，在GNU宽通用公共许可证（LGPL）条款下发布。所有版本都支持广泛的编译器，包括GCC的C++编译器与Visual Studio。

### Qt的安装 

前往[Qt官网](https://www.qt.io/cn)下载Qt，注册一个账号，然后下载下来一个在线安装程序，点击进入安装界面

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Qt_Learn/02.png" />

安装路径等不用说(注意不能有中文路径)，默认选择安装Qt Creator，比较重要的是选择组件这一步：

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Qt_install.png"/> 

(之前我就点了个默认就没管了，然后在Qt Creator里面死活找不到Kit)，这里我选的Qt5的版本，Qt4与Qt5之间差别还是比较大的，但是既然是学习，肯定选择最新的版本了（我选的Qt 5.12.0)。<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Qt_Learn/01.png"/>

然后是开发编译工具：

<img src="https://raw.githubusercontent.com/nature-god/MarkdownPhotos/master/Qt_Learn/03.png"/>

然后静待安装完成。

### QGLViewer

[libQGLViewer](http://libqglviewer.com/)是一个Qt的C++库，用于简化创建OpenGl 3D图像。它提供了一些非常典型的3D视图功能，包括像用鼠标移动相机，坐标轴显示，物体选择等功能。基于Qt工具包，它支持所有平台，其有商业许可，但也可用作开源软件开发。

QGLViewer在上面的官网就可以下到了，官网内也详细写明了各个不同的平台该如何进行操作。我是Windows，所以下载好QGLViewer的项目后，解压后打开QGLVIewer/QGLViewer.pro项目文件(使用Qt Creator，其他工具方式详见官网)，然后点击左下角那个锤子进行构建。一般可以正确运行，如果失败的话，检查一下自己之前安装的kit是否错误，在Qt Creator上面菜单选择：工具->选项，打开后选择Kit，看是否有错误。

在运行完毕之后，会得到两个dll文件：QGLViewer2.dll和QGLViewer2.dll。将其拷贝至C:\Windows\System32文件内，或是在要使用的项目路径下直接添加这两个dll。

此时可以尝试跑一下example，就在QGLViewer文件的example文件内，可以看到QGLViewer的一些比较基本的功能演示。

之后调用该库时，也可以在.pro项目文件中添加如下设置：

> ```
> TARGET = myViewer
> CONFIG *= qt opengl release
> QT *= opengl xml
> 
> HEADERS = myViewer.h
> SOURCES = myViewer.cpp main.cpp
> 
> # Windows
> INCLUDEPATH *= C:/Users/login/Documents/libQGLViewer-2.7.1
> LIBS *= -LC:/Users/login/Documents/libQGLViewer-2.7.1/QGLViewer -lQGLViewer2
> 
> # Linux
> INCLUDEPATH *= /home/login/Documents/libQGLViewer-2.7.1
> LIBS *= -L/home/login/libQGLViewer-2.7.1/QGLViewer -lQGLViewer
> 
> # Mac 
> INCLUDEPATH *= /Users/login/Documents/libQGLViewer-2.7.1
> LIBS *= -F/Users/login/Library/Frameworks -framework QGLViewer
> ```

但是由于Qt5内部封装了Opengl的一些函数，如果你自己电脑之前也装过Opengl的相关东西的话，可能导致编译后链接库失败，此时可以在.pro文件中添加：

>```
>win32-g++ {
>    LIBS += -lopengl32
>}
>win32-msvc*{
>    LIBS += opengl32.lib
>}
>```

即可编译成功。