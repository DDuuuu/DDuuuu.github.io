# SuperSocket1.6 源码解析


注意：该版本为SuperSocket1.6,最新版已有较大改动

## Normal Socket

System.Net.Sockets.dll程序集中使用socket类：

服务器：

1. **创建socket**:`_socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);`
2. **创建IP**：`IPAddress _ip = IPAddress.Parse(ip);_endPoint = new IPEndPoint(_ip, port);`
3. **绑定IP地址：** `_socket.Bind(_endPoint);` //绑定端口
4. **服务开启监听：**`_socket.Listen(BACKLOG);` //开启监听，backlog是监听的最大数列
5. **开启监听线程**：创建新的监听线程，在监听线程中while调用`Socket acceptSocket = _socket.Accept();`
   1. 一旦acceptSocket 不为空，说明有客户端连接成功，保存客户端socket，并查看该socket的isConnected属性是否连接`socket.RemoteEndPoint.ToString();`
   2. 一旦连接创建接收线程，并启动线程，在该线程中创建while`while (sInfo.isConnected){sInfo.socket.BeginReceive(sInfo.buffer, 0, sInfo.buffer.Length, SocketFlags.None, ReceiveCallBack, sInfo.socket.RemoteEndPoint);}`来接收客户端传来的消息。
6. **BeginReceive()**有一个回调函数ReceiveCallBack()通过读取byte[]buffer
7. 向客户端发送信息`socket.Send(Encoding.ASCII.GetBytes(text));`

receivebuffer默认值8192

## SocketAsyncEventArgs

异步套接字操作

1. **创建IPEndPoint**
2. **创建socket**`ListenerSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);`
3. **绑定IP地址**`ListenerSocket.Bind(e);`
4. **开始监听**`ListenerSocket.Listen(10);`
5. **创建异步套接字,并绑定异步完成事件**`Args = new SocketAsyncEventArgs();Args.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;(ProcessAccept);`
6. **调用socket的AcceptAsync(Args)方法**`ListenerSocket.AcceptAsync(Args);`
7. **在异步套接字完成事件的回调函数中**，创建新的异步套接字用于接收客户端传入消息的异步操作。`var args = new SocketAsyncEventArgs();args.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;(OnIOCompleted);args.AcceptSocket = s;s.ReceiveAsync(args)`s.**ReceiveAsync**(args)，s接收的socket的，新建一个异步套接字，并传入ReceiveAsync()方法。
8. `switch (e.LastOperation)case SocketAsyncOperation.Receive:`

Socket.AcceptAsync(SocketAsyncEventArgs) 方法

返回：如果 I/O 操作挂起，则为 `true`。 操作完成时，将引发 [Completed](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socketasynceventargs.completed?view=netframework-4.8) 参数的 `e` 事件。read和write是异步完成为true，如果过是同步完成则为false.

如果 I/O 操作同步完成，则为 `false`。 将不会引发 [Completed](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socketasynceventargs.completed?view=netframework-4.8) 参数的 `e` 事件，并且可能在方法调用返回后立即检查作为参数传递的 `e` 对象以检索操作的结果。

## SuperSocket Architecture

#### SuperSocket 层次示意图



![SuperSocketLayers](/supersocket/1579317752168.jpg)

- Reusable IO Buffer Pool：BufferManager类

#### SuperSocket 对象模型图示意图



![SuperSocketObjectModel](/supersocket/objectmodel.jpg)

#### SuperSocket 请求处理模型示意图



![SuperSocketRequestHandlingModel](/supersocket/requesthandlingmodel.jpg)

#### SuperSocket 隔离模型示意图



![SuperSocketIsolationModel](/supersocket/isolationmodel.jpg)



### Config

### Command Filters

### Log/LogFactory

### Command Loaders

![1579856396698](/supersocket/1078802-20200127233503613-437203940-1580139446894.png)







### ReceiveFilterFactory

![1579855102888](/supersocket/1078802-20200127233503630-1179617648-1580139446901.png)

### ReceiveFilter

![1579855469740](/supersocket/1078802-20200127233503679-902606586-1580139446911.png)













### Connection Filters

