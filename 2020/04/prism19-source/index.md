# Prism源码解析 数据绑定和命令


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

WPF本身就支持通知、绑定和命令，实现ViewModel和VIew之间的通讯，但相对来说功能比较少，Prism扩充了这些功能并提供更加强有力，简洁的数据绑定和命令。

## 0 绑定通知

WPF的绑定通知需要实现INotifyPropertyChanged接口，也就是实现一个属性改变事件，用来通知UI属性改变了，让UI更新。该事件需要一个事件参数new PropertyChangedEventArgs(propertyName)传入属性的名字，这样的调用方式比较繁琐。

Prism扩充了WPF的绑定通知。提供了BindableBase 实现了 INotifyPropertyChanged接口，并使用CallerMemberName获取属性名字。这样就解决了属性改变事件调用繁琐的问题。同时在内部还是对相等值进行了过滤。

![1585984101837](/prism/1585984101837.png)

![1585985493521](/prism/1585985493521.png)

简单愉快的调用吧

![1585985633084](/prism/1585985633084.png)

只需要使用SetProperty方法就可以自动更新UI了。

值得注意的是OnPropertyChanged还提供了对Expression的支持

![1585986210800](/prism/1585986210800.png)

也就是主动调用OnPropertyChanged(()=&gt;this.PropertyName),也可以触发UI响应。

## 1 命令

### DelegateCommand

WPF命令和通知有点类似，命令需要实现ICommand接口，实现Execute方法，命令状态CanExecute和命令状态改变事件。并且WPF只实现了一个RoutedCommand。Prism提供了一个DelegateCommandBase命令基类实现ICommand，并扩充了子类DelegateCommand，大大简化了命令调用方式

![1585982600756](/prism/1585982600756.png)

![1585982617542](/prism/1585982617542.png)

看到在基类中有_synchronizationContext线程同步上下文，用来保证命令执行的时候线程同步。

重点关注一下子类中的几个方法

![1585987365575](/prism/1585987365575.png)

![1585987399505](/prism/1585987399505.png)

![1585987411359](/prism/1585987411359.png)

1.ExecuteDelegateCommand = new DelegateCommand(Execute, CanExecute);

&#43; 这个命令声明方式，如果命令状态发生变化的时候需要主动调用RaiseCanExecuteChanged方法来触发命令状态改变事件。

2.DelegateCommandObservesProperty = new DelegateCommand(Execute, CanExecute).ObservesProperty(() =&gt; IsEnabled);

&#43; 可以看到这种声明方式，提供了一个ObservesProperty方法，不需要显示调用命令状态改变事件。

3.DelegateCommandObservesCanExecute = new DelegateCommand(Execute).ObservesCanExecute(() =&gt; IsEnabled);

&#43; 这种声明方法提供ObservesCanExecute方法，直接观测命令状态改变事件和属性。

4. ExecuteGenericDelegateCommand = new DelegateCommand&lt;string&gt;(ExecuteGeneric).ObservesCanExecute(() =&gt; IsEnabled);

&#43; 这是一个使用泛型带参数的声明方式，

看一下内部怎么实现这种简单而强大的功能

![1585988022789](/prism/1585988022789.png)

![1585988063481](/prism/1585988063481.png)

通过Expression，内部调用PropertyObserver.Observes()方法并将RaiseCanExecuteChaned方法传入。

在PropertyObserver将Expression保存在一个链表，并每个节点都订阅OnPropertyChanged事件。

![1585988408971](/prism/1585988408971.png)

### CompositeCommand

Prism还提供了一个CompositeCommand命令

![1585989655012](/prism/1585989655012.png)

这个命令的功能跟其名字一样，就是复合命令，命令集合。

&#43; 将DelegateCommand实例放到其中，每当调用CompositeCommand调用的时候会调用它保存的所有命令，
&#43; 命令集合中任何一个命令状态改变，都会触发CompositeCommand命令状态的改变事件，导致CompositeCommand检查集合中所有的命令状态，首先会检查IActiveAware，再检查命令状态，如果任何一个命令状态是False，都会导致组合命令返回False

来看一下源码

![1585989673663](/prism/1585989673663.png)

![1585989692622](/prism/1585989692622.png)

![1585989705258](/prism/1585989705258.png)

注意到，聚合命令也是通过线程上下文保持线程同步，同时看到有检测IActiveAware接口，这个接口是什么意思呢？其实就是查看该命令是否激活。这个接口有一个激活状态属性和一个激活状态改变事件，

![1585989967522](/prism/1585989967522.png)

只要界面主动调用UpdateCommand.IsActive = true;命令就会被激活并触发复合命令的激活状态改变回调函数

![1586001088755](/prism/1586001088755.png)

![1586001120188](/prism/1586001120188.png)

在复合命令的ShouldExcute方法中检查其激活状态，命令集合中没有激活命令，那么复合命令的执行状态也会改变。

命令执行和激活状态都是差不多的接口，有状态和状态改变事件组成，感觉很多地方都有相似的模式，有点像订阅模式，也有点像状态机，包括一些Collection和Storage.

## 总结

主要讲了下Prism提供的绑定通知和命令，学习到如何在WPF框架基础上做一些封装，如果可能甚至可以自己重新封装一些功能更强大的命令来兼容Prism。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/prism19-source/  

