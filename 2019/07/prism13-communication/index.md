# Prism Communication


注意：该版本为Prism6,最新版已有较大改动。

在构建大型复杂WPF应用程序时，常见的方法是将功能划分为离散模块程序集。还希望最小化这些模块之间静态引用的使用，这可以通过使用委托命令，区域上下文，共享服务和事件聚合器来实现。这允许模块被独立地开发，测试，部署和更新，并且它迫使松散耦合的通信。本主题提供有关何时使用委托命令和路由命令以及何时使用事件聚合器和.NET框架事件的指导。

在模块之间进行通信时，了解方法之间的差异非常重要，这样您才能最好地确定在特定方案中使用哪种方法。Prism Library提供以下通信方法：

- **解决方案指挥**。在期望用户交互立即采取行动时使用。
- **地区背景**。使用此选项可在主机区域中的主机和视图之间提供上下文信息。这种方法有点类似于**DataContext**，但它不依赖于它。
- **共享服务**。呼叫者可以在服务上调用一种方法，该方法将事件引发给消息的接收者。如果以上都不适用，请使用此选项。
- **事件聚合**。用于在没有直接的动作反应期望的情况下跨视图模型，演示者或控制器进行通信。

### Solution Commanding

如果您需要响应用户手势，例如单击命令调用程序（例如，按钮或菜单项），并且您希望基于业务逻辑启用调用程序，请使用命令。

Windows Presentation Foundation（WPF）提供**RoutedCommand**，它擅长将命令调用程序（如菜单项和按钮）与命令处理程序连接，命令处理程序与具有键盘焦点的可视树中的当前项相关联。

但是，在复合方案中，命令处理程序通常是视图模型，在可视树中没有任何关联元素，或者不是焦点元素。为了支持这种情况，Prism库提供了**DelegateCommand**，它允许您在执行命令时调用委托方法，而**CompositeCommand**允许您组合多个命令**。**这些命令与内置的**RoutedCommand不同**，后者将在可视树上上下路由命令执行和处理。这允许您在可视树中的某个点触发命令并在更高级别处理它。

该**CompositeCommand**是一个实现**ICommand的**，以便它可以被绑定到调用者。**CompositeCommands**可以连接到几个子命令; 调用**CompositeCommand**时，也会调用子命令。

**CompositeCommands**支持启用。**CompositeCommands**侦听每个连接命令的**CanExecuteChanged**事件。然后它会引发此事件通知其调用者。调用者通过在**CompositeCommand**上调用**CanExecute**来对此事件做出反应。然后，**CompositeCommand**通过在每个子命令上调用**CanExecute**来再次轮询其所有子命令。如果对**CanExecute的**任何调用返回**false**，则**CompositeCommand**将返回**false**，从而禁用调用者。

这对于跨模块通信有何帮助？基于Prism库的应用程序可能具有在shell中定义的全局**CompositeCommands**，它们具有跨模块的含义，例如**Save**，**Save All**和**Cancel**。然后，模块可以使用这些全局命令注册其本地命令并参与其执行。

***注意：** **DelegateCommand**和**CompositeCommand**可以在Prism.Commands命名空间中找到，该命名空间位于Prism.Core NuGet包中。*

&gt; **关于WPF路由事件和路由命令**

&gt; 路由事件是一种事件，可以在元素树中的多个侦听器上调用处理程序，而不是仅通知直接订阅该事件的对象。WPF路由命令通过可视树中的UI元素传递命令消息，但树外的元素将不会接收这些消息，因为它们仅从聚焦元素或明确声明的目标元素向上或向下冒泡。路由事件可用于通过元素树进行通信，因为事件的事件数据会持续到路由中的每个元素。一个元素可能会更改事件数据中的某些内容，并且该更改可用于路径中的下一个元素。

&gt; 因此，您应该在以下方案中使用WPF路由事件：在公共根目录中定义公共处理程序或定义您自己的自定义控件类。

### 创建委托命令

要创建委托命令，请在视图模型的构造函数中实例化**DelegateCommand**字段，然后将其作为**ICommand**属性公开。

```cs
// ArticleViewModel.cs
public class ArticleViewModel : BindableBase
{
    private readonly ICommand showArticleListCommand;

    public ArticleViewModel(INewsFeedService newsFeedService,
                            IRegionManager regionManager,
                            IEventAggregator eventAggregator)
    {
        this.showArticleListCommand = new DelegateCommand(this.ShowArticleList);
    }

    public ICommand ShowArticleListCommand
    {
        get { return this.showArticleListCommand; }
    }
}
```

