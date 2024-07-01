# Prism Composite Commands


 

注意：该版本为Prism6,最新版已有较大改动。

在许多情况下，视图模型定义的命令将绑定到关联视图中的控件，以便用户可以直接从视图中调用该命令。但是，在某些情况下，您可能希望能够从应用程序UI的父视图中的控件调用一个或多个视图模型上的命令。

例如，如果您的应用程序允许用户同时编辑多个项目，您可能希望允许用户使用应用程序工具栏或功能区中的按钮所代表的单个命令来保存所有项目。在这种情况下，Save All命令将调用每个项目的视图模型实例实现的每个Save命令，如下图所示。

![SaveAllå¤åå½ä»¤](/prism/composite-commands-1.png)

Prism通过`CompositeCommand`课程支持这种情况。

`CompositeCommand`类表示从多个子指令构成的指令。调用复合命令时，将依次调用其每个子命令。在需要在UI中将一组命令表示为单个命令或者要调用多个命令来实现逻辑命令的情况下，它非常有用。

本`CompositeCommand`类维护子类命令列表 （`DelegateCommand`实例）。`CompositeCommand`类的`Execute`方法依次调用每个子类的`Execute`命令方法。`CanExecute`方法类似地调用每个子命令的`CanExecute`方法，但是如果无法执行任何子命令，则该`CanExecute`方法将返回`false`。换句话说，默认情况下，`CompositeCommand`只有在可以执行所有子命令时才能执行。

### 创建Composite Commands

要创建复合命令，请实例化一个`CompositeCommand`实例，然后将其公开为`ICommand`或`ComponsiteCommand`属性。

```c#
public class ApplicationCommands
{
    private CompositeCommand _saveCommand = new CompositeCommand();
    public CompositeCommand SaveCommand
    {
    	get { return _saveCommand; }
    }
}
```

### 使得Composite Commands全局可用

通常，CompositeCommands在整个应用程序中共享，需要全局可用。当您`CompositeCommand`在整个应用程序中使用同一个CompositeCommand实例注册子命令时，这一点很重要。这需要在应用程序中将CompositeCommand作为单例取消。这可以通过使用依赖注入（DI）或将CompositeCommand定义为静态类来完成。

#### 使用依赖注入

定义CompositeCommands的第一步是创建一个接口。

```c#
public interface IApplicationCommands
{
    CompositeCommand SaveCommand { get; }
}
```

接下来，创建一个实现该接口的类。

```c#
public class ApplicationCommands : IApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand();
    public CompositeCommand SaveCommand
    {
    	get { return _saveCommand; }
    }
}
```

定义ApplicationCommands类后，必须将其注册为容器的单例。

```c#
public partial class App : PrismApplication
{
	protected override void RegisterTypes(IContainerRegistry containerRegistry)
    {
  		containerRegistry.RegisterSingleton&lt;IApplicationCommands, ApplicationCommands&gt;();
    }
}
```

接下来，请求`IApplicationCommands`ViewModel构造函数中的接口。一旦有了`ApplicationCommands`该类的实例，现在可以使用适当的CompositeCommand注册DelegateCommands。

```c#
public DelegateCommand UpdateCommand { get; private set; }

public TabViewModel(IApplicationCommands applicationCommands)
{
    UpdateCommand = new DelegateCommand(Update);
    applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
}
```

#### 使用静态类

创建一个代表CompositeCommands的静态类

```c#
public static class ApplicationCommands
{
    public static CompositeCommand SaveCommand = new CompositeCommand();
}
```

在ViewModel中，将子命令与静态`ApplicationCommands`类关联。

```cs
    public DelegateCommand UpdateCommand { get; private set; }

    public TabViewModel()
    {
        UpdateCommand = new DelegateCommand(Update);
        ApplicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
    }
```

注意

为了提高代码的可维护性和可测试性，建议您使用可靠性注入方法。

### 绑定全局可用命令

一旦创建了CompositeCommands，就必须将它们绑定到UI元素以调用命令。

#### 使用依赖注入 (Dependency Injection)

使用DI时，必须将`IApplicationCommands`暴露给View。在视图的ViewModel中，请`IApplicationCommands`在构造函数中请求并`IApplicationCommands`为该实例设置type属性。

```c#
public class MainWindowViewModel : BindableBase
{
        private IApplicationCommands _applicationCommands;
        public IApplicationCommands ApplicationCommands
        {
            get { return _applicationCommands; }
            set { SetProperty(ref _applicationCommands, value); }
        }

        public MainWindowViewModel(IApplicationCommands applicationCommands)
        {
            ApplicationCommands = applicationCommands;
        }
}
```

