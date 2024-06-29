# TouchSocket 依赖属性


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

该框架实现了广泛的Socket应用，NAT，各种RPC，文件传输，WebAPI，WebSocket，非常优秀的框架，学习一下，有问题可以相互探讨。

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index

本章主要介绍TouchSocket的主要特性之一：依赖属性

## 依赖属性

用过WPF都知道依赖属性，其是绑定，动画，样式的基础，提供了属性值，更改通知等功能，该框架中的依赖属性相当于简易版本，提供了应用的思路。

依赖属性可以看成是Key和Value的封装。依赖属性类：名称，所属类型，值类型和值，还包含了一个工厂方法，用来创建依赖属性，创建时可以提供初始值。

```C#
/// &lt;summary&gt;
/// 依赖项属性
/// &lt;/summary&gt;
[DebuggerDisplay(&#34;Name={Name},Type={ValueType}&#34;)]
public class DependencyProperty
{
    /// &lt;summary&gt;
    /// 属性名称
    /// &lt;/summary&gt;
    protected string m_name;

    /// &lt;summary&gt;
    /// 所属类型
    /// &lt;/summary&gt;
    protected Type m_owner;

    /// &lt;summary&gt;
    /// 值类型
    /// &lt;/summary&gt;
    protected Type m_valueType;

    private object m_value;

    /// &lt;summary&gt;
    /// 依赖项属性
    /// &lt;/summary&gt;
    private DependencyProperty()
    {
    }

    /// &lt;summary&gt;
    /// 默认值
    /// &lt;/summary&gt;
    public object DefauleValue =&gt; this.m_value;

    /// &lt;summary&gt;
    /// 属性名
    /// &lt;/summary&gt;
    public string Name =&gt; this.m_name;

    /// &lt;summary&gt;
    /// 所属类型
    /// &lt;/summary&gt;
    public Type Owner =&gt; this.m_owner;

    /// &lt;summary&gt;
    /// 值类型
    /// &lt;/summary&gt;
    public Type ValueType =&gt; this.m_valueType;


    internal void DataValidation(object value)
    {
        if (value == null)
        {
            if (typeof(ValueType).IsAssignableFrom(this.m_valueType))
            {
                throw new Exception($&#34;属性“{this.m_name}”赋值类型不允许出现Null&#34;);
            }
        }
        else if (!this.m_valueType.IsAssignableFrom(value.GetType()))
        {
            throw new Exception($&#34;属性“{this.m_name}”赋值类型与注册类型不一致，应当注入“{this.m_valueType}”类型&#34;);
        }
    }

    internal void SetDefauleValue(object value)
    {
        this.DataValidation(value);
        this.m_value = value;
    }

    /// &lt;summary&gt;
    /// 注册依赖项属性。
    /// &lt;para&gt;依赖属性的默认值，可能会应用于所有的&lt;see cref=&#34;IDependencyObject&#34;/&gt;&lt;/para&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;propertyName&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;valueType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;owner&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;value&#34;&gt;依赖项属性值，一般该值应该是值类型，因为它可能会被用于多个依赖对象。&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static DependencyProperty Register(string propertyName, Type valueType, Type owner, object value)
    {
        DependencyProperty dp = new DependencyProperty
        {
            m_name = propertyName,
            m_valueType = valueType,
            m_owner = owner
        };
        dp.SetDefauleValue(value);
        return dp;
    }
}
```

如何管理依赖属性，类中创建依赖属性，并设置和获取依赖属性的值。首先实现接口，可以获取和设置依赖属性的值，实现一个基类，实现该接口，所有包含依赖属性的类继承该基类，就可以实现操作依赖属性了。

该框架做了进一步扩展，在基类中添加了一个依赖属性字典，可以添加外部依赖属性。

也就是在基类中保存了一个Key，Value的字典，通过特定的Key获取到Value，在配置的时候特别有用，配置类Option/Config怎么应对变化，写组件的时候发现配置项多需要添加属性怎么办，修改配置项，违反开放封闭原则；使用继承，显得太重；增加一个配置类，还不如使用继承。

把配置项修改成字典，所有信息通过key,value保存，可以应对开放封闭原则。显然该框架就说这样干的，key是依赖属性，vlaue是依赖属性的值。