### 创建复合命令

要创建复合命令，请在构造函数中实例化**CompositeCommand**字段，向其中添加命令，然后将其作为**ICommand**属性公开。

```cs
public class MyViewModel : BindableBase
{
    private readonly CompositeCommand saveAllCommand;

    public ArticleViewModel(INewsFeedService newsFeedService,
                            IRegionManager regionManager,
                            IEventAggregator eventAggregator)
    {
        this.saveAllCommand = new CompositeCommand();
        this.saveAllCommand.RegisterCommand(new SaveProductsCommand());
        this.saveAllCommand.RegisterCommand(new SaveOrdersCommand());
    }

    public ICommand SaveAllCommand
    {
        get { return this.saveAllCommand; }
    }
}
```

### 使命令在全球范围内可用

通常，要创建全局可用命令，请创建**DelegateCommand**或**CompositeCommand**的实例，并通过静态类公开它。

```cs
public static class GlobalCommands
{
    public static CompositeCommand MyCompositeCommand = new CompositeCommand();
}
```

在您的模块中，将子命令与全局可用命令相关联。

```cs
GlobalCommands.MyCompositeCommand.RegisterCommand(command1);
GlobalCommands.MyCompositeCommand.RegisterCommand(command2);
```

***注意：**要提高代码的可测试性，可以使用代理类访问全局可用的命令并在测试中模拟该代理类。*

### 绑定到全局可用命令

以下代码示例演示如何将按钮绑定到WPF中的命令。

```cs
&lt;Button Name=&#34;MyCompositeCommandButton&#34; Command=&#34;{x:Static local:GlobalCommands.MyCompositeCommand}&#34;&gt;Execute My Composite Command&lt;/Button&gt;
```

***注意：**另一种方法是将命令作为资源存储在**Application.Resources**部分的App.xaml文件中。然后，在视图中 - 必须在设置该资源后创建 - 您可以设置**Command =“{Binding MyCompositeCommand，Source = {StaticResource GlobalCommands}}”**以向命令添加调用者。*

### Region Context

在很多情况下，您可能希望在托管区域的视图和区域内的视图之间共享上下文信息。例如，类似主细节的视图显示业务实体并公开区域以显示该业务实体的其他详细信息。Prism Library使用名为**RegionContext**的概念来共享区域主机与区域内加载的任何视图之间的对象，如下图所示。

![使用RegionContext](/prism/Ch9CommunicationsFig1.png)

根据具体情况，您可以选择共享单条信息（例如标识符）或共享模型。视图可以检索**RegionContext**，然后注册更改通知。视图还可以更改**RegionContext**的值。有几种方法可以公开和使用**RegionContext**：

- 您可以将**RegionContext**公开给可扩展应用程序标记语言（XAML）中的区域。
- 您可以将**RegionContext**公开给代码中的区域。
- 您可以从区域内的视图中使用**RegionContext**。

***注意：**如果该视图是**DependencyObject**，则Prism Library当前仅支持从区域内的视图中使用**RegionContext**。如果您的视图不是**DependencyObject**（例如，您正在使用WPF自动数据模板并直接在区域中添加视图模型），请考虑创建自定义**RegionBehavior**以将**RegionContext**转发到视图对象。*

&gt; **关于数据上下文属性**

&gt; 数据上下文是一种允许元素从其父元素继承有关用于绑定的数据源的信息的概念。子元素自动继承其父元素的**DataContext**。数据沿着可视化树向下流动。

### 共享服务

跨模块通信的另一种方法是通过共享服务。加载模块后，模块会将其服务添加到服务定位器。通常，通过公共接口类型从服务定位器注册和检索服务。这允许模块使用其他模块提供的服务，而无需对模块进行静态引用。服务实例在模块之间共享，因此您可以共享数据并在模块之间传递消息。

在Stock Trader参考实施（Stock Trader RI）中，Market模块提供了**IMarketFeedService**的实现。Position模块通过使用shell应用程序的依赖注入容器来使用这些服务，该容器提供服务位置和解析。所述**IMarketFeedService**是指由其他模块被消耗，因此它可以在找到**StockTraderRI.Infrastructure**共同组装，但该接口的具体实现并不需要被共享的，所以它是市场模块中直接定义，并且可以是独立于其他模块更新。

