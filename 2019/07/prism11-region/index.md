# Prism Region


注意：该版本为Prism6,最新版已有较大改动。

复合应用程序用户界面（UI）由松散耦合的可视组件组成，这些组件称为**View**，通常包含在应用程序模块中，但它们并非必须如此。如果将应用程序划分为模块，则需要一些方法来松散地组合UI，但即使视图不在模块中，您也可以选择使用此方法。对用户而言，该应用程序提供了无缝的用户体验，并提供了完全集成的应用程序。

要构建UI，您需要一个体系结构，允许您创建由在运行时生成的松散耦合的可视元素组成的布局。此外，该体系结构应该为这些可视元素提供以松散耦合方式进行通信的策略。

可以使用以下范例之一构建应用程序UI：

- 表单的所有必需控件都包含在单个可扩展应用程序标记语言（**XAML**）文件中，在设计时组成表单。
- 表单的逻辑区域被分成不同的部分，通常是用户控件。部件由表单引用，表单在设计时组成。
- 表单的逻辑区域被分成不同的部分，通常是用户控件。表单中的部分未知，并在运行时动态添加到表单中。使用此方法的应用程序称为使用UI组合模式的复合应用程序。

股票交易者参考实施（股票交易者RI）是通过将来自不同模块的多个视图加载到由shell公开的区域组成的，如下图所示。

![股票交易者RI地区和视图](/prism/Ch7UIFig11.png)

### UI布局概念

复合应用程序中的根对象称为*shell*。shell充当应用程序的母版页。shell包含一个或多个regions。regions是将在运行时加载的内容的占位符。区域附加到UI元素，例如**ContentControl**，**ItemsControl**，**TabControl**或自定义控件，并管理UI元素的内容。区域内容可以自动或按需加载，具体取决于应用程序要求。

通常，区域的内容是视图。视图将您希望保留的UI的一部分封装为尽可能与应用程序的其他部分分离。您可以将视图定义为用户控件，数据模板甚至自定义控件。

区域管理视图的显示和布局。区域可以通过其名称以分离的方式访问，并支持动态添加或删除视图。区域附加到主机控件。将区域视为动态加载视图的容器。

以下部分介绍了复合应用程序开发的高级核心概念。

### shell

shell是包含主UI内容的应用程序根对象。在Windows Presentation Foundation（WPF）应用程序中，shell是**Window**对象。

shell扮演master page的角色，为应用程序提供布局结构。shell包含一个或多个named regions，其中模块可以指定将显示的视图。它还可以定义某些顶级UI元素，例如背景，主菜单和工具栏。

shell定义了应用程序的整体外观。它可以定义在shell布局本身中存在和可见的样式和边框，还可以定义将应用于插入到shell中的视图的样式，模板和主题。

通常，shell是WPF应用程序项目的一部分。包含shell的程序集可能引用也可能不引用包含要在shell的区域中加载的视图的程序集。

### Views

View是复合应用程序中UI构造的主要单元。您可以将视图定义为 user control，page，data template或custom control。视图将您希望保留的UI的一部分封装为尽可能与应用程序的其他部分分离。您可以根据封装或功能选择视图中的内容，也可以选择将某些内容定义为视图，因为您的应用程序中将包含该视图的多个实例。

由于WPF的内容模型，定义视图所需的Prism库没有特定的内容。定义视图的最简单方法是定义用户控件。要向UI添加视图，您只需要一种方法来构建它并将其添加到容器中。WPF提供了执行此操作的机制。Prism Library添加了定义可在运行时动态添加视图的区域的功能。

#### Composite Views

支持特定功能的视图可能会变得复杂。在这种情况下，您可能希望将视图划分为多个子视图，并让父视图句柄通过将子视图用作部件来构建自身。应用程序可能会在设计时静态地执行此操作，或者它可能支持让模块在运行时通过包含的区域添加子视图。如果某个视图未在单个视图类中完全定义，则可以将其称为复合视图。在许多情况下，复合视图负责构建子视图和协调它们之间的交互。您可以使用Prism Library命令和事件聚合器设计从其兄弟视图及其父组合视图中更松散耦合的子视图。

#### 视图和设计模式

虽然Prism Library不要求您使用它们，但在实现视图时应考虑使用多种UI设计模式之一。Stock Trader RI和QuickStart演示了Model-View-ViewModel（MVVM）模式，作为在视图布局和视图逻辑之间实现清晰分离的一种方式。

建议使用MVVM UI设计模式，因为它非常适合Microsoft XAML平台。这些平台的依赖属性系统和富数据绑定堆栈使视图和视图模型能够以松散耦合的方式进行通信。

将逻辑与视图分离对于可测试性和可维护性非常重要，并且它改进了开发人员 - 设计人员的工作流程。

如果使用用户控件或自定义控件创建视图并将所有逻辑放在代码隐藏文件中，则您的视图可能难以测试，因为您必须创建视图实例以对逻辑进行单元测试。如果视图派生或依赖于将WPF组件作为其执行上下文的一部分运行，则这是一个问题。为了确保您可以在没有这些依赖关系的情况下单独测试视图逻辑，您需要能够创建视图的模型以删除执行上下文的依赖关系，这需要视图和逻辑的单独类。

如果将视图定义为数据模板，则没有与视图本身关联的代码。因此，您必须将关联逻辑放在其他位置。逻辑与可测试性所需的布局清晰分离也有助于使视图更易于维护。

注意：单元测试和UI自动化测试是两种不同类型的测试，具有不同的覆盖范围。*

单元测试最佳实践建议单独测试对象。要实现对象隔离，每个外部依赖项都需要一个模型或存根。然后针对该对象运行粒度单元测试。

*UI自动化测试运行应用程序，将手势应用于UI，然后测试预期结果。此类测试验证UI元素是否正确连接到应用程序逻辑。*

将逻辑与视图分开可以清晰地分离关注点。除了可测试性考虑因素之外，这种分离使设计人员能够独立于开发人员在UI上工作。有关MVVM的更多信息，请参阅[实现MVVM模式](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)。

### Commands, UI Triggers, Actions, and Behaviors

在代码隐藏文件中使用其逻辑实现视图时，可以向服务UI交互添加事件处理程序。但是，使用MVVM时，视图模型无法直接处理UI引发的事件。要将UI手势事件路由到视图模型，您可以使用命令或UI触发器，操作和行为。

##### 命令

