# Prism MVVM


注意：该版本为Prism6,最新版已有较大改动。

Model-View-ViewModel（MVVM）模式可帮助您将应用程序的业务和表示逻辑与其用户界面（UI）完全分离。在应用程序逻辑和UI之间保持清晰的分离有助于解决许多开发和设计问题，并使您的应用程序更容易测试，维护和发展。它还可以极大地改善代码重用机会，并允许开发人员和UI设计人员在开发应用程序的各个部分时更轻松地进行协作。

使用MVVM模式，应用程序的UI以及底层表示和业务逻辑被分成三个独立的类：视图(view)，它封装了UI和UI逻辑; 视图模型(viewmodel)，它封装了表示逻辑和状态; 模型（model）封装了应用程序的业务逻辑和数据。

Prism包含示例和参考实现，展示如何在Windows Presentation Foundation（WPF）应用程序中实现MVVM模式。Prism Library还提供了可以帮助您在自己的应用程序中实现模式的功能。这些功能体现了实现MVVM模式的最常见实践，旨在支持可测试性并与Expression Blend和Visual Studio配合使用。

### Class Responsibilities and Characteristics

类的责任和特征

MVVM模式是Presentation Model模式的近似变体，经过优化以利用WPF的一些核心功能，例如数据绑定，数据模板，命令和行为。

在MVVM模式中，视图封装了UI和任何UI逻辑，视图模型封装了表示逻辑和状态，模型封装了业务逻辑和数据。视图通过数据绑定，命令和更改通知事件与视图模型交互。视图模型查询，观察和协调模型的更新，转换，验证和聚合数据，以便在视图中显示。

下图显示了三个MVVM类及其交互。



![image-20240701162241711](/prism/image-20240701162241711.png)

#### MVVM类及其交互

与所有分离的表示模式一样，有效使用MVVM模式的关键在于理解将应用程序代码分解为正确类的适当方式，以及理解这些类在各种场景中交互的方式。以下部分描述了MVVM模式中每个类的职责和特征。

#### 视图类

视图的职责是定义用户在屏幕上看到的内容的结构和外观。理想情况下，视图的代码隐藏只包含一个调用**InitializeComponent**方法的构造函数。在某些情况下，代码隐藏可能包含UI逻辑代码，该代码实现了在可扩展应用程序标记语言（XAML）中表达难以或低效的视觉行为，例如复杂的动画，或者当代码需要直接操作视觉元素时部分观点。您不应该在视图中放置任何需要进行单元测试的逻辑代码。通常，视图的代码隐藏中的逻辑代码将通过UI自动化测试方法进行测试。

在WPF中，视图中的数据绑定表达式将根据其数据上下文进行评估。在MVVM中，视图的数据上下文设置为视图模型。视图模型实现视图可以绑定的属性和命令，并通过更改通知事件通知视图状态的任何更改。视图与其视图模型之间通常存在一对一的关系。

通常，视图是**Control- derived**或**UserControl**派生的类。但是，在某些情况下，视图可以由数据模板表示，该数据模板指定用于在显示对象时可视地表示对象的UI元素。使用数据模板，可视化设计人员可以轻松定义视图模型的呈现方式，也可以修改其默认的可视化表示，而无需更改底层对象本身或用于显示它的控件的行为。

可以将数据模板视为没有任何代码隐藏的视图。它们旨在绑定到特定的视图模型类型，只要需要在UI中显示一个。在运行时，将自动实例化由数据模板定义的视图，并将其数据上下文设置为相应的视图模型。

在WPF中，您可以在应用程序级别将数据模板与视图模型类型相关联。然后，无论何时在UI中显示，WPF都会自动将数据模板应用于指定类型的任何视图模型对象。这称为隐式数据模板。数据模板可以与使用它的控件一起定义，也可以在父视图外的资源字典中定义，并以声明方式合并到视图的资源字典中。

总而言之，该视图具有以下主要特征：

- 视图是可视元素，例如窗口，页面，用户控件或数据模板。视图定义视图中包含的控件及其可视布局和样式。
- 视图通过其**DataContext**属性引用视图模型。视图中的控件是绑定到视图模型公开的属性和命令的数据。
- 视图可以自定义视图和视图模型之间的数据绑定行为。例如，视图可以使用值转换器来格式化要在UI中显示的数据，或者它可以使用验证规则来向用户提供额外的输入数据验证。
- 视图定义并处理UI视觉行为，例如可以从视图模型中的状态更改或通过用户与UI的交互触发的动画或过渡。
- 视图的代码隐藏可以定义UI逻辑以实现在XAML中难以表达的视觉行为，或者需要直接引用视图中定义的特定UI控件。

#### 视图模型类

MVVM模式中的视图模型封装了视图的表示逻辑和数据。它没有直接引用视图或有关视图的特定实现或类型的任何知识。视图模型实现视图可以绑定数据的属性和命令，并通过更改通知事件通知视图任何状态更改。视图模型提供的属性和命令定义UI提供的功能，但视图确定如何呈现该功能。