要查看这些服务如何导出到MEF，请参阅**MarketFeedService.cs**和**MarketHistoryService.cs**文件，如以下代码示例所示。Position模块的**ObservablePosition**通过构造函数依赖注入接收**IMarketFeedService**服务。

```cs
// MarketFeedService.cs
[Export(typeof(IMarketFeedService))]
[PartCreationPolicy(CreationPolicy.Shared)]
public class MarketFeedService : IMarketFeedService, IDisposable
{
    ...
}
```

这有助于跨模块通信，因为服务使用者不需要对提供服务的模块的静态引用。此服务可用于在模块之间发送或接收数据。

***注意：**某些依赖项注入容器允许使用属性注册依赖项，如此示例所示。其他容器可以使用显式注册。在这些情况下，注册通常在模块加载期间发生，此时Prism调用**IModule.Initialize**方法。有关更多信息，请参阅模块化应用程序开*

### Event Aggregation

Prism库提供了一种事件机制，可以在应用程序中松散耦合的组件之间进行通信。该机制基于事件聚合器服务，允许发布者和订阅者通过事件进行通信，但仍然没有彼此直接引用。

该**EventAggregator**提供组播发布/订阅功能。这意味着可以有多个发布者引发相同的事件，并且可以有多个订阅者收听同一事件。考虑使用**EventAggregator**跨模块发布事件，以及在业务逻辑代码（如控制器和演示者）之间发送消息时。

Stock Trader RI的一个例子就是点击**Process Order**按钮并顺序处理订单; 在这种情况下，其他模块需要知道订单已成功处理，以便他们可以更新他们的视图。

使用Prism Library创建的事件是键入的事件。这意味着您可以在运行应用程序之前利用编译时类型检查来检测错误。在Prism库中，**EventAggregator**允许订阅者或发布者定位特定的**EventBase**。事件聚合器还允许多个发布者和多个订阅者，如下图所示。

![使用事件聚合器](/prism/Ch9CommunicationsFig2.png)

&gt; **关于.NET Framework事件**

&gt; 如果不要求松散耦合，则使用.NET Framework事件是组件之间通信的最简单，最直接的方法。.NET Framework中的事件实现了Publish-Subscribe模式，但是要订阅对象，您需要直接引用该对象，在复合应用程序中，该对象通常驻留在另一个模块中。这导致紧密耦合的设计。因此，.NET Framework事件用于模块内的通信，而不是模块之间的通信。

&gt; 如果使用.NET Framework事件，则必须非常小心内存泄漏，尤其是如果您有一个非静态或短期组件，它可以在静态或长寿命的事件上订阅事件。如果您没有取消订阅订阅者，则发布者将保留该订阅者，这将阻止第一个订阅者被垃圾收集。

### IEventAggregator

所述**EventAggregator**类被提供作为在容器中的服务，并且可以通过检索**IEventAggregator**接口。事件聚合器负责定位或构建事件以及保留系统中事件的集合。

```cs
public interface IEventAggregator
{
    TEventType GetEvent&lt;TEventType&gt;() where TEventType : EventBase;
}
```

该**EventAggregator**构建事件在其第一次访问，如果它尚未建立。这使发布者或订阅者无需确定事件是否可用。

### PubSubEvent

连接发布者和订阅者的真正工作是由**PubSubEvent**类完成的。这是Prism库中包含的**EventBase**类的唯一实现。此类维护订户列表并处理订阅者的事件调度。

所述**PubSubEvent**类是一个一般类，需要将其定义为一般类型的有效载荷类型。这有助于在编译时强制发布者和订阅者提供成功事件连接的正确方法。以下代码显示了**PubSubEvent**类的部分定义。

***注意：** **PubSubEvent**可以在Prism.Events命名空间中找到，该命名空间位于Prism.Core NuGet包中。*

```cs
// PubSubEvent.cs
public class PubSubEvent&lt;TPayload&gt; : EventBase
{
    ...
    public SubscriptionToken Subscribe(Action&lt;TPayload&gt; action);
    public SubscriptionToken Subscribe(Action&lt;TPayload&gt; action, ThreadOption threadOption);
    public SubscriptionToken Subscribe(Action&lt;TPayload&gt; action, bool keepSubscriberReferenceAlive)
    public SubscriptionToken Subscribe(Action&lt;TPayload&gt; action, ThreadOption threadOption, bool keepSubscriberReferenceAlive)

    public virtual SubscriptionToken Subscribe(Action&lt;TPayload&gt; action, ThreadOption threadOption, bool keepSubscriberReferenceAlive);
    public virtual SubscriptionToken Subscribe(Action&lt;TPayload&gt; action, ThreadOption threadOption, bool keepSubscriberReferenceAlive, Predicate&lt;TPayload&gt; filter);
    public virtual void Publish(TPayload payload);
    public virtual void Unsubscribe(Action&lt;TPayload&gt; subscriber);
    public virtual bool Contains(Action&lt;TPayload&gt; subscriber)
    ...
}
```