命令将语义和从执行命令的逻辑调用命令的对象分开。内置命令是指示操作是否可用的能力。UI中的命令是绑定到视图模型上的**ICommand**属性的数据。有关命令的详细信息，请参阅[命令](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#commands)在[实现MVVM模式](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)。

##### UI Triggers, Actions, and Behaviors

触发器，操作和行为是**Microsoft.Expression.Interactivity**命名空间的一部分，随Blend for Visual Studio 2013一起提供。它们也是Blend SDK的一部分。触发器，操作和行为提供了一个用于处理UI事件或命令的综合API，然后将它们路由到**DataContext**公开的**ICommand**属性方法。有关UI触发器，操作和行为的更多信息，请参阅在[高级MVVM方案中](http://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html)[实现MVVM模式](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)和[交互触发器和事件到命令的部分](http://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html#interaction-triggers-and-commands)。

##### 用户交互

用户交互是应用程序向用户呈现的交互。这些交互通常是呈现给用户的弹出窗口。在MVVM场景中，可以从视图或视图模型生成这些用户交互。Prism为视图模型需要请求用户交互时的情况提供**InteractionRequests**和**InteractionRequestTriggers**，为触发指定事件时视图需要调用命令时提供**InvokeCommandAction**操作。

有关用户交互，示例以及如何使用它们的更多信息，请参阅[Interactivity QuickStart](https://msdn.microsoft.com/en-us/library/ff921081(v=pandp.40).aspx)。

#### 数据绑定

数据绑定是XAML平台最重要的框架功能之一。要在XAML平台上成功开发应用程序，您需要充分了解数据绑定。

数据绑定充分利用了依赖属性系统提供的内在更改通知。当与**INotifyPropertyChanged**接口的公共语言运行时（CLR）类实现结合使用时，更改通知将启用参与数据绑定的目标和源对象之间的无代码交互。

通过使用值转换器将一种类型转换为另一种类型，数据绑定可以使不同的目标和源类型与数据绑定。数据绑定在其管道中有多个验证挂钩，可用于验证用户输入。

强烈建议您阅读MSDN上的[依赖项属性概述](http://msdn.microsoft.com/en-us/library/ms752914.aspx)和[数据绑定概述](http://msdn.microsoft.com/en-us/library/ms752347.aspx)主题。完全了解这两个主题对于在Microsoft XAML平台上成功开发应用程序至关重要。有关数据绑定的更多信息，请参阅[数据绑定](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#data-binding)在[实现MVVM模式](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)。

### Regions

通过区域管理器，区域和区域适配器在Prism库中启用区域。接下来的部分将介绍它们如何协同工作。

#### Region Manager

该**RegionManager**类负责创建和维护区域的集合主机控制。该**RegionManager**使用一个新的区域与主机控制相关联的特定控制适配器。下图显示了**RegionManager**设置的区域，控件和适配器之间的关系。

![区域，控件和适配器关系](/prism/Ch7UIFig12.png)

该**RegionManager**可以在代码中创建或XAML地区。所述**RegionManager.RegionName**附加属性是用来通过将附加属性到主机控制以在XAML的区域。

应用程序可以包含**RegionManager的**一个或多个实例。您可以指定要在其中注册区域的**RegionManager**实例。如果要在可视树中移动控件并且不希望在删除附加属性值时清除区域，则此选项非常有用。

所述**RegionManager**提供RegionContext附加属性，允许其区域共享数据。

#### Region Implementation

区域是实现**IRegion**接口的类。术语*区域*表示可以保存UI中呈现的动态数据的容器。区域允许Prism库将包含在模块中的动态内容放置在UI容器中的预定义占位符中。

区域可以包含任何类型的UI内容。模块可以包含作为用户控件呈现的UI内容，与数据模板相关联的数据类型，自定义控件或这些的任何组合。这使您可以定义UI区域的外观，然后让模块将内容放在这些预定区域中。

区域可以包含零个或多个项目。根据区域管理的主机控件的类型，可以看到一个或多个项目。例如，**ContentControl**只能显示单个对象。但是，它所在的区域可以包含许多项目，**ItemsControl**可以显示多个项目。这允许区域中的每个项目在UI中可见。

在下图中，Stock Trader RI shell包含四个区域：**MainRegion**，**MainToolbarRegion**，**ResearchRegion**和**ActionRegion**。这些区域由应用程序中的各种模块填充 - 内容可以随时更改。

![股票交易者RI地区](/prism/Ch7UIFig13.png)

##### 模块用户控制到区域映射

Module User Control to Region Mapping

要演示模块和内容如何与区域关联，请参阅下图。它显示了**WatchModule**和**NewsModule**与shell中相应区域的关联。

所述**MainRegion**包含**WatchListView**用户控制，这被包含在**WatchModule**。所述**ResearchRegion**还包含**ArticleView**用户控制，这被包含在**NewsModule**。

在使用Prism Library创建的应用程序中，这样的映射将成为设计过程的一部分，因为设计人员和开发人员使用它们来确定在特定区域中建议的内容。这允许设计人员确定所需的总空间以及必须添加的任何其他项目，以确保内容在可允许的空间中可见。

![模块用户控制到区域映射](/prism/Ch7UIFig14.png)

#### Default Region Functionality

虽然您不需要完全理解区域实现以使用它们，但了解控件和区域如何关联以及默认区域功能可能很有用：例如，区域如何定位和实例化视图，以及如何在视图时通知视图是活动视图，还是视图生命周期如何与激活相关联。

以下部分描述了区域适配器和区域行为。

##### Region Adapter

要将UI控件公开为区域，它必须具有区域适配器。区域适配器负责创建区域并将其与控件相关联。这允许您使用**IRegion**接口以一致的方式管理UI控件内容。每个区域适配器都适应特定类型的UI控件。Prism库提供以下三种区域适配器：

- **ContentControlRegionAdapter**。此适配器适应**System.Windows.Controls.ContentControl**类型和派生类的控件。
- **SelectorRegionAdapter**。此适配器适应从**System.Windows.Controls.Primitives.Selector**类派生的控件，例如**System.Windows.Controls.TabControl**控件。
- **ItemsControlRegionAdapter**。此适配器适应**System.Windows.Controls.ItemsControl**类型和派生类的控件。

##### Region Behaviors

PrismLibrary介绍了区域行为的概念。这些是可插拔(pluggable )组件，为区域提供了大部分功能。引入了区域行为以支持视图发现和区域上下文（在本主题后面描述），并创建在WPF和Silverlight中一致的API。此外，行为提供了扩展区域实施的有效方式。

区域行为是附加到区域的类，以为该区域提供附加功能。此行为附加到该区域，并在该区域的生命周期内保持活动状态。例如，当**AutoPopulateRegionBehavior**附加到某个区域时，它会自动实例化并添加针对具有该名称的区域注册的任何**ViewType**。对于该区域的生命周期，它会持续监视**RegionViewRegistry**以进行新注册。可以在系统范围或每个区域的基础上轻松添加自定义区域行为或替换现有行为。

下一节将介绍自动添加到所有区域的默认行为。一种行为**SelectorItemsSourceSyncBehavior**仅附加到从**Selector**派生的控件。

##### 注册行为

该**RegionManagerRegistrationBehavior**是负责确保该区域被注册到正确的**RegionManager**。当视图或控件作为另一个控件或区域的子项添加到可视树中时，控件中定义的任何区域都应该在父控件的**RegionManager**中注册。删除子控件后，注册的区域将取消注册。

##### Auto-Population Behavior

有两个类负责实现视图发现。其中之一是**AutoPopulateRegionBehavior**。当它附加到某个区域时，它会检索在该区域名称下注册的所有视图类型。然后，它创建这些视图的实例并将它们添加到该区域。创建区域后，**AutoPopulateRegionBehavior将**监视**RegionViewRegistry**以查找该区域名称的任何新注册的视图类型。

如果您想要更多地控制视图发现过程，请考虑创建自己的**IRegionViewRegistry**实现和**AutoPopulateRegionBehavior**。

##### Region Context Behaviors

区域上下文功能包含在两个行为中：**SyncRegionContextWithHostBehavior**和**BindRegionContextToDependencyObjectBehavior**。这些行为负责监视对区域所做的上下文的更改，然后将上下文与附加到视图的上下文依赖项属性同步。

##### Activation Behavior

所述**RegionActiveAwareBehavior**负责通知的图，如果它是有效或无效。视图必须实现**IActiveAware**才能接收这些更改通知。此主动感知通知是单向的（它从行为传播到视图）。通过更改**IActiveAware**接口上的活动属性，视图不会影响其活动状态。

##### Region Lifetime Behavior

所述**RegionMemberLifetimeBehavior**负责确定如果一个项目应该从区域时被去激活被移除。该**RegionMemberLifetimeBehavior**监控区域的**ActiveViews**收集发现的项目，过渡到非激活状态。该行为检查已删除的项目是否为**IRegionMemberLifetime**或**RegionMemberLifetimeAttribute**（**按此**顺序），以确定它是否应在删除时保持活动状态。

如果集合中的项是**System.Windows.FrameworkElement**，它还将检查其**DataContext**的**IRegionMemberLifetime**或**RegionMemberLifetimeAttribute**。

按以下顺序检查区域项：

1. **IRegionMemberLifetime.KeepAlive**值
2. **DataContext的IRegionMemberLifetime.KeepAlive**值
3. **RegionMemberLifetimeAttribute.KeepAlive**值
4. **DataContext的RegionMemberLifetimeAttribute.KeepAlive**值

##### Control-Specific Behaviors

所述**SelectorItemsSourceSyncBehavior**仅用于从导出的控制**选择器**，例如在一个WPF标签控制。它负责将区域中的视图与选择器的项同步，然后将区域中的活动视图与选择器的选定项同步。

#### Extending the Region Implementation

Prism Library提供了扩展点，允许您自定义或扩展所提供API的默认行为。例如，您可以编写自己的区域适配器，区域行为或更改Navigation API分析URI的方式。有关扩展棱镜库的更多信息，请参阅[扩展棱镜库](http://prismlibrary.github.io/docs/wpf/legacy/Appendix-D-Extending-Prism.html)。

### View Composition

视图合成是视图的构建。在复合应用程序中，必须在运行时在应用程序UI中的特定位置显示来自多个模块的视图。要实现此目的，您需要定义视图的显示位置以及在这些位置创建和显示视图的方式。

可以通过视图发现自动创建和显示视图，也可以通过视图注入以编程方式显示视图。这两种技术决定了各个视图如何映射到应用程序UI中的命名位置。

#### View Discovery

在视图发现中，您可以在**RegionViewRegistry中**在区域名称和视图类型之间建立关系。创建区域时，该区域将查找与该区域关联的所有**ViewType**，并自动实例化并加载相应的视图。因此，使用视图发现时，您无法明确控制何时加载和显示与区域对应的视图。

#### View Injection

在视图注入中，您的代码获取对区域的引用，然后以编程方式向其中添加视图。通常，这在模块初始化或作为用户操作的结果时完成。您的代码将按名称查询**RegionManager**中的特定区域，然后将视图注入其中。通过视图注入，您可以更好地控制何时加载和显示视图。您还可以从该地区删除视图。但是，使用视图注入时，无法将视图添加到尚未创建的区域。

#### View Injection

Prism Library 4.0包含导航API。Navigation API允许您将区域导航到URI，从而简化了视图注入过程。Navigation API实例化视图，将其添加到区域，然后激活它。此外，Navigation API允许导航回包含在区域中的先前创建的视图。有关导航API的更多信息，请参阅[导航](http://prismlibrary.github.io/docs/wpf/legacy/Navigation.html)。

#### 使用View Discovery与View Injection

选择要用于区域的视图加载策略取决于应用程序要求和区域的功能。

在以下情况下使用视图发现：

- 需要或需要自动加载视图。
- 视图的单个实例将加载到该区域中。

在以下情况下使用视图注入：

- 您的应用程序使用导航API。
- 您需要对创建和显示视图的时间进行显式或程序控制，或者需要从区域中删除视图; 例如，作为应用程序逻辑或导航的结果。
- 您需要在区域中显示相同视图的多个实例，其中每个视图实例都绑定到不同的数据。
- 您需要控制添加视图的区域的哪个实例。例如，您要将客户详细信息视图添加到特定客户详细信息区域。（此方案需要实现作用域，如本主题后面所述。）

### UI布局方案

在复合应用程序中，来自多个模块的视图在运行时显示在应用程序UI中的特定位置。要实现此目的，您需要定义视图的显示位置以及在这些位置创建和显示视图的方式。

视图和将在其中显示的UI中的位置的分离允许应用程序的外观和布局独立于区域内出现的视图而发展。

下一节将介绍开发复合应用程序时将遇到的核心方案。适当时，Stock Trader RI的示例将用于演示该场景的解决方案。

### 实现Shell

shell是应用程序根对象，其中包含主UI内容。在Windows Presentation Foundation（WPF）应用程序中，shell是**Window**对象。

shell可以包含命名区域，其中模块可以指定将出现的视图。它还可以定义某些顶级UI元素，例如主菜单和工具栏。shell定义应用程序的整体结构和外观，类似于ASP.NET母版页控件。它可以定义在shell布局本身中存在和可见的样式和边框，还可以定义应用于插入到shell中的视图的样式，模板和主题。

作为应用程序体系结构的一部分，您不需要使用独特的shell来使用Prism库。如果要构建一个全新的复合应用程序，实现一个shell提供了一个定义良好的根和初始化模式，用于设置应用程序的主UI。但是，如果要将Prism Library功能添加到现有应用程序，则无需更改应用程序的基本体系结构即可添加shell。相反，您可以更改现有的窗口定义或控件，以添加可根据需要提取视图的区域。

您的应用程序中也可以有多个shell。如果您的应用程序旨在为用户打开多个顶级窗口，则每个顶级窗口都充当其包含的内容的shell。

#### 股票交易员RI Shell

WPF Stock Trader RI有一个shell作为主窗口。在下图中，将突出显示shell和视图。shell是Stock Trader RI启动时显示的主窗口，其中包含所有视图。它定义了模块添加其视图的区域以及一些顶级UI项目，包括CFI Stock Trader标题和Watch List撕下横幅。

![股票交易者RI壳窗口，区域和视图](/prism/Ch7UIFig18.png)

Stock Trader RI中的shell实现由Shell.xaml及其代码隐藏文件Shell.xaml.cs及其视图模型ShellViewModel.cs提供。Shell.xaml包括作为shell一部分的布局和UI元素，包括模块添加其视图的区域的定义。

以下XAML显示了定义shell的结构和主要XAML元素。请注意，**RegionName**附加属性用于定义四个区域，窗口背景图像为shell提供背景。

```xaml
&lt;!--Shell.xaml (WPF) --&gt;
&lt;Window x:Class=&#34;StockTraderRI.Shell&#34;&gt;

    &lt;!--shell background --&gt;
    &lt;Window.Background&gt;
        &lt;ImageBrush ImageSource=&#34;Resources/background.png&#34; Stretch=&#34;UniformToFill&#34;/&gt;
    &lt;/Window.Background&gt;

    &lt;Grid&gt;

        &lt;!-- logo --&gt;
        &lt;Canvas x:Name=&#34;Logo&#34; ...&gt;
            &lt;TextBlock Text=&#34;CFI&#34; ... /&gt;
            &lt;TextBlock Text=&#34;STOCKTRADER&#34; .../&gt;
        &lt;/Canvas&gt;

        &lt;!-- main bar --&gt;
        &lt;ItemsControl 
            x:Name=&#34;MainToolbar&#34;
            prism:RegionManager.RegionName=&#34;{x:Static inf:RegionNames.MainToolBarRegion}&#34;/&gt;

        &lt;!-- content --&gt;
        &lt;Grid&gt;
            &lt;Controls:AnimatedTabControl
                x:Name=&#34;PositionBuySellTab&#34;
                prism:RegionManager.RegionName=&#34;{x:Static inf:RegionNames.MainRegion}&#34;/&gt;
        &lt;/Grid&gt;

        &lt;!-- details --&gt;
        &lt;Grid&gt;
            &lt;ContentControl
                x:Name=&#34;ActionContent&#34;
                prism:RegionManager.RegionName=&#34;{x:Static inf:RegionNames.ActionRegion}&#34;/&gt;
        &lt;/Grid&gt;

        &lt;!-- sidebar --&gt;
        &lt;Grid x:Name=&#34;SideGrid&#34;&gt;
            &lt;Controls:ResearchControl
                prism:RegionManager.RegionName=&#34;{x:Static inf:RegionNames.ResearchRegion}&#34; /&gt;
        &lt;/Grid&gt;

    &lt;/Grid&gt;
&lt;/Window&gt;
```

在实施**Shell**代码隐藏文件是非常简单的。当引导程序创建**Shell**，它的依赖将被托管扩展框架（MEF）解决。shell具有单一依赖关系 - **ShellViewModel -**在构造期间**注入**，如以下示例所示。

```c#
// Shell.xaml.cs
[Export]
public partial class Shell : Window
{
    public Shell()
    {
        InitializeComponent();
    }

    [Import]
    ShellViewModel ViewModel
    {
        set
        {
            this.DataContext = value;
        }
    }
}
// ShellViewModel.cs
[Export]
public class ShellViewModel : BindableBase
{
    // This is where any view model logic for the shell would go.
}
```

代码隐藏文件中的最小代码说明了复合应用程序体系结构的强大功能和简单性以及shell与其组成视图之间的松散耦合。

### 定义区域

您可以通过定义具有命名位置的布局（称为区域）来定义视图的显示位置。区域充当将在运行时显示的一个或多个视图的占位符。模块可以在布局中的区域中定位和添加内容，而无需知道区域的显示方式和位置。这允许更改布局而不影响将内容添加到布局的模块。

通过将区域名称分配给WPF控件（在上一个Shell.xaml文件中显示的XAML中或代码中）来定义区域。可以通过区域名称访问区域。在运行时，视图将添加到命名的Region控件，然后根据视图实现的布局策略显示视图。例如，选项卡控件区域将以选项卡式排列其子视图。区域支持添加或删除视图。可以通过编程方式或自动方式在区域中创建和显示视图。在Prism Library中，前者通过视图注入实现，后者通过视图发现实现。这两种技术决定了各个视图如何映射到应用程序UI中的命名区域。

应用程序的shell定义了最高级别的应用程序布局; 例如，通过指定主要内容和导航内容的位置，如下图所示。这些高级视图中的布局类似地定义，允许以递归方式组合整个UI。

![shell](/prism/Ch7UIFig16.png)

区域有时用于定义逻辑上相关的多个视图的位置。在这种情况下，区域控件通常是一个**ItemsControl**派生的控件，它将根据它实现的布局策略显示视图，例如以堆叠或选项卡式布局排列。

区域也可用于定义单个视图的位置; 例如，通过使用**ContentControl**。在这种情况下，区域控件一次只显示一个视图，即使多个视图映射到该区域位置也是如此。

#### 股票交易者RI shell Regions

![股票交易者RI壳区域](/prism/Ch7UIFig21.png)

当应用程序购买或出售股票时，股票交易者RI也会演示多视图布局。买/卖区域是一个列表样式区域，显示多个买入/卖出视图（**OrderCompositeView**）作为其列表的一部分，如下图所示。

![ItemsControl区域](/prism/Ch7UIFig22.png)

shell的**ActionRegion**包含**OrdersView**。该**OrdersView**包含**提交所有**和**取消所有**按钮还有**OrdersRegion**。所述**OrdersRegion**附着到**列表框**，其显示多个控制**OrderCompositeViews**。

#### IRegion

区域是实现**IRegion**接口的类。该区域是容纳控件显示内容的容器。以下代码显示了**IRegion**接口。

```c#
public interface IRegion : INavigateAsync, INotifyPropertyChanged
{
    IViewsCollection Views { get; }
    IViewsCollection ActiveViews { get; }
    object Context { get; set; }
    string Name { get; set; }
    Comparison&lt;object&gt; SortComparison { get; set; }
    IRegionManager Add(object view);
    IRegionManager Add(object view, string viewName);
    IRegionManager Add(object view, string viewName, bool createRegionManagerScope);
    void Remove(object view);
    void Deactivate(object view);
    object GetView(string viewName);
    IRegionManager RegionManager { get; set; }
    IRegionBehaviorCollection Behaviors { get; }
    IRegionNavigationService NavigationService { get; set; }
}
```

#### Adding a Region in XAML

该**RegionManager**提供一个附加属性，您可以使用在XAML简单的区域生成。要使用附加属性，必须将Prism Library命名空间加载到XAML中，然后使用**RegionName**附加属性。以下示例显示如何在具有**AnimatedTabControl**的窗口中使用附加属性。

注意使用**x：Static**标记扩展来引用**MainRegion**字符串常量。这种做法消除了XAML中的魔术字符串。

```xaml
&lt;!-- (WPF) --&gt;
&lt;Controls:AnimatedTabControl 
    x:Name=&#34;PositionBuySellTab&#34;
    prism:RegionManager.RegionName=&#34;{x:Static inf:RegionNames.MainRegion}&#34;/&gt;
```

#### Adding a Region by Using Code

所述**RegionManager**可以直接在不使用XAML寄存器区域。以下代码示例演示如何从代码隐藏文件向控件添加区域。首先，获得对区域管理者的引用。然后，使用**RegionManager**静态方法**SetRegionManager**和**SetRegionName**，将该区域附加到UI的**ActionContent**控件，然后将该区域命名为**ActionRegion**。

```cs
IRegionManager regionManager = ServiceLocator.Current.GetInstance&lt;IRegionManager&gt;();
RegionManager.SetRegionManager(this.ActionContent, regionManager);
RegionManager.SetRegionName(this.ActionContent, &#34;ActionRegion&#34;);
```

### Displaying Views in a Region When the Region Loads

使用视图发现方法，模块可以为特定的命名位置注册视图（视图模型或表示模型）。在运行时显示该位置时，将自动创建并在其中显示已为该位置注册的所有视图。

模块使用注册表注册视图。父视图查询此注册表以发现为命名位置注册的视图。发现它们之后，父视图会将这些视图添加到占位符控件中，从而将这些视图放在屏幕上。

加载应用程序后，将通知组合视图以处理已添加到注册表的新视图的放置。

下图显示了视图发现方法。

![查看发现](/prism/Ch7UIFig19.png)

Prism Library定义了一个标准注册表**RegionViewRegistry**，用于注册这些命名位置的视图。

要显示区域中的视图，请使用区域管理器注册视图，如以下代码示例所示。您可以直接向区域注册视图类型，在这种情况下，视图将由依赖项注入容器构造，并在加载托管区域的控件时添加到区域。

```c#
// View discovery
this.regionManager.RegisterViewWithRegion(&#34;MainRegion&#34;, typeof(EmployeeView));
```

（可选）您可以提供一个返回要显示的视图的委托，如下一个示例所示。区域管理器将在创建区域时显示视图。

```c#
// View discovery
this.regionManager.RegisterViewWithRegion(&#34;MainRegion&#34;, () =&gt; this.container.Resolve&lt;EmployeeView&gt;());
```

UI Composition QuickStart在EmployeeModule ModuleInit.cs文件中有一个演练，演示了如何使用**RegisterViewWithRegion**方法。

### Displaying Views in a Region Programmatically

在视图注入方法中，视图以编程方式添加或由管理它们的模块从命名位置删除。要启用此功能，应用程序将在UI中包含命名位置的注册表。模块可以使用注册表查找其中一个位置，然后以编程方式将视图注入其中。为了确保可以类似地访问注册表中的位置，每个命名位置都遵循用于注入视图的公共接口。下图显示了视图注入方法。

![查看注射](/prism/Ch7UIFig20.png)

Prism库定义了一个标准注册表**RegionManager**和一个标准接口**IRegion**，用于访问这些位置。

要使用视图注入向区域添加视图，请从区域管理器中获取区域，然后调用**Add**方法，如以下代码所示。使用视图注入时，仅在将视图添加到区域后才会显示视图，这可能在加载模块或用户操作完成预定义操作时发生。

```c#
// View injection
IRegion region = regionManager.Regions[&#34;MainRegion&#34;];

var ordersView = container.Resolve&lt;OrdersView&gt;();
region.Add(ordersView, &#34;OrdersView&#34;);
region.Activate(ordersView);
```

除了Stock Trader RI之外，UI Composition QuickStart还有一个实现视图注入的演练。

#### Region Navigation

Prism Library 5.0包含导航API，它提供了丰富且一致的API，用于在WPF应用程序中实现导航。

区域导航是视图注入的一种形式。处理导航请求时，它将尝试在可以满足请求的区域中查找视图。如果找不到匹配的视图，它会调用应用程序容器来创建对象，然后将对象注入目标区域并激活它。

Stock Trader RI **ArticleViewModel**的以下代码示例说明了如何发起导航请求。

```c#
this.regionManager.RequestNavigate(RegionNames.SecondaryRegion, new Uri(&#34;/NewsReaderView&#34;, UriKind.Relative));
```

有关区域导航的更多信息，请参阅[导航](http://prismlibrary.github.io/docs/wpf/legacy/Navigation.html)。视图切换导航快速入门和基于状态的导航快速入门也是实现应用程序导航的示例。

#### Ordering Views in a Region

无论是使用视图发现还是查看注入，应用程序都可能需要命令视图在**TabControl**，**ItemsControl**或显示多个活动视图的任何其他控件中的显示方式。默认情况下，视图按其注册顺序显示并添加到区域中。

构建复合应用程序时，通常会从不同的模块注册视图。声明模块之间的依赖关系有助于缓解问题，但是当模块和视图没有任何真正的相互依赖关系时，声明人工依赖会不必要地耦合模块。

为了允许视图参与自己的排序，Prism库提供了**ViewSortHint**属性。此属性包含字符串**Hint**属性，该属性允许视图声明在区域中如何排序的提示。

显示视图时，**Region**类使用默认视图排序例程，该例程使用提示对视图进行排序。这是一个简单的区分大小写的排序。具有sort hint属性的视图在没有排序的视图之前排序。此外，没有属性的那些按照添加到区域的顺序显示。

如果要更改视图的排序方式，**Region**类提供了一个**SortComparison**属性，您可以使用自己的**Comparison &lt; object &gt;**委托方法设置该属性。值得注意的是，区域的**Views**和**ActiveViews**属性的顺序会反映在UI中，因为诸如**ItemsControlRegionAdapter之类的**适配器直接绑定到这些属性。自定义区域适配器可以实现自己的排序和过滤器，它将覆盖区域命令视图的方式。

View Switching QuickStart演示了一种简单的编号方案，用于对左侧导航区域中的视图进行排序。以下代码示例显示应用于每个导航项视图的**ViewSortHint**。

```c#
[Export]
[ViewSortHint(&#34;01&#34;)]
public partial class EmailNavigationItemView { ... }

[Export]
[ViewSortHint(&#34;02&#34;)]
public partial class CalendarNavigationItemView { ... }

[Export]
[ViewSortHint(&#34;03&#34;)]
public partial class ContactsDetailNavigationItemView { ... }

[Export]
[ViewSortHint(&#34;04&#34;)]
public partial class ContactsAvatarNavigationItemView { ... }
```

### 在多个区域之间共享数据

Prism Library提供了多种方法来在视图之间进行通信，具体取决于您的方案。区域管理器提供**RegionContext**属性作为这些方法之一。

当您想要共享父视图和区域中托管的子视图之间的上下文时，**RegionContext**非常有用。**RegionContext**是附加属性。您可以在区域控件上设置上下文的值，以便可以使该区域控件中显示的所有子视图都可以使用它。区域上下文可以是任何简单或复杂的对象，也可以是数据绑定值。该**RegionContext**可以与任一视图中发现或视图注射使用。

注意： WPF中的**DataContext**属性用于设置视图的本地数据上下文。它允许视图使用数据绑定与视图模型，本地演示者或模型进行通信。**RegionContext**用于在多个视图之间共享上下文，而不是单个视图的本地视图。它提供了一种在多个视图之间共享上下文的简单机制。

以下代码显示了如何在XAML中使用**RegionContext**附加属性。

```xaml
&lt;TabControl AutomationProperties.AutomationId=&#34;DetailsTabControl&#34; 
    prism:RegionManager.RegionName=&#34;{x:Static local:RegionNames.TabRegion}&#34;
    prism:RegionManager.RegionContext=&#34;{Binding Path=SelectedEmployee.EmployeeId}&#34;
...&gt;
```

您还可以在代码中设置**RegionContext**，如以下示例所示。

```c#
RegionManager.Regions[&#34;Region1&#34;].Context = employeeId;
```

要在视图中检索**RegionContext**，请使用**RegionContext**类的**GetObservableContext**静态方法。它将视图作为参数传递，然后访问其**Value**属性，如以下代码示例所示。

```c#
private void GetRegionContext()
{
    this.Model.EmployeeId = (int)RegionContext.GetObservableContext(this).Value;
}
```

所述的值**RegionContext**可以从视图中通过简单地分配一个新的值到它的改变**值**属性。通过订阅**GetObservableContext**方法返回的**ObservableObject**上的**PropertyChanged**事件，可以选择通过视图通知**RegionContext**的更改。这允许在更改**RegionContext**时保持多个视图同步。以下代码示例演示了订阅**PropertyChanged**事件。

```c#
ObservableObject&lt;object&gt; viewRegionContext = 
                RegionContext.GetObservableContext(this);
viewRegionContext.PropertyChanged &#43;= this.ViewRegionContext_OnPropertyChangedEvent;

private void ViewRegionContext_OnPropertyChangedEvent(object sender, 
                    PropertyChangedEventArgs args)

{
    if (args.PropertyName == &#34;Value&#34;)
    {
        var context = (ObservableObject&lt;object&gt;) sender;
        int newValue = (int)context.Value;
    }
}
```

注意：该**RegionContext**被设置为在该区域托管内容对象上附加属性。这意味着内容对象必须从**DependencyObject**派生。在前面的示例中，视图是一个可视控件，最终从**DependencyObject**派生。

*如果选择使用WPF数据模板来定义视图，则内容对象将表示**ViewModel**或**PresentationModel**。如果视图模型或表示模型需要检索**RegionContext**，则需要从**DependencyObject**基类派生。*

### 创建区域的多个实例

只有视图注入才能使用范围区域。如果您需要视图以拥有自己的区域实例，则应使用它们。定义具有附加属性的区域的视图会自动继承其父级的**RegionManager**。通常，这是在shell窗口中注册的全局**RegionManager**。如果应用程序创建该视图的多个实例，则每个实例都会尝试使用父**RegionManager**注册其区域。**RegionManager**只允许唯一命名的区域; 因此，第二次注册会产生错误。

相反，使用作用域区域，以便每个视图都有自己的**RegionManager**，其区域将使用该**RegionManager**而不是父**RegionManager注册**，如下图所示。

![父级和范围的RegionManagers](/prism/Ch7UIFig31.png)

要为视图创建本地**RegionManager**，请指定在将视图添加到区域时应创建新的**RegionManager**，如以下代码示例所示。

```c#
IRegion detailsRegion = this.regionManager.Regions[&#34;DetailsRegion&#34;];
View view = new View();
bool createRegionManagerScope = true;
IRegionManager detailsRegionManager = detailsRegion.Add(view, null, createRegionManagerScope);
```

该**添加**方法将返回新**RegionManager**该视图可以保留进一步访问本地范围。

### 创建视图

应用程序的可视化表示形式可以采用多种形式，包括用户控件，自定义控件和数据模板等。对于Stock Trader RI，用户控件通常用于表示主窗口上的不同部分，但这不是标准。在您的应用程序中，您应该使用您最熟悉的方法，这种方法适合您作为设计师的工作方式。无论应用程序中的主要可视化表示如何，您都将不可避免地在整体设计中使用用户控件，自定义控件和数据模板的组合。下图显示了Stock Trader RI使用这些不同项目的位置。此图还可作为以下部分的参考，这些部分描述了每个项目。

![股票交易者RI用户控件，自定义控件和数据模板的使用](/prism/Ch7UIFig32.png)

### 用户控件

Blend for Visual Studio 2013和Visual Studio 2013都为创建用户控件提供了丰富的支持。因此，建议使用这些工具创建的用户控件使用Prism Library创建UI内容。如本主题前面所述，Stock Trader RI广泛使用它们来创建将插入区域的内容。所述**WatchListView.xaml**用户控制是包含内部的简单的用户界面表示的一个很好的例子**WatchModule**。此控件是一个非常简单的控件，使用此模型可以直接创建。

#### 自定义控件

在某些情况下，用户控制太有限。在这些情况下，自定义布局或可扩展性比创建的简便性更重要。这是自定义控件有用的地方。在Stock Trader RI中，饼图控件就是一个很好的例子。该控制由来自头寸的数据组成，并显示整个投资组合的图表。与用户控件相比，这种类型的控件比创建用户控件更具挑战性，与用户控件相比，它在Blend for Visual Studio 2013和Visual Studio 2013中的视觉设计支持有限。

#### 数据模板

数据模板是大多数类型的数据驱动应用程序的重要组成部分。基于列表的控件的数据模板的使用在整个股票交易者RI中很普遍。在许多情况下，您可以使用数据模板来创建完整的可视化表示，而无需创建任何类型的控件。该**ResearchRegion**使用数据模板显示的文章，并与一个联合**项目**的风格，提供了的选择项目的指示。

Visual Studio 2013和Visual Studio 2013的Blend具有对数据模板的完全可视化设计支持。

#### 资源

样式，资源字典和控件模板等资源可以分散在整个应用程序中。复合应用程序尤其如此。在考虑放置资源的位置时，请特别注意UI元素与所需资源之间的依赖关系。Stock Trader RI解决方案（如下图所示）包含指示资源可以存在的各个区域的标签。

![跨解决方案的资源分配](/prism/Ch7UIFig33.png)

##### Application Resources

通常，应用程序资源是整个应用程序可用的资源。这些资源往往集中在根应用程序上，但它们也可以在模型或控件的类型基础上提供默认样式。例如，文本框样式应用于根应用程序中的文本框类型。除非在模块或控件级别覆盖样式，否则此样式将可用于应用程序中的所有文本框。

##### Module Resources

模块资源与根应用程序资源的作用相同，因为它们可以应用于模块中的所有项目。使用此级别的资源可以在整个模块中提供一致的外观，并且还允许在跨越一个或多个可视组件的更具体实例中重用。模块级别的资源使用应包含在单个模块中。在UI元素显示不正确时，创建模块之间的依赖关系可能导致难以找到的问题。

##### Control Resources

控制资源通常包含在控制库中，可供控制库中的所有控件使用。这些资源往往具有最有限的范围，因为控制库通常包含非常特定的控件，并且不包含用户控件。（在使用Prism Library创建的应用程序中，用户控件通常放在使用它们的模块中。）

### UI设计指南

本主题的目标是为正在使用Prism Library和WPF构建应用程序的XAML设计人员和开发人员提供一些高级指导。本主题描述UI布局，可视化表示，数据绑定，资源和表示模型。阅读本主题后，您应该高度了解如何基于Prism库设计应用程序的UI，以及一些可以帮助您在复合应用程序中创建可维护UI的技术。

#### 设计用户界面的指南

使用Prism Library创建的复合应用程序的布局建立在WPF的标准主体上 - 布局使用包含相关项的面板的概念。但是，对于复合应用程序，各种面板内的内容是动态的，在设计时不知道。这迫使设计人员和开发人员创建可以包含布局内容的页面结构，然后分别设计适合布局的每个元素。作为设计人员或开发人员，这意味着您必须考虑Prism库中的两个主要布局概念：容器组合和区域。

#### 容器组成

容器组合实际上只是WPF本身提供的包含模型的扩展。术语*容器*可以表示任何元素，包括窗口，页面，用户控件，面板，自定义控件，控件模板或数据模板，它们可以包含其他元素。

您可视化UI的方式因实现而异，但您会发现突出的重复主题。您将创建包含固定内容和动态内容的窗口，页面或用户控件。固定内容将包含包含UI元素的整体结构，动态内容将放置在区域内。

例如，WPF Stock Trader RI有一个名为Shell.xaml的启动窗口，其中包含应用程序的整体结构。下图显示了Blend for Visual Studio 2013中加载的shell。请注意，只有UI的固定部分可见。当应用程序加载时，shell的其余部分由模块动态插入到各个区域中。

在这种类型的应用程序中，设计时体验有点受限，但是您知道内容将在运行时放置在不同区域这一事实是您需要设计的。要查看此示例，请将下一个插图中主页面的设计器视图与其后的插图中的运行时视图进行比较。在设计器视图中，页面大多是空的。与运行时视图对比，其中存在包含具有位置数据的选项卡控件的位置区域，以及与所选股票相关的趋势线，饼图和新闻区域。设计器视图和运行时视图之间的差异表明了设计人员和开发人员在创建使用Prism Library构建的应用程序时所面临的挑战。

在设计时间内无法看到物品; 因此，确定它们的大小以及它们如何适应应用程序的整体外观有点困难。在为容器创建布局时，请考虑以下事项：

- 是否有任何大小限制会限制内容的大小？如果有，请考虑使用支持滚动的容器。
- 考虑使用扩展器和**ScrollViewer**组合，以适应大量动态内容需要适应受限区域的情况。
- 密切关注内容随着屏幕内容的增长而扩大的程度，以确保应用程序的外观在任何分辨率下都具有吸引力。

![股票交易者RI主窗口在Blend for Visual Studio 2013](/prism/StockTraderRIExpressionBlend.png)

股票交易者RI主窗口在Blend for Visual Studio 2013

![在运行时间的股票交易商RI主窗口](/prism/StockTraderRI.png)

在运行时间的股票交易商RI主窗口

#### 在设计时查看复合应用程序

前面的两个图说明了使用在运行时组成的高级视图的挑战之一。复合应用程序中的每个UI元素必须单独设计。这使得很难直观地看出复合页面或窗口在运行时的外观。要在组合状态下可视化组合视图，可以使用包含要测试的视图的所有UI元素的页面或窗口创建测试项目。

此外，请考虑在Blend for Visual Studio 2013和Visual Studio 2013中使用设计时样本数据功能，以使用数据填充UI元素。使用数据模板，列表控件，图表或图形时，设计时数据非常有用。有关更多信息，请参阅[设计时样本数据指南](http://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html#guidelines-for-design-time-sample-data)部分。

#### 布局

在设计复合应用程序的布局时，请考虑以下事项：

- shell定义了应用程序的主要布局。布局的每个区域都是一个区域，应保留为空容器。不要在设计时将内容放在区域内，因为内容将在运行时加载到那里。
- shell应包含背景，标题和页脚。将shell视为ASP.NET母版页。
- 充当区域的控制容器与它们包含的视图分离。因此，您应该能够在不修改控件的情况下更改视图的大小，并且应该能够在不修改视图的情况下更改控件的大小。定义视图大小时应考虑以下事项：
  - 如果视图将在多个区域中使用，或者如果不确定将在何处使用，请使用动态宽度和高度进行设计。
  - 如果视图具有固定大小，则shell的区域应使用动态大小。
  - 如果shell区域具有固定大小，则视图应使用动态大小。
  - 视图可能需要固定的高度和动态宽度。这方面的一个例子是位于Stock Trader RI侧栏的**PositionPieChart**视图。
  - 其他视图可能需要动态高度和宽度**。**例如，**StockTrader** RI侧栏中的**NewsReader**视图。高度本身取决于标题的长度，宽度应始终适应区域的大小（侧边栏宽度）。这同样适用于**PositionSummaryView**视图，其中网格的宽度应适应屏幕大小，高度应适应网格中的行数。
- 视图通常应具有透明背景，允许shell背景提供应用程序视觉背景。
- 始终使用命名资源来分配颜色，画笔，字体和字体大小，而不是直接在XAML中分配属性值。这使得应用程序维护更容易。它还允许应用程序在运行时响应资源字典中的更改。

#### 动画

在shell或视图中使用动画时，请考虑以下事项：

- 您可以为shell的布局设置动画，但您必须单独为其内容和视图设置动画。
- 分别设计和动画每个视图。
- 使用柔和或温和的动画来提供UI元素被带入视图或从视图中移除的视觉线索。这为应用程序提供了抛光的外观和感觉。

Blend for Visual Studio 2013提供了丰富的行为，简化功能，以及基于可视状态更改或事件动画和转换UI元素的出色编辑体验。有关更多信息，请参阅MSDN上的[VisualStateManager类](https://msdn.microsoft.com/en-us/library/cc626338(v=VS.95).aspx)。

#### 运行时优化

请考虑以下有关性能优化的提示：

- 将任何公共资源放在App.xaml文件或合并字典中以避免重复样式。

#### 设计时优化

以下部分描述了设计时方案，并提供了充分利用设计时体验的解决方案。

#### Large Solutions with Many XAML Resources

在具有许多XAML资源的大型应用程序中，可视化设计器的加载时间可能会受到影响，有时会显着影响。这种性能下降的存在是因为可视化设计器必须解析所有合并的XAML资源。此问题的解决方案是将所有XAML资源移动到另一个解决方案，编译该解决方案，然后从大型解决方案引用新的XAML资源DLL。由于XAML资源位于二进制引用的程序集中，因此可视化设计器不会解析XAML资源，从而提高了设计时性能。将XAML资源移动到外部程序集时，您可能需要考虑为您的资源公开**ComponentResourceKeys**。有关更多信息，请参阅MSDN上的[ComponentResourceKey标记扩展](http://msdn.microsoft.com/en-us/library/ms753186.aspx)。

#### XAMLAssets

XAML是一种功能强大且富有表现力的语言，用于创建图像，图表，绘图和三维场景等资源。一些开发人员和设计人员更喜欢创建XAML资产，而不是使用.ico，.jpg或.png图像文件。他们更喜欢XAML方法的一个原因是利用XAML渲染的分辨率独立性。另一个是他们可以使用一个工具集Blend for Visual Studio 2013来创建所有必需的资产并设计他们的应用程序。

如果解决方案具有许多这些资产，则可能会影响设计时可视化设计器的加载。将资产移动到单独的DLL可以解决性能问题。移动资产还可以跨多个解决方案重用。

#### Visual Designers and Referenced Assemblies

将XAML资源和资产移动到二进制引用程序集的一个令人遗憾的副作用是，Blend for 2013和Visual Studio 2013属性编辑器不会列出位于二进制引用程序集中的资源。这意味着您将无法从工具提供的其中一个资源选择器中选择命名资源。相反，您需要输入资源的名称。

### 创建设计友好视图的指南

以下是设计人员友好（也称为*可混合*或*可**工具*）应用程序的一些特征：

- 它通过使用Visual Studio和Blend设计器提供了高效的编辑体验。
- 它是启用工具的。例如，它允许您使用绑定构建器。
- 它在需要时提供设计时样本数据。
- 它允许在设计时执行代码，而不会导致未处理的异常。

在编辑会话期间多次执行以下操作。非设计友好的用户代码将导致这些操作中的一个或多个失败，从而降低开发人员或设计人员的工作效率和创造力。

- 设计表面动作：
  - 构造对象
  - 加载对象
  - 设置属性值
  - 执行设计表面手势
  - 使用控件作为根元素
  - 在另一个控件内部托管控件
  - 重复打开，关闭和重新打开XAML文件
  - 重建项目
  - 重塑设计师
- 绑定构建器操作：
  - 发现**DataContext**
  - 列出可用的数据源
  - 列出数据源类型属性
- 设计时样本数据操作：
  - 使用设计图面上的控件正确显示样本数据

#### 编码设计时间

为了给您丰富的设计时体验，Visual Studio和Blend设计人员在设计时实例化对象并运行代码。但是，在实例化之前尝试访问引用类型的代码导致的空引用异常会导致高百分比的加载失败和不必要的设计时异常。

下表列出了设计时体验不佳的主要原因。通过避免以下问题并使用这些技术来缓解这些问题，您的设计时体验和生产力将大大提高，开发人员到设计人员的工作流程将更加顺畅。

| **用户代码中避免使用此功能**                                 | **Visual Studio 2013**                                       | **混合Visual Studio 2013**                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 在设计时旋转多个线程。例如，在构造函数中实例化和启动**Timer**或在设计时启动**Loaded**事件。 | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 使用在设计时导致堆栈溢出的控件。 使用尝试递归加载自身的控件。 | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 在转换器或数据模板选择器中抛出空引用异常。                   | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 在构造函数中抛出null引用或其他异常。这些是由：使用调用业务或数据层的代码在设计时从数据库或网络返回数据。在引导或容器初始化代码运行之前，尝试使用MEF，控制反转（IoC）或服务定位器来解决依赖关系。 | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 在控件或用户控件的**Loaded**事件中抛出空引用或其他异常。当您对运行时可能为真的控件状态进行假设但在设计时不正确时会发生这种情况。 | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 尝试在设计时访问**Application**或**Application.Current**对象。 | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |
| 创建非常大的项目。                                           | ![是](http://prismlibrary.github.io/docs/wpf/legacy/images/Yes.png) | ![没有](http://prismlibrary.github.io/docs/wpf/legacy/images/No.png) |

#### 减少设计时用户代码中的问题

一些防御性编码实践将消除上表中描述的大多数问题。但是，在您可以缓解设计时用户代码中的问题之前，您必须了解您的应用程序控件和代码是由设计人员在未初始化的应用程序域中单独执行的。在这种情况下，*未初始化*意味着通常的启动，引导或初始化代码尚未运行。

当您的应用程序在运行时执行时，将运行App.xaml.cs或App.xaml.vb中的启动代码。如果您的应用程序的其余部分依赖于此代码，则此代码将不会在设计时执行。如果您在代码中没有预料到这一点，则会发生不必要的异常。（这就是为什么在设计时尝试在用户代码中访问**Application**或**Application.Current**对象会导致异常。）为了缓解这些问题：

- 永远不要假设引用的对象将在设计时代码中实例化。在可以在设计时执行的代码中，始终在访问任何引用对象之前执行空检查。

- 如果您的代码访问**Application**或**Application.Current**对象，请在访问对象之前执行空引用检查。

- 如果构造函数或**Loaded**事件处理程序需要运行访问数据库或调用网络的复杂代码或代码，请考虑以下解决方案之一：

  - 将代码包装在一个检查中，该检查通过调用**System.ComponentModel DesignerProperties**方法**DesignerProperties.GetIsInDesignMode**来确定代码是否在设计时运行。
  - 而不是直接在构造函数或**Loaded**事件处理程序中运行代码，抽象调用接口后面的类，然后使用许多技术之一在设计时，运行时和测试时以不同方式解析该依赖项。

  例如，不是直接调用数据服务来检索数据，而是将数据服务调用包装在通过接口公开方法的类中。然后，在设计时，使用模拟或设计时对象解析接口。

#### 了解用户控制代码何时在设计时执行

Blend和Visual Studio都使用设计器窗格中显示的根对象的模型。这对于提供所需的设计体验是必要的。因为根对象是模拟的，所以它的构造函数和**Loaded**事件代码不会在设计时执行。但是，场景中的其余控件正常构造，并且它们的**Loaded**事件就像在运行时一样被引发。

在下图中，将不执行根**Windows**构造函数和**已加载**事件代码。子用户控件构造函数和**Loaded**事件代码将被执行。

![用户控制代码在Design-Time执行](/prism/Ch7UIFig35.png)

这些概念很重要，尤其是在构建在运行时动态构建的复合应用程序或应用程序时。

大多数应用程序视图都是独立编码和设计 因为它们是独立设计的，所以它们通常是设计器中的根对象。因此，它们的构造函数和**Loaded**事件代码永远不会执行。

但是，如果您使用相同的用户控件并将其作为另一个控件的子项放置在设计图面上，则曾经隔离的用户控件代码现在正在设计时执行。如果您没有遵循上述减轻设计时代码问题的做法，那么现在托管的用户控件可能会变得不友好并导致设计器加载问题。

#### 设计时属性

内置的“d：”设计时属性为成功的设计时工具体验提供了平稳的道路。

我们需要解决的问题是如何在设计时为Binding Builder工具提供形状。在这种情况下，形状是Binding Builder可以反映的实例化**类型**，然后在构建绑定时列出这些属性以供选择。

形状也由设计时样本数据提供。样本数据包含在[“设计时样本数据指南”](http://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html#guidelines-for-design-time-sample-data)一节中。

以下部分描述了如何使用**d：DataContext**属性和**d：DesignInstance**标记扩展。

属性和标记扩展中的“d：”是设计属性所属的设计命名空间的别名。有关更多信息，请参阅WPDN主题，[WPF设计器中的设计时属性](http://msdn.microsoft.com/en-us/library/ee839627.aspx)。

无法在用户代码中创建或扩展“d：”属性和标记扩展; 它们只能在XAML中使用。“d：”属性和标记扩展名未编译到您的应用程序中; 它们仅由Visual Studio和Blend工具使用。

##### d：DataContext属性

**d：DataContext**，为控件及其子控件指定设计时数据上下文。指定**d：DataContext时**，应始终为设计时**DataContext**提供与运行时**DataContext**相同的形状。

如果为控件指定了**DataContext**和**d：DataContext**，则工具将使用**d：DataContext**。

##### d：DesignInstance标记扩展

如果标记扩展对您来说是新手，请在MSDN上阅读[标记扩展和WPF XAML](http://msdn.microsoft.com/en-us/library/ms747254.aspx)。

**d：DesignInstance**返回一个实例化的Type（“shape”），您希望将其指定为绑定到设计器中控件的数据源。该类型不需要是可创建的以用于建立形状。下表说明了**d：DesignInstance**标记扩展属性。

| **标记扩展属性**          | **定义**                                                     |
| :------------------------ | :----------------------------------------------------------- |
| **类型**                  | 要创建的类型的名称。Type是构造函数中的默认参数。             |
| **IsDesignTimeCreatable** | Can the specified Type be created? If false, a faux Type will be created rather than the real Type. The default is false. |
| **CreateList**            | If true, returns a generic list of the specified Type. The default is false. |

##### Typical d:DataContext Scenario

The following three code examples demonstrate a repeatable pattern for wiring up views and view models and enabling the designer&#39;s tooling.

The **PersonViewModel** is a dependency that the **PersonView** has at run time. While the view model in the example is incredibly simple, real-world view models typically have one or more external dependencies that must be resolved, and those dependencies are typically injected into their constructor.

当**PersonView**被构造，其依赖**PersonViewModel**将建及其依赖由MEF或依赖注入容器解决。

注意：如果视图模型没有需要解析的外部依赖项，则可以在视图的XAML中实例化视图模型，并且不需要其**DataContext**和**d：DataContext**。

```c#
// PersonViewModel.cs
[Export]
public class PersonViewModel 
{
    public String FirstName { get; set; }
    public String LasName { get; set; }
}
// PersonView.xaml.cs
[Export]
public partial class PersonView : UserControl
{
    public PersonView()
    {
        InitializeComponent();
    }

    [Import]
    public PersonViewModel ViewModel
    {
        get { return this.DataContext as PersonViewModel; }
        set { this.DataContext = value; }
    }
}
```

这是一个很好的模式，用于连接视图和视图模型; 但是，它在设计时让视图不知道它的**DataContext**的形状（视图模型）。

在下面的XAML示例中，您可以看到**网格**上使用的**d：DesignInstance**标记扩展，用于返回**PersonViewModel**的虚假实例，然后由**d：DataContext**公开。因此，**Grid的**所有子控件都将继承**d：DataContext**，使设计器工具能够发现并使用其类型和属性，从而为开发人员和设计人员提供更高效的设计体验。

```xaml
&lt;!--PersonView.xaml --&gt;
&lt;UserControl
    xmlns:local=&#34;clr-namespace:WpfApplication1&#34;
    x:Class=&#34;WpfApplication1.PersonView&#34;
    xmlns=&#34;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#34;
    xmlns:x=&#34;http://schemas.microsoft.com/winfx/2006/xaml&#34;
    xmlns:mc=&#34;http://schemas.openxmlformats.org/markup-compatibility/2006&#34;
    xmlns:d=&#34;http://schemas.microsoft.com/expression/blend/2008&#34;
    mc:Ignorable=&#34;d&#34;
    d:DesignHeight=&#34;300&#34; d:DesignWidth=&#34;300&#34;&gt;

    &lt;Border BorderBrush=&#34;LightGray&#34; BorderThickness=&#34;1&#34; CornerRadius=&#34;10&#34; Padding=&#34;10&#34;&gt;
        &lt;Grid d:DataContext=&#34;{d:DesignInstance local:PersonViewModel}&#34;&gt;
            &lt;Grid.RowDefinitions&gt;
                &lt;RowDefinition Height=&#34;Auto&#34; /&gt;
                &lt;RowDefinition Height=&#34;Auto&#34; /&gt;
            &lt;/Grid.RowDefinitions&gt;

            &lt;Grid.ColumnDefinitions&gt;
                &lt;ColumnDefinition Width=&#34;100&#34; /&gt;
                &lt;ColumnDefinition Width=&#34;Auto&#34; /&gt;
            &lt;/Grid.ColumnDefinitions&gt;

            &lt;Label Grid.Column=&#34;0&#34; Grid.Row=&#34;0&#34; Content=&#34;First Name&#34; /&gt;
            &lt;Label Grid.Column=&#34;0&#34; Grid.Row=&#34;1&#34; Content=&#34;Las Name&#34; /&gt;

            &lt;TextBox
                Grid.Column=&#34;1&#34; Grid.Row=&#34;0&#34; Width=&#34;150&#34; MaxLength=&#34;50&#34;
                HorizontalAlignment=&#34;Left&#34; VerticalAlignment=&#34;Top&#34;
                Text=&#34;{Binding Path=FirstName, Mode=TwoWay}&#34; /&gt;
            &lt;TextBox 
                Grid.Column=&#34;1&#34; Grid.Row=&#34;1&#34; Width=&#34;150&#34; MaxLength=&#34;50&#34;
                HorizontalAlignment=&#34;Left&#34; VerticalAlignment=&#34;Top&#34;
                Text=&#34;{Binding Path=LasName, Mode=TwoWay}&#34; /&gt;
        &lt;/Grid&gt;
    &lt;/Border&gt;
&lt;/UserControl&gt;
```

注意：**附加属性和ViewModel定位器解决方案**

*有几种替代技术可用于关联开发人员社区提供的视图和视图模型。其中一个挑战是在运行时运行良好的解决方案并不总是在设计时工作。一种这样的解决方案是使用附加属性和视图模型定位器来分配视图的**DataContext**。视图模型定位器是必需的，以便可以构建视图模型并解析其依赖关系。*

*此解决方案的问题在于您还必须包含**d：DataContext** - **d：DesignInstance**组合，因为可视化设计器工具无法以**d：DesignInstance**的方式反映在附加属性的结果中。*

*无论您在应用程序中使用哪种技术在设计时解决形状，最重要的目标是在整个应用程序中保持一致。一致性将使应用程序维护变得更加容易，并将导致成功的设计人员 - 开发人员工作流程。*

#### 使用设计时样本数据

如果使用可视化设计工具（如Blend或Visual Studio 2013），则设计时样本数据变得非常重要。视图可以填充数据和图像，使设计任务更容易，更快速地完成。这样可以提高生产力和创造力。

包含数据模板的空列表控件将不可见，除非它们填充了数据，使编辑空控件的任务更耗时，因为您需要运行应用程序以查看上次编辑在运行时的外观。

### Using Design-Time Sample Data

如果使用可视化设计工具（如Blend或Visual Studio 2013），则设计时样本数据变得非常重要。视图可以填充数据和图像，使设计任务更容易，更快速地完成。这样可以提高生产力和创造力。

包含数据模板的空列表控件将不可见，除非它们填充了数据，使编辑空控件的任务更耗时，因为您需要运行应用程序以查看上次编辑在运行时的外观。

#### 示例数据源

您可以使用以下任何来源的示例数据：

- 混合Visual Studio 2013 XML示例数据
- 混合Visual Studio 2013和Visual Studio 2013 XAML示例数据
- XAML资源
- 码

以下各小节介绍了每个来源的数据。

#### 混合XML示例数据

Blend使您能够快速创建XML模式并填充相应的XML文件。这是在不依赖任何项目类的情况下完成的。

此类示例数据的目的是让设计人员快速启动他们的项目，而无需等待开发人员或应用程序类可供使用之前。

虽然Blend和Visual Studio设计器都支持大多数示例数据，但XML示例数据是Blend功能，并且不在Visual Studio设计器中呈现。

***注意：** XML样本数据文件在构建时不会编译或添加到程序集中; 但是，XML模式将编译到构建的程序集中。*

#### Visual Studio 2013和Visual Studio 2013 XAML示例数据的混合

从Expression Blend 4和Visual Studio 2010开始，添加了**d：DesignData**标记扩展，以启用XAML样本数据的设计时加载。

示例数据XAML文件包含实例化一个或多个类型的XAML，并为属性分配值。

**d：DesignData**具有**Source**属性，该属性将统一资源标识符（URI）提取到位于项目中的示例数据XAML文件。的**d：DesignData**标记扩展加载XAML文件，解析它，然后返回一个对象图。对象图可以由**d：DataContext**属性，**CollectionViewSource d：DesignSource**属性或**DomainDataSource d：DesignData**属性使用。

**d：DesignData**标记扩展克服的挑战之一是它可以为不可创建的用户类型创建样本数据。例如，无法在代码中创建WCF富Internet应用程序（RIA）服务实体派生对象。此外，开发人员可能拥有自己的类型，这些类型不可创建，但仍希望拥有这些类型的示例数据。

您可以通过在**解决方案资源管理器中**设置示例数据文件上的**Build Action**属性来更改**d：DesignData**处理示例数据文件的方式，如下所示：

- **Build Action = DesignData** - 将创建虚拟类型
- **Build Action = DesignDataWithDesignTimeCreatableTypes** - 将创建真实类型

当Blend用于为类创建示例数据时，它会创建一个XAML示例数据文件，并将**Build Action**设置为**DesignData**。如果需要实际类型，请在Visual Studio中打开解决方案，并将示例数据文件的**Build Action**更改为**DesignDataWithDesignTimeCreatableTypes**。

***注意：**在下图中，“ **自定义工具”**属性为空。这是样本数据正常工作所必需的。默认情况下，Blend正确地将此属性设置为空。*

*使用Visual Studio 2013添加示例数据文件时，通常会添加新的资源字典项并从那里进行编辑。在这种情况下，您必须设置“ **构建操作”**并清除“ **自定义工具”**属性。*

![示例数据文件属性](/prism/Ch7UIFig37.png)

Expression Blend提供了用于快速创建和绑定XAML样本数据的工具。可以在Visual Studio 2013设计器中使用和查看XAML示例数据，如下图所示。

![Blend for Visual Studio 2013中的示例数据](/prism/BlendCreateSampleData.png)

生成样本数据后，数据将显示在“数据”窗格中，如下图所示。

![数据窗格](/prism/Ch7UIFig39.png)

然后，您可以将其拖到视图的根元素（例如**UserControl**）上，并将其设置为**d：DataContext**属性。您还可以将样本数据集合拖放到项目控件上，Blend会将示例数据连接到控件。

***注意**：XAML示例数据文件不会编译到构建的程序集中或包含在构建的程序集中。*

#### XAML资源

您可以在XAML中创建实例化所需类型的资源，然后将该资源绑定到**DataContext**或列表控件。

此技术可用于快速创建用于编辑数据模板的丢弃样本数据，该数据模板在没有样本数据的情况下编辑需要更长时间。

##### Code

如果您更喜欢在代码中创建示例数据，则可以编写一个类，该类公开将样本数据返回给其使用者的属性或方法。例如，您可以编写一个**Customers**类，该类在其默认的空构造函数中填充了**Customer**类的多个实例。每个**Customer**实例也将设置适当的属性值。

可用于使用前面描述的示例数据类的一种技术是使用**d：DataContext**，**d：DesignInstance**组合，确保将**d：** **DesignInstance IsDesignTimeCreatable**属性设置为**True**。**IsDesignTimeCreatable**必须为**True**的原因是您希望执行customer构造函数，以便运行填充该类的代码。如果将客户视为虚假类型，则客户代码将永远不会运行，并且工具只能发现“形状”。

以下XAML示例实例化**Customers**类，然后将其设置为**d：DataContext**。此**Grid的**子控件可以使用**Customers**类公开的数据。

```xaml
&lt;Grid d:DataContext=&#34;{d:DesignInstance local:Customers, IsDesignTimeCreatable=True}&#34;&gt;
```

### UI布局关键决策

当您开始复合应用程序项目时，您需要做出一些UI设计决策，这些决策以后很难更改。通常，这些决策是应用程序范围的，它们的一致性有助于开发人员和设

以下是重要的UI布局决策：

- 确定应用程序流程并相应地定义区域。
- 确定加载每个区域将使用的视图类型。
- 确定是否要使用区域导航API。
- 确定要使用的UI设计模式（MVVM，演示模型等）。
- 确定样本数据策略。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/07/prism11-region/  

