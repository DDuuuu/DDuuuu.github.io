# Prism Modular


注意：该版本为Prism6,最新版已有较大改动。

Modular Application Development Using Prism Library for WPF

模块化应用程序是一个应用程序，它被分成一组松散耦合的功能单元（命名模块），可以集成到更大的应用程序中。客户端模块封装了应用程序的整体功能的一部分，并且通常表示一组相关的问题。它可以包括一组相关组件，例如应用程序功能，包括用户界面和业务逻辑，或应用程序基础结构，例如用于记录或验证用户的应用程序级服务。模块彼此独立，但可以以松散耦合的方式彼此通信。使用模块化应用程序设计，您可以更轻松地开发，测试，部署和维护应用程序。

例如，考虑个人银行应用程序。用户可以访问各种功能，例如在账户之间转账，支付账单以及从单个用户界面（UI）更新个人信息。但是，在幕后，这些功能中的每一个都封装在一个离散模块中。这些模块相互通信，并与**后端系统（如数据库服务器和Web服务）进行通信**。应用服务集成了每个不同模块中的各种组件，并处理与用户的通信。用户看到的视图类似于单个应用程序的集成视图。

下图显示了具有多个模块的模块化应用程序的设计。

![模块组成](/prism/Ch4ModularityFig1.png)

### 构建模块化应用程序的好处

您可能已经使用程序集，接口和类构建了一个架构良好的应用程序，并采用了良好的面向对象设计原则。即便如此，除非非常小心，否则您的应用程序设计可能仍然是“单一的”（所有功能都在应用程序内以紧密耦合的方式实现），这可能使应用程序难以开发，测试，扩展和维护。

另一方面，模块化应用程序方法可以帮助您识别应用程序的大规模功能区域，并允许您独立开发和测试该功能。这可以使开发和测试更容易，但它也可以使您的应用程序更灵活，更容易在未来扩展。模块化方法的好处是它可以使您的整体应用程序架构更加灵活和可维护，因为它允许您将应用程序分解为可管理的部分。每个部分都封装了特定的功能，每个部分都通过清晰但松散耦合的通信渠道进行集成。

### Prism对模块化应用程序开发的支持

Prism为您的应用程序中的模块化应用程序开发和运行时模块管理提供支持。使用Prism的模块化开发功能可以节省您的时间，因为您不必实现和测试自己的模块化框架。Prism支持以下模块化应用程序开发功能：

- 用于**注册命名模块和每个模块位置的模块目录**; 您可以通过以下方式创建模块目录：
- 通过代码或可扩展应用程序标记语言（XAML）**定义模块**
- 通过发现**目录中的模块，您可以加载所有模块，而无需在集中目录中明确定义**
- 通过在**配置文件中定义模块**
- 模块的**声明性元数据属性，以支持初始化模式和依赖性**
- 与依赖注入容器集成以支持模块之间的松散耦合
- 对于模块加载：
  - 依赖管理，包括重复和循环检测，以确保模块以正确的顺序加载，并且只加载和初始化一次
  - 模块的按需和后台下载，以最大限度地减少应用程序启动时间; 其余模块可以在后台加载和初始化，也可以在需要时加载和初始化

### 核心概念

本节介绍与Prism模块化相关的核心概念，包括**IModule**接口，模块加载过程，模块目录，模块之间的通信以及依赖注入容器。

#### IModule：模块化应用程序的构建块

模块是功能和资源的逻辑集合，以可以单独开发，测试，部署和集成到应用程序中的方式打包。包可以是一个或多个程序集。每个模块都有一个中心类，负责初始化模块并将其功能集成到应用程序中。该类实现了**IModule**接口。

***注意：**实现**IModule**接口的类的存在足以将包标识为模块。

该**IModule的**接口只有一个方法，名为**初始化**，您可以在其中实现的任何逻辑需要初始化和模块的功能集成到应用程序。根据模块的用途，它可以将视图注册到组合用户界面，为应用程序提供其他服务，或扩展应用程序的功能。以下代码显示了模块的最低实现。

```c#
public class MyModule : IModule
{
    public void Initialize()
    {
        // Do something here.
    }
}
```

#### 模块寿命

Prism中的模块加载过程包括以下内容：

- **注册/发现模块**。在运行时为**特定应用程序加载的模块在模块目录中定义**。该目录包含有关要加载的模块，其位置以及加载顺序的信息。
- **加载模块**。包含模块的程序集将加载到内存中。此阶段可能需要从某个远程位置或本地目录检索模块。
- **初始化模块**。然后初始化模块。这意味着创建模块类的实例并通过**IModule**接口调用它们的**Initialize**方法。