![1579856311644](/supersocket/1078802-20200127233503698-344557318-1580139446918.png)









## SocketBase.dll

### ISessionBase

![1579318554194](/supersocket/1078802-20200127233503726-859896689-1580139446943.png)

### AppSession

![1580125323588](/supersocket/1078802-20200127233503768-2071780339-1580139446934.png)

![1579319140513](/supersocket/1078802-20200127233504166-1333428366-1580139446941.png)

对AppServer和SocketSession的包装

### ServerConfig

服务参数配置,在serverbase基类SetUp中创建

```c#
/// &lt;summary&gt;
/// Setups with the specified ip and port.
/// &lt;/summary&gt;
/// &lt;param name=&#34;ip&#34;&gt;The ip.&lt;/param&gt;
/// &lt;param name=&#34;port&#34;&gt;The port.&lt;/param&gt;
/// &lt;param name=&#34;socketServerFactory&#34;&gt;The socket server factory.&lt;/param&gt;
/// &lt;param name=&#34;receiveFilterFactory&#34;&gt;The Receive filter factory.&lt;/param&gt;
/// &lt;param name=&#34;logFactory&#34;&gt;The log factory.&lt;/param&gt;
/// &lt;param name=&#34;connectionFilters&#34;&gt;The connection filters.&lt;/param&gt;
/// &lt;param name=&#34;commandLoaders&#34;&gt;The command loaders.&lt;/param&gt;
/// &lt;returns&gt;return setup result&lt;/returns&gt;
public bool Setup(string ip, int port, ISocketServerFactory socketServerFactory = null, IReceiveFilterFactory&lt;TRequestInfo&gt; receiveFilterFactory = null, ILogFactory logFactory = null, IEnumerable&lt;IConnectionFilter&gt; connectionFilters = null, IEnumerable&lt;ICommandLoader&lt;ICommand&lt;TAppSession, TRequestInfo&gt;&gt;&gt; commandLoaders = null)
{
    return Setup(new ServerConfig
                    {
                        Ip = ip,
                        Port = port
                    },
                    socketServerFactory,
                    receiveFilterFactory,
                    logFactory,
                    connectionFilters,
                    commandLoaders);
}
```

![1578468089417](/supersocket/1078802-20200127233505015-945625614-1580139446956.png)





### RootConfig

![1578469017093](/supersocket/1078802-20200127233504737-1332814685-1580139446985.png)

- MaxWorkingThreads：最大工作线程数量
- MaxCompletionPortThreads：线程池中异步 I/O 线程的最大数目。
- PerformanceDataCollectInterval：性能数据收集间隔



### RequestInfo

类图

![1578446037999](/supersocket/1078802-20200127233504676-1620865394-1580139446996.png)

- 基类是RequestInfo，提供了两个方法Key和Body,Body是模板，由子类确定具体类型
- StringRequestInfo,在父类基础上提供了一个参数，String[] Parameters
- RequestInfo&lt;TRequestHeader, TRequestBody&gt;:提供了请求头和请求体类型的模板。
- 三个接口，key属性，body属性，heater属性



### ListenerInfo

![1579947882373](/supersocket/1078802-20200127233503853-531792950-1580139447019.png)

监听节点

### ListenerConfig

![1578474012112](/supersocket/1078802-20200127233503914-974907069-1580139447057.png)





### ReflectCommandLoader

![1578472566238](/supersocket/1078802-20200127233504892-1585734593-1580139447063.png)

- ReflectCommandLoader:通过TryLoadCommands方法反射出程序集中的所有命令