### 创建和发布事件

以下各节介绍如何创建，发布和订阅**PubSubEvent**使用**IEventAggregator**接口。

#### 创建一个事件

所述**PubSubEvent &lt;TPayload&gt;**旨在是为应用程序或模块的特定事件的基类。**TPayLoad**是事件有效负载的类型。有效负载是在事件发布时将传递给订阅者的参数。

例如，以下代码显示了股票交易者参考实现（Stock Trader RI）中的**TickerSymbolSelectedEvent**。有效负载是包含公司符号的字符串。请注意此类的实现是如何为空。

```cs
public class TickerSymbolSelectedEvent : PubSubEvent&lt;string&gt;{}
```

***注意：**在复合应用程序中，事件通常在多个模块之间共享，因此它们在公共位置定义。在Stock Trader RI中，这是在**StockTraderRI.Infrastructure**项目中完成的。*

#### 发布活动

发布者通过从**EventAggregator**检索事件并调用**Publish**方法来引发事件。要访问**EventAggregator**，可以通过向类构造函数添加类型为**IEventAggregator**的参数来使用依赖项注入。

以下代码演示了如何发布**TickerSymbolSelectedEvent**。

```cs
this.eventAggregator.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Publish(&#34;STOCK0&#34;);
```

### 订阅活动

订阅者可以使用**PubSubEvent**类上提供的一个**Subscribe**方法重载来登记事件。有几种方法可以订阅**PubSubEvents**。使用以下标准来帮助确定最适合您需求的选项：

- 如果您需要能够在收到事件时更新UI元素，请订阅以在UI线程上接收事件。
- 如果您需要过滤事件，请在订阅时提供过滤器委托。
- 如果您对事件有性能问题，请考虑在订阅时使用强引用的委托，然后手动取消订阅**PubSubEvent**。
- 如果以上都不适用，请使用默认订阅。

以下部分描述了这些选项。

#### 订阅UI线程

订阅者通常需要更新UI元素以响应事件。在WPF中，只有UI线程可以更新UI元素。

默认情况下，订阅者在发布者的线程上接收事件。如果发布者从UI线程发送事件，则订阅者可以更新UI。但是，如果发布者的线程是后台线程，则订阅者可能无法直接更新UI元素。在这种情况下，订户需要使用**Dispatcher**类在UI线程上安排更新。

Prism Library提供的**PubSubEvent**可以通过允许订阅者在UI线程上自动接收事件来提供帮助。订阅者在订阅期间指示此信息，如以下代码示例所示。

```cs
public void Run()
{
    ...
    this.eventAggregator.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Subscribe(ShowNews, ThreadOption.UIThread);
);
}

public void ShowNews(string companySymbol)
{
    this.articlePresentationModel.SetTickerSymbol(companySymbol);
}
```

**ThreadOption**有以下选项：

- **PublisherThread**。使用此设置可在发布商的主题上接收活动。这是默认设置。
- **BackgroundThread**。使用此设置在.NET Framework线程池线程上异步接收事件。
- **UIThread**。使用此设置可在UI线程上接收事件。

***注意：**为了让**PubSubEvents**在UI线程上发布给订阅者，必须首先在UI线程上构造**EventAggregator**。*

#### 订阅过滤

订阅者可能不需要处理已发布事件的每个实例。在这些情况下，订户可以使用**过滤器**参数。的**滤波器**参数的类型的**System.Predicate &lt;TPayLoad&gt;**和是当事件被发布，以确定是否已发布的事件的有效载荷的一组具有调用的回调订户要求的标准相匹配的被执行的委托。如果有效负载不满足指定的条件，则不执行订户回调。

通常，此过滤器作为lambda表达式提供，如以下代码示例所示。