下图显示了模块加载过程。

![模块加载过程](/prism/Ch4ModularityFig2.png)

#### 模块目录

所述**ModuleCatalog**保存关于能够由应用程序使用的模块的信息。目录本质上是**ModuleInfo**类的集合。**ModuleInfo**类中描述了每个模块，该类记录了模块的其他属性中的名称，类型和位置。使用**ModuleInfo**实例填充**ModuleCatalog**有几种典型方法：

- 在**代码中注册模块**
- 在**XAML中注册模块**
- 在**配置文件中注册模块**
- 在**磁盘上的本地目录**中发现模块

您应该使用的注册和发现机制取决于您的应用程序需要什么。使用配置文件或XAML文件允许您的应用程序不需要引用模块。使用目录可以允许应用程序发现模块，而无需在文件中指定它们。

#### 控制何时加载模块

Prism应用程序可以尽快初始化模块，称为“可用时”，或者当应用程序需要它们时，称为“按需”。请考虑以下加载模块的准则：

- 运行应用程序所需的模块必须与应用程序一起加载，并在应用程序运行时进行初始化。
- 包含几乎总是在应用程序的典型使用中使用的功能的模块可以在**后台加载并在可用时进行初始化**。
- 可以按需加载和初始化包含很少使用的功能（或其他模块可选择依赖的支持模块）的模块。

考虑如何对应用程序进行分区，常见使用方案，应用程序启动时间以及下载的数量和大小，以确定如何配置模块以进行下载和初始化。

#### 将模块与应用程序集成

Prism提供以下类来引导您的应用程序：**UnityBootstrapper**或**MefBootstrapper**。这些类可用于创建和配置模块管理器以发现和加载模块。您可以覆盖配置方法，以在几行代码中注册XAML文件，配置文件或目录位置中指定的模块。

使用模块**Initialize**方法将模块与应用程序的其余部分集成。执行此操作的方式因应用程序的结构和模块的内容而异。以下是将模块集成到应用程序中的常见操作：

- 将模块的视图添加到应用程序的导航结构中。在使用视图发现或视图注入构建复合UI应用程序时，这很常见。
- 订阅应用程序级别的事件或服务。
- 使用应用程序的依赖注入容器注册共享服务。

#### 在模块之间进行通信

即使模块之间的耦合度较低，模块也可以相互通信。有几种松散耦合的通信模式，每种都有自己的优势。通常，这些模式的组合用于创建所得到的解决方案。以下是其中一些模式：

- **松散耦合的事件**。模块可以广播已发生的特定事件。其他模块可以订阅这些事件，以便在事件发生时通知他们。松耦合事件是在两个模块之间建立通信的轻量级方式; 因此，它们很容易实现。但是，过于依赖事件的设计可能变得难以维护，尤其是如果必须协调许多事件以完成单个任务。在这种情况下，考虑共享服务可能更好。
- **共享服务**。共享服务是可以通过公共接口访问的类。通常，共享服务位于共享程序集中，并提供系统范围的服务，例如身份验证，日志记录或配置。
- **共享资源**。如果您不希望模块直接相互通信，您还可以通过共享资源（如数据库或一组Web服务）间接进行通信。

#### 依赖注入和模块化应用程序

Unity应用程序块（Unity）和托管可扩展性框架（MEF）等容器允许您轻松使用控制反转（IoC）和依赖注入，它们是强大的设计模式，有助于以松散耦合的方式组合组件。它允许组件获得对它们所依赖的其他组件的引用，而无需对这些引用进行硬编码，从而促进更好的代码重用和更高的灵活性。在构建松散耦合的模块化应用程序时，依赖注入非常有用。Prism旨在与用于组成应用程序中的组件的依赖注入容器无关。容器的选择取决于您，并且在很大程度上取决于您的应用要求和偏好。然而，

模式和实践Unity Application Block提供了一个功能齐全的依赖注入容器。它支持基于属性和基于构造函数的注入和策略注入，允许您透明地在组件之间注入行为和策略; 它还支持许多其他典型的依赖注入容器功能。

MEF（它是.NET Framework 4.5的一部分）通过支持基于依赖注入的组件组合提供对构建可扩展.NET应用程序的支持，并提供支持模块化应用程序开发的其他功能。它允许应用程序在运行时发现组件，然后以松散耦合的方式将这些组件集成到应用程序中。MEF是一个很好的可扩展性和组合框架。它包括程序集和类型发现，类型依赖性解析，依赖注入以及一些不错的程序集下载功能。Prism支持利用MEF功能，以及以下内容：

