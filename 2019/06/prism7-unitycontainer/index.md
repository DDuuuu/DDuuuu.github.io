# Prism UnityContainer


注意：该版本为Prism6,最新版已有较大改动。

Managing Dependencies Between Components Using the Prism Library for WPF

基于Prism库的应用程序是复合应用程序，可能包含许多松散耦合的类型和服务。他们需要进行交互以提供内容并根据用户操作接收通知。因为它们是松散耦合的，所以它们需要一种相互交互和通信的方式来提供所需的业务功能。为了将这些不同的部分组合在一起，基于Prism库的应用依赖于依赖注入容器。

依赖注入容器通过提供实例化类实例的工具并根据容器的配置管理其生命周期来减少对象之间的依赖关系。在对象创建期间，容器会将对象所需的所有依赖项注入其中。如果尚未创建这些依赖项，则容器首先创建并解析它们的依赖项。在某些情况下，容器本身被解析为依赖项。例如，当使用Unity应用程序块（Unity）作为容器时，模块会注入容器，因此可以使用该容器注册其视图和服务。

使用容器有几个好处：

- 容器不需要组件来定位其依赖项或管理它们的生命周期。
- 容器允许交换已实现的依赖项而不影响组件。
- 容器通过允许模拟依赖项来促进可测试性。
- 容器通过允许将新组件轻松添加到系统中来提高可维护性。

在基于Prism库的应用程序的上下文中，容器具有特定的优点：

- 容器在加载时将模块依赖项注入模块。
- 容器用于注册和解析视图模型和视图。
- 容器可以创建视图模型并注入视图。
- 容器注入组合服务，例如区域管理器和事件聚合器。
- 容器用于注册特定于模块的服务，这些服务是具有模块特定功能的服务。

**注意：** Prism指南中的某些示例依赖Unity应用程序块（Unity）作为容器。其他代码示例（例如Modularity QuickStarts）使用Managed Extensibility Framework（MEF）。Prism库本身不是特定于容器的，您可以将其服务和模式与其他容器一起使用，例如Castle Windsor，StructureMap和Spring.NET。

### 关键决策：选择依赖注入容器

Prism Library为依赖注入容器提供了两个选项：Unity或MEF。棱镜是可扩展的，从而允许使用其他容器而不需要一点工作。Unity和MEF都为依赖注入提供了相同的基本功能，即使它们的工作方式非常不同。两个容器提供的一些功能包括：

- 它们都使用容器注册类型。
- 他们都用容器注册实例。
- 它们都强制创建已注册类型的实例。
- 它们都将注册类型的实例注入到构造函数中。
- 它们都将已注册类型的实例注入属性。
- 它们都具有用于标记需要管理的类型和依赖项的声明性属性。
- 它们都解决了对象图中的依赖关系。

Unity提供了MEF不具备的几种功能：

- 它解决了没有注册的具体类型。
- 它解决了开放的泛型。
- 它使用拦截来捕获对象的调用并向目标对象添加其他功能。

MEF提供了Unity不具备的几种功能：

- 它发现目录中的程序集。
- 它使用XAP文件下载和程序集发现。
- 它会在发现新类型时重新组合属性和集合。
- 它会自动导出派生类型。
- 它与.NET Framework一起部署。

容器具有不同的功能和不同的工作方式，但Prism库将与容器一起使用并提供类似的功能。在考虑使用哪个容器时，请记住前面的功能并确定哪种容量更适合您的方案。

### 使用容器的注意事项

在使用容器之前，您应该考虑以下事项：

- 考虑使用容器注册和解析组件是否合适：
  - 考虑在您的方案中是否可以接受向容器注册和从中解析实例的性能影响。例如，如果需要创建10,000个多边形以在渲染方法的局部范围内绘制曲面，则通过容器解析所有这些多边形实例的成本可能会产生显着的性能成本，因为容器使用反射来创建每个实体。
  - 如果存在许多或深度依赖性，则创建成本会显着增加。
  - 如果组件没有任何依赖关系或者不是其他类型的依赖关系，那么将它放在容器中可能没有意义。
  - 如果组件具有一组与该类型不可分割的依赖关系并且永远不会更改，则将其放入容器中可能没有意义。
- 考虑组件的生命周期是否应该注册为单例或实例：
  - 如果组件是充当单个资源（例如日志记录服务）的资源管理器的全局服务，则可能需要将其注册为单例。
  - 如果组件为多个使用者提供共享状态，您可能希望将其注册为单例。
  - 如果正在注入的对象需要在每次依赖对象需要时注入一个新实例，请将其注册为非单例。例如，每个视图可能需要一个视图模型的新实例。
- 考虑是否要通过代码或配置配置容器：
  - 如果要集中管理所有不同的服务，请通过配置配置容器。
  - 如果要有条件地注册特定服务，请通过代码配置容器。
  - 如果您有模块级服务，请考虑通过代码配置容器，以便仅在加载模块时注册这些服务。

**注意：**某些容器（如MEF）无法通过配置文件进行配置，必须通过代码进行配置。

### 核心情景

