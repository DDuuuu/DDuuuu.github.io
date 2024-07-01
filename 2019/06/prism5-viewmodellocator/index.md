# Prism ViewModel Locator


 

注意：该版本为Prism6,最新版已有较大改动。

`ViewModelLocator`用于绑定视图的`DataContext`，以使用标准命名约定的一个ViewModel的实例。

Prism `ViewModelLocator`有一个`AutoWireViewModel`附加属性，当设置为`true`调用类中的`AutoWireViewModelChanged`方法`ViewModelLocationProvider`来解析视图的ViewModel时，然后将视图的数据上下文设置为该ViewModel的实例。

将`AutoWireViewModel`附加属性添加到每个视图：

```
&lt;Window x:Class=&#34;Demo.Views.MainWindow&#34;
    ...
    xmlns:prism=&#34;http://prismlibrary.com/&#34;
    prism:ViewModelLocator.AutoWireViewModel=&#34;True&#34;&gt;
```

要查找ViewModel，`ViewModelLocationProvider`首先尝试从`ViewModelLocationProvider.Register`方法注册的任何映射中解析ViewModel （请参阅**自定义ViewModel注册**）。如果使用此方法无法解析ViewModel，则会`ViewModelLocationProvider`回退到基于约定的方法来解析正确的ViewModel类型。

该惯例假定：

- **ViewModel与视图类型位于同一个程序集中**
- **ViewModel位于`.ViewModels`子命名空间中**
- **视图位于`.Views`子命名空间中**
- **ViewModel名称与视图名称对应，以“ViewModel”结尾。**

**注意**

本`ViewModelLocationProvider`可以在发现`Prism.Mvvm`命名空间中的**Prism.Core** NuGet包。本`ViewModelLocator`可以在发现`Prism.Mvvm`命名空间中的**Prism.WPF** NuGet包。

**注意**

**ViewModelLocator是必需的**，并且在使用Xamarin.Forms进行开发时会自动应用于每个View，因为它负责向`INavigationService`ViewModel 提供正确的实例。在开发Xamarin.Forms应用程序时，`ViewModelLocator`只能选择退出。

### 更改命名约定

如果您的应用程序不遵循`ViewModelLocator`默认命名约定，则可以更改约定以满足应用程序的要求。本`ViewModelLocationProvider`类提供了一个称为静态方法`SetDefaultViewTypeToViewModelTypeResolver`，可以用来提供自己的约定关联视图查看模型。

要更改`ViewModelLocator`命名约定，请覆盖类中的`ConfigureViewModelLocator`方法`App.xaml.cs`。然后在`ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver`方法中提供自定义命名约定逻辑。

```c#
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();
 ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver((viewType) =&gt;
    {
        var viewName = viewType.FullName.Replace(&#34;.ViewModels.&#34;, &#34;.CustomNamespace.&#34;);//看视频就明白了
        var viewAssemblyName = viewType.GetTypeInfo().Assembly.FullName;
        var viewModelName = $&#34;{viewName}ViewModel, {viewAssemblyName}&#34;;
        return Type.GetType(viewModelName);
    });
}
```



### 自定义ViewModel注册

可能存在您的应用程序遵循`ViewModelLocator`默认命名约定的情况，但您有许多不符合约定的ViewModel。您可以`ViewModelLocator`使用该`ViewModelLocationProvider.Register`方法直接将ViewModel的映射注册到特定视图，而不是尝试自定义命名约定逻辑以有条件地满足所有命名要求。

以下示例显示了在名为`MainWindow`的ViewModel和ViewModel 之间创建映射的各种方法`CustomViewModel`。

**类型/类型**

```c#
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), typeof(CustomViewModel));
```

**类型/工厂**

```c#
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), () =&gt; Container.Resolve&lt;CustomViewModel&gt;());
```

**通用工厂**

```c#
ViewModelLocationProvider.Register&lt;MainWindow&gt;(() =&gt; Container.Resolve&lt;CustomViewModel&gt;());
```

**通用类型**

```c#
ViewModelLocationProvider.Register&lt;MainWindow, CustomViewModel&gt;();
```

##### 注意

直接注册ViewModel `ViewModelLocator`比依赖默认命名约定更快。这是因为命名约定需要使用反射，而自定义映射直接提供类型`ViewModelLocator`。

##### 重要

该`viewTypeName`参数必须是视图的Type（`Type.ToString()`）的完全限定名称。否则映射将失败。



### 控制ViewModel的解析方式

默认情况下，`ViewModelLocator`将使用您选择的DI容器来创建Prism应用程序以解析ViewModels。但是，如果您需要自定义ViewModel的解析方式或完全更改解析器，则可以使用该`ViewModelLocationProvider.SetDefaultViewModelFactory`方法实现此目的。

此示例显示如何更改用于解析ViewModel实例的容器。

```C#
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewModelFactory(viewModelType) =&gt;
    {
        return MyAwesomeNewContainer.Resolve(viewModelType);
    });
}
```

这是一个示例，说明如何检查为其创建ViewModel的视图类型，以及执行逻辑来控制ViewModel的创建方式。

```c#
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewModelFactory((view, viewModelType) =&gt;
    {
        switch (view)
        {
            case Window window:
                //your logic
                break;
            case UserControl userControl:
                //your logic
                break;
        }

        return MyAwesomeNewContainer.Resolve(someNewType);
    });
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism5-viewmodellocator/  