- 通过XAML和代码属性进行模块注册
- 通过配置文件和目录扫描进行模块注册
- **加载模块时的状态跟踪**
- 使用MEF时模块的自定义声明性元数据

Unity和MEF依赖注入容器都可以与Prism无缝协作。

### 关键决定

您要做的第一个决定是您是否要开发模块化解决方案。如上一节所述，构建模块化应用程序有许多好处，但是您需要花费时间和精力来获得这些好处。如果您决定开发模块化解决方案，还有几个需要考虑的事项：

- **确定您将使用的框架**。您可以创建自己的模块化框架，使用Prism，MEF或其他框架。
- **确定如何组织解决方案**。通过定义每个模块的边界来处理模块化体系结构，包括哪些组件是每个模块的一部分。您可以决定使用模块化来简化开发，以及控制应用程序的部署方式或是否支持插件或可扩展体系结构。
- **确定如何对模块进行分区**。可以根据需求对模块进行不同的分区，例如，按功能区域，提供程序模块，开发团队和部署要求进行分区。
- **确定应用程序将为所有模块提供的核心服务**。例如，核心服务可以是错误报告服务或身份验证和授权服务。
- **如果您使用的是Prism，请确定在模块目录中注册模块时使用的方法**。对于WPF，您可以在代码，XAML，配置文件中注册模块，或在磁盘上的本地目录中发现模块。
- **确定您的模块通信和依赖策略**。模块需要相互通信，您需要处理模块之间的依赖关系。
- **确定您的依赖注入容器**。通常，模块化系统需要依赖注入，控制反转或服务定位器，以允许松散耦合和动态加载和创建模块。Prism允许在使用Unity，MEF或其他容器之间进行选择，并为Unity或基于MEF的应用程序提供库。
- **最小化应用程序启动时间**。考虑模块的按需和后台下载，以最大限度地减少应用程序启动时间。
- **确定部署要求**。您需要考虑如何部署应用程序。

下一节提供了有关这些决策的详细信息。

### 将您的应用程序划分为模块

当您以模块化方式开发应用程序时，可以将应用程序组织到单独的客户端模块中，这些模块可以单独开发，测试和部署。每个模块都将封装应用程序的一部分整体功能。您必须做出的首要设计决策之一是决定如何将应用程序的功能划分为离散模块。

模块应该封装一组相关的问题，并具有一组独特的职责。模块可以表示应用程序的垂直切片或水平服务层。大型应用程序可能有两种类型的模块。

![垂直切片应用程序](/prism/Ch4ModularityFig3.png)

围绕垂直切片组织模块的应用程序

![水平分层应用程序](/prism/Ch4ModularityFig4.png)

围绕水平层组织模块的应用程序

较大的应用程序可能具有使用垂直切片和水平层组织的模块。模块的一些示例包括以下内容：

- 包含特定应用程序功能的模块，例如Stock Trader参考实现中的新闻模块（Stock Trader RI）
- 包含特定子系统或功能的模块，用于一组相关用例，例如采购，发票或总帐
- 包含基础结构服务的模块，例如日志记录，缓存和授权服务，或Web服务
- 除了其他内部系统之外，包含调用业务线（LOB）系统（如Siebel CRM和SAP）的服务的模块

模块应该对其他模块具有最小的依赖关系。当模块依赖于另一个模块时，它应该通过使用共享库中定义的接口而不是具体类型来松散耦合，或者通过使用**EventAggregator**通过**EventAggregator**事件类型与其他模块进行通信。

模块化的目标是以一种即使在添加和删除功能和技术时仍保持灵活性，可维护性和稳定性的方式对应用程序进行分区。实现此目的的最佳方法是设计应用程序，使模块尽可能独立，具有良好定义的接口，并尽可能隔离。

#### 确定项目与模块的比率

有几种方法可以创建和打包模块。建议的和最常见的方法是为每个模块创建一个组件。这有助于保持逻辑模块分离并促进适当的封装。它还使得更容易将组件作为模块边界以及如何部署模块的包装进行讨论。但是，没有什么可以阻止单个程序集包含多个模块，在某些情况下，这可能是首选，以最大限度地减少解决方案中的项目数量。对于大型应用程序，拥有10-50个模块并不罕见。将每个模块分离到自己的项目中会增加解决方案的复杂性，并会降低Visual Studio的性能。

### 使用依赖注入来实现松散耦合

模块可以依赖于主机应用程序或其他模块提供的组件和服务。Prism支持在模块之间注册依赖关系的能力，以便以正确的顺序加载和初始化它们。Prism还支持在将模块加载到应用程序时初始化模块。在模块初始化期间，模块可以检索对其所需的附加组件和服务的引用，和/或注册它包含的任何组件和服务，以使其可供其他模块使用。