```c#
/// &lt;summary&gt;
/// Tries to load commands.
/// &lt;/summary&gt;
/// &lt;param name=&#34;commands&#34;&gt;The commands.&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public override bool TryLoadCommands(out IEnumerable&lt;TCommand&gt; commands)
{
    commands = null;
    var commandAssemblies = new List&lt;Assembly&gt;();
    if (m_AppServer.GetType().Assembly != this.GetType().Assembly)
        commandAssemblies.Add(m_AppServer.GetType().Assembly);
    string commandAssembly = m_AppServer.Config.Options.GetValue(&#34;commandAssembly&#34;);
    if (!string.IsNullOrEmpty(commandAssembly))
    {
        OnError(&#34;The configuration attribute &#39;commandAssembly&#39; is not in used, please try to use the child node &#39;commandAssemblies&#39; instead!&#34;);
        return false;
    }
    if (m_AppServer.Config.CommandAssemblies != null &amp;&amp; m_AppServer.Config.CommandAssemblies.Any())
    {
        try
        {
            var definedAssemblies = AssemblyUtil.GetAssembliesFromStrings(m_AppServer.Config.CommandAssemblies.Select(a =&gt; a.Assembly).ToArray());

            if (definedAssemblies.Any())
                commandAssemblies.AddRange(definedAssemblies);
        }
        catch (Exception e)
        {
            OnError(new Exception(&#34;Failed to load defined command assemblies!&#34;, e));
            return false;
        }
    }
    if (!commandAssemblies.Any())
    {
        commandAssemblies.Add(Assembly.GetEntryAssembly());
    }
    var outputCommands = new List&lt;TCommand&gt;();
    foreach (var assembly in commandAssemblies)
    {
        try
        {
            outputCommands.AddRange(assembly.GetImplementedObjectsByInterface&lt;TCommand&gt;());
        }
        catch (Exception exc)
        {
            OnError(new Exception(string.Format(&#34;Failed to get commands from the assembly {0}!&#34;, assembly.FullName), exc));
            return false;
        }
    }
    commands = outputCommands;
    return true;
}
}
```

### StatusInfoCollection

![1578475314305](/supersocket/1078802-20200127233503964-196619068-1580139447075.png)



### AppServerBase

![1580125365801](/supersocket/1078802-20200127233505098-1915472804-1580139447103.png)

![1578487267303](/supersocket/1078802-20200127233505413-292737059-1580139447098.png)

### AppSeverBase\&lt;TAppSession,TRequestInfo\&gt;

m_CommandContainer：命令容器

m_CommandLoaders

m_ConnectionFilters

m_GlobalCommandFilters

m_Listeners

m_SocketServerFactory：在SetupBas





## Facility.dll

### PolicyReceiveFilterFactory

![1579780070988](/supersocket/1078802-20200127233504288-266302679-1580139447121.png)

### PolicyRecieveFilter

![1579780715003](/supersocket/1078802-20200127233504792-1217497193-1580139447122.png)











### Protocol

ReceiveFilterBase

类图

![1578464691790](/supersocket/1078802-20200127233504744-769462672-1580139447191.png)

![1578464252406](/supersocket/1078802-20200127233505220-2047058269-1580139447221.png)

![1578466885687](/supersocket/1078802-20200127233504349-1337008525-1580139447373.png)

- 在SuperSocket.SocketBase.Protocol程序集中
- IReceiveFilter\&lt;TRequestInfo\&gt;接口，接收解析接口
  - Filter方法，解析会话请求的信息，参数包括，读取缓冲，偏移量，长度，是否copy，没有被解析的长度
  - LeftBufferSize属性：空余的缓冲区长度
  - NextReceiveFilter属性，下一个接收解析器
  - Reset方法，恢复初始化
  - State:解析器状态，正常和错误状态
- ArraySegmentEx\&lt;T\&gt;数段类
  - T为数组模板
  - Array数组，count:数量，Offset偏移量，From从，To到
- ArraySegmentList\&lt;T\&gt;数段列表
  - 实现了一个数组段列表
  - m_PrevSegment：当前的数段
  - m_PrevSegmentIndex，数段所在的index
- ReceiveFilterBase\&lt;TRequestInfo\&gt;
  - BufferSegments属性

## SocketEngine.dll

### PerformanceMonitor

![1580126841482](/supersocket/1078802-20200127233504532-151444867-1580139447367.png)







### SocketSession

![1579319465778](/supersocket/1078802-20200127233505321-2113835116-1580139447378.png)



在初始化里对AppSession产生依赖，同时维护Socket和SmartPool(SendingQueue[])，因为维护着socket所以发送接收数据都是通过这个类。

