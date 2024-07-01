# Prism源码解析 RegionContext


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

## 0 RegionContext

Prism为Region构造了一个RegionContext，其功能和DataContext差不多。看看怎么实现的吧

首先声明

![1586063234079](/prism/1586063234079.png)

在RegionManager中定义一个依赖属性RegionContext,

![1586056586194](/prism/1586056586194.png)

![1586059794432](/prism/1586059794432.png)

![1586063271063](/prism/1586063271063.png)

值改变的时候更新RegionContext中的依赖属性

![1586060366772](/prism/1586060366772.png)

这个RegionContext就保存一个依赖属性对象实例，这个依赖属性用ObserableObject包装了一下，提供了一个获取实例的方法。

同时注意到这里面有一个关键的地方

view.SetValue(RegionContext.ObservableRegionContextProperty, (object) observableObject);

其实在view中还保存了ObservableRegionContextProperty依赖属性实例。

看一下依赖属性值改变怎么触发的

![1586064024535](/prism/1586064024535.png)

在Reion的VIew中其实是绑定了RegionContext依赖属性的ValeChanged事件。

看到这我就好奇为啥RegionManager中要有一个依赖属性，看着很多余啊？

我注意到有一个行为SyncRegionContextWithHostBehavior，

![1586065381426](/prism/1586065381426.png)

目前还没看出来怎么用。

我注意到Region的行为中有一个BindRegionContextToDependencyObjectBehavior 行为

![1586064712346](/prism/1586064712346.png)

当Region的View改变时，

![1586064741432](/prism/1586064741432.png)

![1586064747524](/prism/1586064747524.png)

这个context是哪里来的呢？在这个行为中

![1586064874717](/prism/1586064874717.png)

![1586064885585](/prism/1586064885585.png)

第一次载入的时候，只要View的RegionContext值变化就会将新值给Region.Context。

只要有新的View添加进来就将Region.Context的值给View的RegionContext.ObservableRegionContextProperty依赖属性。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/prism22-source/  