模块应使用独立机制来获取外部接口的实例，而不是直接实例化具体类型，例如通过使用依赖注入容器或工厂服务。诸如Unity或MEF之类的依赖注入容器允许类型通过依赖注入自动获取所需的接口和类型的实例。Prism与Unity和MEF集成，允许模块轻松使用依赖注入。

下图显示了加载模块时需要获取或注册组件和服务引用的典型操作顺序。

![依赖注入的示例](/prism/Ch4ModularityFig5.png)

在此示例中，**OrdersModule**程序集定义了**OrdersRepository**类（以及实现顺序功能的其他视图和类）。所述**CustomerModule**组件限定**CustomersViewModel**类依赖于**OrdersRepository**，通常基于由服务暴露的接口上。应用程序启动和引导过程包含以下步骤：

1. 引导程序启动模块初始化过程，模块加载程序加载并初始化**OrdersModule**。
2. 在**OrdersModule**的初始化中，它将**OrdersRepository**注册到容器中。
3. 然后，模块加载器加载**CustomersModule**。模块加载的顺序可以由模块元数据中的依赖项指定。
4. 该**CustomersModule**构建的一个实例**CustomerViewModel**通过容器以解决该问题。该**CustomerViewModel**对一个依赖**OrdersRepository**（通常基于它的接口上），并指示它通过构造或财产注射。容器根据**OrdersModule**注册的类型在视图模型的构造中注入该依赖**项**。最终结果是从**CustomerViewModel**到**OrderRepository**的接口引用，而没有这些类之间的紧密耦合。

***注意：**用于公开**OrderRespository**（**IOrderRepository**）的接口可以驻留在单独的“共享服务”程序集或“订单服务”程序**集中**，该程序集仅包含公开这些服务所需的服务接口和类型。这样，**CustomersModule**和**OrdersModule**之间就没有硬依赖关系。*

请注意，两个模块都依赖于依赖注入容器。在模块构建器中的模块构造期间注入该依赖性。

### 核心情景

本节介绍在应用程序中使用模块时将遇到的常见方案。这些方案包括定义模块，注册和发现模块，加载模块，初始化模块，指定模块依赖关系，按需加载模块，在后台下载远程模块以及检测模块何时已加载。您可以在代码，XAML或应用程序配置文件中注册和发现模块，也可以通过扫描本地目录来注册和发现模块。

### 定义模块

模块是功能和资源的逻辑集合，以可以单独开发，测试，部署和集成到应用程序中的方式打包。每个模块都有一个中心类，负责初始化模块并将其功能集成到应用程序中。该类实现了**IModule**接口，如下所示。

```c#
public class MyModule : IModule
{
    public void Initialize()
    {
        // Initialize module
    }
}
```

实现**Initialize**方法的方式取决于应用程序的要求。模块目录中定义了模块类类型，初始化模式和任何模块依赖性。对于目录中的每个模块，模块加载器创建模块类的实例，然后调用**Initialize**方法。模块按模块目录中指定的顺序处理。运行时初始化顺序基于模块下载，可用和满足依赖性的时间。

根据应用程序使用的模块目录的类型，可以通过模块类本身的声明性属性或模块目录文件中的模块依赖性来设置模块依赖性。以下部分提供了更多详细信息。

#### 创建模块目录

```c#
protected override IModuleCatalog CreateModuleCatalog()
{
    return new AggregateModuleCatalog()
}
```

#### 注册和发现模块

应用程序可以加载的模块在模块目录中定义。Prism Module Loader使用模块目录来确定哪些模块可以加载到应用程序中，何时加载它们以及它们的加载顺序。

模块目录由实现**IModuleCatalog**接口的类表示。模块目录类由应用程序引导程序类在应用程序初始化期间创建。Prism提供了不同的模块目录实现供您选择。您还可以通过调用**AddModule**方法或从**ModuleCatalog**派生来创建具有自定义行为的模块目录，从另一个数据源填充模块目录。

**注意：**通常，Prism中的模块使用依赖注入容器和公共服务定位器来检索模块初始化所需的类型实例。Unity和MEF容器都由Prism支持。虽然注册，发现，下载和初始化模块的整个过程是相同的，但细节可以根据是使用Unity还是MEF而有所不同。本主题将解释方法之间特定于容器的差异。

##### 在代码中注册模块

最基本的模块目录由**ModuleCatalog**类提供。您可以使用此模块目录通过指定模块类类型以编程方式注册模块。您还可以以编程方式指定模块名称和初始化模式。要直接使用**ModuleCatalog**类注册模块，请在应用程序的**Bootstrapper**类中调用**AddModule**方法。以下代码中显示了一个示例。