容器用于两个主要目的，即注册和解析。

#### 注册

在将依赖项注入对象之前，需要向容器注册依赖项的类型。注册类型通常涉及向容器传递接口和实现该接口的具体类型。注册类型和对象主要有两种方法：通过代码或通过配置。具体方式因容器而异。

通常，有两种方法可以通过代码在容器中注册类型和对象：

- 您可以使用容器注册类型或映射。在适当的时候，容器将构建您指定的类型的实例。
- 您可以将容器中的现有对象实例注册为单例。容器将返回对现有对象的引用。

#### 使用Unity容器注册类型

在初始化期间，类型可以注册其他类型，例如视图和服务。注册允许通过容器提供其依赖项，并允许从其他类型访问它们。要做到这一点，类型将需要将容器注入模块构造函数。以下代码显示了命令QuickStart中的**OrderModule**类型如何注册类型。

```c#
// OrderModule.cs
public class OrderModule : IModule
{
    public void Initialize()
    {
        this.container.RegisterType&lt;IOrdersRepository, OrdersRepository&gt;(new ContainerControlledLifetimeManager());
        ...
    }
    ...
}
```

根据您使用的容器，也可以通过配置在代码外部执行注册。有关此示例，请参阅。

**注意：**与配置相比，在代码中注册的优点是只有在模块加载时才会进行注册。

#### 使用MEF注册类型

MEF使用基于属性的系统来向容器注册类型。因此，向容器添加类型注册很简单：它需要在类型中添加**[Export]**属性，如下面的代码示例所示。

```c#
[Export(typeof(ILoggerFacade))]
public class CallbackLogger: ILoggerFacade
{
}
```

使用MEF时的另一个选择是创建类的实例并使用容器注册该特定实例。带有MEF QuickStart的Modularity中的**QuickStartBootstrapper**在**ConfigureContainer**方法中显示了一个示例，如下所示。

```c#
protected override void ConfigureContainer()
{
    base.ConfigureContainer();

    // Because we created the CallbackLogger and it needs to
    // be used immediately, we compose it to satisfy any imports it has.
    this.Container.ComposeExportedValue&lt;CallbackLogger&gt;(this.callbackLogger);
}
```

**注意：**使用MEF作为容器时，建议您使用属性来注册类型。

### Resolving

注册类型后，可以将其解析或注入为依赖项。在解析类型并且容器需要创建新实例时，它会将依赖项注入这些实例。

通常，在解析类型时，会发生以下三种情况之一：

- 如果尚未注册该类型，则容器会引发异常。

  **注意：**某些容器（包括Unity）允许您解析尚未注册的具体类型。

- 如果类型已注册为单例，则容器将返回单例实例。如果这是第一次调用该类型，则容器会创建它并保留它以供将来调用。

- 如果类型尚未注册为单例，则容器将返回新实例。

  **注意：**默认情况下，使用MEF注册的类型是单例，容器包含对象的引用。在Unity中，默认情况下会返回新的对象实例，并且容器不会维护对该对象的引用。

### 使用Unity解析实例

命令快速入门中的以下代码示例显示了从容器中解析**OrdersEditorView**和**OrdersToolBar**视图的位置，以将它们与相应的区域相关联。

```c#
// OrderModule.cs
public class OrderModule : IModule
{
    public void Initialize()
    {
        this.container.RegisterType&lt;IOrdersRepository, OrdersRepository&gt;(new ContainerControlledLifetimeManager());

        // Show the Orders Editor view in the shell&#39;s main region.
        this.regionManager.RegisterViewWithRegion(&#34;MainRegion&#34;,
            () =&gt; this.container.Resolve&lt;OrdersEditorView&gt;());

        // Show the Orders Toolbar view in the shell&#39;s toolbar region.
        this.regionManager.RegisterViewWithRegion(&#34;GlobalCommandsRegion&#34;,
            () =&gt; this.container.Resolve&lt;OrdersToolBar&gt;());
    }
    ...
}
```

该**OrdersEditorViewModel**构造包含以下依赖（订单仓库和订单命令代理），当其解决注入。

```c#
// OrdersEditorViewModel.cs
public OrdersEditorViewModel(IOrdersRepository ordersRepository, OrdersCommandProxy commandProxy)
{
    this.ordersRepository = ordersRepository;
    this.commandProxy     = commandProxy;

    // Create dummy order data.
    this.PopulateOrders();

    // Initialize a CollectionView for the underlying Orders collection.
    this.Orders = new ListCollectionView( _orders );
    // Track the current selection.
    this.Orders.CurrentChanged &#43;= SelectedOrderChanged;
    this.Orders.MoveCurrentTo(null);
}
```

除了前面代码中显示的构造函数注入之外，Unity还允许注入属性。应用**[Dependency]**属性的任何属性将在解析对象时自动解析并注入。

### 使用MEF解析实例

以下代码示例显示了使用MEF QuickStart的Modularity中的**Bootstrapper**如何获取shell的实例。代码可以请求接口的实例，而不是请求具体类型。