- 设置状态：AddStateFlag（）TryAddStateFlag（）RemoveStateFlag（）,AddStateFlag：自旋设置m_State状态，线程安全的
- m_Client:Socket
- SessionID:new guid
- LocalEndPoint:本地Id端
- RemoteEndPoint：远程终结点
- m_SendingQueuePool：实际是SmartPool类的实例，该实例维护者sendingQueue数组
- m_SendingQueue：从SmarlPool中获取一个SendingQueue实例。

方法

Initialize()方法：

- 初始化m_SendingQueuePool和m_SendingQueue

TrySend()方法：参数：IList&lt;ArraySegment\&lt;byte\&gt;&gt; segments：将segments压入sendingqueue队列并调用StartSend最终是调用**SendAsync**或**SendSync**，这个是由子类实现。

### AsyncSocketSession

在子类中维护SocketAsyncEventArgs

- SocketAsyncProxy：维护着SocketAsyncEventArgs
- m_SocketEventArgSend：发送的SocketAsyncEventArgs实例

在初始化中如果同步发送就使用m_SocketEventArgSend，并OnSendingCompleted方法绑定其Completed事件

在SendAsync()方法中将SendingQueue实例给m_SocketEventArgSend的UserToken属性，并调用m_SocketEventArgSend的**SetBuffer**和**SendAsync**方法，发送失败也调用OnSendingCompleted

SocketAsyncProxy中的Completed事件中调用ProcessReceive方法，再调用`this.AppSession.ProcessRequest(e.Buffer, e.Offset, e.BytesTransferred, true);`方法



### AsyncStreamSocketSession



### SocketFactory

![1578467105778](/supersocket/1078802-20200127233504195-344926542-1580139447406.png)

```c#
/// &lt;summary&gt;
/// Creates the socket server.
/// &lt;/summary&gt;
/// &lt;typeparam name=&#34;TRequestInfo&#34;&gt;The type of the request info.&lt;/typeparam&gt;
/// &lt;param name=&#34;appServer&#34;&gt;The app server.&lt;/param&gt;
/// &lt;param name=&#34;listeners&#34;&gt;The listeners.&lt;/param&gt;
/// &lt;param name=&#34;config&#34;&gt;The config.&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public ISocketServer CreateSocketServer&lt;TRequestInfo&gt;(IAppServer appServer, ListenerInfo[] listeners, IServerConfig config)
    where TRequestInfo : IRequestInfo
{
    if (appServer == null)
        throw new ArgumentNullException(&#34;appServer&#34;);
    if (listeners == null)
        throw new ArgumentNullException(&#34;listeners&#34;);
    if (config == null)
        throw new ArgumentNullException(&#34;config&#34;);
    switch(config.Mode)
    {
        case(SocketMode.Tcp):
            return new AsyncSocketServer(appServer, listeners);
        case(SocketMode.Udp):
            return new UdpSocketServer&lt;TRequestInfo&gt;(appServer, listeners);
        default:
            throw new NotSupportedException(&#34;Unsupported SocketMode:&#34; &#43; config.Mode);
    }
}
```

### SocketServers

![1578487919018](/supersocket/1078802-20200127233505121-377801937-1580139447431.png)

### AsyncSocketServer

- 缓存管理器m_BufferManager
- 线程安全的SocketAsyncEventArgsProxy栈



构造函数，父类

```c#
public TcpSocketServerBase(IAppServer appServer, ListenerInfo[] listeners)
    : base(appServer, listeners)
{
    var config = appServer.Config;

    uint dummy = 0;
    m_KeepAliveOptionValues = new byte[Marshal.SizeOf(dummy) * 3];
    m_KeepAliveOptionOutValues = new byte[m_KeepAliveOptionValues.Length];
    //whether enable KeepAlive
    BitConverter.GetBytes((uint)1).CopyTo(m_KeepAliveOptionValues, 0);
    //how long will start first keep alive
    BitConverter.GetBytes((uint)(config.KeepAliveTime * 1000)).CopyTo(m_KeepAliveOptionValues, Marshal.SizeOf(dummy));
    //keep alive interval
    BitConverter.GetBytes((uint)(config.KeepAliveInterval * 1000)).CopyTo(m_KeepAliveOptionValues, Marshal.SizeOf(dummy) * 2);

    m_SendTimeOut = config.SendTimeOut;
    m_ReceiveBufferSize = config.ReceiveBufferSize;
    m_SendBufferSize = config.SendBufferSize;
}
```

