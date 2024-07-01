# Prism Commanding


 

注意：该版本为Prism6,最新版已有较大改动。

除了提供对要在视图中显示或编辑的数据的访问之外，ViewModel还可能定义可由用户执行的一个或多个动作或操作。用户可以通过UI执行的动作或操作通常被定义为命令。命令提供了一种方便的方法来表示可以轻松绑定到UI中的控件的操作或操作。它们封装了实现操作或操作的实际代码，并有助于使其与视图中的实际可视化表示分离。

当用户与视图交互时，用户可以以多种不同的方式直观地表示和调用命令。在大多数情况下，它们是通过鼠标单击调用的，但也可以通过快捷键按下，触摸手势或任何其他输入事件来调用它们。视图中的控件是绑定到ViewModel命令的数据，以便用户可以使用控件定义的任何输入事件或手势来调用它们。视图中的UI控件与命令之间的交互可以是双向的。在这种情况下，可以在用户与UI交互时调用该命令，并且可以在启用或禁用基础命令时自动启用或禁用UI。

ViewModel可以将**命令**实现为**命令对象**（实现`ICommand`接口的对象）。可以以声明方式定义视图与命令的交互，而无需在视图的代码隐藏文件中使用复杂的事件处理代码。例如，某些控件固有地支持命令并提供`Command`可以是绑定到`ICommand`ViewModel提供的对象的数据的属性。在其他情况下，命令行为可用于将控件与ViewModel提供的命令方法或命令对象相关联。

实现`ICommand`界面很简单。Prism提供了`DelegateCommand`这个界面的实现，您可以在应用程序中轻松使用它。

### DelegateCommand

Prism `DelegateCommand`类封装了两个委托，每个委托引用在ViewModel类中实现的方法。它通过调用这些委托来实现`ICommand`接口`Execute`和`CanExecute`方法。您可以在`DelegateCommand`类构造函数中指定ViewModel方法的委托。例如，以下代码示例显示如何`DelegateCommand`通过指定OnSubmit和CanSubmit ViewModel方法的委托来构造表示Submit命令的实例。然后，该命令通过只读属性公开给视图，该属性返回对该参数的引用`DelegateCommand`。

```c#
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand&lt;object&gt;(Submit, CanSubmit);
    }

    void Submit(object parameter)
    {
        //implement logic
    }

    bool CanSubmit(object parameter)
    {
        return true;
    }
}
```

在`DelegateCommand`对象上调用Execute方法时，它只是通过您在构造函数中指定的委托将调用转发到ViewModel类中的方法。同样，`CanExecute`调用该方法时，将调用ViewModel类中的相应方法。`CanExecute`构造函数中方法的委托是可选的。如果没有指定一个委托，`DelegateCommand`将始终返回`true`了`CanExecute`。

该`DelegateCommand`班是一个泛型类型。type参数指定传递给`Execute`和`CanExecute`方法的命令参数的类型。在前面的示例中，command参数是type `object`。**非通用**的版本`DelegateCommand`类也通过棱镜用于提供当没有所需的命令参数，并且被定义为如下：

```c#
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit, CanSubmit);
    }

    void Submit()
    {
        //implement logic
    }

    bool CanSubmit()
    {
        return true;
    }
}
```

所述`DelegateCommand`故意阻止使用值类型（int，双，布尔等）。因为`ICommand`需要一个`object`具有值类型`T`将导致当意外行为`CanExecute(null)`XAML初始化命令绑定期间被调用。使用`default(T)`被认为是被拒绝作为解决方案，因为实现者无法区分有效值和默认值。如果要将值类型用作参数，则必须使用`DelegateCommand&lt;Nullable&lt;int&gt;&gt;`或使用简写`?`语法（`DelegateCommand&lt;int?&gt;`）使其可为空。

### 从View调用DelegateCommands

有许多方法可以将视图中的控件与ViewModel提供的命令对象相关联。某些WPF，Xamarin.Forms和UWP控件可以通过`Command`属性轻松地绑定到命令对象。

