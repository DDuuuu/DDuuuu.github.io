# Prism 使用Prism


##  使用Prism

注意：该版本为Prism6,最新版已有较大改动。

使用Prism Library开发复合应用程序时所需的常见活动。

![活动](/prism/Ch1IntroFig4.png)

此示例，描述了创建由定义单个视图的单个模块组成的基本Prism应用程序所需的步骤。

### 定义Shell

应用程序shell提供应用程序的基本布局。此布局使用模块可用于放置视图的区域定义。Views和shells可以使用区域来定义可添加内容的可发现区域，如下图所示。Shells通常设置整个应用程序的外观，并包含整个应用程序中使用的样式。

![](/prism/Ch1IntroFig5.png)

### 创建Bootstrapper

引导程序是连接应用程序与Prism Library服务和Unity或MEF容器的粘合剂。每个应用程序都会创建一个特定于应用程序的引导程序，它通常从**UnityBootstrapper**或**MefBootstrapper**继承，如下图所示。您需要确定要用于填充模块目录的方法。最低限度，每个应用程序将提供模块目录和shell。

默认情况下，引导程序使用.NET Framework **Trace**类记录事件。大多数应用程序都希望提供自己的日志记录服务，例如Enterprise Library日志记录。应用程序可以在其引导程序中提供其日志记录服务。

默认情况下，**UnityBootstrapper**和**MefBootstrapper**启用Prism Library服务。可以在特定于应用程序的引导程序中禁用或替换它们。

![](/prism/Ch1IntroFig6.png)

### 创建Module

该模块包含特定于应用程序功能的视图和服务。通常，这些包含在单独的程序集中，由不同的团队开发。模块由实现**IModule**接口的类表示。这些模块在初始化期间注册其视图和服务，并可能向shell添加一个或多个视图。根据您的模块发现方法，您可能需要将属性应用于模块类或定义模块之间的依赖关系。

### 将Module View添加到Shell

模块利用shell的区域来放置内容。在初始化期间，模块使用**RegionManager**定位shell中的区域，并向这些区域添加一个或多个视图，或者注册要在这些区域内创建的一个或多个视图类型。该**RegionManager**是负责跟踪区域的整个应用程序，是从引导程序初始化的核心服务。

文档中的其余主题提供了有关Prism关键概念的详细信息。

## 使用Prism库为WPF初始化应用程序

本主题介绍了为WPF应用程序启动和运行Prism所需要做的事情。Prism应用程序需要在应用程序启动过程中进行注册和配置 - 这称为Bootstrapper应用程序。Prism Bootstrapper过程包括创建和配置模块目录，创建依赖注入容器（如Unity），为UI组合配置默认区域适配器，创建和初始化shell视图以及初始化模块。

### Bootstrapper

引导程序是一个类，负责初始化使用Prism库构建的应用程序。通过使用引导程序，您可以更好地控制Prism库组件如何连接到您的应用程序。

Prism库包含一个默认的抽象**Bootstrapper**基类，可以专门用于任何容器。Bootstrapper类中的许多方法都是虚方法。您可以在自己的自定义引导程序实现中适当地覆盖这些方法。

![èªä¸¾è¿ç¨çåºæ¬é¶æ®µ](/prism/Ch2BootstrapperFig1.png)

Prism库提供了一些派生自**Bootstrapper**的附加基类，这些基类具有适用于大多数应用程序的默认实现。应用程序引导程序实现的唯一阶段是创建和初始化shell。

###  Dependency Injection

使用Prism Library构建的应用程序依赖于容器提供的依赖注入。该库提供了与Unity应用程序块（Unity）或托管扩展性框架（MEF）一起使用的程序集，它允许您使用其他依赖项注入容器。引导过程的一部分是配置此容器并使用容器注册类型。

Prism库包括**UnityBootstrapper**和**MefBootstrapper**类，它们实现了在应用程序中使用Unity或MEF作为依赖注入容器所需的大部分功能。除了上图中显示的阶段之外，每个引导程序还添加了一些特定于其容器的步骤。

###  Creating the Shell

在传统的Windows Presentation Foundation（WPF）应用程序中，启动主窗口的App.xaml文件中指定了启动统一资源标识符（URI）。