```c#
public override bool Start()
{
    try
    {
        int bufferSize = AppServer.Config.ReceiveBufferSize;

        if (bufferSize &lt;= 0)
            bufferSize = 1024 * 4;

        m_BufferManager = new BufferManager(bufferSize * AppServer.Config.MaxConnectionNumber, bufferSize);

        try
        {
            m_BufferManager.InitBuffer();
        }
        catch (Exception e)
        {
            AppServer.Logger.Error(&#34;Failed to allocate buffer for async socket communication, may because there is no enough memory, please decrease maxConnectionNumber in configuration!&#34;, e);
            return false;
        }

        // preallocate pool of SocketAsyncEventArgs objects
        SocketAsyncEventArgs socketEventArg;

        var socketArgsProxyList = new List&lt;SocketAsyncEventArgsProxy&gt;(AppServer.Config.MaxConnectionNumber);

        for (int i = 0; i &lt; AppServer.Config.MaxConnectionNumber; i&#43;&#43;)
        {
            //Pre-allocate a set of reusable SocketAsyncEventArgs
            socketEventArg = new SocketAsyncEventArgs();
            m_BufferManager.SetBuffer(socketEventArg);

            socketArgsProxyList.Add(new SocketAsyncEventArgsProxy(socketEventArg));
        }

        m_ReadWritePool = new ConcurrentStack&lt;SocketAsyncEventArgsProxy&gt;(socketArgsProxyList);

        if (!base.Start())
            return false;

        IsRunning = true;
        return true;
    }
    catch (Exception e)
    {
        AppServer.Logger.Error(e);
        return false;
    }
}
```

### SocketAsyncEventArgsProxy

![1579316308225](/supersocket/1078802-20200127233504284-2051183622-1580139447431.png)

SocketAsyncEventArgs的代理

维护着一个SocketAsyncEventArgs对象，并订阅了该对象的Completed事件(异步完成事件)

IsRecyclable：是否可以循环使用

OrigOffset：原始偏移量

每当异步完成的时候调用SocketAsyncEventArgs实例中的**UserToken属性**，该**属性实际上保存着SocketSession**实例，并调用SocketSession的**ProcessReceive()**和**AsyncRun()**方法；**socketSession.AsyncRun(() =&gt; socketSession.ProcessReceive(e));**

UserToken属性是在SocketAsyncEventArgsProxy的**初始化方法中定义的**

```c#
public void Initialize(IAsyncSocketSession socketSession)
{
    SocketEventArgs.UserToken = socketSession;
}
```



代理模式

![img](/supersocket/1078802-20200127233504292-415600494-1580139447454.png)

### BootstrapFactory

![1580106366845](/supersocket/1078802-20200127233504128-1448853299-1580139447486.png)

### DefaultBootStrap



![1579403530245](/supersocket/1078802-20200127233505051-410216803-1580139447503.png)

引导配置文件并通过配置实例化各个server和factory,在CreateWorkItemInstance方法通过Activator.CreateInstance(serviceType)实例化



### ConfigurationWatcher

![1580108736600](/supersocket/1078802-20200127233504295-437996370-1580139447742.png)



### SocketListenerBase

![1579439987190](/supersocket/1078802-20200127233504852-779876721-1580139447762.png)

### TcpAsyncSocketListener

监听类，由三个事件：监听错误，监听停止，新的客户端连接

m_ListrnSocket：监听Socket



### WorkItemFactoryInfoLoader

![1580119613894](/supersocket/1078802-20200127233504625-1535726155-1580139447839.png)

配置文件载入 LoadResult,载入配置的connectionFilter,logfactory,commandloaderfactory，将appserver转化成IworkItem接口，

## Common.dll

### BufferManager

![1578487794792](/supersocket/1078802-20200127233504286-854569341-1580139447912.png)

此类创建一个大缓冲区，该缓冲区可以分配给每个套接字I / O操作使用，并分配给SocketAsyncEventArgs对象。 这使得bufffer可以轻松地重用，并且可以**防止堆内存碎片化**。