```c#
protected override void ConfigureModuleCatalog()
{
    Type moduleCType = typeof(ModuleC);
    ModuleCatalog.AddModule(
    new ModuleInfo()
    {
        ModuleName = moduleCType.Name,
        ModuleType = moduleCType.AssemblyQualifiedName,
    });
}
```

**注意：**如果您的应用程序直接引用模块类型，您可以按类型添加它，如上所示; 否则，您需要提供完全限定的类型名称和程序集的位置。

要查看在代码中定义模块目录的另一个示例，请参阅Stock Trader参考实现（Stock Trader RI）中的StockTraderRIBootstrapper.cs。

**注：**该**引导程序**基类提供了**CreateModuleCatalog**方法来帮助创建的**ModuleCatalog**。默认情况下，此方法创建**ModuleCatalog**实例，但可以在派生类中重写此方法，以便创建不同类型的模块目录。

##### 使用XAML文件注册模块

您可以通过在XAML文件中指定模块目录来以声明方式定义模块目录。XAML文件指定要创建的模块目录类类型以及要添加到哪个模块。通常，.xaml文件作为资源添加到shell项目中。模块目录由引导程序创建，并调用**CreateFromXaml**方法。从技术角度来看，这种方法非常类似于在代码中定义**ModuleCatalog**，因为XAML文件只是定义了要实例化的对象的层次结构。

以下代码示例显示了指定模块目录的XAML文件。

```xaml
&lt;--! ModulesCatalog.xaml --&gt;
&lt;Modularity:ModuleCatalog xmlns=&#34;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#34;
    xmlns:x=&#34;http://schemas.microsoft.com/winfx/2006/xaml&#34;
    xmlns:sys=&#34;clr-namespace:System;assembly=mscorlib&#34;
    xmlns:Modularity=&#34;clr-namespace:Microsoft.Practices.Prism.Modularity;assembly=Microsoft.Practices.Prism&#34;&gt;

    &lt;Modularity:ModuleInfoGroup Ref=&#34;file://DirectoryModules/ModularityWithMef.Desktop.ModuleB.dll&#34; InitializationMode=&#34;WhenAvailable&#34;&gt;
        &lt;Modularity:ModuleInfo ModuleName=&#34;ModuleB&#34; ModuleType=&#34;ModularityWithMef.Desktop.ModuleB, ModularityWithMef.Desktop.ModuleB, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; /&gt;
    &lt;/Modularity:ModuleInfoGroup&gt;

    &lt;Modularity:ModuleInfoGroup InitializationMode=&#34;OnDemand&#34;&gt;
        &lt;Modularity:ModuleInfo Ref=&#34;file://ModularityWithMef.Desktop.ModuleE.dll&#34; ModuleName=&#34;ModuleE&#34; ModuleType=&#34;ModularityWithMef.Desktop.ModuleE, ModularityWithMef.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; /&gt;
        &lt;Modularity:ModuleInfo Ref=&#34;file://ModularityWithMef.Desktop.ModuleF.dll&#34; ModuleName=&#34;ModuleF&#34; ModuleType=&#34;ModularityWithMef.Desktop.ModuleF, ModularityWithMef.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34;&gt;
            &lt;Modularity:ModuleInfo.DependsOn&gt;
                &lt;sys:String&gt;ModuleE&lt;/sys:String&gt;
            &lt;/Modularity:ModuleInfo.DependsOn&gt;
        &lt;/Modularity:ModuleInfo&gt;
    &lt;/Modularity:ModuleInfoGroup&gt;

    &lt;!-- Module info without a group --&gt;
    &lt;Modularity:ModuleInfo Ref=&#34;file://DirectoryModules/ModularityWithMef.Desktop.ModuleD.dll&#34; ModuleName=&#34;ModuleD&#34; ModuleType=&#34;ModularityWithMef.Desktop.ModuleD, ModularityWithMef.Desktop.ModuleD, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; /&gt;
&lt;/Modularity:ModuleCatalog&gt;
```

**注意：** **ModuleInfoGroups**提供了一种方便的方法来对同一程序集中的模块进行分组，以相同的方式进行初始化，或者只对同一组中的模块具有依赖性。模块之间的依赖关系可以在同一**ModuleInfoGroup中的**模块中定义; 但是，**您无法在不同的ModuleInfoGroups中定义模块之间的依赖关系**。将模块放在模块组中是可选的。为组设置的属性将应用于其包含的所有模块。请注意，模块也可以在不在组内的情况下进行注册。

在应用程序的**Bootstrapper**类中，您需要指定XAML文件是**ModuleCatalog**的源，如以下代码所示。