```c#
protected override DependencyObject CreateShell()
{
    return this.Container.GetExportedValue&lt;Shell&gt;();
}
```

在MEF解析的任何类中，您也可以使用构造函数注入，如下面的模块化与MEF QuickStart中的ModuleA中的代码示例所示，其中注入了**ILoggerFacade**和**IModuleTracker**。

```c#
[ImportingConstructor]
public ModuleA(ILoggerFacade logger, IModuleTracker moduleTracker)
{
    if (logger == null)
    {
        throw new ArgumentNullException(&#34;logger&#34;);
    }
    if (moduleTracker == null)
    {
        throw new ArgumentNullException(&#34;moduleTracker&#34;);
    }
    this.logger = logger;
    this.moduleTracker = moduleTracker;
    this.moduleTracker.RecordModuleConstructed(WellKnownModuleNames.ModuleA);
}
```

另一种选择是使用属性注入，如Modularity with MEF QuickStart 中的**ModuleTracker**类所示，其中注入了**ILoggerFacade**的实例。

```c#
[Export(typeof(IModuleTracker))]
public class ModuleTracker : IModuleTracker
{
    [Import] private ILoggerFacade Logger;
}
```

### 在Prism中使用依赖注入容器和服务

依赖注入容器（通常称为“容器”）用于满足组件之间的依赖关系; 满足这些依赖性通常涉及注册和解决。Prism Library提供对Unity容器和MEF的支持，但它不是特定于容器的。因为库通过**IServiceLocator**接口访问容器，所以可以替换容器。为此，您的容器必须实现**IServiceLocator**接口。通常，如果要更换容器，则还需要提供自己的容器特定引导程序。该**IServiceLocator**接口在Common Service Locator Library中定义。这是一项开源工作，旨在提供IoC（控制反转）容器的抽象，例如依赖注入容器和服务定位器。使用此库的目的是利用IoC和服务位置，而不必与特定实现相关联。

Prism库提供**UnityServiceLocatorAdapter**和**MefServiceLocatorAdapter**。两个适配器都通过扩展**ServiceLocatorImplBase**类型来实现**ISeviceLocator**接口。下图显示了类层次结构。

![image-20240701161454029](/prism/image-20240701161454029.png)

虽然Prism Library不引用或依赖于特定容器，但应用程序通常依赖于特定容器。这意味着特定应用程序引用容器是合理的，但Prism Library不直接引用容器。例如，Stock Trader RI和Prism附带的几个QuickStart依赖Unity作为容器。其他样品和快速入门依赖于MEF。

### IServiceLocator

以下代码显示了**IServiceLocator**接口。

```cs
public interface IServiceLocator : IServiceProvider
{
    object GetInstance(Type serviceType);
    object GetInstance(Type serviceType, string key);
    IEnumerable&lt;object&gt; GetAllInstances(Type serviceType);
    TService GetInstance&lt;TService&gt;();
    TService GetInstance&lt;TService&gt;(string key);
    IEnumerable&lt;TService&gt; GetAllInstances&lt;TService&gt;();
}
```

服务定位器在Prism库中扩展，扩展方法如下面的代码所示。您可以看到**IServiceLocator**仅用于解析，这意味着它用于获取实例; 它不用于注册。

```c#
// ServiceLocatorExtensions
public static class ServiceLocatorExtensions
{
    public static object TryResolve(this IServiceLocator locator, Type type)
    {
        try
        {
            return locator.GetInstance(type);
        }
        catch (ActivationException)
        {
            return null;
        }
    }

    public static T TryResolve&lt;T&gt;(this IServiceLocator locator) where T: class
    {
        return locator.TryResolve(typeof(T)) as T;
    }
}
```

Unity容器不支持的**TryResolve**扩展方法 - 如果已注册，则返回要解析的类型的实例; 否则，它返回**null**。

所述**ModuleInitializer**使用**IServiceLocator**为加载模块期间解析模块，作为显示在下面的代码示例。

```c#
// ModuleInitializer.cs - Initialize()
IModule moduleInstance = null;
try
{
    moduleInstance = this.CreateModule(moduleInfo);
    moduleInstance.Initialize();
}
...
// ModuleInitializer.cs - CreateModule()
protected virtual IModule CreateModule(string typeName)
{
    Type moduleType = Type.GetType(typeName);

    if (moduleType == null)
    {
        throw new ModuleInitializeException(string.Format(CultureInfo.CurrentCulture, Properties.Resources.FailedToGetType, typeName));
    }

    return (IModule)this.serviceLocator.GetInstance(moduleType);
}
```

### 使用IServiceLocator的注意事项

**IServiceLocator**并不是通用容器。容器具有不同的使用语义，这通常决定了为什么选择容器。考虑到这一点，Stock Trader RI直接使用依赖注入容器而不是使用**IServiceLocator**。这是您的应用程序开发的推荐方法。

在以下情况下，您可能适合使用**IServiceLocator**：

- 您是一家独立软件供应商（ISV），负责设计需要支持多个容器的第三方服务。
- 您正在设计一个服务，以便在使用多个容器的组织中使用。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism7-unitycontainer/  

