# Prism 导航


注意：该版本为Prism6,最新版已有较大改动。

当用户与富客户端应用程序交互时，其用户界面（UI）将不断更新以反映用户正在处理的当前任务和数据。随着用户与应用程序内的各种任务交互并完成各种任务，UI可能会随着时间发生相当大的变化。应用程序协调这些UI更改的过程通常称为*导航*。本主题描述如何使用Prism库实现复合Model-View-ViewModel（MVVM）应用程序的导航。

通常，导航意味着删除UI中的某些控件，同时添加其他控件。在其他情况下，导航可以意味着更新一个或多个现有控件的视觉状态 - 例如，一些控件可以被简单地隐藏或折叠，而其他控件被显示或扩展。类似地，导航可能意味着控件显示的数据被更新以反映应用程序的当前状态 - 例如，在主 - 细节场景中，详细视图中显示的数据将基于当前选择的项目进行更新在主视图中。所有这些场景都可以被视为导航，因为更新了用户界面以反映用户的当前任务和应用程序的当前状态。

应用程序内的导航可以由用户与UI的交互（通过鼠标事件或其他UI手势）或由于内部逻辑驱动的状态改变而从应用程序本身引起。在某些情况下，导航可能涉及非常简单的UI更新，不需要自定义应用程序逻辑。在其他情况下，应用程序可以实现复杂的逻辑以编程方式控制导航以确保强制执行某些业务规则 - 例如，应用程序可能不允许用户离开某个表单而不首先确保输入的数据是正确的。

在Windows Presentation Foundation（WPF）应用程序中实现所需的导航行为通常可以相对简单，因为它提供了对导航的直接支持。但是，在使用Model-View-ViewModel（MVVM）模式的应用程序中或在使用多个松散耦合模块的复合应用程序中实现导航可能更复杂。棱镜提供了在这些情况下实施导航的指导。

### prism  Navigation

导航定义为应用程序通过其与应用程序或内部应用程序状态更改进行交互而更改其UI的过程。

UI更新可以通过在应用程序的可视树中添加或删除元素，或者通过对可视树中的现有元素应用状态更改来完成。WPF是一个非常灵活的平台，通常可以使用这种方法实现特定的导航场景。但是，最适合您应用的方法取决于多种因素。

Prism区分了前面描述的两种导航方式。通过对可视树中的现有控件的状态更改完成的导航被称为*基于状态的导航*。通过从可视树中添加或移除元素来完成的导航被称为*基于视图的导航*。Prism提供了实现两种导航样式的指导，重点关注应用程序使用Model-View-ViewModel（MVVM）模式将UI（封装在视图中）与表示逻辑和数据（封装在视图中）分开的情况模型）。

### 基于状态的导航

State-Based Navigation

在基于状态的导航中，表示UI的视图可以通过视图模型中的状态更改或通过视图本身内的用户交互来更新。在这种导航方式中，不是用另一个视图替换视图，而是更改视图的状态。根据视图状态的更改方式，更新的UI可能会让用户感觉像导航一样。

这种导航方式适用于以下情况：

- 视图需要以不同的样式或格式显示相同的数据或功能。
- 视图需要根据视图模型的基础状态更改其布局或样式。
- 视图需要在视图的上下文中启动与用户的有限模态或非模态交互。

这种导航方式不适用于UI必须向用户呈现不同数据或用户必须执行不同任务的情况。在这些情况下，最好实现单独的视图（和视图模型）来表示数据或任务，然后使用基于视图的导航在它们之间导航，如本主题后面所述。类似地，如果实现导航所需的UI状态更改的数量过于复杂，则这种导航方式不适合，因为视图的定义可能变得很大并且难以维护。在这种情况下，最好通过使用基于视图的导航在不同的视图中实现导航。

以下部分描述了可以使用基于状态的导航的典型情况。这些部分中的每一部分都涉及基于状态的导航快速入门，它实现了即时消息传递式应用程序，允许用户管理和与其联系人聊天。

### 以不同的格式或样式显示数据

Displaying Data in Different Formats or Styles

您的应用程序可能经常需要向用户显示相同的数据，但格式或样式不同。在这种情况下，您可以在视图中使用基于状态的导航在不同样式之间切换，可能使用它们之间的动画过渡。例如，基于状态的导航快速入门允许用户选择联系人的显示方式 - 简单文本列表或头像（图标）。用户可以通过单击“列表”按钮或“头像”按钮在这些可视表示之间切换。该视图提供了两个表示之间的动画过渡，如下图所示。

![视图之间的动画过渡](/prism/Ch8NavigationFig1.png)

#### 联系基于状态的导航快速入门中的视图导航

Contact view navigation in the State-Based Navigation QuickStart

由于视图呈现的是相同的数据，但是在不同的可视化表示中，视图模型不需要参与表示之间的导航。在这种情况下，导航完全在视图本身内处理。这种方法为UI设计人员提供了很大的灵活性，可以设计出引人注目的用户体验，而无需更改应用程序的代码。