在使用Prism Library创建的应用程序中，**引导程序负责创建shell或主窗口**。这是因为shell依赖于需要在显示shell之前注册的服务（例如Region Manager）。

### 关键决定

在您决定在应用程序中使用Prism库之后，还需要做出一些额外的决定：

- 您需要决定是否使用MEF，Unity或其他容器作为**依赖注入容器**。这将确定您应该使用哪个提供的引导程序类，以及是否需要为另一个容器创建引导程序。
- 您应该考虑应用程序中所需的特定于应用程序的**服务**。这些将需要在容器中注册。
- 确定内置日志记录服务是否足以满足您的需求，或者是否需要创建**其他日志记录**服务。
- 确定应用程序如何发现模块：通过显式**代码声明**，通过**目录扫描**，配置或XAML发现的模块上的代码属性。

本主题的其余部分提供了更多详细信息。

### Core Scenarios

创建启动序列是构建Prism应用程序的重要部分。本节介绍如何创建引导程序并对其进行自定义以创建shell，配置依赖项注入容器，注册应用程序级服务以及如何加载和初始化模块。

### 创建Bootstrapper 

如果您选择使用Unity或MEF作为依赖注入容器，则可以轻松地为您的应用程序创建一个简单的引导程序。您需要创建一个派生自**MefBootstrapper**或**UnityBootstrapper**的新类。然后，实现**CreateShell**方法。（可选）您可以覆盖**InitializeShell**方法以进行特定于shell的初始化。

#### 实现CreateShell方法

该**CreateShell**方法允许开发者指定一个棱镜应用顶层窗口。shell通常是**MainWindow**或**MainPage**。通过返回应用程序的shell类的实例来实现此方法。在Prism应用程序中，您可以创建shell对象，或者根据应用程序的要求从容器中解析它。

以下代码示例中显示了使用**ServiceLocator**解析shell对象的示例。

```c#
protected override DependencyObject CreateShell()
{
    return ServiceLocator.Current.GetInstance&lt;Shell&gt;();
}
```

**注意：**您经常会看到**ServiceLocator**用于解析类型的实例而不是特定的依赖注入容器。该**服务定位**是通过调用容器实现的，所以它使容器无关代码一个不错的选择。您也可以直接引用和使用容器而不是**ServiceLocator**。

#### 实现InitializeShell方法

创建shell后，可能需要运行初始化步骤以确保准备好显示shell。对于WPF应用程序，您将创建shell应用程序对象并将其设置为应用程序的主窗口，如此处所示（来自WPF的Modularity QuickStarts）。

```c#
protected override void InitializeShell()
{
    Application.Current.MainWindow = Shell;
    Application.Current.MainWindow.Show();
}
```

**InitializeShell**的基本实现什么都不做。不调用基类实现是安全的。

#### 创建和配置模块目录

Creating and Configuring the **Module Catalog**

如果要构建模块应用程序，则需要创建和配置模块目录。Prism使用具体的**IModuleCatalog**实例来跟踪应用程序可用的模块，可能需要下载的模块以及模块所在的位置。

该**引导程序**提供了一个受保护的**ModuleCatalog**属性来引用目录以及一个基实现虚拟的**CreateModuleCatalog**方法。基础实现返回一个新的**ModuleCatalog** ; 但是，可以重写此方法以提供不同的**IModuleCatalog**实例，如下面的代码来自模块化中的**QuickStartBootstrapper**和MEF for WPF QuickStart。

```c#
protected override IModuleCatalog CreateModuleCatalog()
{
    // When using MEF, the existing Prism ModuleCatalog is still
    // the place to configure modules via configuration files.
    return new ConfigurationModuleCatalog()
}
```

在**UnityBootstrapper**和**MefBootstrapper**类中，Run方法调用**CreateModuleCatalog**方法，然后使用返回的值设置类的**ModuleCatalog**属性。如果重写此方法，则无需调用基类的实现，因为您将替换提供的功能。有关模块化的更多信息，请参阅“模块化应用程序开发”。

#### 创建和配置容器

Creating and Configuring the Container