BufferManager类上公开的操作不是线程安全的。我觉得这个类不需要线程安全，因为每个socket获得数据基本不会并发执行。

- m_buffer:所有的字节缓存
- m_bufferSize:单个片段的缓存大小
- m_currentIndex：当前字节在总缓存中的索引
- m_freeIndexPool:空闲索引池
- m_numBytes：缓存片段的数目

主要提供两个方法：一个是SetBuffer和FreeBuffer

SetBuffer：

- 检查空闲索引栈中是否有值，有值就直接使用空闲索引栈中的值，并将其值从栈中推出，
- 如果没有空闲栈的值就先检查剩余的缓存是否有一个片段大小，有的化就设置并改变m_currentIndex索引，没有返回false

FreeBuffer：

- 将当前索引添加到空闲索引栈中，并释放SocketAsyncEventArgs中用的缓存片段。

### ArraySegmentList

![1579323401012](/supersocket/1078802-20200127233504855-1506344631-1580139447911.png)

方法：

IndexOf：T在所有缓存中的索引

### ArraySegmentEx

- 数组，是保存着所有缓存，T[]
- 偏移，该片段在缓存中的位置
- 数量，该片段的长度

### SendingQueue

![1579329295160](/supersocket/1078802-20200127233504887-724206769-1580139447914.png)

维护ArraySegment\&lt;byte\&gt;[] globalQueue， globalQueue中包含着所有所有缓存

入栈，出战，开始入栈，开始出栈。

所有的发送队列内存片组成一个大的arraysegment，由SendingQueueSourceCreator创建，并由SmartPool维护

### SendingQueueSourceCreator

实际就是SmartPoolSourceCreator,发送队列**创建者**，默认有5个发送队列，其实每个连接一个发送队列，这边的所有sendingQueue组数是由SmartPool维护的

m_SendingQueueSize：发送队列大小，默认为5

```c#
/// &lt;summary&gt;
/// Creates the specified size.
/// &lt;/summary&gt;
/// &lt;param name=&#34;size&#34;&gt;The size.&lt;/param&gt;
/// &lt;param name=&#34;poolItems&#34;&gt;The pool items.&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public ISmartPoolSource Create(int size, out SendingQueue[] poolItems)
{
    var source = new ArraySegment&lt;byte&gt;[size * m_SendingQueueSize];//256*5
    poolItems = new SendingQueue[size];//size=256
    for (var i = 0; i &lt; size; i&#43;&#43;)
    {
        poolItems[i] = new SendingQueue(source, i * m_SendingQueueSize, m_SendingQueueSize);//SendingQueue中的source是所有的队列缓存，发送队列偏移量和发送队列容量
    }
    return new SmartPoolSource(source, size);
}
```



### SmartPool

![1579327943978](/supersocket/1078802-20200127233504824-1163773032-1580139447929.png)

其中维护了一个T（实际是SendingQueue）线程安全栈(m_GlobalStack)。由此看出SmartPool就是SendingQueue的池

m_MinPoolSize：Math.Max(config.MaxConnectionNumber / 6, 256)

m_MaxPoolSize：Math.Max(config.MaxConnectionNumber * 2, 256)

m_SourceCreator：new SendingQueueSourceCreator(config.SendingQueueSize)

m_ItemsSource：保存着SmartPoolSource[]对象,该对象实际上是所有的sendingqueue缓存。

**m_GlobalStack**：保存着单个SendingQueuep对象的数组

Initialize()：初始化函数，初始化上面的变量



### SmartPoolSource

维护所有的发送队列缓存，并保存sendingQueue的个数

Source：是object类型，实际上是ArraySegment\&lt;byte\&gt;[]，实际上是所有的sendingqueue的缓存，大小为`size*sendingqueuesize=256*5`，

Count:为默认值5

## Other.dll

### [SocketAsyncEventArgs](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socketasynceventargs?f1url=https%3A%2F%2Fmsdn.microsoft.com%2Fquery%2Fdev15.query%3FappId%3DDev15IDEF1%26l%3DZH-CN%26k%3Dk(System.Net.Sockets.SocketAsyncEventArgs);k(TargetFrameworkMoniker-.NETFramework,Version%3Dv4.5);k(DevLang-csharp)%26rd%3Dtrue&amp;view=netframework-4.8) 类