混合行为提供了在视图中实现此导航样式的好方法。基于状态的导航QuickStart应用程序使用Blend的**DataStateBehavior**数据绑定到单选按钮，在使用可视状态管理器定义的两个可视状态之间切换，一个按钮将联系人显示为列表，一个按钮将联系人显示为图标。

```xaml
&lt;ei:DataStateBehavior Binding=&#34;{Binding IsChecked, ElementName=ShowAsListButton}&#34; 
            Value=&#34;True&#34;
            TrueState=&#34;ShowAsList&#34; 
            FalseState=&#34;ShowAsIcons&#34;/&gt;
```

![基于状态的导航示例截图](/prism/Ch8NavigationFig2.png)

当用户单击“ **联系人”**或“ **头像”**单选按钮时，可视状态在**ShowAsList**可视状态和**ShowAsIcons**可视状态之间切换。还使用可视状态管理器定义这些状态之间的翻转过渡动画。

当用户切换到当前所选联系人的详细信息视图时，基于状态的导航快速入门应用程序会显示此类导航的另一个示例。下图显示了此示例。

##### 基于状态的导航快速入门中的“联系人详细信息”视图

同样，这可以使用Blend **DataStateBehavior**轻松实现; 但是，这次它被绑定到视图模型上的**ShowDetails**属性，该属性使用翻转过渡动画在**ShowDetails**和**ShowContacts**视觉状态之间切换。

### Reflecting Application State

类似地，应用程序中的视图有时可能需要根据对内部应用程序状态的更改来更改其布局或样式，而内部应用程序状态又由视图模型上的属性表示。基于状态的导航快速入门中显示了此方案的示例，其中使用**ConnectionStatus**属性在Chat视图模型类上表示用户的连接状态。当用户的连接状态发生变化时，将通知视图（通过属性更改通知事件），允许视图在适当的位置直观地表示当前连接状态，如下图所示。

![连接状态表示](/prism/Ch8NavigationFig3.png)

#### 基于状态的导航快速入门中的连接状态表示

Connection state representation in the State-Based Navigation QuickStart

为实现此目的，视图定义绑定到视图模型的**ConnectionStatus**属性的**DataStateBehavior**数据，以在适当的可视状态之间切换。

```xaml
&lt;ei:DataStateBehavior Binding=&#34;{Binding ConnectionStatus}&#34; 
                        Value=&#34;Available&#34; 
                        TrueState=&#34;Available&#34; FalseState=&#34;Unavailable&#34;/&gt;
```

请注意，用户可以通过UI或应用程序根据某些内部逻辑或事件更改连接状态。例如，如果用户在特定时间段内没有与视图交互或者当用户的日历指示他或她在会议中时，应用程序可以移动到“不可用”状态。基于状态的导航快速入门通过使用计时器随机切换连接状态来模拟此场景。更改连接状态后，将更新视图模型上的属性，并通过属性更改事件通知视图。然后更新UI以反映当前连接状态。

前面的所有示例都涉及在视图中定义视觉状态，以及由于用户与视图的交互或视图模型定义的属性更改而在视图之间切换。此方法允许UI设计器在视图中实现类似导航的可视行为，而无需替换视图或要求对应用程序代码进行任何代码更改。当需要视图以不同的样式或布局呈现相同的数据时，此方法是合适的。它不适用于向用户呈现不同数据或应用程序功能或导航到应用程序的不同部分的情况。

### 与用户交互

通常，应用程序需要以有限的方式与用户交互。在这些情况下，通常更适合在当前视图的上下文中与用户交互，而不是导航到新视图。例如，在基于状态的导航快速入门中，用户可以通过单击“ **发送消息”**按钮向联系人**发送消息**。然后，视图将显示一个弹出窗口，允许用户键入消息，如下图所示。因为与用户的这种交互是有限的并且逻辑上发生在父视图的上下文中，所以它可以容易地实现为基于状态的导航。

![使用弹出窗口与用户交互](/prism/Ch8NavigationFig4.png)

#### 使用弹出窗口与用户交互

使用基于状态的导航快速入门中的弹出窗口与用户交互要实现此行为，基于状态的导航快速入门实现**SendMessage**命令，该命令绑定到“ **发送消息”**按钮。调用此命令时，视图模型将与视图交互以显示弹出窗口。这是使用[实现MVVM模式中](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)描述的交互请求模式[实现的](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)。

以下代码示例显示了基于状态的导航QuickStart应用程序中的视图如何响应视图模型提供的**SendMessageRequest**交互请求对象。收到请求事件后，**SendMessageChildWindow**将显示为弹出窗口。

```xaml
&lt;prism:InteractionRequestTrigger SourceObject=&#34;{Binding SendMessageRequest}&#34;&gt;
    &lt;prism:PopupWindowAction IsModal=&#34;True&#34;&gt;
        &lt;prism:PopupWindowAction.WindowContent&gt;
            &lt;vs:SendMessagePopupView /&gt;
        &lt;/prism:PopupWindowAction.WindowContent&gt;
    &lt;/prism:PopupWindowAction&gt;
&lt;/prism:InteractionRequestTrigger&gt;
```

### 基于视图的导航

