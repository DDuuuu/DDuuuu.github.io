# TouchSocket 依赖注入容器


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

容器现在已经成为各种组件的标配了，为什么容器这么火，确实是太好用了。

表面上容器解决的是耦合问题，实际上容器实现了框架对流程的控制。

表面的耦合问题：没有容器，高层逻辑想要实现某个具体功能，只能依赖某个具体类或者具体工厂。当变化越来越多，改动越来越大，具体类之间依赖关系就像麻绳。里氏替换和依赖倒置根本无法实现。容器的出现将所有的具体类都保存在容器中，实现依赖倒置和里氏替换，系统高层和底层实现解耦。

框架对流程的控制问题：框架实现了整个应用流程的编排，流程中肯定需要具体的执行类。一旦依赖某个执行类就无法应对变化，具体类和流程严重耦合。整个框架就像被焊死，无法应对任何变化。容器就像是活页，连接着具体类和框架流程，当具体类发生变化，对框架没有任何影响，这也就使得框架的应用范围更广，实现的功能更多，框架中任何部件都是可以改变的。

唯一不变的就是改变，如果没有改变，也就不需要任何模式了。我们痛恨变化，同时也热爱变化，在痛苦中追求无限可能。

言归正传，TouchSocket框架实现了一个简易版的依赖注入容器。一起来看一下。

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index

## 依赖注入

简单点，容器其实就是一个字典，一个包含一个key和具体类的字典，将框架中所有依赖具体类的地方换成key，当执行到key的时候去字典中根据key取出具体类就可以了。

当然字典也需要生成，就是在应用开始的时候根据需求构建出字典。以后即使变化，也只变化开始构建的部分，最大程度减少修改。

真正的容器就是将字典改造一下，既然由容器管理，就应该负责到底，管理具体类的生命周期，什么时候创建，什么时候销毁，怎么创建。

1. 首先字典中的具体类需要改造一下，变成描述类。

生命周期分为:单例，瞬态，域。

```C#
/// &lt;summary&gt;
/// 注入依赖对象
/// &lt;/summary&gt;
public class DependencyDescriptor
{
    /// &lt;summary&gt;
    /// 初始化一个单例实例。
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;instance&#34;&gt;&lt;/param&gt;
    public DependencyDescriptor(Type fromType, object instance)
    {
        this.FromType = fromType;
        this.ToInstance = instance;
        this.Lifetime = Lifetime.Singleton;
        this.ToType = instance.GetType();
    }

    /// &lt;summary&gt;
    /// 初始化一个完整的服务注册
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;toType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;lifetime&#34;&gt;&lt;/param&gt;
    public DependencyDescriptor(Type fromType, Type toType, Lifetime lifetime)
    {
        this.FromType = fromType;
        this.Lifetime = lifetime;
        this.ToType = toType;
    }

    /// &lt;summary&gt;
    /// 实例类型
    /// &lt;/summary&gt;
    public Type ToType { get; }

    /// &lt;summary&gt;
    /// 实例
    /// &lt;/summary&gt;
    public object ToInstance { get; set; }

    /// &lt;summary&gt;
    /// 生命周期
    /// &lt;/summary&gt;
    public Lifetime Lifetime { get; }

    /// &lt;summary&gt;
    /// 注册类型
    /// &lt;/summary&gt;
    public Type FromType { get; }
}

/// &lt;summary&gt;
/// 注入项的生命周期。
/// &lt;/summary&gt;
public enum Lifetime
{
    /// &lt;summary&gt;
    /// 单例对象
    /// &lt;/summary&gt;
    Singleton,

    /// &lt;summary&gt;
    /// 以&lt;see cref=&#34;IScopedContainer&#34;/&gt;接口为区域实例单例。
    /// &lt;/summary&gt;
    Scoped,

    /// &lt;summary&gt;
    /// 瞬时对象
    /// &lt;/summary&gt;
    Transient
}
```

2. 字典中的key改成FromType&#43;Name。

最终字典变成了` private readonly ConcurrentDictionary&lt;string, DependencyDescriptor&gt; registrations = new ConcurrentDictionary&lt;string, DependencyDescriptor&gt;();`这样。

