# Prism Event Aggregator


 

注意：该版本为Prism6,最新版已有较大改动。

Prism库提供了一种事件机制，可以在应用程序中松散耦合的组件之间进行通信。该机制基于事件聚合器服务，允许发布者和订阅者通过事件进行通信，但仍然没有彼此直接引用。

在`EventAggregator`提供多种发布/订阅功能。这意味着可以有多个发布者引发相同的事件，并且可以有多个订阅者收听同一事件。考虑使用`EventAggregator`跨模块发布事件以及在业务逻辑代码（如控制器和演示者）之间发送消息时。

使用Prism Library创建的事件是键入的事件。这意味着您可以在运行应用程序之前利用编译时类型检查来检测错误。在Prism Library中，`EventAggregator`允许订阅者或发布者定位特定的`EventBase`。事件聚合器还允许多个发布者和多个订阅者，如下图所示。

![ä½¿ç¨äºä»¶èåå¨](/prism/event-aggregator-1.png)

###  IEventAggregator

`EventAggregator`类被提供作为在容器中的服务，并且可以通过检索`IEventAggregator`界面。事件聚合器负责定位或构建事件以及保留系统中事件的集合。

```cs
public interface IEventAggregator
{
    TEventType GetEvent&lt;TEventType&gt;() where TEventType : EventBase;
}
```

如果`EventAggregator`事件尚未构造，则在第一次访问时构造事件。这使发布者或订阅者无需确定事件是否可用。

### PubSubEvent

连接发布者和订阅者的真正工作是由`PubSubEvent`班级完成的。`EventBase`是Prism Library中包含的类的唯一实现。此类维护订户列表并处理订阅者的事件调度。

`PubSubEvent`类是一个通用类，需要将其定义为一般类型的有效载荷类型。这有助于在编译时强制发布者和订阅者提供成功事件连接的正确方法。以下代码显示了PubSubEvent类的部分定义。

### 创建一个事件

`PubSubEvent&lt;TPayload&gt;`意图是为应用程序或模块的特定事件的基类。`TPayLoad`是事件的有效负载的类型。有效负载是在事件发布时将传递给订阅者的参数。

例如，以下代码显示了`TickerSymbolSelectedEvent`。有效负载是包含公司符号的字符串。请注意此类的实现是如何为空。

```c#
public class TickerSymbolSelectedEvent : PubSubEvent&lt;string&gt;{}
```

##### 注意

在复合应用程序中，事件经常在多个模块之间共享，因此它们在公共位置定义。通常的做法是在共享程序集中定义这些事件，例如“核心”或“基础结构”项目。

### 发布事件

发布者通过从`EventAggregator`调用`Publish`方法中检索事件来引发事件。要访问`EventAggregator`，可以通过向`IEventAggregator`类构造函数添加类型参数来使用依赖项注入。

```c#
public class MainPageViewModel
{
    IEventAggregator _eventAggregator;
    public MainPageViewModel(IEventAggregator ea)
    {
        _eventAggregator = ea;
    }
}
```

以下代码演示了如何发布TickerSymbolSelectedEvent。

```c#
_eventAggregator.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Publish(&#34;STOCK0&#34;);
```

### 订阅事件

订阅者可以使用类中`Subscribe`可用的方法重载之一来参与事件`PubSubEvent`。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Subscribe(ShowNews);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

有几种订阅方式`PubSubEvents`。使用以下标准来帮助确定最适合您需求的选项：

- 如果您需要能够在收到事件时更新UI元素，请订阅以在UI线程上接收事件。
- 如果您需要过滤事件，请在订阅时提供过滤器委托。
- 如果您对事件有性能问题，请考虑在订阅时使用强引用的委托，然后手动取消订阅PubSubEvent。
- 如果以上都不适用，请使用默认订阅。

以下部分描述了这些选项。

#### 在UI线程订阅

订阅者通常需要更新UI元素以响应事件。在WPF中，只有UI线程可以更新UI元素。