视图模型负责协调视图与所需的任何模型类的交互。通常，视图模型和模型类之间存在一对多关系。视图模型可以选择直接将模型类公开给视图，以便视图中的控件可以直接将数据绑定到它们。在这种情况下，需要设计模型类以支持数据绑定和相关的更改通知事件。有关此方案的详细信息，请参阅本主题后面的“ [数据绑定](https://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#data-binding) ”一节。

视图模型可以转换或操纵模型数据，以便视图可以轻松地使用它。视图模型可以定义其他属性以专门支持视图; 这些属性通常不属于模型的一部分（或不能添加到模型中）。例如，视图模型可以组合两个字段的值以使视图更容易呈现，或者它可以计算具有最大长度的字段的输入剩余字符数。视图模型还可以实现数据验证逻辑以确保数据一致性。

视图模型还可以定义视图可以用于在UI中提供视觉变化的逻辑状态。视图可以定义反映视图模型状态的布局或样式更改。例如，视图模型可以定义指示数据异步提交到Web服务的状态。视图可以在此状态期间显示动画，以向用户提供视觉反馈。

通常，视图模型将定义可在UI中表示并且用户可以调用的命令或操作。一个常见示例是视图模型提供允许用户将数据提交到Web服务或数据存储库的**Submit**命令。视图可以选择用按钮表示该命令，以便用户可以单击按钮来提交数据。通常，当命令变得不可用时，其关联的UI表示将被禁用。命令提供了一种封装用户操作并将其与UI中的可视表示清晰分离的方法。

总而言之，视图模型具有以下关键特征：

- 视图模型是非可视类，不是从任何WPF基类派生的。它封装了支持应用程序中的用例或用户任务所需的表示逻辑。视图模型可以独立于视图和模型进行测试。
- 视图模型通常不直接引用视图。它实现了视图可以绑定数据的属性和命令。它通过**INotifyPropertyChanged**和**INotifyCollectionChanged**接口通过更改通知事件通知视图任何状态更改。
- 视图模型协调视图与模型的交互。它可以转换或操作数据，以便视图可以轻松使用它，并可以实现模型上可能不存在的其他属性。它还可以通过**IDataErrorInfo**或**INotifyDataErrorInfo**接口实现数据验证。
- 视图模型可以定义视图可以在视觉上向用户表示的逻辑状态。

**注意：查看或查看模型？** 很多时候，确定应该实现某些功能的地方并不明显。一般的经验法则是：任何与屏幕上UI的特定视觉外观有关并且可以在以后重新设置样式的内容（即使您当前没有计划重新设置样式）也应该进入视图; 对应用程序的逻辑行为很重要的任何内容都应该进入视图模型。此外，由于视图模型应该不具有视图中特定可视元素的明确知识，因此以编程方式操作视图中的可视元素的代码应驻留在视图的代码隐藏中或封装在行为中。同样，检索或操作要通过数据绑定在视图中显示的数据项的代码应驻留在视图模型中。

#### 模型类

MVVM模式中的模型封装了业务逻辑和数据。业务逻辑被定义为与应用程序数据的检索和管理有关的任何应用程序逻辑，并确保强制执行确保数据一致性和有效性的任何业务规则。为了最大化重用机会，模型不应包含任何特定于用例或特定于用户任务的行为或应用程序逻辑。

通常，模型表示应用程序的客户端域模型。它可以基于应用程序的数据模型和任何支持业务和验证逻辑来定义数据结构。该模型还可以包括支持数据访问和缓存的代码，尽管通常使用单独的数据存储库或服务。通常，模型和数据访问层是作为数据访问或服务策略的一部分生成的，例如ADO.NET实体框架，WCF数据服务或WCF RIA服务。

通常，模型实现了可以轻松绑定到视图的工具。这通常意味着它通过**INotifyPropertyChanged**和**INotifyCollectionChanged**接口支持属性和集合更改通知。表示对象集合的模型类通常派生自**ObservableCollection &lt;T&gt;**类，该类提供**INotifyCollectionChanged**接口的实现。

该模型还可以通过**IDataErrorInfo**（或**INotifyDataErrorInfo**）接口支持数据验证和错误报告。该**IDataErrorInfo的**和**INotifyDataErrorInfo**接口使WPF数据绑定时通知值发生改变，这样的UI可以更新。它们还支持UI层中的数据验证和错误报告。

**注意：如果您的模型类没有实现所需的接口，该怎么办？** 有时您需要使用未实现**INotifyPropertyChanged**，**INotifyCollectionChanged**，**IDataErrorInfo**或**INotifyDataErrorInfo**接口的模型对象。在这些情况下，视图模型可能需要包装模型对象并将所需的属性公开给视图。这些属性的值将由模型对象直接提供。视图模型将为它公开的属性实现所需的接口，以便视图可以轻松地将数据绑定到它们。

该模型具有以下主要特征：

- 模型类是非可视类，它封装了应用程序的数据和业务逻辑。他们负责管理应用程序的数据，并通过封装所需的业务规则和数据验证逻辑来确保其一致性和有效性。
- 模型类不直接引用视图或视图模型类，也不依赖于它们的实现方式。
- 模型类通常通过**INotifyPropertyChanged**和**INotifyCollectionChanged**接口提供属性和集合更改通知事件。这允许它们在视图中容易地数据绑定。表示对象集合的模型类通常派生自**ObservableCollection &lt;T&gt;**类。
- 模型类通常通过**IDataErrorInfo**或**INotifyDataErrorInfo**接口提供数据验证和错误报告。
- 模型类通常与封装数据访问和缓存的服务或存储库结合使用。

#### View,ViewModel,Model互动

MVVM模式通过将每个应用程序的用户界面，其表示逻辑以及业务逻辑和数据分离为单独的类，提供了清晰的分离。因此，在实现MVVM时，重要的是将应用程序的代码分解为正确的类，如上一节所述。

精心设计的视图，视图模型和模型类不仅会封装正确类型的代码和行为; 它们的设计也使它们可以通过数据绑定，命令和数据验证接口轻松地相互交互。

视图与其视图模型之间的交互可能是最重要的考虑因素，但模型类和视图模型之间的交互也很重要。以下部分描述了这些交互的各种模式，并描述了在应用程序中实现MVVM模式时如何设计它们。

### 数据绑定

数据绑定在MVVM模式中起着非常重要的作用。WPF提供强大的数据绑定功能。您的视图模型和（理想情况下）您的模型类应设计为支持数据绑定，以便它们可以利用这些功能。通常，这意味着它们必须实现正确的接口。

WPF数据绑定支持多种数据绑定模式。通过单向数据绑定，可以将UI控件绑定到视图模型，以便在呈现显示时它们反映基础数据的值。当用户在UI中修改基础数据时，双向数据绑定也将自动更新基础数据。

为确保在视图模型中数据发生更改时UI保持最新，它应实现相应的更改通知界面。如果它定义了可以绑定数据的属性，它应该实现**INotifyPropertyChanged**接口。如果视图模型**表示集合**，则它应实现**INotifyCollectionChanged**接口，或者从提供此接口实现的**ObservableCollection &lt;T&gt;**类派生。这两个接口都定义了每当基础数据发生更改时引发的事件。引发这些事件时，将自动更新任何数据绑定控件。

在许多情况下，视图模型将定义返回对象的属性（反过来，可以定义返回其他对象的属性）。WPF数据绑定支持通过**Path**属性绑定到嵌套属性。因此，视图的视图模型返回对其他视图模型或模型类的引用是很常见的。视图可访问的所有视图模型和模型类应根据需要实现**INotifyPropertyChanged**或**INotifyCollectionChanged**接口。

以下部分描述了如何实现所需的接口以支持MVVM模式中的数据绑定。

#### 实现INotifyPropertyChanged

在视图模型或模型类中实现**INotifyPropertyChanged**接口允许它们在基础属性值更改时向视图中的任何数据绑定控件提供更改通知。实现此接口非常简单，如以下代码示例所示。

```c#
public class Questionnaire : INotifyPropertyChanged
{
    private string favoriteColor;
    public event PropertyChangedEventHandler PropertyChanged;
    ...
    public string FavoriteColor
    {
        get { return this.favoriteColor; }
        set
        {
            if (value != this.favoriteColor)
            {
                this.favoriteColor = value;

                var handler = this.PropertyChanged;
                if (handler != null)
                {
                    handler(this,
                            new PropertyChangedEventArgs(&#34;FavoriteColor&#34;));
                }
            }
        }
    }
}
```

由于需要在event参数中指定属性名称，因此在许多视图模型类上实现**INotifyPropertyChanged**接口可能是重复且容易出错的。Prism库提供了BindableBase基类，您可以从中派生以类型安全的方式实现**INotifyPropertyChanged**接口的视图模型类，如此处所示。

```c#
public abstract class BindableBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;
    ...
    protected virtual bool SetProperty&lt;T&gt;(ref T storage, T value,
                            [CallerMemberName] string propertyName = null)
    {...}
    protected void OnPropertyChanged&lt;T&gt;(
                            Expression&lt;Func&lt;T&gt;&gt; propertyExpression)
    {...}

    protected void OnPropertyChanged(string propertyName)
    {...}
}
```

派生视图模型类可以通过调用**SetProperty**方法在setter中引发属性更改事件。所述的**SetProperty**方法检查被设定的值支持字段是否是不同的。如果不同，则更新后备字段并引发**PropertyChanged**事件。

下面的代码示例演示如何设置属性，并通过在**OnPropertyChanged**方法中使用lambda表达式同时发出另一个属性的更改。此示例来自Stock Trader RI。该**TransactionInfo**和**TickerSymbol**属性相关。如果**TransactionInfo**属性更改，则**TickerSymbol**也可能会更新。通过调用**OnPropertyChanged**的**TickerSymbol**中的setter属性**TransactionInfo**财产，二的**PropertyChanged**事件将提高，一个用于**TransactionInfo**，一个用于**TickerSymbol**。

```c#
public TransactionInfo TransactionInfo
{
    get { return this.transactionInfo; }
    set
    {
        SetProperty(ref this.transactionInfo, value);
        this.OnPropertyChanged(() =&gt; this.TickerSymbol);
    }
}
```

**注意：**以这种方式使用lambda表达式会产生很小的性能成本，因为必须为每次调用计算lambda表达式。好处是，如果重命名属性，此方法可提供编译时类型安全性和重构支持。虽然性能成本很低，并且通常不会影响您的应用程序，但如果您有许多更改通知，则会产生成本。在这种情况下，您应该考虑使用非lambda方法重载。

通常，模型或视图模型将包含其值从模型或视图模型中的其他属性计算的属性。处理属性更改时，请务必同时为任何计算属性引发通知事件。

#### 实现INotifyCollectionChanged

您的视图模型或模型类可以表示项的集合，也可以定义一个或多个返回项集合的属性。在任何一种情况下，您可能希望在**ItemsControl中**显示集合，例如**ListBox**，或者在视图中的**DataGrid**控件中。这些控件可以是绑定到视图模型的数据，该视图模型表示集合或通过**ItemSource**属性返回集合的属性。

```xml
    &lt;DataGrid ItemsSource=&#34;{Binding Path=LineItems}&#34; /&gt;
```

为了正确支持更改通知请求，视图模型或模型类（如果它表示集合）应实现**INotifyCollectionChanged**接口（除了**INotifyPropertyChanged**接口）。如果视图模型或模型类定义了返回对集合的引用的属性，则返回的集合类应实现**INotifyCollectionChanged**接口。

但是，实现**INotifyCollectionChanged**接口可能具有挑战性，因为它必须在集合中添加，删除或更改项目时提供通知。它不是直接实现接口，而是通常更容易使用或派生自已实现它的集合类。所述**的ObservableCollection &lt;T&gt;**类提供这个接口的实现和通常用作任一个基类或执行该代表项的集合的性质。

如果需要为视图提供数据绑定的集合，并且不需要跟踪用户的选择或支持对集合中项目的过滤，排序或分组，则只需在视图模型上定义属性即可返回对**ObservableCollection &lt;T&gt;**实例的引用。

```c#
public class OrderViewModel : BindableBase
{
    public OrderViewModel( IOrderService orderService )
    {
        this.LineItems = new ObservableCollection&lt;OrderLineItem&gt;(
                                orderService.GetLineItemList() );
    }

    public ObservableCollection&lt;OrderLineItem&gt; LineItems { get; private set; }
}
```

如果获得对集合类的引用（例如，来自未实现**INotifyCollectionChanged的**其他组件或服务），则通常可以使用其中一个构造函数将该集合包装在**ObservableCollection &lt;T&gt;**实例中，该构造函数采用**IEnumerable &lt;T&gt;**或**List &lt;T&gt;**参数。

**注意：BindableBase**可以在Prism.Mvvm命名空间中找到，该命名空间位于Prism.Core NuGet包中。

#### 实现ICollectionView

上面的代码示例演示如何实现一个简单的视图模型属性，该属性返回可以通过视图中的数据绑定控件显示的项集合。由于**ObservableCollection &lt;T&gt;**类实现了**INotifyCollectionChanged**接口，因此在添加或删除项目时，视图中的控件将自动更新以反映集合中的当前项目列表。

但是，您通常需要更精细地控制项目集合在视图中的显示方式，或者在视图模型本身内跟踪用户与显示的项目集合的交互。例如，您可能需要根据视图模型中实现的表示逻辑来过滤或排序项目集合，或者您可能需要跟踪视图中当前选定的项目，以便在视图模型中实现命令可以对当前选定的项目采取行动。

WPF通过提供实现**ICollectionView**接口的各种类来支持这些场景。此接口提供允许对集合进行过滤，排序或分组的属性和方法，并允许跟踪或更改当前选定的项目。WPF使用**ListCollectionView**类提供此接口的实现。

集合视图类通过包装基础项目集合来工作，以便它们可以为它们提供自动选择跟踪和排序，过滤和分页。可以使用**CollectionViewSource**类在XAML中以编程方式或声明方式创建这些类的实例。

**注意：**在WPF中，只要控件绑定到集合，就会自动创建默认集合视图。

视图模型可以使用集合视图类来跟踪底层集合的重要状态信息，同时保持视图中的UI与模型中的基础数据之间的关注点的清晰分离。实际上，**CollectionViews**是专为支持集合而设计的视图模型。

因此，如果需要在视图模型中对集合中的项目进行过滤，排序，分组或选择跟踪，则视图模型应为要向视图公开的每个集合创建集合视图类的实例。然后，您可以使用视图模型中集合视图类提供的方法订阅选择更改的事件（例如**CurrentChanged**事件）或控制过滤，排序或分组。

视图模型应该实现一个只读属性，该属性返回**ICollectionView**引用，以便视图中的控件可以将数据绑定到集合视图对象并与之交互。从ItemsControl基类派生的所有WPF控件都可以自动与ICollectionView类交互。

以下代码示例显示了在WPF中使用**ListCollectionView**来跟踪当前选定的客户。

```c#
public class MyViewModel : BindableBase
{
    public ICollectionView Customers { get; private set; }

    public MyViewModel( ObservableCollection&lt;Customer&gt; customers )
    {
        // Initialize the CollectionView for the underlying model
        // and track the current selection.
        Customers = new ListCollectionView( customers );

        Customers.CurrentChanged &#43;=SelectedItemChanged;
    }

    private void SelectedItemChanged( object sender, EventArgs e )
    {
        Customer current = Customers.CurrentItem as Customer;
        ...
    }
    ...
}
```

在视图中，您可以通过**ItemsSource**属性将**ItemsControl**（如**ListBox）**绑定到视图模型上的**Customers**属性，如此处所示。

```xaml
&lt;ListBox ItemsSource=&#34;{Binding Path=Customers}&#34;&gt;
    &lt;ListBox.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text=&#34;{Binding Path=Name}&#34;/&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ListBox.ItemTemplate&gt;
&lt;/ListBox&gt;
```

当用户在UI中选择客户时，将通知视图模型，以便它可以应用与当前所选客户相关的命令。视图模型还可以通过调用集合视图对象上的方法以编程方式更改UI中的当前选择，如以下代码示例所示。

```c#
Customers.MoveCurrentToNext();
```

当选择在集合视图中更改时，UI会自动更新以直观地表示项目的选定状态。

### 命令

除了提供对要在视图中显示或编辑的数据的访问之外，视图模型还可能定义可由用户执行的一个或多个动作或操作。在WPF中，用户可以通过UI执行的操作或操作通常被定义为命令。命令提供了一种方便的方法来表示可以轻松绑定到UI中的控件的操作或操作。它们封装了实现操作或操作的实际代码，并有助于使其与视图中的实际可视化表示分离。

当用户与视图交互时，用户可以以多种不同的方式直观地表示和调用命令。在大多数情况下，它们是通过鼠标单击调用的，但也可以通过快捷键按下，触摸手势或任何其他输入事件来调用它们。视图中的控件是绑定到视图模型命令的数据，以便用户可以使用控件定义的任何输入事件或手势来调用它们。视图中的UI控件与命令之间的交互可以是双向的。在这种情况下，可以在用户与UI交互时调用该命令，并且可以在启用或禁用基础命令时自动启用或禁用UI。

视图模型可以将命令实现为**命令方法**或**命令对象**（实现**ICommand**接口的对象）。在任何一种情况下，视图与命令的交互都可以以声明方式定义，而不需要在视图的代码隐藏文件中使用复杂的事件处理代码。例如，WPF中的某些控件本身支持命令并提供**Command**属性，该属性可以是绑定到视图模型提供的**ICommand**对象的数据。在其他情况下，命令行为可用于将控件与视图模型提供的命令方法或命令对象相关联。

**注意：**行为是一种功能强大且灵活的可扩展性机制，可用于封装交互逻辑和行为，然后可以与视图中的控件进行声明性关联。命令行为可用于将命令对象或方法与未专门设计用于与命令交互的控件相关联。

以下部分描述了如何在视图中实现命令，命令方法或命令对象，以及如何将它们与视图中的控件相关联。

#### 实现基于任务的委托命令

在许多情况下，命令使用长时间运行的事务调用代码，这些事务无法阻止UI线程。对于这种情况，你应该使用**FromAsyncHandler**的方法**DelegateCommand**类，它创建的新实例**DelegateCommand**从一个异步处理方法。

```c#
// DelegateCommand.cs
public static DelegateCommand FromAsyncHandler(Func&lt;Task&gt; executeMethod, Func&lt;bool&gt; canExecuteMethod)
{
    return new DelegateCommand(executeMethod, canExecuteMethod);
}
```

例如，以下代码显示如何通过指定**SignInAsync**和**CanSignIn**视图模型方法的委托来构造表示登录命令的**DelegateCommand**实例。然后，该命令通过只读属性公开给视图，该属性返回对[**ICommand**](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.input.icommand.aspx)的引用。

```c#
// SignInFlyoutViewModel.cs
public DelegateCommand SignInCommand { get; private set;  }

...
SignInCommand = DelegateCommand.FromAsyncHandler(SignInAsync, CanSignIn);
```

### 实现命令对象

命令对象是实现**ICommand**接口的对象。该接口定义了一个**Execute**方法，它封装了操作本身，以及一个**CanExecute**方法，它指示是否可以在特定时间调用该命令。这两种方法都只使用一个参数作为命令的参数。对命令对象中的操作的实现逻辑的封装意味着它可以更容易地进行单元测试和维护。

实现**ICommand**接口非常简单。但是，您可以在应用程序中轻松使用此接口的许多实现。例如，您可以使用Blend for Visual Studio SDK中的**ActionCommand**类或Prism提供的**DelegateCommand**类。

**注意：DelegateCommand**可以在Prism.Commands命名空间中找到，该命名空间位于Prism.Core NuGet包中。

Prism **DelegateCommand**类封装了两个委托，每个委托引用在视图模型类中实现的方法。它继承自**DelegateCommandBase**类，**该类**通过调用这些委托来实现**ICommand**接口的**Execute**和**CanExecute**方法。您可以在**DelegateCommand**类构造函数中为视图模型方法指定委托，其定义如下。

```c#
// DelegateCommand.cs
public class DelegateCommand&lt;T&gt; : DelegateCommandBase
{
    public DelegateCommand(Action&lt;T&gt; executeMethod,Func&lt;T,bool&gt; canExecuteMethod ): base((o) =&gt; executeMethod((T)o), (o) =&gt; canExecuteMethod((T)o))
    {
        ...
    }
}
```

例如，以下代码示例显示如何通过为**OnSubmit**和**CanSubmit**视图模型方法指定委托来构造表示**Submit**命令的**DelegateCommand**实例。然后，该命令通过只读属性公开给视图，该属性返回对**ICommand**的引用。

```c#
public class QuestionnaireViewModel
{
    public QuestionnaireViewModel()
    {
        this.SubmitCommand = new DelegateCommand&lt;object&gt;(
                                        this.OnSubmit, this.CanSubmit );
    }

    public ICommand SubmitCommand { get; private set; }

    private void OnSubmit(object arg)   {...}
    private bool CanSubmit(object arg)  { return true; }
}
```

当在**DelegateCommand**对象上调用**Execute**方法时，它只是通过您在构造函数中指定的委托将调用转发到视图模型类中的方法。同样，调用**CanExecute**方法时，将调用视图模型类中的相应方法。构造函数中**CanExecute**方法的委托是可选的。如果未指定**委托**，则**DelegateCommand**将始终为**CanExecute**返回**true**。

该**DelegateCommand**类是一个泛型类型。type参数指定传递给**Execute**和**CanExecute**方法的命令参数的类型。在前面的示例中，command参数的类型为**object**。Prism还提供了非泛型版本的**DelegateCommand**类，以便在不需要命令参数时使用。

视图模型可以通过调用**DelegateCommand**对象上的**RaiseCanExecuteChanged**方法来指示命令的**CanExecute**状态的更改。这会导致引发**CanExecuteChanged**事件。UI中绑定到该命令的任何控件都将更新其启用状态以反映绑定命令的可用性。

可以使用**ICommand**接口的其他实现。Expression Blend SDK提供的**ActionCommand**类与前面描述的Prism的**DelegateCommand**类类似，但它仅支持单个**Execute**方法委托。Prism还提供了**CompositeCommand**类，它允许将**DelegateCommands**组合在一起执行。有关使用**CompositeCommand**类的更多信息，请参阅“ [高级MVVM方案](https://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html) ”中的“复合命令” 。

#### 从视图调用命令对象

有许多方法可以将视图中的控件与视图模型提供的命令对象相关联。某些WPF控件，特别是**ButtonBase**派生控件，如**Button**或**RadioButton**，以及**Hyperlink**或**MenuItem**派生控件，可以通过**Command**属性轻松地将数据绑定到命令对象。WPF还支持将视图模型**ICommand**绑定到**KeyGesture**。

```xml
&lt;Button Command=&#34;{Binding Path=SubmitCommand}&#34; CommandParameter=&#34;SubmitOrder&#34;/&gt;
```

也可以使用**CommandParameter**属性选择性地定义命令参数。预期参数的类型在**Execute**和**CanExecute**目标方法中指定。当用户与该控件交互时，控件将自动调用目标命令，并且命令参数（如果提供）将作为参数传递给命令的**Execute**方法。在前面的示例中，按钮将在**单击**时自动调用**SubmitCommand**。此外，如果指定了**CanExecute**处理程序，则在**CanExecute**返回**false时**将自动禁用该按钮，如果返回**true**，将启用它。

另一种方法是使用Blend for Visual Studio 2013交互触发器和**InvokeCommandAction**行为。有关**InvokeCommandAction**行为以及将命令与事件关联的更多信息，请参阅“ [高级MVVM方案](https://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html)”中的“交互触发器和命令” 。

#### 数据验证和错误报告

通常需要您的视图模型或模型来执行数据验证并向视图发出任何数据验证错误信号，以便用户可以采取行动纠正它们。

WPF支持管理更改绑定到视图中控件的各个属性时发生的数据验证错误。对于与控件数据绑定的单个属性，视图模型或模型可以通过拒绝传入的错误值并抛出异常来表示属性设置器中的数据验证错误。如果数据绑定上的**ValidatesOnExceptions**属性为true，则WPF中的数据绑定引擎将处理该异常并向用户显示存在数据验证错误的可视提示。

但是，应尽可能避免以这种方式抛出属性异常。另一种方法是在视图模型或模型类上实现IDataErrorInfo或INotifyDataErrorInfo接口。这些接口允许您的视图模型或模型对一个或多个属性值执行数据验证，并向视图返回错误消息，以便可以通知用户错误。

#### 实现IDataErrorInfo

该**IDataErrorInfo的**接口提供了性能数据验证和错误报告的基本支持。它定义了两个只读属性：一个索引器属性，其属性名称为索引器参数，以及一个**Error**属性。两个属性都返回一个字符串值。

indexer属性允许视图模型或模型类提供特定于命名属性的错误消息。空字符串或空返回值向视图指示已更改的属性值有效。的**错误**属性允许视图模型或模型类，以提供对整个对象的错误消息。但请注意，WPF数据绑定引擎当前不会调用此属性。

所述**IDataErrorInfo的**时首先显示数据绑定属性索引器属性被访问，并且每当它随后被更改。因为为所有更改的属性调用了indexer属性，所以应该小心确保数据验证尽可能快速有效。

将视图中的控件绑定到要通过**IDataErrorInfo**接口**验证的**属性时，请将数据绑定上的**ValidatesOnDataErrors**属性设置为**true**。这将确保数据绑定引擎将请求数据绑定属性的错误信息。

```xaml
&lt;TextBox
    Text=&#34;{Binding Path=CurrentEmployee.Name, Mode=TwoWay, ValidatesOnDataErrors=True, NotifyOnValidationError=True }&#34;
/&gt;
```

#### 实现INotifyDataErrorInfo

该**INotifyDataErrorInfo**界面比更灵活**IDataErrorInfo的**接口。它支持属性的多个错误，异步数据验证，以及在对象的错误状态更改时通知视图的能力。

所述**INotifyDataErrorInfo**接口定义了一个**HasErrors**属性，该属性允许视图模型，以指示用于任何性质的误差（或多个误差）是否存在，和一个**GetErrors**方法，其允许视图模型返回错误消息的列表的特定属性。

所述**INotifyDataErrorInfo**接口还限定**ErrorsChanged**事件。这通过允许视图或视图模型通过**ErrorsChanged**事件**指示**特定属性的错误状态更改来支持异步验证方案。可以通过多种方式更改属性值，而不仅仅是通过数据绑定 - 例如，作为Web服务调用或后台计算的结果。该**ErrorsChanged**事件使得一旦数据验证错误已被确定视图模型告知错误的观点。

要支持**INotifyDataErrorInfo**，您需要维护每个属性的错误列表。Model-View-ViewModel参考实现（MVVM RI）演示了一种使用**ErrorsContainer**集合类来实现此目的的方法，该集合类跟踪对象中的所有验证错误。如果错误列表发生更改，它还会引发通知事件。以下代码示例显示了**DomainObject**（根模型对象），并使用**ErrorsContainer**类显示了**INotifyDataErrorInfo**的示例实现。

```c#
public abstract class DomainObject : INotifyPropertyChanged, 
                                        INotifyDataErrorInfo
{
    private ErrorsContainer&lt;ValidationResult&gt; errorsContainer =
                    new ErrorsContainer&lt;ValidationResult&gt;(
                        pn =&gt; this.RaiseErrorsChanged( pn ) );

    public event EventHandler&lt;DataErrorsChangedEventArgs&gt; ErrorsChanged;

    public bool HasErrors
    {
        get { return this.ErrorsContainer.HasErrors; }
    }

    public IEnumerable GetErrors( string propertyName )
    {
        return this.errorsContainer.GetErrors( propertyName );
    }

    protected void RaiseErrorsChanged( string propertyName )
    {
        var handler = this.ErrorsChanged;
        if (handler != null)
        {
            handler(this, new DataErrorsChangedEventArgs(propertyName) );
        }
    }
    ...
}
```

### Construction and Wire-Up

MVVM模式可以帮助您将UI与表示和业务逻辑和数据完全分离，因此在正确的类中实现正确的代码是有效使用MVVM模式的重要第一步。通过数据绑定和命令管理视图和视图模型类之间的交互也是需要考虑的重要方面。下一步是考虑如何在运行时实例化视图，视图模型和模型类并将它们相互关联。

**注意：**如果在应用程序中使用依赖项注入容器，则选择适当的策略来管理此步骤尤为重要。托管可扩展性框架（MEF）和Unity应用程序块（Unity）都提供了指定视图，视图模型和模型类之间的依赖关系以及使容器满足它们的能力。有关更高级的方案，请参阅[高级MVVM方案](https://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html)。

通常，视图与其视图模型之间存在一对一的关系。视图和视图模型通过视图的数据上下文属性松散耦合; 这允许视图中的可视元素和行为是绑定到视图模型上的属性，命令和方法的数据。您将需要决定如何在运行时通过**DataContext**属性来管理视图的实例化以及查看模型类及其关联。

在构建和连接视图和视图模型时也必须小心，以确保保持松耦合。如前一节所述，视图模型理想情况下不应依赖于视图的任何特定实现。同样，理想情况下，视图应该不依赖于视图模型的任何特定实现。

**注意：**但是，应该注意，视图将*隐式*依赖于视图模型上的特定属性，命令和方法，因为它定义了数据绑定。如果视图模型未实现所需的属性，命令或方法，则数据绑定引擎将生成运行时异常，该异常将在调试期间显示在Visual Studio输出窗口中。

可以通过多种方式在运行时构建视图和视图模型并将其关联。适合您的应用程序的方法在很大程度上取决于您是首先创建视图还是视图模型，以及是以编程方式还是以声明方式创建视图模型。以下部分描述了在运行时可以创建视图和视图模型类以及相互关联的常用方法。

#### 使用XAML创建视图模型

也许最简单的方法是视图以声明方式在XAML中实例化其对应的视图模型。构造视图时，还将构造相应的视图模型对象。您还可以在XAML中指定将视图模型设置为视图的数据上下文。

```xaml
&lt;UserControl.DataContext&gt;
    &lt;my:MyViewModel/&gt;
&lt;/UserControl.DataContext&gt;
```

创建此视图时，将自动构建**MyViewModel**的实例并将其设置为视图的数据上下文。此方法要求您的视图模型具有默认（无参数）构造函数。

视图的声明性构造和视图模型的分配具有以下优点：它很简单并且在诸如Microsoft Expression Blend或Microsoft Visual Studio的设计时工具中运行良好。此方法的缺点是视图具有相应视图模型类型的知识，并且视图模型类型必须具有默认构造函数。

#### 以编程方式创建视图模型

另一种方法是视图在其构造函数中以编程方式实例化其对应的视图模型实例。然后，它可以将其设置为其数据上下文，如以下代码示例所示。

```c#
public MyView()
{
    InitializeComponent();
    this.DataContext = new MyViewModel();
}
```

#### 使用视图模型定位器创建视图模型

创建视图模型实例并将其与视图关联的另一种方法是使用视图模型定位器。

Prism视图模型定位器具有**AutoWireViewModel**附加属性，在设置时调用**ViewModelLocationProvider**类中的**AutoWireViewModelChanged**方法来解析视图的视图模型。默认情况下，它使用基于约定的方法。

在Basic MVVM QuickStart中，MainWindow.xaml使用视图模型定位器来解析视图模型。

```xml
&lt;Window x:Class=&#34;QuickStart.Views.MainWindow&#34;
    ...
    xmlns:prism=&#34;http://prismlibrary.com/&#34;
    prism:ViewModelLocator.AutoWireViewModel=&#34;True&#34;&gt;
```

Prism的**ViewModelLocator**类有一个附加属性**AutoWireViewMode** l，当设置为true时，将尝试定位视图的视图模型，然后将视图的数据上下文设置为视图模型的实例。若要查找相应的视图模型，**ViewModelLocationProvider**首先尝试从**ViewModelLocationProvider**类的**Register**方法注册的任何映射中解析视图模型。如果使用此方法无法解析视图模型，例如，如果未创建映射，则**ViewModelLocationProvider**回归到基于约定的方法来解决正确的视图模型类型。此约定假定视图模型与视图类型在同一个程序集中，视图模型位于a。**ViewModels**子命名空间，该视图位于。**查看**子命名空间，该视图模型名称与视图名称对应，以“ViewModel”结尾。有关如何更改Prism的视图模型定位器约定的说明，请参阅[附录D：扩展棱镜](https://prismlibrary.github.io/docs/wpf/legacy/Appendix-D-Extending-Prism.html)。

**注意：ViewModelLocationProvider**可以在**Prism.Core** NuGet包中的**Prism.Mvvm**命名空间中找到。**ViewModelLocator**可以在**Prism.WPF** NuGet包中的**Prism.Mvvm**命名空间中找到。

#### 创建定义为数据模板的视图

视图可以定义为数据模板并与视图模型类型相关联。数据模板可以定义为资源，也可以在显示视图模型的控件中内联定义。控件的“内容”是视图模型实例，数据模板用于直观地表示它。WPF将自动实例化数据模板，并在运行时将其数据上下文设置为视图模型实例。此技术是首先实例化视图模型，然后创建视图的情况的示例。

数据模板灵活轻便。UI设计人员可以使用它们轻松定义视图模型的可视化表示，而无需任何复杂的代码。数据模板仅限于不需要任何UI逻辑（代码隐藏）的视图。Microsoft Blend for Visual Studio 2013可用于可视化设计和编辑数据模板。

以下示例显示绑定到客户列表的**ItemsControl**。底层集合中的每个客户对象都是一个视图模型实例。客户的视图由内联数据模板定义。在以下示例中，每个客户视图模型的视图由一个**StackPanel**组成，其中标签和文本框控件绑定到视图模型上的**Name**属性。

```xaml
&lt;ItemsControl ItemsSource=&#34;{Binding Customers}&#34;&gt;
    &lt;ItemsControl.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel Orientation=&#34;Horizontal&#34;&gt;
                &lt;TextBlock VerticalAlignment=&#34;Center&#34; Text=&#34;Customer Name: &#34; /&gt;
                &lt;TextBox Text=&#34;{Binding Name}&#34; /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ItemsControl.ItemTemplate&gt;
&lt;/ItemsControl&gt;
```

您还可以将数据模板定义为资源。以下示例显示了数据模板定义的资源，并通过**StaticResource**标记扩展应用于内容控件。

```xaml
&lt;UserControl ...&gt;
    &lt;UserControl.Resources&gt;
        &lt;DataTemplate x:Key=&#34;CustomerViewTemplate&#34;&gt;
            &lt;local:CustomerContactView /&gt;
        &lt;/DataTemplate&gt;
    &lt;/UserControl.Resources&gt;

    &lt;Grid&gt;
        &lt;ContentControl Content=&#34;{Binding Customer}&#34;
                ContentTemplate=&#34;{StaticResource CustomerViewTemplate}&#34; /&gt;
    &lt;/Grid&gt;
&lt;/UserControl&gt;
```

这里，数据模板包装了一个具体的视图类型。这允许视图定义代码隐藏行为。通过这种方式，数据模板机制可用于从外部提供视图和视图模型之间的关联。虽然前面的示例显示了**UserControl**资源中的模板，但它通常会放在应用程序的资源中以供重用。

### 关键决定

当您选择使用MVVM模式构建应用程序时，您将不得不做出某些难以在以后更改的设计决策。通常，这些决策是应用程序范围的，并且它们在整个应用程序中的一致使用将提高开发人员和设 以下总结了实现MVVM模式时最重要的决策：

- 确定查看和查看您将使用的模型构造的方法。您需要确定您的应用程序是首先构造视图还是视图模型，以及是否使用依赖注入容器，例如Unity或MEF。您通常希望这在整个应用程序范围内保持一致。有关详细信息，请参阅部分，[建设和线向上](https://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#construction-and-wire-up)，这个主题和部分先进施工和线向上，在[高级MVVM方案](https://prismlibrary.github.io/docs/wpf/legacy/Advanced-MVVM.html)。
- 确定是否将视图模型中的命令作为命令方法或命令对象公开。命令方法很容易公开，可以通过视图中的行为来调用。命令对象可以巧妙地封装命令和启用/禁用逻辑，并且可以通过行为或通过**ButtonBase**派生控件上的**Command**属性调用。为了使开发人员和设计人员更容易，最好将其作为应用程序范围内的选择。有关更多信息，请参阅本主题中的“ [命令](https://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#commands) ”一节。
- 确定视图模型和模型如何向视图报告错误。您的模型可以支持**IDataErrorInfo**或**INotifyDataErrorInfo**。并非所有模型都需要报告错误信息，但对于那些模型，最好为开发人员提供一致的方法。有关详细信息，请参阅本主题中的“ [数据验证和错误报告](https://prismlibrary.github.io/docs/wpf/legacy/Implementing-MVVM.html#data-validation-and-error-reporting) ”部分。
- 确定Microsoft Blend for Visual Studio 2013设计时数据支持对您的团队是否重要。如果您将使用Blend来设计和维护UI并希望查看设计时数据，请确保您的视图和视图模型提供的构造函数没有参数，并且您的视图提供了设计时数据上下文。或者，考虑使用Microsoft Blend for Visual Studio 2013提供的设计时功能，使用设计时属性，例如**d：DataContext**和**d：DesignSource**。有关更多信息，请参阅在[编写用户界面中](https://prismlibrary.github.io/docs/wpf/legacy/Composing-the-UI.html)创建设计器友好视图的准则。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/07/prism9-mvvm/  

