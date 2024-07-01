# Prism源码解析 View的加载和控制


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

上一篇讲了入了门，这一篇，让我们不多BB直接开始

## 4、ViewDiscovery

在创建好Region后需要将View添加到Region中。先补充几个概念

在上一篇将了如何创建Region，现在让我们看看Region类是什么

![1585492014664](/prism/1585492014664.png)

&#43; private ViewsCollection views;
&#43; private ViewsCollection activeViews;

![1585492216249](/prism/1585492216249.png)

这是一个View集合，集合改变会触发CollectionChanged事件

![1585495811302](/prism/1585495811302.png)

其完全依赖ObservableCollection对象

![1585495861232](/prism/1585495861232.png)

&#43; this.Behaviors = (IRegionBehaviorCollection) new RegionBehaviorCollection((IRegion) this);

![1585492307886](/prism/1585492307886.png)

这是一个行为集合，每当添加进行为的时候，会主动调用Attach（）

![1585492378755](/prism/1585492378755.png)

&#43; PropertyChanged事件，每当Context,Name, RegionManager，会触发该事件

下面来看一个好玩的行为AutoPopulateRegionBehavior

![1585493348169](/prism/1585493348169.png)

可以看到这个行为对RegionViewRegistry有依赖，这个是通过构造注入的方式注入的。

![1585493881067](/prism/1585493881067.png)

该RegionViewRegistry保存着所有的View，是名副其实的Registry.

![1585494113642](/prism/1585494113642.png) 

该Registry有一个事件ContentRegistered,

![1585494177640](/prism/1585494177640.png)

每当调用这个方法的时候就会触发这个事件。

不能跑偏了，回到AutoPopulateRegionBehavior

![1585494258548](/prism/1585494258548.png)

在行为Attach的时候，已经对RegionViewRegistry进行了订阅。

![1585494345106](/prism/1585494345106.png)

![1585494374088](/prism/1585494374088.png)

看看this.Region.Add()

![1585494424733](/prism/1585494424733.png)

![1585494443683](/prism/1585494443683.png)

![1585494495775](/prism/1585494495775.png)

这个ItemMetadataCollection的改变会影响Views和ActiveViews

![1585495029949](/prism/1585495029949.png)

首先它是一个ObservableCollection，

![1585495066707](/prism/1585495066707.png)

![1585495082020](/prism/1585495082020.png)

其次ViewCollection就是依赖ItemMetadataCollection创建的，所以改变自然会影响ViewCollection

那这个VIewCollection是怎么来影响界面的呢，这就要看看另一个行为RegionActiveAwareBehavior

![1585495262471](/prism/1585495262471.png)

![1585495273837](/prism/1585495273837.png)

![1585495286661](/prism/1585495286661.png)

![1585495314530](/prism/1585495314530.png)

![1585495354241](/prism/1585495354241.png)

至此可能会一头雾水，这讲了什么啊，一会是Region，一会是Behavior，到底想说什么啊？其实就是讲了View是如何被自动注入到对应的Region。

下面让我们跟着Samples中的ViewDiscovery并结合刚刚讲的源码梳理一下。

&#43; 在程序开始的时候向行为工厂中注入了相应的行为

![1585496457588](/prism/1585496457588.png)

&#43; 在创建Region的时候RegionAdapter向其添加了所有的行为

![1585496767032](/prism/1585496767032.png)

&#43; 现在只需调用RegionManager.RegisterViewWithRegion方法就可以自动向Region中添加VIew并显现出来

![1585497087938](/prism/1585497087938.png)

![1585496954834](/prism/1585496954834.png)

可以看到就是调用RegionViewRegistry中Register&#39;VIew&#39;With&#39;Region方法

&#43; 下面就等着AutoPopulateRegionBehavior和RegionActiveAwareBehavior按照上面的方式工作就可以了。

可以看出为什么Region有这么强大的功能就是因为Prism给Region提供了很多的行为，行为作为WPF的一个特性，其作用是非常强大的。后面的View生命周期管理也是通过行为来完成的

## 5、ViewInjection

View手动加载到Region，通过一个点击事件，通过RegionManager的Regions属性添加View

![1585497500713](/prism/1585497500713.png)

这个就更简单了，因为没有走RegionVIewRegistry，而是直接通过Region添加View，会直接添加到对应的RegionView上,然后通过RegionActiveAwareBehavior显示，上面有就不再详尽叙述了。

## 6、ViewActivationDeactivation

激活或停用View

这个也不多BB直接看怎么调用

![1585498127903](/prism/1585498127903.png)

首先先用手动的方式向Region中添加两个View

![1585498162692](/prism/1585498162692.png)

就是两个方法Activate和Deactivate

![1585498267866](/prism/1585498267866.png)

这实现也太巧妙了吧，通过ItemMetadata直接影响了View和ActiveView，然后通过RegionActiveAwareBehavior行为实现。真帅

![1585498410120](/prism/1585498410120.png)

就不再叙述了。

## 总结

本章主要讲了View的加载方式，可以手动加载，可以自动加载，并可以控制View的Activate和DeActivate。其主要实现都是依靠行为，也从侧面反映出行为的强大，行为能做的事情实在太多了。下一章会对Modules的实现进行介绍。

## 补充

在创建Shell的时候，已经调用RegionManager.SetRegionName方法，然后触发RegionManager.OnSetRegionNameCallback方法

![1585815261494](/prism/1585815261494.png)

![1585815270764](/prism/1585815270764.png)

![1585815291842](/prism/1585815291842.png)

![1585815303024](/prism/1585815303024.png)


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/03/prism16-source/  