虽然基于状态的导航对于前面概述的场景可能很有用，但是应用程序中的导航通常通过将应用程序UI中的一个视图替换为另一个视图来实现。在Prism中，这种导航方式称为基于视图的导航。

根据应用程序的要求，此过程可能相当复杂，需要仔细协调。以下是在实现基于视图的导航时经常需要解决的常见挑战：

- 导航的目标 - 要添加或移除的视图的容器或主机控件 - 可以在添加或移除视图时以不同方式处理导航，或者它们可以以不同方式可视地表示导航。在许多情况下，导航目标将是简单的**Frame**或**ContentControl**，导航视图将简单地显示在这些控件中。但是，在许多情况下，导航操作的目标是不同类型的容器控件，例如**TabControl**或**ListBox**控件。在这些情况下，导航可能需要激活或选择现有视图，或者添加新视图是一种特定方式。
- 应用程序还经常必须定义如何识别要导航到的视图。例如，在Web应用程序中，要导航到的页面通常由统一资源标识符（URI）直接标识。在客户端应用程序中，可以通过类型名称，资源位置或以各种不同方式来标识视图。此外，在由松散耦合的模块组成的复合应用程序中，视图通常将在单独的模块中定义。需要以不会在模块之间引入紧密耦合和依赖关系的方式识别单个视图。
- 在识别视图之后，必须仔细协调实例化和初始化新视图的过程。在使用MVVM模式时，这尤其重要。在这种情况下，视图和视图模型可能需要在导航操作期间通过视图的数据上下文进行实例化并相互关联。在应用程序利用依赖注入容器（例如Unity应用程序块（Unity）或托管可扩展性框架（MEF））的情况下，视图和/或视图模型（以及其他依赖类）的实例化可能必须使用特定的构造机制来实现。
- MVVM模式提供了应用程序的UI与其表示和业务逻辑之间的分离。但是，应用程序的导航行为通常会跨越应用程序的UI和表示逻辑部分。用户通常会在视图中启动导航，并且视图将作为导航的结果进行更新，但通常还需要从视图模型中启动或协调导航。在整个视图和视图模型中清晰地分离应用程序的导航行为的能力是一个需要考虑的重要方面。
- 应用程序通常还需要将参数或上下文传递给视图，以便可以正确初始化它。例如，如果用户导航到视图以更新特定客户的详细信息，则必须将客户的ID或数据传递给视图，以便它可以显示正确的信息。
- 许多应用程序还必须仔细协调导航，以确保遵守某些业务规则。例如，在远离视图导航之前可能会提示用户，以便他们可以纠正任何无效数据，或者提示他们提交或放弃他们在该视图中所做的任何数据更改。此过程需要在先前视图和新视图之间进行仔细协调。
- 最后，大多数现代应用程序允许用户轻松地向后（或向前）导航到先前显示的视图。类似地，一些应用程序使用一系列视图或表单实现其工作流，并允许用户向前或向后导航，在完成任务并一次提交所有更改之前添加或更新数据。这些场景需要某种日记（或历史）机制，以便可以存储，重放或预定义导航序列。

Prism通过扩展Prism的区域机制来支持导航，为这些挑战提供支持和指导。以下部分提供了Prism区域的简要概述，并描述了它们如何扩展以支持基于视图的导航。

### Prism Region概述

Prism Region旨在通过允许应用程序的整体UI以松散耦合的方式构建来支持复合应用程序（即，由多个模块组成的应用程序）的开发。区域允许在模块中定义的视图显示在应用程序的UI中，而无需模块明确了解应用程序的整体UI结构。它们允许轻松更改应用程序UI的布局，从而允许UI设计人员为应用程序选择最合适的UI设计和布局，而无需更改模块本身。

Prism Region基本上named placeholders，在其中可以显示视图。通过简单地**向其**添加**RegionName**附加属性，应用程序UI中的任何控件都可以声明为区域，如此处所示。

```xaml
&lt;ContentControl prism:RegionManager.RegionName=&#34;MainRegion&#34; ... /&gt;
```

对于指定为区域的每个控件，Prism创建一个**Region**对象来表示区域，并创建一个**RegionAdapter**对象，该对象管理视图在指定控件中的放置和激活。Prism Library 为大多数常见的WPF控件提供**RegionAdapter**实现。您可以创建自定义**RegionAdapter**以支持其他控件，或者在需要定义自定义行为时。该**RegionManager**类提供给接入**地区**的应用程序中的对象。

在许多情况下，区域控件将是一个简单的控件，例如**ContentControl**，它可以一次显示一个视图。在其他情况下，**Region**控件将是一个能够同时显示多个视图的**控件**，例如**TabControl**或**ListBox**控件。

区域适配器管理关联区域内的视图列表。这些视图中的一个或多个将根据其定义的布局策略显示在区域控件中。可以为视图分配一个名称，该名称可用于稍后检索该视图。区域适配器管理区域内视图的活动状态。活动视图是选定视图或最顶视图 - 例如，在**TabControl中**，活动视图是显示在所选选项卡中的视图; 在**ContentControl中**，活动视图是当前显示为控件内容的视图。

**注意：**在导航期间，视图的活动状态很重要。通常，您希望活动视图参与导航，以便它可以在用户导航之前保存数据，或者可以确认或取消导航操作。