表示异步套接字操作。



## 设置IP和Port调用流程

1. 创建ServerConfig实例，RootConfig实例
2. 设置m_State状态，线程安全的，通过Interlocked.CompareExchange方法设置
3. 在setbasic中设置RootConfig,m_Name,Config，设置currentculture,设置线程池参数，设置m_socketfactory,设置textencoding,
4. 设置logfactory
5. 在setMedium中设置**ReceiveFilterFactory**，m_ConnectionFilters，m_CommandLoaders（add **ReflectCommandLoader**）
6. 在SetupAdvanced中设置BaseSecurity和Certificate，设置**listners(ListenerInfo)** 设置CommandFilterAttribute，遍历m_CommandLoaders，订阅Error,Updated事件，调用Initialize方法，通过TryLoadCommands方法获取命令集合commands，遍历命令集合添加命令到discoveredCommands集合中
7. 遍历discoveredCommands集合，将其添加到命令容器 m_CommandContainer中，使用Interlocked.Exchange方法保证线程安全
8. 在SetupFinal中设置ReceiveFilterFactory=new CommandLineReceiveFilterFactory(TextEncoding)，设置m_ServerStatus,通过socketfactory获得serverfactory。

## start调用流程

1. 调用SuperSocket.SocketBase.AppServer中start()方法，调用基类AppServerBase的start()方法，该方法中调用socketserver的start方法
2. 在socketserver的start方法中设置BufferManager,创建**SocketAsyncEventArg**,并通过buffermanager设置其buffer,并创建**SocketAsyncEventArgProxys**, **SocketAsyncEventArgProxys**集合赋值给**m_ReadWritePool**。调用SocketServer基类中的start
3. 在socketserver基类的start中创建**SendingQueuePool**并初始化,实际是初始化队列池中的sendingqueue队列；通过遍历ListenerInfo集合创建TcpAsyncSocketListener监听者，订阅监听者的**stop，error,NewClientAccepted**事件，并开始监听Listener.Start,也添加到容器中。
4. Listener.Start中创建一个监听Listen_socket和**new异步套接字SocketAsyncEventArgs**,并订阅Compeleted事件，启用socket监听，并调用AcceptAsync方法，异步完成触发compeleted事件，调用ProcessAccept方法，原来的方法异步已经触发**重新调用一下AcceptAsync方法，通过函数递归实现while**，判定acceptsocket是否正常，触发NewClientAccepted事件，
5. 事件触发AsyncSocketServer 类中的ProcessNewClient方法，从m_ReadWritePool池中取一个空闲的SocketAsyncEventArgProxy，并通过代理，socket创建AsyncSocketSession，并通过socketsession创建Appsession，在创建过程中做连接过滤，初始化app&#39;session，通过receivefactory创建receivefilter，同时初始化socketsession,主要是订阅SocketAsyncEventArgProxy中的compeleted事件。调用socketsession的start方法
6. 在socketsession中调用**startreceive**方法,中调用socket.ReceiveAsync方法，当异步完成时调用socketProxy的SocketEventArgs_Completed方法，该方法调用SocketSession的ProcessReceive方法，在该方法中执行过滤FilterRequest,执行命令，再一次**调用startReceive方法，如此不停通过异步直接实现接收循环**

## send调用流程

在订阅了NewRequestReceived事件之后，该事件会有两个参数，一个是appsession，一个是requestinfo,

appsession和socketsession完成，

在appsession的InteralSend函数中对sendtimeout进行限制。

在socketsession中将消息压入消息栈对消息进行校验，最终是通过socket.send和socket.sendasync两个方法将消息发送。

## Stop调用流程

先调用stop再调用close

socketserver的stop，释放m_ReadWritePool中所有SocketAsyncEventArgs，所有listener的stop，释放其SocketAsyncEventArgs

socket&#39;session的closed，回收所有sendingqqueue到pool中


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/01/supersocket1-source/  