```c#
protected override IModuleCatalog CreateModuleCatalog()
{
    return ModuleCatalog.CreateFromXaml(new Uri(&#34;/MyProject;component/ModulesCatalog.xaml&#34;, UriKind.Relative));
}
```

##### 使用配置文件注册模块

在WPF中，可以在App.config文件中指定模块信息。此方法的优点是此文件未编译到应用程序中。这使得在运行时添加或删除模块非常容易，无需重新编译应用程序。

以下代码示例显示了指定模块目录的配置文件。如果要自动加载模块，请设置**startupLoaded =“true”**。

```xaml
&lt;!-- ModularityWithUnity.Desktop\\app.config --&gt;
&lt;xml version=&#34;1.0&#34; encoding=&#34;utf-8&#34; ?&gt;
&lt;configuration&gt;
    &lt;configSections&gt;
        &lt;section name=&#34;modules&#34; type=&#34;Prism.Modularity.ModulesConfigurationSection, Prism.Wpf&#34;/&gt;
    &lt;/configSections&gt;

    &lt;modules&gt;
        &lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleE.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleE&#34; startupLoaded=&#34;false&#34; /&gt;
        &lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleF.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleF, ModularityWithUnity.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleF&#34; startupLoaded=&#34;false&#34;&gt;
            &lt;dependencies&gt;
                &lt;dependency moduleName=&#34;ModuleE&#34;/&gt;
            &lt;/dependencies&gt;
        &lt;/module&gt;
    &lt;/modules&gt;
&lt;/configuration&gt;
```

**注意：**即使程序集位于全局程序集缓存中或与应用程序位于同一文件夹中，也需要**assemblyFile**属性。该属性用于将**moduleType**映射到要使用的正确**IModuleTypeLoader**。

在应用程序的**Bootstrapper**类中，您需要指定配置文件是**ModuleCatalog**的源。为此，请使用**ConfigurationModuleCatalog**类，如以下代码所示。

```c#
protected override IModuleCatalog CreateModuleCatalog()
{
    return new ConfigurationModuleCatalog();
}
```

**注意：**您仍然可以在代码中将模块添加到**ConfigurationModuleCatalog**。例如，您可以使用它来确保应用程序绝对需要运行的模块在目录中定义。

##### 发现目录中的模块

Prism **DirectoryModuleCatalog**类允许您将本地目录指定为WPF中的模块目录。此模块目录将扫描指定的文件夹并搜索定义应用程序模块的程序集。要使用此方法，您需要在模块类上使用声明性属性来指定模块名称以及它们具有的任何依赖项。以下代码示例显示了通过发现目录中的程序集来填充的模块目录。

```c#
protected override IModuleCatalog CreateModuleCatalog()
{
    return new DirectoryModuleCatalog() {ModulePath = @&#34;.\\Modules&#34;};
}
```

###### 注意：**所在文件夹不要带特殊字符，不识别**

### 加载模块

填充**ModuleCatalog**后，可以加载和初始化模块。模块加载意味着模块组件从磁盘传输到内存。该**ModuleManager**会负责协调加载和初始化过程。

### 初始化模块

模块加载后，它们被初始化。这意味着创建了模块类的实例并调用了其**Initialize**方法。初始化是将模块与应用程序集成的地方。考虑以下模块初始化的可能性：

- **使用应用程序注册模块的视图**。如果您的模块使用视图发现或视图注入参与用户界面（UI）组合，则您的模块将需要将其视图或视图模型与相应的区域名称相关联。这允许视图在应用程序中的菜单，工具栏或其他可视区域上动态显示。
- **订阅应用程序级别的事件或服务**。通常，应用程序会公开您的模块感兴趣的特定于应用程序的服务和/或事件。使用**Initialize**方法将模块的功能添加到那些应用程序级别的事件和服务。

例如，应用程序可能会在关闭时引发事件，并且您的模块想要对该事件做出反应。您的模块也可能必须向应用程序级服务提供一些数据。例如，如果您已创建**MenuService**（它负责添加和删除菜单项），则可以在模块的**Initialize**方法中添加正确的菜单项。

***注意：**默认情况下，模块实例生存期是短暂的。在加载过程中调用**Initialize**方法后，将释放对模块实例的引用。如果您没有为模块实例建立强引用链，则会进行垃圾回收。如果您订阅包含对模块的弱引用的事件，则此行为可能会导致调试有问题，因为您的模块在垃圾收集器运行时“消失”。*

- **使用依赖项注入容器注册类型**。如果使用依赖注入模式（如Unity或MEF），则模块可以为应用程序或其他模块注册要使用的类型。它还可能要求容器解析所需类型的实例。