```c#
/// &lt;summary&gt;
/// 依赖对象接口
/// &lt;/summary&gt;
public interface IDependencyObject : System.IDisposable
{
    /// &lt;summary&gt;
    /// 获取依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    object GetValue(DependencyProperty dp);

    /// &lt;summary&gt;
    /// 获取依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public T GetValue&lt;T&gt;(DependencyProperty dp);

    /// &lt;summary&gt;
    /// 设置依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
    public DependencyObject SetValue(DependencyProperty dp, object value);
}
```

```C#
/// &lt;summary&gt;
/// 依赖项对象.
/// 线程安全。
/// &lt;/summary&gt;
public class DependencyObject : DisposableObject, IDependencyObject, System.IDisposable
{
    /// &lt;summary&gt;
    /// 构造函数
    /// &lt;/summary&gt;
    public DependencyObject()
    {
        this.m_dp = new ConcurrentDictionary&lt;DependencyProperty, object&gt;();
    }

    [System.Diagnostics.DebuggerBrowsable(System.Diagnostics.DebuggerBrowsableState.Never)]
    private readonly ConcurrentDictionary&lt;DependencyProperty, object&gt; m_dp;

    /// &lt;summary&gt;
    /// 获取依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public object GetValue(DependencyProperty dp)
    {
        if (this.m_dp.TryGetValue(dp, out object value))
        {
            return value;
        }
        else
        {
            return dp.DefauleValue;
        }
    }

    /// &lt;summary&gt;
    /// 获取依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public T GetValue&lt;T&gt;(DependencyProperty dp)
    {
        try
        {
            return (T)this.GetValue(dp);
        }
        catch
        {
            return default;
        }
    }

    /// &lt;summary&gt;
    /// 设置依赖注入的值
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dp&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
    public DependencyObject SetValue(DependencyProperty dp, object value)
    {
        dp.DataValidation(value);
        if (this.m_dp.ContainsKey(dp))
        {
            this.m_dp[dp] = value;
        }
        else
        {
            this.m_dp.TryAdd(dp, value);
        }
        return this;
    }

    /// &lt;summary&gt;
    /// &lt;inheritdoc/&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;disposing&#34;&gt;&lt;/param&gt;
    protected override void Dispose(bool disposing)
    {
        this.m_dp.Clear();
        base.Dispose(disposing);
    }
}
```

再来看应用

对于socket配置项确实不少，不同应用还要有不同的配置项。解决方案：

1. 首先创建需要配置的依赖属性，该依赖属性使用外部静态类

2. 然后让Config类继承DependencyObject，
3. 通过扩展方法为Config类提供功能，以应对变化。



静态类管理所有依赖属性

