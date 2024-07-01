# Prism源码解析 Bootstrapper和Region的创建


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

之前也研究过Prism框架但是一直没有深入理解，现在项目上想把一个Winform的桌面应用程序改造成WPF程序，同时我希望程序是可测试可维护架构良好的，Prism的这些设计理念正好符合我的需求，其主要用于WPF和Xamarin，用于构建松耦合，可维护，可测试的应用程序框架，在我看到源码后也深受启发，欢迎大家一起交流探讨。

Prism的整体架构

![Typical Prism application architecture](/prism/Ch1IntroFig3.png)

## 开始

我将从官方的Samples的顺序，看介绍中的每个功能是怎么实现的。

[Github地址](https://github.com/PrismLibrary/Prism-Samples-Wpf)

## 0、PrismApplicationBase

首先介绍一下这个类，这是Startup，这个类中构建了所有的Prism功能和整体的框架。这个类中大多数的方法都是虚方法，可以重载加入自定义的一些功能，Prism也希望我们如此进行设计。

在整个Prism中，UnityContainer无处不再，它就是一个大的容器，保存着所有类，在需要的时候Resolver出来。

![1585470691950](/prism/1585470691950.png)

有两个字段，ContainerExtension就UnityContainer,当然也可以自定义其他的容器，在文章中都默认为UnityContainer容器，关于容器也给一个官方说明

![Common Service Locator implementations in Prism](/prism/Ch3DIFig1.png)

ModuleCatalog是定义了加载Module的方式，模块是Prism的一大优势，给一张官方说明，意图胜千言

![Module composition](/prism/Ch4ModularityFig1.png)

所有的一切都是从一个类开始PrismApplicationBase，在这个类中加载了Prism的所有功能。

![1585471475741](/prism/1585471475741.png)

看一下最重要的一个方法，正是在这个方法中完成了大部分功能，其主要工作就是将基础架构模块，RequireTypes，RegionAdapterMappings,RegionBehaviors,注入到相应的容器中。

下面几个方法感受一下

![1585471851178](/prism/1585471851178.png)

![1585471862192](/prism/1585471862192.png)

![1585471872010](/prism/1585471872010.png)

![1585471899983](/prism/1585471899983.png)

![1585471908276](/prism/1585471908276.png)

在PrismApplicationBase的子类中看一下

![1585472041462](/prism/1585472041462.png)

正如我前面所说，重写某个方法，先调用Base.Method,然后再加入自己功能

对于开发者来说必须重写的就两个方法

![1585472248282](/prism/1585472248282.png)

这个Shell就是主窗体，窗体的构成Prism的窗体都是由一个个Region构成，每个Region中都包含若干个View

![Shells, views, and regions](/prism/Ch1IntroFig5.png)

创建主窗体和RegisterTypes方法，在第二个方法里可以加入我们所必须的一些基础构建，ContainerRegistry其实就是注册的Unity容器

![1585472365902](/prism/1585472365902.png)

下面让我们愉快的看例子吧。例子都在Prism-Samples-Wpf-master中一共29个，&lt;https://github.com/PrismLibrary/Prism-Samples-Wpf&gt;

## 1、BootstrapperShell

![1585472950515](/prism/1585472950515.png)

![1585472994216](/prism/1585472994216.png)

看着就是创建了一个Bootstraper然后Run了一下，通过容器创建了主窗体，Show了一下。

看一下如何实现的，其实文章都是在Bootstrapper中

![1585473149592](/prism/1585473149592.png)

看到这两个类是不是有一种恍然大悟的感觉，原来Bootstrapper是啥？就是将PrismApplicationBase中的方法全部从Application中抽出来，在这重新实现了一下，难道这就是单一职责原则？

来看一下Run，这些方法太熟悉了吧。

![1585473531825](/prism/1585473531825.png)

日志怎么用，当然是创建然后记录了，所有的信息都放到资源里，创建的方式有很多种，选择最简单的一种new，

![1585473705313](/prism/1585473705313.png)

BootStrapper的主要职责

![Connecting to the Prism Library](/prism/Ch1IntroFig6.png)

## 2、Regions

这个就是简单的创建一个Region

![1585473969090](/prism/1585473969090.png)

一目了然啊，就是在ContentControl中用RegionManager的依赖属性创建的一个Region。

用经典的三个问题来看看RegionManager，你是谁，你从哪里来，要到哪里去。

![1585474161899](/prism/1585474161899.png)

哦！小伙子你很张狂啊，很强大。看一眼把关了就可以了。

看一眼知道了有一个RegionName依赖属性，当这个属性变化的时候调用

![1585474975966](/prism/1585474975966.png)

IsInDesignMode就是判断是否在VS的设计模式，调用

![1585475107593](/prism/1585475107593.png)

哦，还是用了延迟加载，还是用容器创建的，Prism里几乎所有的类都是通过容器创建的

![1585475296411](/prism/1585475296411.png)

Behavior？WPF里面的行为就是服务啊，就是先把一个依赖属性存着，需要的时候盘它。去看看

![1585475415828](/prism/1585475415828.png)

很标准的服务。

![1585475467136](/prism/1585475467136.png)

使用了弱引用，很棒的设计，想想也是如此，

![1585475585421](/prism/1585475585421.png)

![1585475621940](/prism/1585475621940.png)

通过Load事件实现延迟加载，嗯，很棒，在界面载入的时候创建Region。

![1585475679204](/prism/1585475679204.png)

![1585475701564](/prism/1585475701564.png)

载入触发一次就好。很喜欢这个单词WireUp，缠绕，UnWire，

![1585475877997](/prism/1585475877997.png)

![1585476017158](/prism/1585476017158.png)

通过名字创建，到RegionAdapterMapping中找到RegionAdapter然后通过Adapter的Initialize创建。

![1585476694306](/prism/1585476694306.png)

RegionAdapterMapping就是RegionAdapter的集合

![1585476749370](/prism/1585476749370.png)

![1585478744531](/prism/1585478744531.png)

![1585476852940](/prism/1585476852940.png)

先创建Region，然后添加行为

![1585477085692](/prism/1585477085692.png)

![1585476992679](/prism/1585476992679.png)

都有哪些行为呢？

![1585477032237](/prism/1585477032237.png)

创建Region工作完成啦。

看看官方文档关于Region

![Region, control, and adapter relationship](/prism/Ch7UIFig2-1585477472550.png)



## 3、CustomRegions

想要自定义一个Regin，那肯定要创建一个RegionAdapter，自定义一个StackPanelRegionAdapter

![1585477844435](/prism/1585477844435.png)

![1585477898475](/prism/1585477898475.png)

![1585477950980](/prism/1585477950980.png)

通过刚刚的源码解读这些理解起来好像都不困难了。

## 总结

通过源码探索了下Bootstapper，Region的创建及如何自定义一个RegionAdapter，轻轻揭开了Prism一点点面纱。后面还有很多的功能和想法，欢迎大家和我一起探讨学习。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/03/prism15-source/  

