# Prism源码解析 导航


注意：该版本为Prism6,最新版已有较大改动。

## 介绍

Prism提供了一个非常强大的功能导航，导航的意思就是指定对应的View显示。这个导航的强大之处有：

&#43; 可以设置导航前后的动作
&#43; 可以指定View实例的生命周期，可以是否导航到新的View实例
&#43; 提供了确认导航接口。

![Confirming Navigation Using an InteractionRequest Object ](/prism/gg430861.430462631ab962dccbce65c298a2eed8(en-us,pandp.40).png)

&#43; 导航前后均有相应的事件通知
&#43; 提供了回退前进的导航功能

## 导航

直接看代码

![1586101856797](/prism/1586101856797.png)

![1586101868352](/prism/1586101868352.png)

可以看到直接通过RequstNavigate来请求，参数是View的TypeName

![1586101921254](/prism/1586101921254.png)

转到了Region.RequestNavigate里

![1586101954617](/prism/1586101954617.png)

这边出现了NavigationService，几乎所有的导航功能都是在这个服务中实现的，

![1586102614643](/prism/1586102614643.png)

![1586101993357](/prism/1586101993357.png)

![1586102012245](/prism/1586102012245.png)

这边将导航的一些信息封装成NavigationContext,

![1586102044383](/prism/1586102044383.png)

在这出现了第一个功能，实现ICon&#39;firm&#39;NavigationRequest接口，确认导航。

最后来到了最重要的函数ExecuteNavigation

![1586102124583](/prism/1586102124583.png)

这个函数每一行都很重要，每一行都是一个功能。

![1586102153448](/prism/1586102153448.png)

调用OnNavigateFrom,可以在导航前做一些操作

![1586102182080](/prism/1586102182080.png)

获取导航内容，先从Region的View中找，没找到就到容器中找，然后添加到Region的View。

![1586102280441](/prism/1586102280441.png)

激活界面

创建条目，保存条目，主要用来进行前进后退

触发导航完成事件。

整个导航功能的顺序：

![Prism region navigation sequence](/prism/gg430861.e8eaa02d7a2d9a1370c3bf6d79636697(en-us,pandp.40).png)

## 总结

​	Prism提供的这个导航功能非常强大，但是代码却不复杂，通过一些简单的接口，实现了非常强大的功能。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/prism21-source/  