容器在使用Prism Library创建的应用程序中起着关键作用。Prism Library和构建在它上面的应用程序都依赖于一个容器来注入所需的依赖项和服务。在容器配置阶段，注册了几个核心服务。除了这些核心服务之外，您还可以使用特定于应用程序的服务，这些服务提供与组合相关的其他功能。

##### 核心服务

下表列出了Prism库中的核心非应用程序特定服务。

| 服务界面               | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| **IModuleManager**     | 定义将检索和初始化应用程序模块的服务的接口。                 |
| **IModuleCatalog**     | 包含有关应用程序中模块的元数据。Prism Library提供了几种不同的目录。 |
| **IModuleInitializer** | 初始化模块。                                                 |
| **IRegionManager**     | 注册和检索区域，这些区域是布局的可视容器。                   |
| **IEventAggregator**   | 在发布者和订阅者之间松散耦合的事件集合。                     |
| **ILoggerFacade**      | 日志记录机制的包装器，因此您可以选择自己的日志记录机制。Stock Trader参考实施（Stock Trader RI）通过**EnterpriseLibraryLoggerAdapter**类使用Enterprise Library Logging Application Block 作为如何使用您自己的记录器的示例。使用**CreateLogger**方法返回的值，通过引导程序的**Run**方法向容器注册日志记录服务。向容器注册另一个记录器将不起作用; 而是覆盖引导程序上的**CreateLogger**方法。 |
| **IServiceLocator**    | 允许Prism库访问容器。如果要自定义或扩展库，这可能很有用。    |

Prism，**UnityBootstrapper**和**MefBootstrapper中**有两个Bootstrapper派生类。创建和配置不同的容器涉及以不同方式实现的类似概念。

####  在UnityBootstrapper中创建和配置容器

Creating and Configuring the Container in the **UnityBootstrapper**

该**UnityBootstrapper**类的**CreateContainer**方法简单地创建并返回的新实例**UnityContainer**。在大多数情况下，您无需更改此功能; 但是，该方法是虚拟的，从而允许这种灵活性。

创建容器后，可能需要为您的应用程序配置容器。**UnityBootstrapper中**的**ConfigureContainer**实现**默认**注册了许多核心Prism服务，如下所示。

***注意：**此示例是模块在其**Initialize**方法中注册模块级服务的情况。*

```c#
   // UnityBootstrapper.cs
protected virtual void ConfigureContainer()
{
    ...

    if (useDefaultConfiguration)
    {
         RegisterTypeIfMissing(typeof(IServiceLocator), typeof(UnityServiceLocatorAdapter), true);
         RegisterTypeIfMissing(typeof(IModuleInitializer), typeof(ModuleInitializer), true);
         RegisterTypeIfMissing(typeof(IModuleManager), typeof(ModuleManager), true);
         RegisterTypeIfMissing(typeof(RegionAdapterMappings), typeof(RegionAdapterMappings), true)
         RegisterTypeIfMissing(typeof(IRegionManager), typeof(RegionManager), true);
         RegisterTypeIfMissing(typeof(IEventAggregator), typeof(EventAggregator), true);
         RegisterTypeIfMissing(typeof(IRegionViewRegistry), typeof(RegionViewRegistry), true);
         RegisterTypeIfMissing(typeof(IRegionBehaviorFactory), typeof(RegionBehaviorFactory), true);
         RegisterTypeIfMissing(typeof(IRegionNavigationJournalEntry), typeof(RegionNavigationJournalEntry), false);
         RegisterTypeIfMissing(typeof(IRegionNavigationJournal), typeof(RegionNavigationJournal), false);
         RegisterTypeIfMissing(typeof(IRegionNavigationService), typeof(RegionNavigationService), false);
         RegisterTypeIfMissing(typeof(IRegionNavigationContentLoader), typeof(UnityRegionNavigationContentLoader), true);

    }
}
```

引导程序的**RegisterTypeIfMissing**方法确定服务是否已经注册 - 它不会注册两次。这允许您通过配置覆盖默认注册。您也可以默认关闭注册任何服务; 为此，请使用重载的**Bootstrapper.Run**方法传入**false**。您还可以覆盖**ConfigureContainer**方法并禁用您不想使用的服务，例如事件聚合器。