先前版本的Prism允许以两种方式在区域中显示视图。第一种称为*视图注入*，允许以编程方式在区域中显示视图。此方法对于动态内容很有用，根据应用程序的表示逻辑，要在区域中显示的视图会频繁更改。

通过**Region**类的**Add**方法支持视图注入。下面的代码示例演示如何通过**RegionManager**类获取对**Region**对象的引用，并以编程方式向其添加视图。在此示例中，使用依赖项注入容器创建视图。

```cs
    IRegionManager regionManager = ...;
    IRegion mainRegion = regionManager.Regions[&#34;MainRegion&#34;];
    InboxView view = this.container.Resolve&lt;InboxView&gt;();
    mainRegion.Add(view);
```

第二种方法称为*视图发现*，它允许模块针对区域名称注册视图类型。每当显示具有指定名称的区域时，将自动创建指定视图的实例并在该区域中显示。此方法对于相对静态的内容很有用，其中要在区域中显示的视图不会更改。

通过**RegionManager**类上的**RegisterViewWithRegion**方法支持视图发现。此方法允许您指定在显示命名区域时将调用的回调方法。以下代码示例显示了在首次显示主区域时如何创建视图（通过依赖项注入容器）。

```cs
    IRegionManager regionManager = ...;
    regionManager.RegisterViewWithRegion(&#34;MainRegion&#34;, () =&gt;
                       container.Resolve&lt;InboxView&gt;());
```