```xml
&lt;Button Command=&#34;{Binding SubmitCommand}&#34; CommandParameter=&#34;OrderId&#34;/&gt;
```

也可以使用`CommandParameter`属性选择性地定义命令参数。期望参数的类型在`DelegateCommand&lt;T&gt;`泛型声明中指定。当用户与该控件交互时，控件将自动调用目标命令，并且命令参数（如果提供）将作为参数传递给命令的`Execute`方法。在前面的示例中，按钮将`SubmitCommand`在单击时自动调用。此外，如果`CanExecute`指定了委托，则`CanExecute`返回时将自动禁用该按钮，如果返回`false`则将启用该按钮`true`。

### Raising Change Notifications

ViewModel通常需要指示命令`CanExecute`状态的更改，以便UI中绑定到该命令的任何控件都将更新其启用状态以反映绑定命令的可用性。在`DelegateCommand`提供了几种这些通知发送到用户界面。

#### RaiseCanExecuteChanged

`RaiseCanExecuteChanged`每当需要手动更新绑定的UI元素的状态时，请使用该方法。例如，当`IsEnabled`属性值更改时，我们调用`RaiseCanExecuteChanged`属性的setter来通知UI状态更改。

```c#
private bool _isEnabled;
public bool IsEnabled
{
    get { return _isEnabled; }
    set
    {
        SetProperty(ref _isEnabled, value);
        SubmitCommand.RaiseCanExecuteChanged();
    }
}
```

#### ObservesProperty

如果命令应在属性值更改时发送通知，则可以使用该`ObservesProperty`方法。使用该`ObservesProperty`方法时，只要提供的属性的值发生更改，`DelegateCommand`将自动调用`RaiseCanExecuteChanged`以通知UI状态更改。

```c#
public class ArticleViewModel : BindableBase
{
    private bool _isEnabled;
    public bool IsEnabled
    {
        get { return _isEnabled; }
        set { SetProperty(ref _isEnabled, value); }
    }

    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit, CanSubmit).ObservesProperty(() =&gt; IsEnabled);
    }

    void Submit()
    {
        //implement logic
    }

    bool CanSubmit()
    {
        return IsEnabled;
    }
}
```

注意

使用该`ObservesProperty`方法时，您可以链接注册多个属性以供观察。示例：`ObservesProperty(() =&gt; IsEnabled).ObservesProperty(() =&gt; CanSave)`。

#### ObservesCanExecute

如果您`CanExecute`是简单`Boolean`属性的结果，则可以省去声明`CanExecute`委托，并使用该`ObservesCanExecute`方法。`ObservesCanExecute`当注册的属性值发生变化时，它不仅会向UI发送通知，而且还会使用与实际`CanExecute`委托相同的属性。

```c#
public class ArticleViewModel : BindableBase
{
    private bool _isEnabled;
    public bool IsEnabled
    {
        get { return _isEnabled; }
        set { SetProperty(ref _isEnabled, value); }
    }

    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit, CanSubmit).ObservesCanExecute(() =&gt; IsEnabled);
    }

    void Submit()
    {
        //implement logic
    }
}
```

##### 警告

不要尝试链式注册`ObservesCanExecute`方法。`CanExcute`代表只能观察到一个属性。

### 基于Task-Based的DelegateCommand

`async`/`await`调用`Execute`委托内部的异步方法是一个非常常见的要求。每个人的第一直觉是他们需要一个`AsyncCommand`，但这种假设是错误的。`ICommand`本质上是同步的，并且代表`Execute`和`CanExecute`代表应被视为事件。这意味着这`async void`是一个非常有效的语法用于命令。使用异步方法有两种方法`DelegateCommand`。

&#43; 方法一

```c#
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit);
    }

    async void Submit()
    {
        await SomeAsyncMethod();
    }
}
```

&#43; 方法二

```c#
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(async ()=&gt; await Submit());
    }

    Task Submit()
    {
        return SomeAsyncMethod();
    }
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism2-commanding/  