### 指定模块依赖项

模块可能依赖于其他模块。如果模块A依赖于模块B，则必须在模块A之前初始化模块B. **ModuleManager**会跟踪这些依赖关系并相应地初始化模块。根据您定义模块目录的方式，您可以在代码，配置或XAML中定义模块依赖性。

##### 在代码中指定依赖项

对于在代码中注册模块或按目录发现模块的WPF应用程序，Prism提供了在创建模块时使用的声明性属性，如以下代码示例所示。

```c#
// (when using Unity)
[Module(ModuleName = &#34;ModuleA&#34;)\]
[ModuleDependency(&#34;ModuleD&#34;)\]
public class ModuleA: IModule
{
    ...
}
```

##### 在XAML中指定依赖项

以下XAML显示了模块F依赖于模块E的位置。

```xml
    &lt;-- ModulesCatalog.xaml --&gt;
    &lt;Modularity:ModuleInfo Ref=&#34;file://ModularityWithMef.Desktop.ModuleE.dll&#34; moduleName=&#34;ModuleE&#34; moduleType=&#34;ModularityWithMef.Desktop.ModuleE, ModularityWithMef.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34;&gt;

    &lt;Modularity:ModuleInfo Ref=&#34;file://ModularityWithMef.Desktop.ModuleF.dll&#34; moduleName=&#34;ModuleF&#34; moduleType=&#34;ModularityWithMef.Desktop.ModuleF, ModularityWithMef.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34;&gt;
    &lt;Modularity:ModuleInfo.DependsOn&gt;
        &lt;sys:String&gt;ModuleE&lt;/sys:String&gt;
    &lt;/Modularity:ModuleInfo.DependsOn&gt;
&lt;/Modularity:ModuleInfo&gt;
. . .
```

##### 在配置中指定依赖项

以下示例App.config文件显示了模块F依赖于模块E的位置。

```xml
&lt;!-- App.config --&gt;
&lt;modules&gt;
    &lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleE.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleE&#34; startupLoaded=&#34;false&#34; /&gt;

    &lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleF.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleF, ModularityWithUnity.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleF&#34; startupLoaded=&#34;false&#34;&gt;
        &lt;dependencies&gt;
            &lt;dependency moduleName=&#34;ModuleE&#34; /&gt;
        &lt;/dependencies&gt;
    &lt;/module&gt;
&lt;/modules&gt;
```

#### 按需加载模块

要按需加载模块，您需要指定将它们加载到模块目录中，并将**InitializationMode**设置为**OnDemand**。执行此操作后，您需要在应用程序中编写请求加载模块的代码。

##### 在代码中指定按需加载

使用属性将模块指定为按需，如以下代码示例所示。

```c#
// Boostrapper.cs
protected override void ConfigureModuleCatalog()
{
    . . .
    Type moduleCType = typeof(ModuleC);
    this.ModuleCatalog.AddModule(new ModuleInfo()
    {
        ModuleName = moduleCType.Name,
        ModuleType = moduleCType.AssemblyQualifiedName,
        InitializationMode = InitializationMode.OnDemand
    });
    . . .
}
```

##### 在XAML中指定按需加载

在XAML中定义模块目录时，可以指定**InitializationMode.OnDemand**，如以下代码示例所示。

```xaml
&lt;!-- ModulesCatalog.xaml --&gt;
...
&lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleE.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleE&#34; startupLoaded=&#34;false&#34; /&gt;
...
```

##### 在配置中指定按需加载

在App.config文件中定义模块目录时，可以指定**InitializationMode.OnDemand**，如以下代码示例所示。

```xaml
&lt;!-- App.config --&gt;
&lt;module assemblyFile=&#34;ModularityWithUnity.Desktop.ModuleC.dll&#34; moduleType=&#34;ModularityWithUnity.Desktop.ModuleC, ModularityWithUnity.Desktop.ModuleC, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null&#34; moduleName=&#34;ModuleC&#34; startupLoaded=&#34;false&#34; /&gt;
```

##### 请求按需加载模块

在按需指定模块后，应用程序可以请求加载模块。想要启动加载的代码需要获取对引导程序向容器注册的**IModuleManager**服务的引用。

```c#
private void OnLoadModuleCClick(object sender, RoutedEventArgs e)
{
    moduleManager.LoadModule(&#34;ModuleC&#34;);
}
```

### 检测模块何时加载

所述**ModuleManager**会服务提供了一个用于事件的应用程序的模块负载时来跟踪或无法加载。您可以通过依赖注入**IModuleManager**接口来获取对此服务的引用。