***注意：**如果关闭默认注册，则需要手动注册所需的服务。*

为了扩展的默认行为**ConfigureContainer**，只是一个覆盖添加到您的应用程序的引导程序和可选调用基实现，如从下面的代码**QuickStartBootstrapper**从模块化的WPF（使用Unity）快速入门。此实现调用基类的实现，注册**ModuleTracker**类型的具体实施**IModuleTracker**，并注册**callbackLogger**作为的单一实例**CallbackLogger**与统一。

```cs
protected override void ConfigureContainer()
{
    base.ConfigureContainer();

    this.RegisterTypeIfMissing(typeof(IModuleTracker), typeof(ModuleTracker), true);
    this.Container.RegisterInstance&lt;CallbackLogger&gt;(this.callbackLogger);
}
```

#### 在MefBootstrapper中创建和配置容器

该**MefBootstrapper**类的**CreateContainer**方法做几件事情。首先，它创建一个**AssemblyCatalog**和一个**CatalogExportProvider**。该**CatalogExportProvider**允许**MefExtensions**组件提供默认出口一批棱镜类型，并且还可以让你覆盖默认类型注册。然后，**CreateContainer**使用**CatalogExportProvider**创建并返回**CompositionContainer**的新实例。在大多数情况下，您无需更改此功能; 但是，该方法是虚拟的，从而允许这种灵活性。

创建容器后，需要为您的应用程序配置容器。默认情况下，**MefBootstrapper中**的**ConfigureContainer**实现注册了许多核心Prism服务，如以下代码示例所示。如果重写此方法，请仔细考虑是否应调用基类的实现来注册核心Prism服务，或者是否在实现中提供这些服务。

```cs
protected virtual void ConfigureContainer()
{
    this.RegisterBootstrapperProvidedTypes();
}

protected virtual void RegisterBootstrapperProvidedTypes()
{
    this.Container.ComposeExportedValue&lt;ILoggerFacade&gt;(this.Logger);
    this.Container.ComposeExportedValue&lt;IModuleCatalog&gt;(this.ModuleCatalog);
    this.Container.ComposeExportedValue&lt;IServiceLocator&gt;(new MefServiceLocatorAdapter(this.Container));
    this.Container.ComposeExportedValue&lt;AggregateCatalog&gt;(this.AggregateCatalog);
}
```

***注意：**在**MefBootstrapper中**，Prism的核心服务作为单例添加到容器中，因此可以在整个应用程序中通过容器定位它们。*

除了提供**CreateContainer**和**ConfigureContainer**方法之外，**MefBootstrapper**还提供了两种方法来创建和配置MEF使用的**AggregateCatalog**。该**CreateAggregateCatalog**方法简单地创建并返回一个**AggregateCatalog**对象。与**MefBootstrapper中**的其他方法一样，**CreateAggregateCatalog**是虚拟的，如果需要可以覆盖。

该**ConfigureAggregateCatalog**方法允许您注册类型添加到**AggregateCatalog**势在必行。例如，**QuickStartBootstrapper**从MEF快速入门模块性明确增加了ModuleA和ModuleC到**AggregateCatalog**，如下图所示。

```cs
protected override void ConfigureAggregateCatalog()
{
    base.ConfigureAggregateCatalog();

    // Add this assembly to export ModuleTracker
    this.AggregateCatalog.Catalogs.Add(
        new AssemblyCatalog(typeof(QuickStartBootstrapper).Assembly));

    // Module A is referenced in in the project and directly in code.
    this.AggregateCatalog.Catalogs.Add(
        new AssemblyCatalog(typeof(ModuleA.ModuleA).Assembly));

    this.AggregateCatalog.Catalogs.Add(
        new AssemblyCatalog(typeof(ModuleC.ModuleC).Assembly));

    // Module B and Module D are copied to a directory as part of a post-build step.
    // These modules are not referenced in the project and are discovered by inspecting a directory.
    // Both projects have a post-build step to copy themselves into that directory.
    DirectoryCatalog catalog = new DirectoryCatalog(&#34;DirectoryModules&#34;);
    this.AggregateCatalog.Catalogs.Add(catalog);
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism6-usingprism/  