```cs
FundAddedEvent fundAddedEvent = this.eventAggregator.GetEvent&lt;FundAddedEvent&gt;();

fundAddedEvent.Subscribe(FundAddedEventHandler, ThreadOption.UIThread, false,
fundOrder =&gt; fundOrder.CustomerId == this.customerId);
```

***注：**该**订阅**方法返回类型的订阅令牌**Prism.Events.SubscriptionToken**可用于以后删除订阅的事件。当您使用匿名委托或lambda表达式作为回调委托时，或者使用不同的过滤器订阅相同的事件处理程序时，此标记特别有用。*

***注意：**建议不要在回调委托中修改有效负载对象，因为多个线程可能同时访问有效负载对象。您可以使有效负载不可变，以避免并发错误。*

#### 订阅使用强引用

如果您在短时间内提出多个事件并注意到它们的性能问题，则可能需要使用强委托引用进行订阅。如果您这样做，则需要在处置订户时手动取消订阅该事件。

默认情况下，**PubSubEvent**维护对订阅者处理程序的弱委托引用，并对订阅进行过滤。这意味着**PubSubEvent**所持有的引用不会阻止订阅者的垃圾收集。使用弱委托引用可以使订户免于取消订阅并允许正确的垃圾收集。

但是，维护此弱委托引用比相应的强引用要慢。对于大多数应用程序，此性能不会很明显，但如果您的应用程序在短时间内发布大量事件，则可能需要对**PubSubEvent**使用强引用。如果您确实使用了强委托引用，则订阅者应该取消订阅，以便在不再使用订阅对象时启用正确的垃圾回收。

要使用强引用进行**订阅**，请在**Subscribe**方法上使用**keepSubscriberReferenceAlive**参数，如以下代码示例所示。

```cs
FundAddedEvent fundAddedEvent = eventAggregator.GetEvent&lt;FundAddedEvent&gt;();

bool keepSubscriberReferenceAlive = true;

fundAddedEvent.Subscribe(FundAddedEventHandler, ThreadOption.UIThread, keepSubscriberReferenceAlive, fundOrder =&gt; fundOrder.CustomerId == _customerId);
```

所述**keepSubscriberReferenceAlive**参数的类型的**布尔**：

- 设置为**true时**，事件实例会保留对订户实例的强引用，从而不允许它进行垃圾回收。有关如何取消订阅的信息，请参阅本主题后面的“ [取消订阅事件](http://prismlibrary.github.io/docs/wpf/legacy/Communication.html#unsubscribing-from-an-event) ”一节。
- 当设置为**false**（省略此参数时的默认值）时，事件维护对订户实例的弱引用，从而允许垃圾收集器在没有其他引用时配置订阅者实例。收集订户实例后，将自动取消订阅该事件。

#### 默认订阅

对于最小订阅或默认订阅，订阅者必须提供具有接收事件通知的适当签名的回调方法。例如，**TickerSymbolSelectedEvent**的处理程序要求该方法采用字符串参数，如以下代码示例所示。

```cs
public TrendLineViewModel(IMarketHistoryService marketHistoryService, IEventAggregator eventAggregator)
{
    ... eventAggregator.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Subscribe(this.TickerSymbolChanged);
}

public void TickerSymbolChanged(string newTickerSymbol)
{
    MarketHistoryCollection newHistoryCollection = this.marketHistoryService.GetPriceHistory(newTickerSymbol);

    this.TickerSymbol = newTickerSymbol;
    this.HistoryCollection = newHistoryCollection;
}
```

#### 取消订阅活动

如果您的订户不再想要接收活动，您可以使用订阅者的处理程序取消订阅，也可以使用订阅令牌取消订阅。

以下代码示例演示如何直接取消订阅处理程序。

```cs
FundAddedEvent fundAddedEvent = this.eventAggregator.GetEvent&lt;FundAddedEvent&gt;();

fundAddedEvent.Subscribe(FundAddedEventHandler, ThreadOption.PublisherThread);

fundAddedEvent.Unsubscribe(FundAddedEventHandler);
```

以下代码示例演示如何取消订阅订阅令牌。令牌作为Subscribe方法的返回值提供。

```cs
FundAddedEvent fundAddedEvent = this.eventAggregator.GetEvent&lt;FundAddedEvent&gt;();

subscriptionToken = fundAddedEvent.Subscribe(FundAddedEventHandler, ThreadOption.UIThread, false, fundOrder =&gt; fundOrder.CustomerId == this.customerId);

fundAddedEvent.Unsubscribe(subscriptionToken);
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/07/prism13-communication/  