在视图中，将按钮绑定到`ApplicationCommands.SaveCommand`属性。这`SaveCommand`是在`ApplicationCommands`类上定义的属性。

```xaml
&lt;Button Content=&#34;Save&#34; Command=&#34;{Binding ApplicationCommands.SaveCommand}&#34;/&gt;
```

#### 使用静态类

如果您使用的是静态类方法，则以下代码示例演示如何将按钮绑定到WPF中的静态ApplicationCommands类。

```xml
&lt;Button Content=&#34;Save&#34; Command=&#34;{x:Static local:ApplicationCommands.SaveCommand}&#34; /&gt;
```

### 取消注册命令

如前面的示例所示，使用该`CompositeCommand.RegisterCommand`方法注册子命令。但是，当您不再希望响应CompositeCommand或者您要销毁View / ViewModel以进行垃圾回收时，您应该使用该`CompositeCommand.UnregisterCommand`方法取消注册子命令。

```c#
public void Destroy()
{
    _applicationCommands.UnregisterCommand(UpdateCommand);
}
```

##### 重要

您必须从`CompositeCommand`不再需要View / ViewModel（为GC准备好）时取消注册您的命令。否则你会引入内存泄漏。

### 在活动视图上执行命令

父视图级别的复合命令通常用于协调如何调用子视图级别的命令。在某些情况下，您将需要执行所有显示视图的命令，如前面所述的Save All命令示例中所示。在其他情况下，您将希望仅在活动视图上执行该命令。在这种情况下，复合命令将仅对被视为活动的视图执行子命令; 它不会在非活动的视图上执行子命令。例如，您可能希望在应用程序的工具栏上实现缩放命令，该命令仅导致当前活动项目被缩放，如下图所示。

![Executing a CompositeCommand on a single child](/prism/composite-commands-2.png)

为了支持这种情况，Prism提供了`IActiveAware`界面。该`IActiveAware`接口定义了一个`IsActive`返回属性`true`时实施者是活动的，并且`IsActiveChanged`当活动状态改变时引发事件。

您可以`IActiveAware`在视图或ViewModel上实现该接口。它主要用于跟踪视图的活动状态。视图是否处于活动状态由特定控件内的视图决定。例如，对于Tab控件，有一个适配器将当前选定选项卡中的视图设置为活动状态。

本`DelegateCommand`类还实现了`IActiveAware`接口。`CompositeCommand`可被配置成评估子类DelegateCommands的活动状态（除了`CanExecute`通过指定状态）`true`为`monitorCommandActivity`在构造参数。当此参数设置为时`true`，`CompositeCommand`类在确定方法的返回值以及在`CanExecute`方法中执行子命令时将**考虑每个子DelegateCommand的活动状态**`Execute`。

```cs
    public class ApplicationCommands : IApplicationCommands
    {
        private CompositeCommand _saveCommand = new CompositeCommand(true);
        public CompositeCommand SaveCommand
        {
            get { return _saveCommand; }
        }
    }
```

当`monitorCommandActivity`参数为时`true`，`CompositeCommand`该类表现出以下行为：

- `CanExecute`：`true`**仅在可以执行所有活动命令时返回**。根本不会考虑不活动的子命令。
- `Execute`：执行所有活动命令。根本不会考虑不活动的子命令。

通过`IActiveAware`在ViewModel上实现界面，当您的视图变为活动或非活动时，您将收到通知。当视图的活动状态更改时，您可以更新子命令的活动状态。然后，当用户调用复合命令时，将调用活动子视图上的命令。

```c#
public class TabViewModel : BindableBase, IActiveAware
    {
        private bool _isActive;
        public bool IsActive
        {
            get { return _isActive; }
            set
            {
                _isActive = value;
                OnIsActiveChanged();
            }
        }

        public event EventHandler IsActiveChanged;

        public DelegateCommand UpdateCommand { get; private set; }

        public TabViewModel(IApplicationCommands applicationCommands)
        {
            UpdateCommand = new DelegateCommand(Update);
            applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
        }

        private void Update()
        {
            //implement logic
        }

        private void OnIsActiveChanged()
        {
            UpdateCommand.IsActive = IsActive; //set the command as active
            IsActiveChanged?.Invoke(this, new EventArgs()); //invoke the event for al listeners
        }
    }
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism3-compositecommands/  