看一下怎么实现注入和解析的。下面是原代码，Resolve方法根据生命周期实现解析逻辑；Create方法负责创建，同时各种注入方式；Register实现构建字典。

该容器有一些特点：

1. 在注入容器时，提供了一个Name，key被构建成FromType&#43;Name。这样的好处是可以根据Name来获取具体类。

2. 同时对于多态的问题，同一个接口的多个实现没有实现注入，只能通过Name来区别了。
3. 如果FromType不是抽象类，不需要注入，也可以Resolve出来。
4. Resolve会根据所有构造函数参数的个数最多的那个构造函数。
5. 可以使用DependencyInject，DependencyParamterInject特性进行控制。
6. 实现了构造函数注入，属性注入和方法注入

```C#
/// &lt;summary&gt;
/// IOC容器
/// &lt;/summary&gt;
public class Container : IContainer
{
    private readonly ConcurrentDictionary&lt;string, DependencyDescriptor&gt; registrations = new ConcurrentDictionary&lt;string, DependencyDescriptor&gt;();

    /// &lt;summary&gt;
    /// 初始化一个IOC容器
    /// &lt;/summary&gt;
    public Container()
    {
        this.RegisterSingleton&lt;IContainer&gt;(this);
    }

    /// &lt;summary&gt;
    /// &lt;inheritdoc/&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public bool IsRegistered(Type fromType, string key = &#34;&#34;)
    {
        return this.registrations.ContainsKey($&#34;{fromType.FullName}{key}&#34;);
    }

    /// &lt;summary&gt;
    /// &lt;inheritdoc/&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;descriptor&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    public void Register(DependencyDescriptor descriptor, string key = &#34;&#34;)
    {
        string k = $&#34;{descriptor.FromType.FullName}{key}&#34;;
        this.registrations.AddOrUpdate(k, descriptor, (k, v) =&gt; { return descriptor; });
    }

    /// &lt;summary&gt;
    /// &lt;inheritdoc/&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;ps&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public object Resolve(Type fromType, object[] ps = null, string key = &#34;&#34;)
    {
        if (fromType == typeof(IScopedContainer))
        {
            return this.GetScopedContainer();
        }
        string k;
        DependencyDescriptor descriptor;
        if (fromType.IsGenericType)
        {
            Type type = fromType.GetGenericTypeDefinition();
            k = $&#34;{type.FullName}{key}&#34;;
            if (this.registrations.TryGetValue(k, out descriptor))
            {
                if (descriptor.Lifetime == Lifetime.Singleton)
                {
                    if (descriptor.ToInstance != null)
                    {
                        return descriptor.ToInstance;
                    }
                    lock (descriptor)
                    {
                        if (descriptor.ToInstance != null)
                        {
                            return descriptor.ToInstance;
                        }
                        if (descriptor.ToType.IsGenericType)
                        {
                            return descriptor.ToInstance = this.Create(descriptor.ToType.MakeGenericType(fromType.GetGenericArguments()), ps);
                        }
                        else
                        {
                            return descriptor.ToInstance = this.Create(descriptor.ToType, ps);
                        }
                    }
                }
                if (descriptor.ToType.IsGenericType)
                {
                    return this.Create(descriptor.ToType.MakeGenericType(fromType.GetGenericArguments()), ps);
                }
                else
                {
                    return this.Create(descriptor.ToType, ps);
                }
            }
        }
        k = $&#34;{fromType.FullName}{key}&#34;;
        if (this.registrations.TryGetValue(k, out descriptor))
        {
            if (descriptor.Lifetime == Lifetime.Singleton)
            {
                if (descriptor.ToInstance != null)
                {
                    return descriptor.ToInstance;
                }
                lock (descriptor)
                {
                    if (descriptor.ToInstance != null)
                    {
                        return descriptor.ToInstance;
                    }
                    return descriptor.ToInstance = this.Create(descriptor.ToType, ps);
                }
            }
            return this.Create(descriptor.ToType, ps);
        }
        else if (fromType.IsPrimitive || fromType == typeof(string))
        {
            return default;
        }
        else if (fromType.IsClass &amp;&amp; !fromType.IsAbstract)
        {
            if (fromType.GetCustomAttribute&lt;DependencyParamterInjectAttribute&gt;() is DependencyParamterInjectAttribute attribute)
            {
                if (attribute.ResolveNullIfNoRegistered &amp;&amp; !this.IsRegistered(fromType, key))
                {
                    return default;
                }
            }
            return this.Create(fromType, ps);
        }
        else
        {
            return default;
        }
    }

    /// &lt;summary&gt;
    /// &lt;inheritdoc/&gt;
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;toType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;ops&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    private object Create(Type toType, object[] ops)
    {
        ConstructorInfo ctor = toType.GetConstructors().FirstOrDefault(x =&gt; x.IsDefined(typeof(DependencyInjectAttribute), true));
        if (ctor is null)
        {
            //如果没有被特性标记，那就取构造函数参数最多的作为注入目标
            if (toType.GetConstructors().Length == 0)
            {
                throw new Exception($&#34;没有找到类型{toType.FullName}的公共构造函数。&#34;);
            }
            ctor = toType.GetConstructors().OrderByDescending(x =&gt; x.GetParameters().Length).First();
        }
        else
        {
            if (ops == null)
            {
                ops = ctor.GetCustomAttribute&lt;DependencyInjectAttribute&gt;().Ps;
            }
        }

        DependencyTypeAttribute dependencyTypeAttribute = null;
        if (toType.IsDefined(typeof(DependencyTypeAttribute), true))
        {
            dependencyTypeAttribute = toType.GetCustomAttribute&lt;DependencyTypeAttribute&gt;();
        }

        var parameters = ctor.GetParameters();
        object[] ps = new object[parameters.Length];

        if (dependencyTypeAttribute == null || dependencyTypeAttribute.Type.HasFlag(DependencyType.Constructor))
        {
            for (int i = 0; i &lt; parameters.Length; i&#43;&#43;)
            {
                if (ops != null &amp;&amp; ops.Length - 1 &gt;= i)
                {
                    ps[i] = ops[i];
                }
                else
                {
                    if (parameters[i].ParameterType.IsPrimitive || parameters[i].ParameterType == typeof(string))
                    {
                        if (parameters[i].HasDefaultValue)
                        {
                            ps[i] = parameters[i].DefaultValue;
                        }
                        else
                        {
                            ps[i] = default;
                        }
                    }
                    else
                    {
                        if (parameters[i].IsDefined(typeof(DependencyParamterInjectAttribute), true))
                        {
                            DependencyParamterInjectAttribute attribute = parameters[i].GetCustomAttribute&lt;DependencyParamterInjectAttribute&gt;();
                            Type type = attribute.Type == null ? parameters[i].ParameterType : attribute.Type;

                            if (attribute.ResolveNullIfNoRegistered &amp;&amp; !this.IsRegistered(type, attribute.Key))
                            {
                                ps[i] = default;
                            }
                            else
                            {
                                ps[i] = this.Resolve(type, attribute.Ps, attribute.Key);
                            }
                        }
                        else
                        {
                            ps[i] = this.Resolve(parameters[i].ParameterType, null);
                        }
                    }
                }
            }
        }
        object instance = Activator.CreateInstance(toType, ps);

        if (dependencyTypeAttribute == null || dependencyTypeAttribute.Type.HasFlag(DependencyType.Property))
        {
            var propetys = toType.GetProperties().Where(x =&gt; x.IsDefined(typeof(DependencyInjectAttribute), true));
            foreach (var item in propetys)
            {
                if (item.CanWrite)
                {
                    object obj;
                    if (item.IsDefined(typeof(DependencyParamterInjectAttribute), true))
                    {
                        DependencyParamterInjectAttribute attribute = item.GetCustomAttribute&lt;DependencyParamterInjectAttribute&gt;();
                        Type type = attribute.Type == null ? item.PropertyType : attribute.Type;
                        if (attribute.ResolveNullIfNoRegistered &amp;&amp; !this.IsRegistered(type, attribute.Key))
                        {
                            obj = null;
                        }
                        else
                        {
                            obj = this.Resolve(type, attribute.Ps, attribute.Key);
                        }
                    }
                    else
                    {
                        obj = this.Resolve(item.PropertyType, null);
                    }
                    item.SetValue(instance, obj);
                }
            }
        }

        if (dependencyTypeAttribute == null || dependencyTypeAttribute.Type.HasFlag(DependencyType.Method))
        {
            var methods = toType.GetMethods().Where(x =&gt; x.IsDefined(typeof(DependencyInjectAttribute), true)).ToList();
            foreach (var item in methods)
            {
                parameters = item.GetParameters();
                ops = item.GetCustomAttribute&lt;DependencyInjectAttribute&gt;().Ps;
                ps = new object[parameters.Length];
                for (int i = 0; i &lt; ps.Length; i&#43;&#43;)
                {
                    if (ops != null &amp;&amp; ops.Length - 1 &gt;= i)
                    {
                        ps[i] = ops[i];
                    }
                    else
                    {
                        if (parameters[i].ParameterType.IsPrimitive || parameters[i].ParameterType == typeof(string))
                        {
                            if (parameters[i].HasDefaultValue)
                            {
                                ps[i] = parameters[i].DefaultValue;
                            }
                            else
                            {
                                ps[i] = default;
                            }
                        }
                        else
                        {
                            if (parameters[i].IsDefined(typeof(DependencyParamterInjectAttribute), true))
                            {
                                DependencyParamterInjectAttribute attribute = parameters[i].GetCustomAttribute&lt;DependencyParamterInjectAttribute&gt;();
                                Type type = attribute.Type == null ? parameters[i].ParameterType : attribute.Type;

                                if (attribute.ResolveNullIfNoRegistered &amp;&amp; !this.IsRegistered(type, attribute.Key))
                                {
                                    ps[i] = default;
                                }
                                else
                                {
                                    ps[i] = this.Resolve(type, attribute.Ps, attribute.Key);
                                }
                            }
                            else
                            {
                                ps[i] = this.Resolve(parameters[i].ParameterType, null);
                            }
                        }
                    }
                }
                item.Invoke(instance, ps);
            }
        }
        return instance;
    }

    private IScopedContainer GetScopedContainer()
    {
        Container container = new Container();
        foreach (var item in this.registrations)
        {
            if (item.Value.Lifetime == Lifetime.Scoped)
            {
                container.registrations.AddOrUpdate(item.Key, new DependencyDescriptor(item.Value.FromType, item.Value.ToType, Lifetime.Singleton), (k, v) =&gt; { return v; });
            }
            else
            {
                container.registrations.AddOrUpdate(item.Key, item.Value, (k, v) =&gt; { return v; });
            }
        }
        return new ScopedContainer(container);
    }
}
```

