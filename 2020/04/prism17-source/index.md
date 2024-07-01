# Prism源码解析 Modules加载


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

在软件开发过程中，总想组件式的开发方式，各个组件之间最好互不影响，独立测试。Prism的Modules很好的满足了这一点。

![Module composition](/prism/Ch4ModularityFig1.png)

这个架构图很好了讲解了Prism的Modules的概念

Prism支持通过配置文件，文件夹，手动载入Module的方式，并且对Module的载入进行验证，包括重复和循环依赖验证

Prism加载模块的顺序

![Module loading process](/prism/Ch4ModularityFig2.png)

直接看源码吧

## 0、Modules加载

&#43; Modules的加载主要依靠ModuleCatalog来发现模块，
&#43; 通过ModuleManager来加载模块并对模块进行验证以确保模块的加载顺序，
&#43; ModuleInitializer负责模块的初始化，包括加载模块所必须的类和显示UI Elements等等。

在Prism.PrismApplicationBase 的Initialize方法中调用

![1585574598775](/prism/1585574598775.png)

创建目录

![1585574672183](/prism/1585574672183.png)

RegisterRequiredTypes方法中向容器注入ModuleManager，ModuleInitializer，

![1585574816406](/prism/1585574816406.png)

最后调用了InitializeModules方法，并在其中调用了ModuleManager的Run方法

![1585574852698](/prism/1585574852698.png)

看着两个名字就明白了，第一个是发现模块并验证模块，第二个是加载模块并初始化。

![1585574917778](/prism/1585574917778.png)

看一下ModuleCatalogBase的Initialize方法，果然

![1585574973443](/prism/1585574973443.png)

![1585574987953](/prism/1585574987953.png)

而验证就更加有意思了

![1585575114834](/prism/1585575114834.png)

重复性验证

![1585571571246](/prism/1585571571246.png)

通过模块名字ModuleNames来判断是否被加载过，，如果存在就抛出异常

加载顺序验证

![1585572866324](/prism/1585572866324.png)

同时看一下ModuleCatalogBase

![1585575240730](/prism/1585575240730.png)

每当items发生变化都会进行验证

![1585575308583](/prism/1585575308583.png)

![1585575327939](/prism/1585575327939.png)

发现验证完了来看一下ModuleManage的LoadModulesWhenAvailable方法

![1585575416018](/prism/1585575416018.png)

![1585575456472](/prism/1585575456472.png)

![1585575508444](/prism/1585575508444.png)

![1585575521164](/prism/1585575521164.png)

看到最终使用了ModuleInitializer来初始化Module。其过程通过Linq实现延迟加载技术。

![1585575588013](/prism/1585575588013.png)

在这个方法中发现Module必须实现IModle接口。并在这儿调用了RegisterTypes和OnInitialized方法。

模块的加载看完了，下面来看例子吧

## 1、通过AppSetting加载

先看一下配置文件

![1585575781099](/prism/1585575781099.png)

在初始化时

![1585575803440](/prism/1585575803440.png)

看到重写了CreateModuleCatalog，前面已经介绍过ModuleCatalog就是控制Module发现和验证的。

![1585575869468](/prism/1585575869468.png)

![1585575909447](/prism/1585575909447.png)

![1585575962111](/prism/1585575962111.png)

可以看到section的名称必须是modules。

先解析Module依赖逻辑，最后调用AddModule方法

![1585577943221](/prism/1585577943221.png)

再ModuleAModule中载入相关的UIElement。

## 2、通过代码加载

通过代码加载就更简单了，直接在ConfigureModuleCatalog方法中调用默认的ModuleCatalog加载相关的Module就可以了。

![1585578152349](/prism/1585578152349.png)

在ModuleAModule中代码不变

这其中的逻辑在0节中已经解释清楚了，就不在叙述。

## 3、通过目录加载

通过目录加载，如果不看源码怎么设计，需要创建一个ModuleCatalog，在创建的时候将目录地址传入。在内部InnerLoad方法中找到对应目录，然后通过遍历程序集找到实现IModule接口的类，加载这个类就可以了。

看了下源码也正是这么做的

![1585578439577](/prism/1585578439577.png)

![1585578580233](/prism/1585578580233.png)

看了源码发现官方考虑了更多的问题，比如创建了AppDoamin来加载程序集以保证隔离和数据安全。甚至还为其创建了一个InnerModuleInfoLoader类来反射程序集

![1585578854546](/prism/1585578854546.png)

这样的指责分配非常好，我们甚至可以写一个通过网络来加载Module的ModuleCatalog类。

## 4、通过手动方式加载

![1585579279370](/prism/1585579279370.png)

先在ConfigureModuleCatalog中将所有的Module加载进来，并将InitializationMode的方式设置为按需，

![1585579366391](/prism/1585579366391.png)

那么就可以在需要的时候利用LoadModule方法载入之前加载的Module

![1585579427491](/prism/1585579427491.png)

![1585579463038](/prism/1585579463038.png)

值得注意的是并没有提供卸载Module的接口。

## 总结

这一篇介绍了下Modules加载的原理，其实就是

&#43; ModuleCatalog负责发现Module。
&#43; 通过ModuleManager来加载模块并对模块进行验证以确保模块的加载顺序，
&#43; ModuleInitializer负责模块的初始化，包括加载模块所必须的类和显示UI Elements等等。

下一篇开始将介绍MVVM的实现。

## 添加

ModuleManager负责对ModuleCatalog和ModuleInitializer的引用。

ModuleCatalog维护一个ModuleCatalogItemCollection和ModuleInfo的集合


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/prism17-source/  

