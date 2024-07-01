# Prism 高级MVVM


注意：该版本为Prism6,最新版已有较大改动。

上一个主题描述了如何通过将应用程序的用户界面（UI），表示逻辑和业务逻辑分成三个独立的类（View，ViewModel和Model）来实现ModelViewViewModel（MVVM）模式的基本元素，实现这些类之间的交互（通过数据绑定，命令和数据验证接口），并实施处理构造和连接的策略。本主题描述了一些复杂的场景，并描述了MVVM模式如何支持它们。下一节将介绍如何将命令链接在一起或与子视图关联，以及如何扩展命令以支持自定义要求。

“ 使用依赖注入容器”（例如Unity应用程序块（Unity））或使用托管可扩展性框架（MEF）时，“ [高级构造和连接](http://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html#advanced-construction-and-wire-up) ”一节提供了有关处理构造和连接的指导。最后一节描述了如何通过提供单元测试应用程序的视图模型和模型类以及测试行为的指导来测试MVVM应用程序。

### 命令

命令提供了一种将命令的实现逻辑与其UI表示分开的方法。数据绑定或行为提供了一种方法，用于声明性地将视图中的元素与视图模型提供的命令相关联。的部分中，[命令](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#commands)和[实施MVVM模式](http://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html)描述了如何命令可以作为命令对象或在视图模型命令的方法来实现，以及它们是如何可以从控制在视图中通过使用调用的内置**命令**由某些控件提供的属性。

**WPF路由命令**：应该注意的是，MVVM模式中作为命令对象或命令方法实现的命令与WPF的路由命令的内置实现略有不同。WPF路由命令通过在UI树中的元素（特别是[逻辑树](https://msdn.microsoft.com/en-us/library/ms753391.aspx)）路由来传递命令消息。因此，命令消息在UI树中从聚焦元素向上或向下路由到显式指定的目标元素; 默认情况下，它们不会路由到UI树之外的组件，例如与视图关联的视图模型。但是，WPF路由命令可以使用视图代码隐藏中定义的命令处理程序将命令调用转发到视图模型类。

#### 复合命令

在许多情况下，视图模型定义的命令将绑定到关联视图中的控件，以便用户可以直接从视图中调用该命令。但是，在某些情况下，您可能希望能够从应用程序UI的父视图中的控件调用一个或多个视图模型上的命令。

例如，如果您的应用程序允许用户同时编辑多个项目，您可能希望允许用户使用应用程序工具栏或功能区中的按钮所代表的单个命令来保存所有项目。在这种情况下，**Save All**命令将调用每个项目的视图模型实例实现的每个**Save**命令，如下图所示。

![SaveAll复合命令](/prism/Ch6AdvMVVMFig12.png)

Prism通过**CompositeCommand**类支持这种情况。

所述**CompositeCommand**类表示从多个子指令构成的指令。调用复合命令时，将依次调用其每个子命令。在需要在UI中将一组命令表示为单个命令或者要调用多个命令来实现逻辑命令的情况下，它非常有用。

例如，**CompositeCommand**类用于股票交易者参考实现（Stock Trader RI），以实现买入/卖出视图中的“ **全部提交”**按钮所代表的**SubmitAllOrders**命令。当用户单击“ **全部提交”**按钮时，将执行由各个买/卖交易定义的每个**SubmitCommand**。

该**CompositeCommand**类维护子类的命令（列表**DelegateCommand**实例）。在**执行**该方法**CompositeCommand**类只是调用**执行**每个反过来子命令的方法。在**CanExecute**方法同样调用**CanExecute**每个子命令的方法，但是如果有子命令不能执行时，**CanExecute**方法将返回**错误**。换句话说，默认情况下，**只有在可以执行所有子命令时才能执行CompositeCommand**。

#### 注册和取消子命令

使用**RegisterCommand**和**UnregisterCommand**方法注册或取消注册子命令。例如，在Stock Trader RI中，每个买/卖订单的**提交**和**取消**命令都使用**SubmitAllOrders**和**CancelAllOrders**复合命令进行注册，如以下代码示例所示（请参阅**OrdersController**类）。

```c#
// OrdersController.cs
commandProxy.SubmitAllOrdersCommand.RegisterCommand(
                        orderCompositeViewModel.SubmitCommand );
commandProxy.CancelAllOrdersCommand.RegisterCommand(
                        orderCompositeViewModel.CancelCommand );
```

注意：前面的**commandProxy**对象提供对**Submit**和**Cancel**复合命令的实例访问，这些命令是静态定义的。有关更多信息，请参阅类文件StockTraderRICommands.cs。

#### 在活动子视图上执行命令

Executing Commands on Active Child Views

通常，您的应用程序需要在应用程序的UI中显示子视图的集合，其中每个子视图将具有相应的视图模型，而该视图模型又可以实现一个或多个命令。复合命令可用于表示应用程序UI中子视图实现的命令，并有助于协调从父视图中调用它们的方式。为了支持这些场景，Prism **CompositeCommand**和**DelegateCommand**类被设计为与Prism区域一起使用。

Prism的Regin（[区域](http://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html#regions)，在[Composing接口](http://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html)）提供了一种方法用于子视图与在应用程序的UI逻辑占位符相关联。它们通常用于将子视图的特定布局与其逻辑占位符及其在UI中的位置分离。区域基于附加到特定布局控件的命名占位符。下图显示了一个示例，其中每个子视图都已添加到名为**EditRegion**的区域，并且UI设计器已选择使用**Tab**控件来布局该区域内的视图。

![使用Tab控件定义EditRegion](/prism/Ch6AdvMVVMFig13.png)

父视图级别的复合命令通常用于协调如何调用子视图级别的命令。在某些情况下，您将需要执行所有显示视图的命令，如前面所述的**Save All**命令示例中所示。在其他情况下，您将希望仅在活动视图上执行该命令。在这种情况下，复合命令将仅对被视为活动的视图执行子命令; 它不会在非活动的视图上执行子命令。例如，您可能希望在应用程序的工具栏或功能区上实现**缩放**命令，该命令仅导致当前活动项目被缩放，如下图所示。

![在单个子节点上执行CompositeCommand](/prism/Ch6AdvMVVMFig14.png)

为了支持这种情况，Prism提供了**IActiveAware**接口。所述**IActiveAware**接口定义的**IsActive**属性，返回**True**时实施者是活动的，以及一个**IsActiveChanged**每当激活状态改变时引发事件。

您可以在子视图或视图模型上实现**IActiveAware**接口。它主要用于跟踪区域内子视图的活动状态。视图是否处于活动状态由区域适配器确定，该适配器协调特定区域控件内的视图。对于前面显示的**Tab**控件，有一个区域适配器，例如，它将当前所选选项卡中的视图设置为**活动状态**。

该**DelegateCommand**类还实现了**IActiveAware**接口。可以将**CompositeCommand**配置为通过为构造函数中的**monitorCommandActivity**参数指定True来评估子**DelegateCommands**的活动状态（除了**CanExecute**状态）。当此参数设置为**true**，**CompositeCommand**类将在确定**CanExecute**方法的返回值以及在**Execute**方法中执行子命令时考虑每个子**DelegateCommand**的活动状态。

当**monitorCommandActivity**参数为**true时**，**CompositeCommand**类会出现以下行为：

- **CanExecute**。仅在可以执行所有活动命令时返回**true**。根本不会考虑不活动的子命令。
- **执行**。执行所有活动命令。根本不会考虑不活动的子命令。

您可以使用此功能来实现前面描述的示例。通过在子视图模型上实现**IActiveAware**接口，当您的子视图对该区域变为活动或非活动时，将通知您。当子视图的活动状态更改时，您可以更新子命令的活动状态。然后，当用户调用**缩放**复合命令时，将调用活动子视图上的**缩放**命令。

#### 集合中的命令

在视图中显示项目集合时经常遇到的另一种常见情况是，您需要将集合中每个项目的UI与父视图级别（而不是项目级别）的命令相关联。

例如，在下图所示的应用程序中，视图显示**ListBox**控件中的项集合，用于显示每个项的数据模板定义了一个**Delete**按钮，允许用户从集合中删除单个项。

![集合中的绑定命令](/prism/Ch6AdvMVVMFig15.png)

因为该视图模型实现的**删除**命令，面临的挑战是要连接的**删除**按钮在每个项目的用户界面中，对**删除**由视图模型实现的命令。之所以出现这种困难是因为**ListBox中**每个项的数据上下文引用了集合中的项而不是实现**Delete**命令的父视图模型。

解决此问题的一种方法是使用**ElementName**绑定属性将数据模板中的按钮绑定到父视图中的命令，以确保绑定相对于父控件而不是相对于数据模板。以下XAML说明了这种技术。

```xaml
&lt;Grid x:Name=&#34;root&#34;&gt;
    &lt;ListBox ItemsSource=&#34;{Binding Path=Items}&#34;&gt;
        &lt;ListBox.ItemTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;Button Content=&#34;{Binding Path=Name}&#34;
                    Command=&#34;{Binding ElementName=root, 
                    Path=DataContext.DeleteCommand}&#34; /&gt;
            &lt;/DataTemplate&gt;
        &lt;/ListBox.ItemTemplate&gt;
    &lt;/ListBox&gt;
&lt;/Grid&gt;
```

数据模板中按钮控件的内容绑定到集合中项目的**Name**属性。但是，按钮的命令通过根元素的数据上下文绑定到**Delete**命令。这允许按钮在父视图级别而不是在项目级别绑定到命令。您可以使用**CommandParameter**属性指定要应用命令的项目，也可以实现命令以对当前所选项目进行操作（通过**CollectionView**）。

### 交互触发器和命令

Interaction Triggers and Commands

另一种命令方法是使用Blend for Visual Studio 2013交互触发器和**InvokeCommandAction**操作。

```xml
&lt;Button Content=&#34;Submit&#34; IsEnabled=&#34;{Binding CanSubmit}&#34;&gt;
    &lt;i:Interaction.Triggers&gt;
        &lt;i:EventTrigger EventName=&#34;Click&#34;&gt;
            &lt;i:InvokeCommandAction Command=&#34;{Binding SubmitCommand}&#34;/&gt;
        &lt;/i:EventTrigger&gt;
    &lt;/i:Interaction.Triggers&gt;
&lt;/Button&gt;
```

此方法可用于您可以附加交互触发器的任何控件。如果要将命令附加到未实现**ICommandSource**接口的控件，或者要在默认事件以外的事件上调用命令，则此功能特别有用。同样，如果需要为命令提供参数，可以使用**CommandParameter**属性。

下面显示了如何使用配置为侦听ListBox的**SelectionChanged**事件的Blend EventTrigger 。发生此事件时，**InvokeCommandAction**将调用**SelectedCommand**。

```xml
&lt;ListBox ItemsSource=&#34;{Binding Items}&#34; SelectionMode=&#34;Single&#34;&gt;
    &lt;i:Interaction.Triggers&gt;
        &lt;i:EventTrigger EventName=&#34;SelectionChanged&#34;&gt;
            &lt;i:InvokeCommandAction Command=&#34;{Binding SelectedCommand}&#34; /&gt;
        &lt;/i:EventTrigger&gt;
    &lt;/i:Interaction.Triggers&gt;
&lt;/ListBox&gt;
```

**启用命令的控件与行为**

支持命令的WPF控件允许您以声明方式将控件挂接到命令。当用户以特定方式与控件交互时，这些控件将调用指定的命令。例如，对于**Button**控件，将在用户单击按钮时调用该命令。与命令关联的此事件是固定的，无法更改。

行为还允许您以声明方式将控件连接到命令。但是，行为可以与控件引发的一系列事件相关联，并且它们可以用于在视图模型中有条件地调用关联的命令对象或命令方法。换句话说，行为可以解决许多与启用命令的控件相同的场景，并且它们可以提供更大程度的灵活性和控制。

您需要选择何时使用启用命令的控件以及何时使用行为以及要使用的行为类型。如果您希望使用单一机制将视图中的控件与视图模型中的功能相关联或者为了保持一致性，则可以考虑使用行为，即使对于本身支持命令的控件也是如此。

如果您只需要使用启用命令的控件来调用视图模型上的命令，并且如果您对调用命令的默认事件感到满意，则可能不需要行为。同样，如果您的开发人员或UI设计人员不使用Blend for Visual Studio 2013，您可能会支持启用命令的控件（或自定义附加行为），因为Blend行为需要额外的语法。

### 将EventArgs参数传递给命令

当您需要调用命令以响应位于视图中的控件引发的事件时，您可以使用Prism的**InvokeCommandAction**。Prism的**InvokeCommandAction**与Blend SDK中的同名类有两种不同。首先，Prism **InvokeCommandAction**根据命令的**CanExecute**方法的返回值更新关联控件的启用状态。其次，Prism **InvokeCommandAction**使用从父触发器传递给它的**EventArgs**参数，如果未设置**CommandParameter，**则将其传递给关联的命令。

有时你需要一个参数传递给来自父触发，如**EventArgs**作为**EventTrigger**参数。在这种情况下，您不能使用Blend的**InvokeCommandAction**操作。

在下面的代码中，您可以看到Prism的**InvokeCommandAction**具有一个名为**TriggerParameterPath**的属性，该属性用于指定作为命令参数传递的参数的成员（可能是嵌套的）。在以下示例中，**SelectionChanged** EventArgs 的**AddedItems**属性将传递给**SelectedCommand**命令。

```xaml
&lt;ListBox Grid.Row=&#34;1&#34; Margin=&#34;5&#34; ItemsSource=&#34;{Binding Items}&#34; SelectionMode=&#34;Single&#34;&gt;
    &lt;i:Interaction.Triggers&gt;
        &lt;i:EventTrigger EventName=&#34;SelectionChanged&#34;&gt;
            &lt;!-- This action will invoke the selected command in the view model and pass the parameters of the event to it. --&gt;
            &lt;prism:InvokeCommandAction Command=&#34;{Binding SelectedCommand}&#34; TriggerParameterPath=&#34;AddedItems&#34; /&gt;
        &lt;/i:EventTrigger&gt;
    &lt;/i:Interaction.Triggers&gt;
&lt;/ListBox&gt;
```

### 处理异步交互

Handling Asynchronous Interactions

您的视图模型通常需要与应用程序中的服务和组件进行交互，这些服务和组件是异步通信而不是同步通信。如果您通过网络与Web服务或其他资源交互，或者您的应用程序使用后台任务执行计算或I / O，则尤其如此。异步执行这些操作可确保您的应用程序保持响应，这对于提供良好的用户体验至关重要。

当用户启动异步请求或后台任务时，很难预测响应何时到达（或者即使它将到达），并且通常很难预测它将返回什么线程。因为UI只能在UI线程中更新，所以通常需要通过在UI线程上调度请求来更新UI。

### 检索数据并与Web服务交互

Retrieving Data and Interacting with Web Services

在与Web服务或其他远程访问技术交互时，您将经常遇到**IAsyncResult**模式。在此模式中，您可以使用**BeginGetQuestionnaire**和**EndGetQuestionnaire**等方法，而不是调用**GetQuestionnaire**等方法。要启动异步调用，请调用**BeginGetQuestionnaire**。要在调用目标方法时获取结果或确定是否存在异常，请在调用完成时调用**EndGetQuestionnaire**。

要确定何时调用**EndGetQuestionnaire**，您可以轮询完成或（最好）在调用**BeginGetQuestionnaire**期间指定回调。使用回调方法，当目标方法的执行完成时，将调用您的回调方法，允许您从那里调用**EndGetQuestionnaire**，如此处所示。

```c#
IAsyncResult asyncResult = this.service.BeginGetQuestionnaire(GetQuestionnaireCompleted, null); // object state, not used in this example

private void GetQuestionnaireCompleted(IAsyncResult result)
{
    try
    {
        questionnaire = this.service.EndGetQuestionnaire(result);
    }
    catch (Exception ex)
    {
        // Do something to report the error.
    }
}
```

需要注意的是，在调用**End**方法（在本例中为**EndGetQuestionnaire**）时，将引发在执行请求期间发生的任何异常。您的应用程序必须处理这些，并且可能需要通过UI以线程安全的方式报告它们。如果您不处理这些，线程将结束，您将无法处理结果。

由于响应通常不在UI线程上，如果您计划修改任何会影响UI状态的内容，则需要使用线程**Dispatcher**或**SynchronizationContext**对象将响应分派给UI线程。在WPF中，您通常会使用调度程序。

在下面的代码示例中，异步检索**Questionnaire**对象，然后将其设置为**QuestionnaireView**的数据上下文。您可以使用调度程序的**CheckAccess**方法来查看您是否在UI线程上。如果不是，则需要使用**BeginInvoke**方法在UI线程上执行请求。

```c#
var dispatcher = System.Windows.Deployment.Current.Dispatcher;
if (dispatcher.CheckAccess())
{
    QuestionnaireView.DataContext = questionnaire;
}
else
{
    dispatcher.BeginInvoke(
        () =&gt; { Questionnaire.DataContext = questionnaire; });
}
```

Model-View-ViewModel参考实现（MVVM RI）显示了如何使用类似于前面示例的基于**IAsyncResult**的服务接口的示例。它还包装服务，为消费者提供更简单的回调机制，并处理调用者对调用者线程的调度。例如，以下代码示例显示了问卷的检索。

```c#
this.questionnaireRepository.GetQuestionnaireAsync(
    (result) =&gt;
    {
        this.Questionnaire = result.Result;
    });
```

返回的**结果**对象将包装检索到的结果以及可能发生的错误。以下代码示例显示了如何评估错误。

```c#
this.questionnaireRepository.GetQuestionnaireAsync(
    (result) =&gt;
    {
        if (result.Error == null) 
        {
            this.Questionnaire = result.Result;
            ...
        }
        else
        {
            // Handle error.
        }
    });
```

### 用户交互模式

通常，应用程序需要在继续操作之前通知用户事件的发生或要求确认。这些交互通常是简短的交互，旨在简单地告知应用程序应用程序的更改或从中获取简单的响应。这些交互中的一些可以对用户呈现模态，例如当显示对话框或消息框时，或者它们对于用户可能看起来是非模态的，例如当显示Toast通知或弹出窗口时。

在这些情况下，有多种方法可以与用户进行交互，但是在基于MVVM的应用程序中以保持关注点清晰分离的方式实现它们可能具有挑战性。例如，在非MVVM应用程序中，您经常在UI的代码隐藏文件中使用**MessageBox**类来简单地提示用户进行响应。在MVVM应用程序中，这不合适，因为它会破坏视图和视图模型之间关注点的分离。

就MVVM模式而言，视图模型负责启动与用户的交互以及消费和处理任何响应，而视图负责使用适当的任何用户体验实际管理与用户的交互。保持视图模型中实现的表示逻辑与视图实现的用户体验之间的关注点分离有助于提高可测试性和灵活性。

在MVVM模式中实现这些类型的用户交互有两种常用方法。一种方法是实现视图模型可以使用的服务来启动与用户的交互，从而保持其在视图的实现上的独立性。另一种方法使用视图模型引发的事件来表达与用户交互的意图，以及视图中绑定到这些事件并管理交互的可视方面的组件。以下各节将介绍这些方法中的每一种。

#### 使用交互服务

在该方法中，视图模型依赖于交互服务组件以通过消息框发起与用户的交互。此方法通过将交互的可视实现封装在单独的服务组件中来支持关注点和可测试性的清晰分离。通常，视图模型依赖于交互服务接口。它经常通过依赖注入或服务定位器获取对交互服务实现的引用。

在视图模型引用交互服务之后，它可以在必要时以编程方式请求与用户的交互。交互服务实现交互的可视方面，如下图所示。根据用户界面的实现要求，在视图模型中使用接口引用允许使用不同的实现。例如，可以提供WPF的交互服务的实现，允许更多地重用应用程序的表示逻辑。

![使用交互服务与用户交互](/prism/Ch6AdvMVVMFig17.png)

**模态交互**（例如，在执行可以继续之前向用户呈现**MessageBox**或模态弹出窗口以获取特定响应的位置）可以使用阻塞方法调用以同步方式实现，如以下代码示例所示。

```c#
var result = interactionService.ShowMessageBox(
        &#34;Are you sure you want to cancel this operation?&#34;,
        &#34;Confirm&#34;,
        MessageBoxButton.OK );
if (result == MessageBoxResult.Yes)
{
    CancelRequest();
}
```

然而，这种方法的一个缺点是它迫使同步编程模型。另一种异步实现允许视图模型提供回调以在完成交互时执行。以下代码说明了这种方法。

**非模态交互**

```cs
interactionService.ShowMessageBox(
    &#34;Are you sure you want to cancel this operation?&#34;,
    &#34;Confirm&#34;,
    MessageBoxButton.OK,
    result =&gt;
    {
        if (result == MessageBoxResult.Yes)
        {
            CancelRequest();
        }
    });
```

通过允许实现模态和非模态交互，异步方法在实现交互服务时提供了更大的灵活性。例如，在WPF中，**MessageBox**类可用于实现与用户的真正模态交互。

#### 使用交互请求对象

在MVVM模式中实现简单用户交互的另一种方法是允许视图模型通过与视图中的行为耦合的交互请求对象直接向视图本身发出交互请求。交互请求对象封装交互请求的详细信息及其响应，并通过事件与视图进行通信。视图订阅这些事件以启动交互的用户体验部分。视图通常会将交互的用户体验封装在与视图模型提供的交互请求对象数据绑定的行为中，如下图所示。

![使用交互请求对象与用户交互](/prism/Ch6AdvMVVMFig18.png)

这种方法提供了一种简单但灵活的机制，可以保持视图模型和视图之间的清晰分离 - 它允许视图模型封装应用程序的表示逻辑，包括任何所需的用户交互，同时允许视图完全封装视觉互动的各个方面。可以轻松测试视图模型的实现，包括通过视图与用户的预期交互，并且UI设计器在通过使用封装不同用户的不同行为选择如何在视图中实现交互时具有很大的灵活性互动的经验。

此方法与MVVM模式一致，使视图能够反映它在视图模型上观察到的状态更改，并使用双向数据绑定来实现两者之间的数据通信。在交互请求对象中封装交互的非可视元素，以及使用相应的行为来管理交互的可视元素，与命令对象和命令行为的使用方式非常相似。

这种方法是Prism采用的方法。Prism Library通过**IInteractionRequest**接口和**InteractionRequest &lt;T&gt;**类直接支持此模式。所述**IInteractionRequest**接口定义的事件来发起的相互作用。视图中的行为绑定到此接口并订阅它公开的事件。所述**InteractionRequest &lt;T&gt;**类实现**IInteractionRequest**接口和定义了两个**Raise**方法来允许视图模型发起交互，并指定为请求的上下文中，并且任选地，一个回调委托。

#### 从ViewModel启动交互请求

所述**InteractionRequest &lt;T&gt;**类坐标的交互请求期间视图模型的与该视图的相互作用。所述**抬起**方法允许视图模型来启动交互和指定一个上下文对象，交互完成后调用的回调方法。上下文对象允许视图模型将数据和状态传递给视图，以便在与用户交互期间使用它。如果指定了回调方法，则上下文对象将被传递回视图模型; 这允许用户在交互期间进行的任何更改都传递回视图模型。

```c#
public interface IInteractionRequest
{
    event EventHandler&lt;InteractionRequestedEventArgs&gt; Raised;
}

public class InteractionRequest&lt;T&gt; : IInteractionRequest
    where T : INotification
{
    public event EventHandler&lt;InteractionRequestedEventArgs&gt; Raised;

    public void Raise(T context)
    {
        this.Raise(context, c =&gt; { });
    }

    public void Raise(T context, Action&lt;T&gt; callback)
    {
        var handler = this.Raised;
        if (handler != null)
        {
            handler(
            this,
            new InteractionRequestedEventArgs(
                context,
                () =&gt; { if (callback != null) callback(context); } ));
        }
    }
}
```

Prism提供预定义的上下文类，支持常见的交互请求场景。该**INotification**接口用于所有上下文对象。当交互请求用于通知用户应用程序中的重要事件时。它提供了两个属性 - **Title**和**Content**- 将显示给用户。通常，通知是单向的，因此预计用户不会在交互期间更改这些值。该**通知**类是这个接口的默认实现。

所述**IConfirmation**接口扩展了**INotification**接口，增加了第三属性**Confirmed** -其用于表示用户已确认或拒绝该操作。在**确认**类，所提供的**IConfirmation**实现，用于实现**MessageBox**用户想获得一个Yes/No风格的交互。您可以定义实现**INotification**接口的自定义上下文类，以封装支持交互所需的任何数据和状态。

要使用**InteractionRequest &lt;T&gt;**类，视图模型类将创建**InteractionRequest &lt;T&gt;**类的实例，并定义只读属性以允许视图对其进行数据绑定。当视图模型想要发起请求时，它将调用**Raise**方法，传入上下文对象和（可选）回调委托。

```c#
public InteractionRequestViewModel()
{
    this.ConfirmationRequest = new InteractionRequest&lt;IConfirmation&gt;();
    ...
    // Commands for each of the buttons. Each of these raise a different interaction request.
    this.RaiseConfirmationCommand = new DelegateCommand(this.RaiseConfirmation);
    ...
}

public InteractionRequest&lt;IConfirmation&gt; ConfirmationRequest { get; private set; }

private void RaiseConfirmation()
{
    this.ConfirmationRequest.Raise(
        new Confirmation { Content = &#34;Confirmation Message&#34;, Title = &#34;Confirmation&#34; },
        c =&gt; { InteractionResultMessage = c.Confirmed ? &#34;The user accepted.&#34; : &#34;The user cancelled.&#34;; });
}
```

[交互性快速入门](https://msdn.microsoft.com/en-us/library/ff921081(v=pandp.40).aspx)示出了如何**IInteractionRequest**接口和**InteractionRequest &lt;T&gt;**类用于实现视图和视图模型（参见InteractionRequestViewModel.cs）之间的用户交互。

#### 使用Behaviors实现交互用户体验

由于交互请求对象表示逻辑交互，因此在视图中定义了交互的确切用户体验。行为通常用于封装用户体验以进行交互; 这允许UI设计者选择适当的行为并将其绑定到视图模型上的交互请求对象。

必须设置视图以检测交互请求事件，然后为请求显示适当的可视界面。触发器用于在引发特定事件时启动操作。

Blend提供的标准**EventTrigger**可用于通过绑定到视图模型公开的交互请求对象来监视交互请求事件。但是，Prism Library定义了一个名为**InteractionRequestTrigger**的自定义**EventTrigger**，它自动连接到**IInteractionRequest**接口的相应**Raised**事件。这减少了所需的可扩展应用程序标记语言（XAML）的数量，并减少了无意中输入错误事件名称的可能性。

引发事件后，**InteractionRequestTrigger**将调用指定的操作。对于WPF，Prism Library提供**PopupWindowAction**类，该类向用户显示弹出窗口。显示窗口时，其数据上下文设置为交互请求的上下文参数。使用**PopupWindowAction**类的**WindowContent**属性，可以指定将在弹出窗口中显示的视图。弹出窗口的标题绑定到上下文对象的Title属性。

注意：默认情况下，**PopupWindowAction**类显示的特定弹出窗口类型取决于上下文对象的类型。对于**Notification**上下文对象，将显示**DefaultNotificationWindow**，而对于**Confirmation**上下文对象，将显示**DefaultConfirmationWindow**。该**DefaultNotificationWindow**显示一个简单的弹出窗口显示通知，而**DefaultConfirmationWindow**还包括**接受**和**取消**按钮来捕获用户的响应。您可以通过使用**WindowContent**指定自定义弹出窗口来覆盖此行为**PopupWindowAction**类的属性。*

以下示例显示了如何使用**InteractionRequestTrigger**和**PopupWindowAction**在Interactivity QuickStart中向用户显示确认弹出窗口。

```xaml
&lt;i:Interaction.Triggers&gt;
    &lt;prism:InteractionRequestTrigger SourceObject=&#34;{Binding ConfirmationRequest, Mode=OneWay}&#34;&gt;
        &lt;prism:PopupWindowAction IsModal=&#34;True&#34; CenterOverAssociatedObject=&#34;True&#34;/&gt;
    &lt;/prism:InteractionRequestTrigger&gt;
&lt;/i:Interaction.Triggers&gt;
```

注：该**PopupWindowAction**有三个重要特性，**IsModal**其弹出时模态设置为true; **CenterOverAssociatedObject**，当设置为true时，显示以父窗口为中心的弹出窗口。最后，未指定**WindowContent**属性，因此将显示**DefaultConfirmationWindow**。*

所述**PopupWindowAction**设置**Notification**对象作为的数据上下文**DefaultNotificationWindow**，它显示**Notification**对象的**Content**属性。在用户关闭弹出窗口后，通过回调方法将上下文对象与任何更新的值一起传递回视图模型。在Interactivity QuickStart的确认示例中，**DefaultConfirmationWindow**负责在单击“ **OK”**按钮时将**Confirmation**对象上的**Confirmed**属性设置为**true**。

可以定义不同的触发器和动作以支持其他交互机制。Prism **InteractionRequestTrigger**和**PopupWindowAction**类的实现可以用作开发自己的触发器和操作的基础。

### Advanced Construction and Wire-Up

要成功实现MVVM模式，您需要完全理解视图，模型和视图模型类的职责，以便您可以在正确的类中实现应用程序的代码。实现正确的模式以允许这些类进行交互（通过数据绑定，命令，交互请求等）也是一个重要的要求。最后一步是考虑如何在运行时实例化视图，视图模型和模型类并将它们相互关联。

如果在应用程序中使用依赖项注入容器，则选择适当的策略来管理此步骤尤为重要。托管可扩展性框架（MEF）和Unity应用程序块（Unity）都提供了指定视图，视图模型和模型类之间的依赖关系以及在运行时由容器实现它们的能力。

通常，您将视图模型定义为视图的依赖项，以便在构造视图时（使用容器）它自动实例化所需的视图模型。反过来，视图模型所依赖的任何组件或服务也将由容器实例化。成功实例化视图模型后，视图会将其设置为其数据上下文。

#### 使用MEF创建视图和视图模型

使用MEF，您可以使用**import**属性指定视图对视图模型的依赖性，并且可以指定要通过**export**属性实例化的具体视图模型类型。您可以通过属性或构造函数参数将视图模型导入视图。

例如，StockTrader参考实现中的**Shell**视图声明了视图模型的只写属性以及**import**属性。实例化视图时，MEF会创建相应导出视图模型的实例并设置属性值。setter属性将视图模型指定为视图的数据上下文，如此处所示。

```c#
[Import]
ShellViewModel ViewModel
{
    set { this.DataContext = value; }
}
```

定义并导出视图模型，如此处所示。

```c#
[Export]
public class ShellViewModel : BindableBase
{
   ...
}
```

另一种方法是在视图上定义导入构造函数，如此处所示。

```c#
public Shell()
{
    InitializeComponent();
}

[ImportingConstructor]
public Shell(ShellViewModel viewModel) : this()
{
    this.DataContext = viewModel;
}
```

然后，视图模型将由MEF实例化，并作为参数传递给视图的构造函数。

注意：您可以在MEF和Unity中使用属性注入或构造函数注入; 但是，您可能会发现属性注入更简单，因为您不必维护两个构造函数。设计时工具（如Visual Studio和Expression Blend）要求控件具有默认的无参数构造函数，以便在设计器中显示它们。您定义的任何其他构造函数应确保调用默认构造函数，以便可以通过**InitializeComponent**方法正确初始化视图。*

#### 使用Unity创建视图和视图模型

使用Unity作为依赖注入容器与使用MEF类似，并且支持基于属性和基于构造函数的注入。主要区别在于通常不会在运行时隐式发现类型; 相反，他们必须在容器中注册。

通常，您在视图模型上定义接口，以便视图模型的特定具体类型可以与视图分离。例如，视图可以通过构造函数参数定义其对视图模型的依赖性，如此处所示。

```c#
public Shell()
{
    InitializeComponent();
}

public Shell(ShellViewModel viewModel) : this()
{
    this.DataContext = viewModel;
}
```

注意：默认的无参数构造函数是允许视图在设计时工具中工作所必需的，例如Visual Studio和Blend for Visual Studio 2013。

或者，您可以在视图上定义只写视图模型属性，如此处所示。Unity将实例化所需的视图模型，并在实例化视图后调用setter属性。

```cs
public Shell()
{
    InitializeComponent();
}

[Dependency]
public ShellViewModel ViewModel
{
    set { this.DataContext = value; }
}
```

视图模型类型已在Unity容器中注册，如此处所示。

```c#
IUnityContainer container;
container.RegisterType&lt;ShellViewModel&gt;();
```

然后可以通过容器实例化视图，如此处所示。

```c#
IUnityContainer container;
var view = container.Resolve&lt;Shell&gt;();
```

#### Creating the View and View Model Using an External Class

通常，您会发现定义控制器或服务类以协调视图的实例化和视图模型类很有用。此方法可以与依赖项注入容器（如MEF或Unity）一起使用，或者在视图显式创建其所需的视图模型时使用。

在您的应用程序中实现导航时，此方法特别有用。在这种情况下，控制器与UI中的占位符控件或区域相关联，并且它协调视图的构造和放置到该占位符或区域中。

例如，服务类可用于使用容器构建视图并在主页面中显示它们。在此示例中，视图由视图名称指定。通过在UI服务上调用**ShowView**方法启动导航，如此简单示例所示。

```c#
private void NavigateToQuestionnaireList()
{
    // Ask the UI service to go to the &#34;questionnaire list&#34; view.
    this.uiService.ShowView(ViewNames.QuestionnaireTemplatesList);
}
```

UI服务与应用程序的UI中的占位符控件相关联; 它封装了所需视图的创建，并在UI中协调其外观。所述**ShowView**所述的**UIService**创建（使得其视图模型和其他依赖可以实现）通过所述容器中的视图的一个实例，并然后显示它在适当的位置，如下所示。

```c#
public void ShowView(string viewName)
{
    var view = this.ViewFactory.GetView(viewName);
    this.MainWindow.CurrentView = view;
}
```

注意：Prism为区域内的导航提供广泛的支持。区域导航使用与前一种方法非常相似的机制，区域管理器负责协调特定区域中视图的实例化和放置。欲了解更多信息，请参见基于视图导航，在导航。

### 测试MVVM应用程序

从MVVM应用程序测试模型和视图模型与测试任何其他类相同，并且可以使用相同的工具和技术 - 例如单元测试和模拟框架。但是，有一些测试模式是典型的模型和视图模型类，可以从标准测试技术和测试助手类中受益。

#### 测试INotifyPropertyChanged实现

实现**INotifyPropertyChanged**接口允许视图对模型和视图模型中发生的更改做出反应。这些更改不仅限于控件中显示的域数据; 它们还用于控制视图，例如视图模型状态，可以启动动画或禁用控件。

#### 简单案例

可以通过将事件处理程序附加到**PropertyChanged**事件并检查在为属性设置新值之后是否引发事件来测试可以由测试代码直接更新的属性。辅助类（如**PropertyChangeTracker**类）可用于附加处理程序并收集结果; 这可以避免编写测试时的重复性任务。以下代码示例显示了使用此类助手类的测试。

```c#
var changeTracker = new PropertyChangeTracker(viewModel);

viewModel.CurrentState = &#34;newState&#34;;

CollectionAssert.Contains(changeTracker.ChangedProperties, &#34;CurrentState&#34;);
```

作为保证**INotifyPropertyChanged**接口实现的代码生成过程的结果的属性（例如模型设计者生成的代码中的属性）通常不需要进行测试。

#### Computed and Non-Settable Properties

当测试代码无法设置属性时 - 例如具有非公共设置器的属性或只读，计算属性 - 测试代码需要激发测试对象导致属性及其相应通知的更改。但是，测试的结构与更简单的情况相同，如下面的代码示例所示，其中模型对象的更改会导致视图模型中的属性发生更改。

```c#
var changeTracker = new PropertyChangeTracker(viewModel);

var question = viewModel.Questions.First() as OpenQuestionViewModel;
question.Question.Response = &#34;some text&#34;;

CollectionAssert.Contains(changeTracker.ChangedProperties, &#34;UnansweredQuestions&#34;);
```

#### Whole Object Notifications

实现**INotifyPropertyChanged**接口时，允许对象使用null或空字符串作为已更改的属性名称引发**PropertyChanged**事件，以指示对象中的所有属性可能已更改。这些案例可以像通知各个属性名称的案例一样进行测试。

### 测试INotifyDataErrorInfo实现

有几种机制可用于使绑定能够执行输入验证，例如在设置属性时抛出异常，实现**IDataErrorInfo**接口以及实现**INotifyDataErrorInfo**接口。实现**INotifyDataErrorInfo**接口允许更复杂，因为它支持指示每个属性的多个错误并执行异步和跨属性验证; 因此，它也需要最多的测试。

测试**INotifyDataErrorInfo**实现有两个方面：测试验证规则是否正确实现，并测试接口实现的要求，例如当**GetErrors**方法的结果不同时引发**ErrorsChanged**事件。

#### 测试验证规则

验证逻辑通常很容易测试，因为它通常是一个自包含的过程，其输出取决于输入。对于与验证规则关联的每个属性，应该对使用有效值，无效值，边界值等的验证属性名称调用**GetErrors**方法的结果进行测试。如果共享验证逻辑，就像使用数据注释的验证属性以声明方式表示验证规则一样，更详尽的测试可以集中在共享验证逻辑上。另一方面，必须彻底测试自定义验证规则。

```c#
// Invalid case
var notifyErrorInfo = (INotifyDataErrorInfo)question;

question.Response = -15;

Assert.IsTrue(notifyErrorInfo.GetErrors(&#34;Response&#34;).Cast&lt;ValidationResult&gt;().Any());

// Valid case
var notifyErrorInfo = (INotifyDataErrorInfo)question;

question.Response = 15;
Assert.IsFalse(notifyErrorInfo.GetErrors(&#34;Response&#34;).Cast&lt;ValidationResult&gt;().Any());
```

跨属性验证规则遵循相同的模式，通常需要更多测试来适应不同属性的值组合。

#### 测试INotifyDataErrorInfo实现的要求

除了为**GetErrors**方法生成正确的值之外，**INotifyDataErrorInfo**接口的实现必须确保正确引发**ErrorsChanged**事件，例如**GetErrors**的结果不同时。此外，**HasErrors**属性必须反映实现该接口的对象的整体错误状态。

实现**INotifyDataErrorInfo**接口没有强制方法。但是，依赖于累积验证错误并执行必要通知的对象的实现通常是首选，因为它们更易于测试。这是因为没有必要验证每个验证属性上的每个验证规则是否满足**INotifyDataErrorInfo**接口的所有成员的要求（当然，因为错误管理对象已经过适当测试）。

测试接口要求至少应包括以下验证：

- 该**HasErrors**属性反映对象的整体错误状态。如果其他属性仍具有无效值，则为此属性设置有效值不会导致此属性发生更改。
- 所述**ErrorsChanged**当通过在结果为一个变化反映了一个属性更改错误状态，事件被引发**GetErrors**方法。错误状态更改可能从有效状态（即无错误）变为无效状态，反之亦然，或者它可能从无效状态变为不同的无效状态。**GetErrors**的更新结果可用于**ErrorsChanged**事件的处理程序。

在测试**INotifyPropertyChanged**接口的实现时，辅助类（例如MVVM示例项目中的**NotifyDataErrorInfoTestHelper**类）通常通过处理重复的内务操作和标准检查来更轻松地为**INotifyDataErrorInfo**接口的实现编写测试。在实现接口时，它们特别有用，而不依赖于某种可重用的错误管理器。以下代码示例显示了此类型的帮助程序类。

```c#
var helper =
    new NotifyDataErrorInfoTestHelper&lt;NumericQuestion, int?&gt;(
        question,
        q =&gt; q.Response);

helper.ValidatePropertyChange(
    6,
    NotifyDataErrorInfoBehavior.Nothing);

helper.ValidatePropertyChange(
    20,
    NotifyDataErrorInfoBehavior.FiresErrorsChanged
    | NotifyDataErrorInfoBehavior.HasErrors
    | NotifyDataErrorInfoBehavior.HasErrorsForProperty);

helper.ValidatePropertyChange(
    null,
    NotifyDataErrorInfoBehavior.FiresErrorsChanged
    | NotifyDataErrorInfoBehavior.HasErrors
    | NotifyDataErrorInfoBehavior.HasErrorsForProperty);

helper.ValidatePropertyChange(
    2,
    NotifyDataErrorInfoBehavior.FiresErrorsChanged);
```

#### 测试异步服务调用

在实现MVVM模式时，视图模型通常会异步调用服务上的操作。对调用这些操作的代码的测试通常使用模拟或存根作为实际服务的替代。

用于实现异步操作的标准模式提供了有关线程的不同保证，在该线程中发生有关操作状态的通知。虽然[基于事件的异步设计模式](http://msdn.microsoft.com/en-us/library/wewwczdw.aspx)保证在适合应用程序的线程上调用事件的处理程序，但[IAsyncResult设计模式](http://msdn.microsoft.com/en-us/library/ms228963.aspx)不提供任何此类保证，强制发起调用的视图模型代码以确保任何更改会影响视图发布到UI线程。

处理线程问题需要更复杂，因此通常更难以测试代码。它通常还要求测试本身是异步的。当保证在UI线程中发生通知时，或者因为使用了基于标准事件的异步模式，或者因为视图模型依赖于服务访问层来将通知编组到适当的线程，所以可以简化测试并且可以基本上发挥作用一个“UI线程的调度程序”。

服务被模拟的方式取决于用于实现其操作的异步事件模式。如果使用基于方法的模式，则使用标准模拟框架创建的服务接口的模拟通常就足够了，但如果使用基于事件的模式，则基于实现添加和删除处理程序的方法的自定义类进行模拟对于服务事件通常是首选。

以下代码示例显示了对使用mocks for services在UI线程中通知的异步操作成功完成时的相应行为的测试。在此示例中，测试代码在进行异步服务调用时捕获视图模型提供的回调。然后，测试通过调用回调来模拟测试后期调用的完成。此方法允许测试使用异步服务的组件，而不会使测试异步。

```c#
questionnaireRepositoryMock
    .Setup(
        r =&gt;
            r.SubmitQuestionnaireAsync(
                It.IsAny&lt;Questionnaire&gt;(),
                It.IsAny&lt;Action&lt;IOperationResult&gt;&gt;()))
    .Callback&lt;Questionnaire, Action&lt;IOperationResult&gt;&gt;(
        (q, a) =&gt; callback = a);

uiServiceMock
    .Setup(svc =&gt; svc.ShowView(ViewNames.QuestionnaireTemplatesList))
    .Callback&lt;string&gt;(viewName =&gt; requestedViewName = viewName);

submitResultMock
    .Setup(sr =&gt; sr.Error)
    .Returns&lt;Exception&gt;(null);

CompleteQuestionnaire(viewModel);
viewModel.Submit();

// Simulate callback posted to the UI thread.
callback(submitResultMock.Object);

// Check expected behavior – request to navigate to the list view.
Assert.AreEqual(ViewNames.QuestionnaireTemplatesList, requestedViewName);
```

***注意：**使用此测试方法仅执行被测对象的功能; 它不测试代码是否是线程安全的。*



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/07/prism10-advancemvvm/  

