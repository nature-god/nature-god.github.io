---
title: Unity委托简单使用
date: 2018-11-22
tag: Unity
---
在C/C++体系中，回调(callback)函数是十分重要的一个部分，以前用cocos2d或是其他一些windows相关的编程，很多地方都要用到类似的回调函数。在C#中，它同样是十分重要的。回调函数实际上是方法调用的指针，也称为函数指针，是一个十分强大的函数特性，同样也是C/C++体系最难的一部分。.NET以委托的形式实现了函数指针的概念，它们之间的区别在于，.NET委托是类型安全的。这说明，C中的函数指针是一个指向存储单元的指针，我们无法说明这个指针实际指向的是什么，更不用说参数和返回类型了。
#### 委托
什么是委托，通俗的来讲，就是将方法"委托"给另一个"对象"来执行。我们都知道函数的写法，在参数列表中可以传入参数，而委托则是不过是将其参数变成传入方法罢了。依旧是考虑我们游戏编程中可能遇到的一个实际性的需求：血量变化。
>假设这里有一个玩家控制的角色Player，其拥有当前生命值属性CurrentHealth，最大生命值属性MaxHealth，在UI层面，我们有一个血条UI，要求这个血条UI随着当前生命值的变化而进行长短与颜色的变化。
* CurrentHealth > 0.5*MaxHealth 血条颜色变成绿色
* 0.2*MaxHealth< CurrentHealth < 0.5*MaxHealth 血条颜色要变成黄色
* 0 < CurrentHealth < 0.1*MaxHealth 血条变成红色

这是一个很常见常见的需求，大家有做血条设计时有很大可能会遇到该需求,最简单的可能就是弄一个函数，每次有有血量值变化时就调用：
```C
void CurrentHealthChange(float _health)
{
    CurrentHealth = _health;
    //血条长度变化
    //血条颜色变化
}

//调用方式
Player.CurrentHealthChange(250.0f);
```
简单易懂，很好的实现了需求，未来有新的变化就直接往这个函数加内容就好了。
<br/>但是啊，这看上去很Low，而且未来如果新增属性：魔法值显示，体力值显示等问题的话，这个Player会变的很大，维护很困难——至少你会觉得写的不好，没有美感不是么？而且由于血条逻辑直接写在了该函数内，与UI层连接很紧密，未来每次UI层有变化，这边的代码逻辑也都需要进行变化。那好，我们现在换种方式：<br/>
<br/>不用delegate，我们可以用set/get访问器属性来解决这个问题:
```C
public class Player
{
    private float currentHealth;
    private float maxHealth;
    // accessors 
    public float CurrentHealth
    {
        set
        {
            currentHealth = value;
            //血条长度变化代码
            //血条颜色变化代码
            //if(value < 0.5*MaxHalth)
            //...
        }
        get
        {
            return currentHealth;
        }
    }
    public float MaxHealth
    {
        set
        {
            maxHealth = value;
        }
        get
        {
            return maxHealth;
        }
    }
}

```
这样，我们只要每次进行
```C
Player.CurrentHealth -= 950.0f;
```
这样的指令操作，就能进行UI相关的逻辑变化了！这个看上去好了不少，至少比较直观，血条逻辑代码似乎也看上去很不错？但是实际上它不过比上面方法的形式好了那么一点点，只不过是"看上去"厉害了一些。这依旧与UI层的耦合很紧密，我们追求的是松耦合，即一个模块的代码变化应该尽可能小地影响其他模块。这是代码编程地艺术，也是设计模式追求的简洁清晰与易扩展。<br/>
<br/>这个时候，就是我们的委托(delegate)出场的时候了。
<br/>先来简单介绍一下委托的基本结构与语法。在C#中使用一个类时，分两个阶段——首先要先定义这个类，即告诉编译器这个类由哪些成员变量与方法所组成。然后实例化类的一个对象。委托也是这样，首先要定义一个委托，然后创建该委托的一个或多个实例：
```C
delegate void NumChange(float changeNum);
```
这就是定义好了一个委托NumChange，它带有一个参数changeNum，返回类型为void。可见，我们给出了该委托的几乎所有细节：它要什么参数，返回什么值——因此它们的类型安全非常高。因此声明一个委托就跟一个函数声明类似，不过前面加了一个delegate的关键字罢了。此外与类相似的，委托也有访问级别：public，private，protected等。
>注意，实际上，定义一个委托是指定义一个新类。委托实现为派生自基类System.MulticastDelegate，而System.MylticastDelegate又派生自基类System.Delegate。

我们称呼类的实例为"对象"，但是委托的实例还是叫做"委托"，下面请根据上下文理解此处委托是指委托还是委托的实例啊。接下来我们就可以使用委托了。首先我们定义两个函数，一个用于血条长度控制，一个用于颜色控制：
```C
void HealthSliderController(float num)
{
    //血条长度逻辑
}
void HealthColorController(float num)
{
    //血条颜色控制
}
```
然后我们实例化一个委托对象，并赋初值
```C
public NumChange HealthNumChange;
HealthNumChange = HealthSliderController;
HealthNUmChange += HealthColorController;
```
调用委托
```C
HealthNumChange(500.0f);
```
这样就实现了同时调用多个方法的效果。这就是简单的委托用法，后续的魔法值，体力值逻辑我们依旧可以照葫芦画瓢就行，后续血条变化而引起的更多的UI逻辑，我们也可以再写函数新加入委托之中即可，无需对原有代码进行修改。在C#与Unity中的委托写法有点不同，另外委托可以直接使用匿名函数的方式进行赋值。