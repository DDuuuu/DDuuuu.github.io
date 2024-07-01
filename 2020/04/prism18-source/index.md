# Prism源码解析 ViewModel注入


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

介绍一个Prism的MVVM实现，主要介绍Prism如何在WPF上进行的一些封装，以实现MVVM。MVVM到底是什么呢？看一下这一幅经典的图

![MVVM classes and their interactions](/prism/Ch5MvvmFig1.png)

以前没有ViewModel这个概念，就是将Model传递到View显示，这样软件也可以工作，但却很混乱，一旦VIew要改动，一点点的改动都会造成很多代码需要改动，不利于维护。再者VIew层充斥着各种解析Model的代码，这些代码完全不属于View啊。平白无故的给View增加了很多职责。这是坏代码的味道。所以就有了ViewModel。ViewModel负责干什么，必须要干什么，其实ViewModel的职责就是将自己的数据绑定到View显示，同时数据变化需要通知View，View上客户的操作及时响应，至于数据怎么解析，从哪里获取，View的响应都应该方法后一层，可以是Controller，可以是Servicer,可以是Presenter。也就是业务逻辑尽量推到后一层。

试想一下，系统里的Model有很多，有数据库对应的数据库模型，有业务于对应的领域模型，有用于数据交互的DTO也是模型，那么对应的View有一个ViewModel也不觉得奇怪。

## 0 ViewModel定位

MVVM的第一步就是要解决ViewModel的依赖注入问题，框架如何不着痕迹的将View对应的VIewModel注入到依赖属性DataContext。

还记得PrismApplicationBase类吗，就是继承Application，将整个Prism框架组件注入到Unity的那个类，

![1585748254700](/prism/1585748254700.png)

![1585748299765](/prism/1585748299765.png)

![1585748279636](/prism/1585748279636.png)

看到第一步是啥?ConfigureViewModelLocator,配置ViewModelLocator,急人之所急，Prism框架的第一步配置ViewModelLocator，

![1585748453172](/prism/1585748453172.png)

好吧，第一步就是设置ViewModelFactory，这个工厂就是通过View的类型和实例从Unity容器中获取ViewModel实例。

![1585748824956](/prism/1585748824956.png)

![1585748835165](/prism/1585748835165.png)

噢！这个View参数还没用上。

再来看看这个包含ViewModelFactory的ViewModelLocationProvider。

![1585748957518](/prism/1585748957518.png)

![1585749152851](/prism/1585749152851.png)

从这个名字我们可以大胆猜测，这个类应该是负责真正解析ViewModel的位置的，看到这个类的方法，有ViewModelFactory，有Register，有GetViewModelByXXX。

![1585749500406](/prism/1585749500406.png)

这个类中一个委托字段_defaultViewTypeToViewModelTypeResolver，从这个字段我们可以看出是默认VIewModel解析方式，可以看出就是把View完整类型名中的Views替换成ViewModels，然后返回Type，从这里面我们知道View的名字一定要含有Views，ViewModel一定要含有ViewModels。

![1585751286821](/prism/1585751286821.png)

好吧，知道了哪里解析的再来看看哪里调用的。

![1585749866991](/prism/1585749866991.png)

prism:ViewModelLocator.AutoWireViewModel=&#34;True&#34;，看到了，将ViewModelLocator的依赖属性AutoWireViewModel至为True，可以进一步推测ViewModelLocator里面肯定调用了ViewModelLocationProvider的相关方法以获得ViewModel的类型或实例。

![1585750033315](/prism/1585750033315.png)

![1585750054551](/prism/1585750054551.png)

![1585750076882](/prism/1585750076882.png)

依赖属性改变触发了AutoWireViewModelChanged方法，然后调用ViewModelLocationProvider.AutoWireViewModelChanged

![1585750149579](/prism/1585750149579.png)

![1585750194664](/prism/1585750194664.png)

![1585750209281](/prism/1585750209281.png)

![1585750223719](/prism/1585750223719.png)

先去查看两个字典，一个字典key是View是实例，另一个字典key是View的Type，都没有调用，然后调用ViewModelLocationProvider.**_defaultViewTypeToViewModelTypeResolver**，也就是默认解析，在这边解析获得VIewModel的类型，然后通过默认工厂获得ViewModel实例。并绑定到VIew的DataContext。

至此，知道了整个默认VIewModel解析的全部过程，梳理一下

&#43; 在程序开始向ViewModelLocationProvider中设置ViewModel类型工厂，也就是Unity。
&#43; ViewModelLocationProvider就是ViewModel获取的地方有两个字典都应该是存放viewmodel，有一个默认解析是通过View的type解析出ViewModel的type。
&#43; 在Xaml中通过ViewModelLocator的依赖属性AutoWireViewModel调用ViewModelLocationProvider的AutoWireViewModelChanged来实现绑定。

## 1 自定义ViewModel定位

通过0的介绍，想一下怎么自定义实现VIewModel定位，有几种方法，

1. 提前向ViewModelLocationProvider的字典中添加ViewModel的类型
2. 改变_defaultViewTypeToViewModelTypeResolver解析方式
3. 修改工厂。这个不能从根本上改变。

这个例子用的是第二种。

![1585751155024](/prism/1585751155024.png)

在程序的开始重写ConfigureViewModelLocator方法，除了向ViewModelLocationProvider中添加ViewModelFactory外，还修改了_defaultViewTypeToViewModelTypeResolver解析方式。直接就通过View的type后面家长ViewModel，简单粗暴。

![1585751325000](/prism/1585751325000.png)

## 2 自定义ViewModel解析

这种方法就是上面提到的1方法

&#43; 提前向ViewModelLocationProvider的字典中添加ViewModel的类型

![1585751470206](/prism/1585751470206.png)

这张方法显然有很大的弊端，当程序中有很多View时怎么能手动添加呢，只能适用与特殊的View和ViewModel的解析，如Shell的VIewModel的解析。

这种解析方法也不用在意View和ViewModel的名字了。

## 总结

从ViewModel的解析中，我们看到一种设计模式，View依赖ViewModelLocator，ViewModelLocator依赖ViewModelLocationProvider，ViewModelLocationProvider负责具体解析出对应的实例，相当于ViewModelRegistry，其中当然以有对工厂的依赖。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/prism18-source/  