有关Prism区域支持的详细概述以及有关如何利用视图注入和发现来利用区域组成应用程序UI的信息，请参阅[编写用户界面](http://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html)。本主题的其余部分描述了如何扩展区域以支持基于视图的导航，以及如何解决前面描述的各种挑战。

### 基本区域导航

视图注入和视图发现都可以被认为是有限形式的导航 - 视图注入是一种显式的，程序化的导航形式，而视图发现是一种隐式或延迟导航的形式。但是，在Prism 4.0中，基于URI和可扩展的导航机制，区域已经扩展到支持更一般的导航概念。

区域内的导航意味着将在该区域内显示新视图。要显示的视图通过URI标识，默认情况下，URI指的是要创建的视图的名称。您可以使用**INavigateAsync**接口定义的**RequestNavigate**方法以编程方式启动导航。

**注意：**尽管名称如此，但**INavigateAsync**接口并不代表在单独的后台线程上执行的异步导航。相反，**INavigateAsync**接口表示执行伪异步导航的能力。该**RequestNavigate**方法可以返回同步导航操作完成之后，或者它可以返回而导航操作仍悬而未决，如在用户需要确认导航的情况。通过允许您在导航期间指定回调和延续，Prism提供了一种机制来启用这些方案，而无需在后台线程上导航的复杂性。

**INavigateAsync**接口由**Region**类实现，允许您在**该区域内**启动导航。

```cs
IRegion mainRegion = ...;
mainRegion.RequestNavigate(new Uri(&#34;InboxView&#34;, UriKind.Relative));
```

您也可以拨打**RequestNavigate**的方法**RegionManager**，它允许你指定要驾驶的区域的名称。这个方便的方法获取对指定区域的引用，然后调用**RequestNavigate**方法，如上面的代码示例所示。

```cs
IRegionManager regionManager = ...;
regionManager.RequestNavigate(&#34;MainRegion&#34;,
                                new Uri(&#34;InboxView&#34;, UriKind.Relative));
```

默认情况下，导航URI指定在容器中注册的视图的名称。

使用MEF，您只需导出具有指定名称的视图类型即可。

```cs
[Export(&#34;InboxView&#34;)]
public partial class InboxView : UserControl { ... }
```

在导航期间，指定视图通过容器或MEF以及其对应的视图模型和其他相关服务和组件进行实例化。在实例化视图之后，将其添加到指定区域并激活（本主题后面将更详细地描述激活）。

**注意：**前面的描述说明了视图优先导航，其中URI指的是视图类型的名称，因为它是随容器导出或注册的。使用视图优先导航，依赖视图模型将创建为视图的依赖项。另一种方法是使用视图模型优先导航，其中导航URI指的是视图模型类型的名称，因为它是随容器导出或注册的。当视图定义为数据模板时，或者您希望独立于视图定义导航方案时，查看模型优先导航非常有用。

该**RequestNavigate**方法还允许您指定一个回调方法，或委托，当导航完成后，将被调用。

```cs
private void SelectedEmployeeChanged(object sender, EventArgs e)
{
    ...
    regionManager.RequestNavigate(RegionNames.TabRegion,
                        &#34;EmployeeDetails&#34;, NavigationCompleted);
}
private void NavigationCompleted(NavigationResult result)
{
    ...
}
```

所述**NavigationResult**类定义了提供有关导航操作信息的属性。该**结果**属性指示导航是否成功。如果导航成功，则**Result**属性为*true*。如果导航失败，通常是因为在**IConfirmNavigationResult.ConfirmNavigationRequest**方法中返回&#39;continuationCallBack（false）&#39; ，则**Result**属性将为*false*。如果由于异常导致导航失败，则**Result**属性将为*false*并且为**Error**property提供对导航期间抛出的任何异常的引用。的**语境**属性提供给导航URI，它包含任何参数，并且向协调导航操作导航服务的引用。

### 查看和查看模型参与导航

通常，应用程序中的视图和视图模型将要参与导航。该**INavigationAware**接口支持这个。您可以在视图上或（更常见地）视图模型上实现此接口。通过实现此界面，您的视图或视图模型可以选择参与导航过程。

**注意：**在下面的描述中，尽管在视图之间导航期间引用了对此接口的调用，但应注意，无论是由视图还是视图模型实现，都将在导航期间调用**INavigationAware**接口。在导航过程中，Prism会检查视图是否实现了**INavigationAware**接口; 如果是，它会在导航期间调用所需的方法。Prism还会检查设置为视图的**DataContext**的对象是否实现了此接口; 如果是，它会在导航期间调用所需的方法。

此界面允许视图或视图模型参与导航操作。所述**INavigationAware**接口定义了三个方法。

```cs
public interface INavigationAware
{
    bool IsNavigationTarget(NavigationContext navigationContext);
    void OnNavigatedTo(NavigationContext navigationContext);
    void OnNavigatedFrom(NavigationContext navigationContext);
}
```

该**IsNavigationTarget**方法允许现有的（显示的）视图或视图模型，以指示它是否能够处理所述导航请求。在您可以重复使用现有视图来处理导航操作或导航到已存在的视图时，这非常有用。例如，可以更新显示客户信息的视图以显示不同的客户信息。有关使用此方法的详细信息，请参阅本主题后面的“ [导航到现有视图](http://prismlibrary.github.io/docs/wpf/legacy/Navigation.html#navigating-to-existing-views) ”一节。

该**OnNavigatedFrom**和**的OnNavigatedTo**方法是导航操作期间调用。如果区域中当前活动的视图实现此接口（或其视图模型），则在导航发生之前调用其**OnNavigatedFrom**方法。该**OnNavigatedFrom**方法允许一个视图保存任何状态，或为它的失活或去除从UI制备，例如，以保存用户已经到web服务或数据库所作的任何更改。

如果新创建的视图实现此接口（或其视图模型），则在导航完成后调用其**OnNavigatedTo**方法。所述**的OnNavigatedTo**方法允许新显示的视图初始化自身，可能使用传递给它上的导航URI任何参数。有关详细信息，请参阅下一节“ [导航期间传递参数”](http://prismlibrary.github.io/docs/wpf/legacy/Navigation.html#passing-parameters-during-navigation)。

在实例化，初始化并添加到目标区域之后，它将成为活动视图，并且停用先前视图。有时您会希望从区域中删除已停用的视图。Prism提供**IRegionMemberLifetime**接口，允许您指定是否要从区域中删除已停用的视图或仅将其标记为已停用，从而控制区域内视图的生命周期。

```cs
public class EmployeeDetailsViewModel : IRegionMemberLifetime
{
    public bool KeepAlive
    {
        get { return true; }
    }
}
```

所述**IRegionMemberLifetime**接口定义的单个只读属性，**的KeepAlive**。如果此属性返回**false**，则在停用视图时将从该区域中删除该视图。由于该区域不再具有对视图的引用，因此它有资格进行垃圾回收（除非应用程序中的某些其他组件维护对它的引用）。您可以在视图或视图模型类上实现此接口。虽然**IRegionMemberLifetime**接口主要用于在激活和取消激活期间管理区域内视图的生命周期，但在目标区域中激活新视图后，还会在导航期间考虑**KeepAlive**属性。

**注意：**可以显示多个视图的区域（例如使用**ItemsControl**或**TabControl的区域**）将同时显示非活动视图和活动视图。从这些类型的区域中删除非活动视图将导致视图从UI中移除。

### 导航期间传递参数

要在应用程序中实现所需的导航行为，通常需要在导航请求期间指定其他数据，而不仅仅是目标视图名称。所述**NavigationContext**对象提供对导航URI，并在其内被指定的或外部的任何参数。您可以从**IsNavigationTarget**，**OnNavigatedFrom**和**OnNavigatedTo**方法中访问**NavigationContext**。

Prism提供**NavigationParameters**类以帮助指定和检索导航参数。该**NavigationParameters**类维护名称-值对，每个参数列表。您可以使用此类将参数作为导航URI的一部分传递或传递对象参数。

以下代码示例演示如何将单个字符串参数添加到**NavigationParameters**实例，以便将其附加到导航URI。

```cs
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var navigationParameters = new NavigationParameters();
    navigationParameters.Add(&#34;ID&#34;, employee.Id);
    _regionManager.RequestNavigate(RegionNames.TabRegion,
            new Uri(&#34;EmployeeDetailsView&#34; &#43; navigationParameters.ToString(), UriKind.Relative));
}
```

此外，您可以通过将对象参数添加到**NavigationParameters**实例并将其作为**RequestNavigate**方法的参数传递来传递对象参数。这在以下代码中显示。

```cs
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var parameters = new NavigationParameters();
    parameters.Add(&#34;ID&#34;, employee.Id);
    parameters.Add(&#34;myObjectParameter&#34;, new ObjectParameter());
    regionManager.RequestNavigate(RegionNames.TabRegion,
            new Uri(&#34;EmployeeDetailsView&#34;, UriKind.Relative), parameters);
}
```

您可以使用**NavigationContext**对象上的**Parameters**属性检索导航参数。此属性返回**NavigationParameters**类的实例，该类提供索引器属性以允许轻松访问各个参数，而**不管**它们是通过查询还是通过**RequestNavigate**方法传递。

```cs
public void OnNavigatedTo(NavigationContext navigationContext)
{
    string id = navigationContext.Parameters[&#34;ID&#34;];
    ObjectParameter myParameter = navigationContext.Parameters[&#34;myObjectParameter&#34;];
}
```

### 导航到现有视图

通常，在导航期间重复使用，更新或激活应用程序中的视图更合适，而不是由新视图替换。这通常是您导航到相同类型的视图但需要向用户显示不同信息或状态的情况，或者当UI中已有适当视图但需要激活（即，选择或制作）时最顶部）。

对于第一个场景的示例，假设您的应用程序允许用户使用**EditCustomer**视图编辑客户记录，并且用户当前正在使用该视图编辑客户ID 123.如果客户决定编辑客户的客户记录ID 456，用户可以直接导航到**EditCustomer**视图并输入新的客户ID。然后，**EditCustomer**视图可以检索新客户的数据并相应地更新其UI。

第二种情况的示例是应用程序允许用户一次编辑多个客户记录。在这种情况下，应用程序在选项卡控件中显示多个**EditCustomer**视图实例 - 例如，一个用于客户ID 123，另一个用于客户ID 456.当用户导航到**EditCustomer**视图并输入客户ID 456时，相应的视图将是激活（即，将选择其相应的选项卡）。如果用户导航到**EditCustomer**视图并输入客户ID 789，则将创建一个新实例并显示在选项卡控件中。

由于各种原因，导航到现有视图的能力很有用。更新现有视图通常更有效，而不是使用相同类型的新实例替换它。同样，激活现有视图而不是创建重复视图可提供更一致的用户体验。此外，无需多少自定义代码即可无缝处理这些情况，这意味着应用程序更易于开发和维护。

棱镜支持通过前面描述的两种方案**IsNavigationTarget** on方法**INavigationAware**接口。在导航期间，在与目标视图类型相同的区域中的所有视图上调用此方法。在前面的示例中，视图的目标类型是**EditCustomer**视图，因此将在当前位于该区域中的所有现有**EditCustomer**视图实例上调用**IsNavigationTarget**方法。Prism从视图URI确定目标类型，它假定它是目标类型的短类型名称。

**注意：**要使Prism确定目标视图的类型，导航URI中的视图名称应与实际目标类型的短类型名称相同。例如，如果您的视图由**MyApp.Views.EmployeeDetailsView**类实现，则导航URI中指定的视图名称应为**EmployeeDetailsView**。这是Prism提供的默认行为。您可以通过实现自定义内容加载器类来自定义此行为; 您可以通过实现**IRegionNavigationContentLoader**接口或从**RegionNavigationContentLoader**类派生来实现此**目的**。

**IsNavigationTarget**方法的实现可以使用**NavigationContext**参数来确定它是否可以处理导航请求。所述**NavigationContext**对象提供对导航URI和导航参数。在前面的示例中，**EditCustomer**视图模型中此方法的实现将当前客户ID与导航请求中指定的ID进行比较，如果匹配则返回**true**。

```cs
public bool IsNavigationTarget(NavigationContext navigationContext)
{
    string id = navigationContext.Parameters[&#34;ID&#34;];
    return _currentCustomer.Id.Equals(id);
}
```

如果**IsNavigationTarget**方法始终返回**true**，则无论导航参数如何，都将始终重用该视图实例。这允许您确保在特定区域中仅显示特定类型的一个视图。

### 确认或取消导航

您经常会发现在导航操作期间需要与用户进行交互，以便用户可以确认或取消它。例如，在许多应用中，用户可以在输入或编辑数据的过程中尝试导航。在这些情况下，您可能想要询问用户是否要在继续离开页面之前保存或丢弃已输入的数据，或者用户是否想要完全取消导航操作。Prism通过**IConfirmNavigationRequest**接口支持这些场景。

所述**IConfirmNavigationRequest**接口从所述派生**INavigationAware**接口并添加**ConfirmNavigationRequest**方法。通过在视图或视图模型类上实现此接口，您允许它们以允许用户与用户交互的方式参与导航序列，以便用户可以确认或取消导航。您将经常使用**交互请求**对象（如[高级MVVM方案中所述](http://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html)）来显示确认弹出窗口。

**注意：**该**ConfirmNavigationRequest**方法称为活动视图或视图模型，类似于**OnNavigatedFrom**前面描述的方法。

该**ConfirmNavigationRequest**方法提供了两个参数，如前文所述当前导航背景下，当你想要导航，继续，你可以调用回调方法的参考。因此，回调称为延续回调。您可以存储对continuation回调的引用，以便应用程序在完成与用户交互后调用它。如果您的应用程序通过**交互请求**对象与用户**交互**，您可以将调用链接到交互请求中的回调的连续回调。下图说明了整个过程。

![使用InteractionRequest对象确认导航](/prism/Ch8NavigationFig5.png)

以下步骤总结了使用**InteractionRequest**对象确认导航的过程：

1. 导航操作通过**RequestNavigate**调用启动。
2. 如果视图或视图模型实现**IConfirmNavigation**，则调用**ConfirmNavigationRequest**。
3. 视图模型引发交互请求事件。
4. 视图显示确认弹出窗口并等待用户的响应。
5. 当用户关闭弹出窗口时，将调用交互请求回调。
6. 调用继续回调以继续或取消挂起的导航操作。
7. 导航操作已完成或取消。

为了说明这一点，请查看View-Switching Navigation Quick Start。此应用程序使用户能够使用**ComposeEmailView**和**ComposeEmailViewModel**类撰写新电子邮件。视图模型类实现**IConfirmNavigation**接口。如果用户导航（例如通过单击“ **日历”**按钮），则在编写电子邮件时，将调用**ConfirmNavigationRequest**方法，以便视图模型可以确认与用户的导航。为了支持这一点，视图模型类定义了交互请求，如以下代码示例所示。

```cs
public class ComposeEmailViewModel : NotificationObject, IConfirmNavigationRequest
{
    . . .
    private readonly InteractionRequest&lt;Confirmation&gt; confirmExitInteractionRequest;

    public ComposeEmailViewModel(IEmailService emailService)
    {
        . . .
        this.confirmExitInteractionRequest = new InteractionRequest&lt;Confirmation&gt;();
    }

    public IInteractionRequest ConfirmExitInteractionRequest
    {
        get { return this.confirmExitInteractionRequest; }
    }
}
```

在**ComposeEmailView**类中，定义了交互请求触发器，并将数据绑定到视图模型上的**ConfirmExitInteractionRequest**属性。当进行交互请求时，将向用户显示简单的弹出窗口。

```xaml
&lt;UserControl.Resources&gt;
    &lt;DataTemplate x:Key=&#34;ConfirmExitDialogTemplate&#34;&gt;
        &lt;TextBlock HorizontalAlignment=&#34;Center&#34; VerticalAlignment=&#34;Center&#34;
                    Text=&#34;{Binding}&#34;/&gt;
    &lt;/DataTemplate&gt;
&lt;/UserControl.Resources&gt;

&lt;Grid x:Name=&#34;LayoutRoot&#34; Background=&#34;White&#34;&gt;
&lt;ei:Interaction.Triggers&gt;
        &lt;prism:InteractionRequestTrigger SourceObject=&#34;{Binding  
            ConfirmExitInteractionRequest}&#34;&gt;
        &lt;prism:PopupWindowAction IsModal=&#34;True&#34; CenterOverAssociatedObject=&#34;True&#34;/&gt;
        &lt;/prism:InteractionRequestTrigger&gt;
&lt;/ei:Interaction.Triggers&gt;
...
```

在**ConfirmNavigationRequest**对方法**ComposeEmailVewModel**类，如果用户尝试导航而电子邮件正在被由被调用。此方法的实现调用先前定义的交互请求，以便用户可以确认或取消导航操作。

```cs
void IConfirmNavigationRequest.ConfirmNavigationRequest(
            NavigationContext navigationContext, Action&lt;bool&gt; continuationCallback)
{
    . . .
    this.confirmExitInteractionRequest.Raise(
                new Confirmation {Content = &#34;...&#34;, Title = &#34;...&#34;},
                c =&gt; {continuationCallback(c.Confirmed);});
}
```

当用户单击确认弹出窗口中的按钮以确认或取消操作时，将调用交互请求的回调。此回调只调用continuation回调，传入**Confirmed**标志的值，并导致导航继续或被取消。

注意：应该注意，在引发交互请求事件之后，立即返回**ConfirmNavigationRequest**方法，以便用户可以继续与应用程序的UI交互。当用户单击弹出窗口上的“ **确定”**或“ **取消”**按钮时，将生成交互请求的回调方法，该方法又调用继续回调以完成导航操作。在UI线程上调用所有方法。使用此技术，不需要后台线程。

使用此机制，您可以控制导航请求是立即执行还是延迟执行，等待与用户的交互或某些其他异步交互（例如，作为Web服务请求的结果）。要启用导航，您只需调用continuation回调方法，传递**true**即表示它可以继续。同样，您可以传递**false**以指示应取消导航。

```cs
void IConfirmNavigationRequest.ConfirmNavigationRequest(
            NavigationContext navigationContext, Action&lt;bool&gt; continuationCallback)
{
    continuationCallback(true);
}
```

如果要延迟导航，可以存储对继续回调的引用，然后在与用户（或Web服务）的交互完成时调用。在您调用continuation回调之前，导航操作将处于暂挂状态。

如果用户同时启动另一个导航操作，则导航请求将被取消。在这种情况下，调用continuation回调没有任何效果，因为它所涉及的导航操作不再是最新的。同样，如果您决定不调用延续回调，则导航操作将处于暂挂状态，直到将其替换为新的导航操作。

### 使用导航日志

所述**NavigationContext**类提供进入该地区导航服务，它负责的区域内导航期间协调操作的序列。它提供对正在进行导航的区域以及与该区域相关联的导航日志的访问。区域导航服务实现**IRegionNavigationService**，其定义如下。

```cs
public interface IRegionNavigationService : INavigateAsync
{
    IRegion Region {get; set;}
    IRegionNavigationJournal Journal {get;}
    event EventHandler&lt;RegionNavigationEventArgs&gt; Navigating;
    event EventHandler&lt;RegionNavigationEventArgs&gt; Navigated;
    event EventHandler&lt;RegionNavigationFailedEventArgs&gt; NavigationFailed;
}
```

由于区域导航服务实现了**INavigateAsync**接口，因此您可以通过调用其**RequestNavigate**方法在父区域内启动导航。在**Navigated **开始导航操作时引发事件。在**导航中**的区域内的导航结束时引发事件。该**NavigationFailed**如果导航过程中遇到的错误引发。

该**Journal **属性提供与该区域相关的导航日记。导航日志实现**IRegionNavigationJournal**接口，其定义如下。

```cs
public interface IRegionNavigationJournal
{
    bool CanGoBack { get; }
    bool CanGoForward { get; }
    IRegionNavigationJournalEntry CurrentEntry { get; }
    INavigateAsync NavigationTarget { get; set; }
    void Clear();
    void GoBack();
    void GoForward();
    void RecordNavigation(IRegionNavigationJournalEntry entry);
}
```

您可以在导航期间通过**OnNavigatedTo**方法调用获取并存储对视图中区域导航服务的引用。默认情况下，Prism提供了一个简单的基于堆栈的日志，允许您在区域内向前或向后导航。

您可以使用导航日志允许用户从视图本身进行导航。在以下示例中，视图模型实现了**GoBack**命令，该命令使用主机区域中的导航日志。因此，视图可以显示“ **后退”**按钮，允许用户轻松导航回区域内的上一个视图。同样，您可以实现**GoForward**命令来实现向导样式工作流。

```cs
public class EmployeeDetailsViewModel : INavigationAware
{
    ...
    private IRegionNavigationService navigationService;

    public void OnNavigatedTo(NavigationContext navigationContext)
    {
        navigationService = navigationContext.NavigationService;
    }

    public DelegateCommand&lt;object&gt; GoBackCommand { get; private set; }

    private void GoBack(object commandArg)
    {
        if (navigationService.Journal.CanGoBack)
        {
            navigationService.Journal.GoBack();
        }
    }

    private bool CanGoBack(object commandArg)
    {
        return navigationService.Journal.CanGoBack;
    }
}
```

如果需要在该区域内实现特定的工作流模式，则可以为区域实现自定义日记。

**注意：**导航日志只能用于由区域导航服务协调的基于区域的导航操作。如果使用视图发现或视图注入来实现区域内的导航，则导航日志将不会在导航期间更新，也不能用于在该区域内向前或向后导航。

### 选择退出导航日志

使用“导航日志”时，显示中间页面（如启动画面，加载页面或对话框）非常有用。希望通过调用**IRegionNavigationJournal.GoForward（）**或**IRegionNavigationJournal.GoBack（）**来重新访问这些页面。通过实现**IJournalAware**接口可以实现此行为。

```
public interface IJournalAware
{
    bool PersistInHistory();
}
```

通过在**View**或**View Model**上实现**IJournalAware**并从**PersistInHistory（）**返回false，Pages可以选择不添加到日志历史记录中。

```
public class IntermediaryPage : IJournalAware
{
    public bool PersistInHistory() =&gt; false;
}
```

### 使用WPF导航框架

Prism区域导航旨在解决在使用MVVM模式和依赖注入容器（如Unity）或Managed Extensibility Framework的松散耦合的模块化应用程序中实现导航时可能遇到的各种常见场景和挑战。 （MEF）。它还旨在支持导航确认和取消，导航到现有视图，导航参数和导航日志。

通过支持Prism区域内的导航，它还支持在各种布局控件中进行导航，并支持在不影响其导航结构的情况下更改应用程序UI的布局。它还支持伪同步导航，允许在导航期间进行丰富的用户交互。

但是，棱镜区域导航并非旨在取代WPF的导航框架。相反，Prism区域导航被设计为与WPF导航框架并排使用。

WPF导航框架很难用于支持MVVM模式和依赖注入。它还基于**Frame**控件，在日记和导航UI方面提供类似的功能。您可以将WPF导航框架与Prism区域导航一起使用，但仅使用Prism区域实现导航可能更容易，也更灵活。

### 区域导航序列

The Region Navigation Sequence

下图提供了导航操作期间操作顺序的概述。它仅供参考，以便您可以在导航请求期间查看棱镜区域导航的各种元素如何协同工作。

![Prism导航序列](/prism/Ch8NavigationFig6.png)



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/07/prism12-navigation/  