```C#
/// &lt;summary&gt;
/// TouchSocketConfigBuilder配置扩展
/// &lt;/summary&gt;
public static class TouchSocketConfigExtension
{
    #region 数据

    /// &lt;summary&gt;
    /// 接收缓存容量，默认1024*64，其作用有两个：
    /// &lt;list type=&#34;number&#34;&gt;
    /// &lt;item&gt;指示单次可接受的最大数据量&lt;/item&gt;
    /// &lt;item&gt;指示常规申请内存块的长度&lt;/item&gt;
    /// &lt;/list&gt;
    /// 所需类型&lt;see cref=&#34;int&#34;/&gt;
    /// &lt;/summary&gt;
    public static readonly DependencyProperty BufferLengthProperty =
        DependencyProperty.Register(&#34;BufferLength&#34;, typeof(int), typeof(TouchSocketConfigExtension), 1024 * 64);

    /// &lt;summary&gt;
    /// 数据处理适配器，默认为获取&lt;see cref=&#34;NormalDataHandlingAdapter&#34;/&gt;
    /// 所需类型&lt;see cref=&#34;Func{TResult}&#34;/&gt;
    /// &lt;/summary&gt;
    public static readonly DependencyProperty DataHandlingAdapterProperty =
        DependencyProperty.Register(&#34;DataHandlingAdapter&#34;, typeof(Func&lt;DataHandlingAdapter&gt;), typeof(TouchSocketConfigExtension),
           (Func&lt;DataHandlingAdapter&gt;)(() =&gt; { return new NormalDataHandlingAdapter(); }));

    /// &lt;summary&gt;
    /// 接收类型，默认为&lt;see cref=&#34;ReceiveType.Auto&#34;/&gt;
    /// &lt;para&gt;&lt;see cref=&#34;ReceiveType.Auto&#34;/&gt;为自动接收数据，然后主动触发。&lt;/para&gt;
    /// &lt;para&gt;&lt;see cref=&#34;ReceiveType.None&#34;/&gt;为不投递IO接收申请，用户可通过&lt;see cref=&#34;ITcpClientBase.GetStream&#34;/&gt;，获取到流以后，自己处理接收。注意：连接端不会感知主动断开&lt;/para&gt;
    /// 所需类型&lt;see cref=&#34;TouchSocket.Sockets. ReceiveType&#34;/&gt;
    /// &lt;/summary&gt;
    public static readonly DependencyProperty ReceiveTypeProperty =
        DependencyProperty.Register(&#34;ReceiveType&#34;, typeof(ReceiveType), typeof(TouchSocketConfigExtension), ReceiveType.Auto);
            /// &lt;summary&gt;
    /// 服务名称，用于标识，无实际意义，所需类型&lt;see cref=&#34;string&#34;/&gt;
    /// &lt;/summary&gt;
    public static readonly DependencyProperty ServerNameProperty =
        DependencyProperty.Register(&#34;ServerName&#34;, typeof(string), typeof(TouchSocketConfigExtension), &#34;RRQMServer&#34;);

    /// &lt;summary&gt;
    /// 多线程数量，默认为10。
    /// &lt;para&gt;TCP模式中，该值等效于&lt;see cref=&#34;ThreadPool.SetMinThreads(int, int)&#34;/&gt;&lt;/para&gt;
    /// &lt;para&gt;UDP模式中，该值为重叠IO并发数&lt;/para&gt;
    /// 所需类型&lt;see cref=&#34;int&#34;/&gt;
    /// &lt;/summary&gt;
    public static readonly DependencyProperty ThreadCountProperty =
        DependencyProperty.Register(&#34;ThreadCount&#34;, typeof(int), typeof(TouchSocketConfigExtension), 10);

    ....  

}
```

Config类继承DependencyObject

```C#
/// &lt;summary&gt;
/// 配置文件基类
/// &lt;/summary&gt;
public class TouchSocketConfig : DependencyObject
{
    ....
}
```

为Config提供扩展方法

```C#
/// &lt;summary&gt;
/// 接收缓存容量，默认1024*10，其作用有两个：
/// &lt;list type=&#34;number&#34;&gt;
/// &lt;item&gt;指示单次可接受的最大数据量&lt;/item&gt;
/// &lt;item&gt;指示常规申请内存块的长度&lt;/item&gt;
/// &lt;/list&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;config&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static TouchSocketConfig SetBufferLength(this TouchSocketConfig config, int value)
{
    config.SetValue(BufferLengthProperty, value);
    return config;
}

/// &lt;summary&gt;
/// 设置(Tcp系)数据处理适配器。
/// &lt;/summary&gt;
/// &lt;param name=&#34;config&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static TouchSocketConfig SetDataHandlingAdapter(this TouchSocketConfig config, Func&lt;DataHandlingAdapter&gt; value)
{
    config.SetValue(DataHandlingAdapterProperty, value);
    return config;
}
        /// &lt;summary&gt;
/// 服务名称，用于标识，无实际意义
/// &lt;/summary&gt;
/// &lt;param name=&#34;config&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static TouchSocketConfig SetServerName(this TouchSocketConfig config, string value)
{
    config.SetValue(ServerNameProperty, value);
    return config;
}

/// &lt;summary&gt;
/// 多线程数量，默认为10。
/// &lt;para&gt;TCP模式中，该值等效于&lt;see cref=&#34;ThreadPool.SetMinThreads(int, int)&#34;/&gt;&lt;/para&gt;
/// &lt;para&gt;UDP模式中，该值为重叠IO并发数&lt;/para&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;config&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static TouchSocketConfig SetThreadCount(this TouchSocketConfig config, int value)
{
    config.SetValue(ThreadCountProperty, value);
    return config;
}

```

## 总结

依赖属性这样的应用方式非常优秀，满足了扩展开放，单一职责。如果有其他的应用方式欢迎评论。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket1-dependencyproperties/  