默认情况下，订阅者在发布者的线程上接收事件。如果发布者从UI线程发送事件，则订阅者可以更新UI。但是，如果发布者的线程是后台线程，则订阅者可能无法直接更新UI元素。在这种情况下，订户需要使用Dispatcher类在UI线程上安排更新。

该`PubSubEvent`具有棱镜图书馆可以通过允许用户自动接收UI线程上的事件协助。订阅者在订阅期间指示此信息，如以下代码示例所示。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;().Subscribe(ShowNews, ThreadOption.UIThread);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

以下选项适用于`ThreadOption`：

- `PublisherThread`：使用此设置在发布商的主题上接收活动。这是默认设置。
- `BackgroundThread`：使用此设置在.NET Framework线程池线程上异步接收事件。
- `UIThread`：使用此设置在UI线程上接收事件。

注意

为了`PubSubEvent`在UI线程上发布给订阅者，`EventAggregator`最初必须在UI线程上构建。

#### 订阅过滤

订阅者可能不需要处理已发布事件的每个实例。在这些情况下，订户可以使用过滤器参数。filter参数是类型的，`System.Predicate&lt;TPayLoad&gt;`并且是在发布事件时执行的委托，以确定已发布事件的有效负载是否与调用订阅者回调所需的一组条件匹配。如果有效负载不满足指定的条件，则不执行订户回调。

通常，此过滤器作为lambda表达式提供，如以下代码示例所示。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, false, 
                              companySymbol =&gt; companySymbol == &#34;STOCK0&#34;);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

##### 注意

该`Subscribe`方法返回一个类型的订阅令牌`Prism.Events.SubscriptionToken`，可用于稍后删除对该事件的订阅。当您使用匿名委托或lambda表达式作为回调委托时，或者使用不同的过滤器订阅相同的事件处理程序时，此标记特别有用。

##### 注意

建议不要在回调委托中修改有效负载对象，因为多个线程可能同时访问有效负载对象。您可以使有效负载不可变，以避免并发错误。

#### 使用强引用订阅

如果您在短时间内提出多个事件并注意到它们的性能问题，则可能需要使用强委托引用进行订阅。如果您这样做，则需要在处置订户时手动取消订阅该事件。

默认情况下，`PubSubEvent`维护对订阅者处理程序的弱委托引用并对订阅进行过滤。这意味着`PubSubEvent`保留的引用不会阻止订户的垃圾收集。使用弱委托引用可以使订户免于取消订阅并允许正确的垃圾收集。

但是，维护此弱委托引用比相应的强引用要慢。对于大多数应用程序，此性能不会很明显，但如果您的应用程序在短时间内发布大量事件，则可能需要对PubSubEvent使用强引用。如果您确实使用了强委托引用，则订阅者应该取消订阅，以便在不再使用订阅对象时启用正确的垃圾回收。

要使用强引用进行订阅，请使用方法`keepSubscriberReferenceAlive`上的参数`Subscribe`，如以下代码示例所示。

```c#
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        bool keepSubscriberReferenceAlive = true;
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, keepSubscriberReferenceAlive, 
                              companySymbol =&gt; companySymbol == &#34;STOCK0&#34;);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

`keepSubscriberReferenceAlive`参数的类型的`bool`：

- 设置为时`true`，事件实例保留对订户实例的强引用，从而不允许它收集垃圾。有关如何取消订阅的信息，请参阅本主题后面的“取消订阅事件”一节。
- 当设置为`false`（省略此参数时的默认值）时，事件维护对订户实例的弱引用，从而允许垃圾收集器在没有其他引用时配置订阅者实例。收集订户实例后，将自动取消订阅该事件。

### 取消事件订阅

如果您的订户不再想要接收活动，您可以使用订阅者的处理程序取消订阅，也可以使用订阅令牌取消订阅。

以下代码示例演示如何直接取消订阅处理程序。

```c#
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;();
        _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(ShowNews);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

以下代码示例演示如何取消订阅订阅令牌。令牌作为方法的返回值提供`Subscribe`。

```c#
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    SubscriptionToken _token;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent&lt;TickerSymbolSelectedEvent&gt;();
        _token = _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(_token);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/06/prism4-eventaggregator/  