依赖注入一些便捷使用的方法封装还是依靠扩展方法实现

```C#
/// &lt;summary&gt;
/// IContainerExtensions
/// &lt;/summary&gt;
public static class ContainerExtensions
{
    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTo&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;instance&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom, TTo&gt;(this IContainer container, TTo instance)
          where TFrom : class
          where TTo : class, TFrom
    {
        RegisterSingleton(container, typeof(TFrom), instance);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTo&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;instance&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom, TTo&gt;(this IContainer container, string key, TTo instance)
          where TFrom : class
          where TTo : class, TFrom
    {
        RegisterSingleton(container, typeof(TFrom), instance, key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;instance&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton(this IContainer container, Type fromType, object instance, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(fromType, instance), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;instance&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom&gt;(this IContainer container, object instance, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(typeof(TFrom), instance), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom&gt;(this IContainer container, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(typeof(TFrom), typeof(TFrom), Lifetime.Singleton), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;toType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton(this IContainer container, Type fromType, Type toType, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(fromType, toType, Lifetime.Singleton), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom, TTO&gt;(this IContainer container)
         where TFrom : class
         where TTO : class, TFrom
    {
        RegisterSingleton(container, typeof(TFrom), typeof(TTO));
        return container;
    }

    /// &lt;summary&gt;
    /// 注册单例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterSingleton&lt;TFrom, TTO&gt;(this IContainer container, string key)
         where TFrom : class
         where TTO : class, TFrom
    {
        RegisterSingleton(container, typeof(TFrom), typeof(TTO), key);
        return container;
    }

    #region Transient

    /// &lt;summary&gt;
    /// 注册临时映射
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterTransient&lt;TFrom, TTO&gt;(this IContainer container)
         where TFrom : class
         where TTO : class, TFrom
    {
        RegisterTransient(container, typeof(TFrom), typeof(TTO));
        return container;
    }

    /// &lt;summary&gt;
    /// 注册临时映射
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterTransient&lt;TFrom, TTO&gt;(this IContainer container, string key)
        where TFrom : class
        where TTO : class, TFrom
    {
        RegisterTransient(container, typeof(TFrom), typeof(TTO), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册临时映射
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;toType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterTransient(this IContainer container, Type fromType, Type toType, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(fromType, toType, Lifetime.Transient), key);
        return container;
    }

    #endregion Transient

    #region Scoped

    /// &lt;summary&gt;
    /// 注册区域映射
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterScoped&lt;TFrom, TTO&gt;(this IContainer container)
         where TFrom : class
         where TTO : class, TFrom
    {
        RegisterScoped(container, typeof(TFrom), typeof(TTO));
        return container;
    }

    /// &lt;summary&gt;
    /// 注册区域映射
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TFrom&#34;&gt;&lt;/typeparam&gt;
    /// &lt;typeparam name=&#34;TTO&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterScoped&lt;TFrom, TTO&gt;(this IContainer container, string key)
         where TFrom : class
         where TTO : class, TFrom
    {
        RegisterScoped(container, typeof(TFrom), typeof(TTO), key);
        return container;
    }

    /// &lt;summary&gt;
    /// 注册区域映射
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;fromType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;toType&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static IContainer RegisterScoped(this IContainer container, Type fromType, Type toType, string key = &#34;&#34;)
    {
        container.Register(new DependencyDescriptor(fromType, toType, Lifetime.Scoped), key);
        return container;
    }

    #endregion Scoped

    /// &lt;summary&gt;
    /// 创建类型对应的实例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;ps&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static T Resolve&lt;T&gt;(this IScopedContainer container, object[] ps, string key = &#34;&#34;)
    {
        return (T)container.Resolve(typeof(T), ps, key);
    }

    /// &lt;summary&gt;
    /// 创建类型对应的实例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static T Resolve&lt;T&gt;(this IScopedContainer container)
    {
        return Resolve&lt;T&gt;(container, null);
    }

    /// &lt;summary&gt;
    /// 创建类型对应的实例
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    /// &lt;param name=&#34;container&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static T Resolve&lt;T&gt;(this IScopedContainer container, string key)
    {
        return Resolve&lt;T&gt;(container, null, key);
    }
}
```

使用：

在框架中，将容器放置在Config中，这边Config也充当了Context的作用，同时默认注入了Logger和PluginsManager。框架中的各个组件可以通过Config进行注入和结构。

```C#
/// &lt;summary&gt;
/// 构造函数
/// &lt;/summary&gt;
public TouchSocketConfig()
{
    this.SetContainer(new Container());
}

/// &lt;summary&gt;
/// 设置注入容器。
/// &lt;/summary&gt;
/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public TouchSocketConfig SetContainer(IContainer value)
{
    this.m_container = value;
    this.m_container.RegisterTransient&lt;ILog, ConsoleLogger&gt;();
    this.SetPluginsManager(new PluginsManager(this.m_container));
    return this;
}
```



## 总结

该容器虽然简单，但是足够使用，小巧紧致，非常值得学习。不需要注入直接解析对象的方式，虽然有点违反依赖倒置原则，但是在使用中确实是一个非常实用的功能。有一点缺陷是没有实现Dispose和瞬态对象的弱引用，当域生命周期结束的时候，Dispose容器同时销毁所有域创建出的瞬态对象。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket2-dependencyinjection/  