```c#
this.moduleManager.LoadModuleCompleted &#43;= this.ModuleManager_LoadModuleCompleted;
void ModuleManager_LoadModuleCompleted(object sender, LoadModuleCompletedEventArgs e)
{
    ...
}
```

为了使应用程序和模块保持松散耦合，应用程序应避免使用此事件将模块与应用程序集成。相反，模块的**Initialize**方法应该处理与应用程序的集成。

该**LoadModuleCompletedEventArgs**包含**IsErrorHandled**财产。如果模块无法加载并且应用程序想要阻止**ModuleManager**记录错误并抛出异常，则可以将此属性设置为**true**。

***注意：**加载并初始化模块后，无法卸载模块组件。Prism库不会保存模块实例引用，因此初始化完成后可能会对模块类实例进行垃圾回收。*

### MEF中的模块

如果您选择使用MEF作为依赖注入容器，本节仅突出显示差异。

***注意：**使用MEF时，**MefBootstrapper**使用**MefModuleManager**。它扩展了**ModuleManager**并实现了**IPartImportsSatisfiedNotification**接口，以确保在MEF导入新类型时更新**ModuleCatalog**。*

#### 使用MEF在代码中注册模块

使用MEF时，可以将**ModuleExport**属性应用于模块类，以使MEF自动发现类型。以下是一个例子。

```c#
[ModuleExport(typeof(ModuleB), InitializationMode = InitializationMode.OnDemand)]
public class ModuleB : IModule
{
    ...
}
```

您还可以使用MEF来发现和加载模块，使用**AssemblyCatalog**类（可用于发现程序集中的所有导出的模块类）和**AggregateCatalog**类（允许将多个目录组合到一个逻辑目录中）。默认情况下，Prism **MefBootstrapper**类创建一个**AggregateCatalog**实例。然后，您可以覆盖**ConfigureAggregateCatalog**方法以注册程序集，如以下代码示例所示。

```c#
protected override void ConfigureAggregateCatalog()
{
    base.ConfigureAggregateCatalog();
    //Module A is referenced in in the project and directly in code.
    this.AggregateCatalog.Catalogs.Add(new AssemblyCatalog(typeof(ModuleA).Assembly));
    this.AggregateCatalog.Catalogs.Add(new AssemblyCatalog(typeof(ModuleC).Assembly));
    . . .
}
```

Prism **MefModuleManager**实现使MEF **AggregateCatalog**和Prism **ModuleCatalog保持**同步，从而允许Prism发现通过**ModuleCatalog**或**AggregateCatalog**添加的模块。

***注意：** MEF 广泛使用**Lazy &lt;T&gt;**来防止导出和导入类型的实例化，直到使用**Value**属性。*

#### 使用MEF在目录中发现模块

MEF提供了一个**DirectoryCatalog**，可用于检查包含模块（以及其他MEF导出类型）的程序集的目录。在这种情况下，您将覆盖**ConfigureAggregateCatalog**方法以注册该目录。此方法仅适用于WPF。

要使用此方法，首先需要使用**ModuleExport**属性将模块名称和依赖项应用于模块，如以下代码示例所示。这允许MEF导入模块并允许Prism 更新**ModuleCatalog**。

```c#
protected override void ConfigureAggregateCatalog()
{
    base.ConfigureAggregateCatalog();
    . . .
    DirectoryCatalog catalog = new DirectoryCatalog(&#34;DirectoryModules&#34;);
    this.AggregateCatalog.Catalogs.Add(catalog);
}
```

#### 使用MEF在代码中指定依赖关系

对于使用MEF的WPF应用程序，请使用**ModuleExport**属性，如下所示。

```c#
// (when using MEF)
[ModuleExport(typeof(ModuleA), DependsOnModuleNames = new string[] { &#34;ModuleD&#34; })]
public class ModuleA : IModule
{
    ...
}
```

因为MEF允许您在运行时发现模块，所以您还可以在运行时发现模块之间的新依赖关系。虽然您可以在**ModuleCatalog**旁边使用MEF ，但重要的是要记住**ModuleCatalog**在从XAML或配置加载时（在加载任何模块之前）验证依赖关系链。如果**ModuleCatalog中**列出了一个模块，然后使用MEF加载，则将使用**ModuleCatalog**依赖项，并忽略**DependsOnModuleNames**属性。

#### 使用MEF指定按需加载

如果使用MEF和**ModuleExport**属性来指定模块和模块依赖关系，则可以使用**InitializationMode**属性指定应按需加载模块，如此处所示。

```c#
[ModuleExport(typeof(ModuleC), InitializationMode = InitializationMode.OnDemand)]
public class ModuleC : IModule
{
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism8-modular/  

