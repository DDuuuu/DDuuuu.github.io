# NetCLRVia 线程




## 线程基础

### Windows为什么要支持线程

在单核单线程系统中有两个问题：1. 如果系统需要执行某些长时间任务或死循环，就没办法响应其他任何，造成系统“假死”；2.当系统重启或任务崩溃的时候所有的数据都丢失。

针对第一个问题：多线程解决，线程的职责是对CPU进行虚拟化。所有线程共享物理CPU，Windows在某个一个时刻只将一个线程分配给CPU，一个时刻称为时间片quantum,时间一到，Windows就进行上下文切换到另一个线程。

针对第二个问题：进程解决，每个进程都被赋予一个虚拟地址控件，一个进程中使用的代码和数据不能被另一个进程使用。

Windows只能调度线程， 不能调度进程。

### 线程开销

线程有以下几个要素：

&#43; 线程内核对象(thread kernel object)：这是一个数据结构，包含：1.**线程描述**；2.**线程上下文**：存储CPU寄存器数据的内存块，不同CPU架构大小不一样，X64大概1240字节。
&#43; 线程环境块(thread environment block,TEB)：存储线程环境相关的块：包括异常处理链首，线程本地数据以及GDI和OpenGL图形相关的数据，不同CPU架构大概都4KB。
&#43; 用户模式栈(User-mode stack)：存储局部变量和实参，方法返回时线程的线程的执行地址，这个结构占用内存最大为1M。
&#43; 内核模式栈(kernel-mode stack)：操作系统内核模式函数时，需要将用户模式栈的实参，局部变量和返回地址复制到内核模式栈，用户不能访问内核模式也就不能修改实参了。不同架构CPU内存不一样，X64为24KB
&#43; DLL线程链接(attach)和分离(detach):在创建线程，会向进程中所有加载的非托管dll的DllMain方法传递一个DLL_THREAD_ATTACH标志。线程终止时也会向该方法传递一个DLL_THREAD_DETACH标志。

所有线程共享物理CPU，Windows在某个一个时刻只将一个线程分配给CPU，一个时刻称为时间片quantum(大概30ms，取决于CPU架构),时间一到，Windows就进行上下文切换到另一个线程。

上下文切换时Windows的操作：

&#43; 将CPU寄存器中的值复制到线程内核对象的线程上下文结构中。
&#43; 在现有线程集合中调度一个线程执行，如果该线程在另一个进程中，就切换进程的虚拟空间地址。
&#43; 将选中线程的线程内核对象上下文结构中的数据加载到CPU寄存器。

上下文切换还有一个操作是：CPU高速缓存的中的数据存到RAM中，新线程执行前从RAM中将数据存储到CPU高速缓存中。

线程可以提前终止时间片，什么也不做。如等待用户输入。只有用户输入的时候，CPU才会调用该线程。

垃圾回收的时候CLR会挂起所有线程。

结论：尽量少使用线程，线程上下文切换和其他的一些操作消耗了大量时间和内存。但有的时候又必须使用线程。

### CPU发展趋势

可以查看windows的进程和线程情况：任务选项卡-性能；详细信息。

CPU的发展正在面临着：多个CPU,超线程芯片，多核芯片。

### CLR线程和Windows线程

CLR线程就是映射的Windows线程的逻辑线程，完全等价。在以前Microsoft做过一些努力使得CLR线程做一些额外的操作，但是失败了。

### 使用专用线程执行异步的计算限制操作

必须要避免创建线程来执行异步的操作，相反需要使用线程池执行异步的操作。

有一些特殊情况需要创建线程：

&#43; 线程需要以非普通优先级运行。线程池都以普通优先级运行。
&#43; 线程需要表现为前台线程。
&#43; 执行长时间的任务。
&#43; 启动线程并可能调用Thread的Abort来终止它。

```c#
using System;
using System.Threading;

public static class ThreadBasics {
   public static void Main() {
      FirstThread.Go();
      BackgroundDemo.Go(true);
      BackgroundDemo.Go(false);
   }
}

internal static class FirstThread {
   public static void Go() {
      Console.WriteLine(&#34;Main thread: starting a dedicated thread &#34; &#43;
         &#34;to do an asynchronous operation&#34;);
      Thread dedicatedThread = new Thread(ComputeBoundOp);
      dedicatedThread.Start(5);

      Console.WriteLine(&#34;Main thread: Doing other work here...&#34;);
      Thread.Sleep(10000);     // Simulating other work (10 seconds)

      dedicatedThread.Join();  // Wait for thread to terminate等待线程终止
      Console.ReadLine();
   }

   // This method&#39;s signature must match the ParametizedThreadStart delegate
   private static void ComputeBoundOp(Object state) {
      // This method is executed by another thread

      Console.WriteLine(&#34;In ComputeBoundOp: state={0}&#34;, state);
      Thread.Sleep(1000);  // Simulates other work (1 second)

      // When this method returns, the dedicated thread dies
   }
}

internal static class BackgroundDemo {
   public static void Go(Boolean background) {
      // Create a new thread (defaults to Foreground)
      Thread t = new Thread(new ThreadStart(ThreadMethod));

      // Make the thread a background thread if desired
      if (background) t.IsBackground = true;

      t.Start(); // Start the thread
      return;   // NOTE: the application won&#39;t actually die for about 10 seconds
   }

   private static void ThreadMethod() {
      Thread.Sleep(10000); // Simulate 10 seconds of work
      Console.WriteLine(&#34;ThreadMethod is exiting&#34;);
   }
}
```

### 线程调度和优先级

可以使用Microsoft Spy&#43;&#43;工具查看线程的调度情况。在vs开发人员命名提示里输入spyxx就可以启动

![1582059696253](/dotnet/1582059696253.png)

Windows系统又称为抢占式多线程操作系统(preemptive multithreaded)。同时Windows不是实时操作系统。

每个线程都分配了从0（最低）到31（最高）的优先级。Windows先以轮流(round-robin)方法调用所有高优先级线程，直到该优先级所有线程不需要运行，才会调度下一层优先级中的线程。如果在执行低优先级线程的时候，有高优先级的线程需要运行，系统会立刻挂起(暂停）低优先级然后调度高优先级。

系统有一个零页线程(zero page thread)，在系统没有线程需要运行的时候，零页线程负责将系统的RAM所有空闲页清零。

为了更好的规划优先级，Microsoft将进程和线程都划分了优先级类：进程：Idle,Below Normal,Normal,Above Normal,High,Realtime;线程：Idle,Lowest,Below Normal,Normal,Above Normal,Highest,Time-Critical。

优先级类与优先级的对应关系：

![1582060261208](/dotnet/1582060261208.png)

CLR终结器以线程Time-Critical优先级运行。

设置Thread的Priority属性向其传递ThreadPriority枚举类型定义的5个值来控制线程优先级。

大多数进程都是再Normal优先级，大多数线程也是Normal，所有系统中大多数线程的优先级是8

### 前台线程和后台线程

**一个进程中所有的前台线程停止时，所有的后台线程会立刻被终止**。

每个AppDomain都可以运行一个单独应用程序，该应用程序有自己前台线程。所有前台线程都停止的时候，进程才会退出销毁。

线程可以从前台线程编程后台线程或则反向。**应用程序主线程以及通过构造一个Thread对象显式创建的任何线程都默认为前台线程。线程池的线程都默认为后台线程。进入托管环境的本机代码创建的线程都是后台线程。**

## 计算限制的异步操作

### CLR线程池基础

每个CLR拥有一个线程池，也就是一个线程池为该CLR控制的所有AppDomain共享。

CLR的工作模式：

&#43; CLR初始化时，线程池没有线程
&#43; 当有异步操作时，将一个记录项(entry)追加到线程池队列中。
&#43; 线程池代码将获得记录项将其派发(dispatch)给一个线程。
&#43; 如果没有线程就创建一个线程。
&#43; 完成任务后，线程回到线程池，进入空闲状态(不进行销毁)。

&#43; 如果线程一直空闲，一段时间后会醒来释放自己。

### 执行简单的计算限制操作

ThreadPool定义的异步方法和回调委托：

```c#
static Boolean QueueUserWorkItem(WaitCallback callBack);
static Boolean QueueUserWorkItem(WaitCallback callBack,Object state);
delegate void WaitCallback(Object state);
```

```c#
//执行
ExecutionContexts.Go();

internal static class ThreadPoolDemo {
   public static void Go() {
      Console.WriteLine(&#34;Main thread: queuing an asynchronous operation&#34;);
      ThreadPool.QueueUserWorkItem(ComputeBoundOp, 5);
      Console.WriteLine(&#34;Main thread: Doing other work here...&#34;);
      Thread.Sleep(10000);  // Simulating other work (10 seconds)
      Console.ReadLine();
   }

   // This method&#39;s signature must match the WaitCallback delegate
   private static void ComputeBoundOp(Object state) {
      // This method is executed by a thread pool thread

      Console.WriteLine(&#34;In ComputeBoundOp: state={0}&#34;, state);
      Thread.Sleep(1000);  // Simulates other work (1 second)

      // When this method returns, the thread goes back
      // to the pool and waits for another task
   }
}
//结果
Main thread: queuing an asynchronous operation
Main thread: Doing other work here...
In ComputeBoundOp: state=5

```

### 执行上下文Execution context

每个线程都**关联**一个执行上下文（Execution context）,执行上下文包括很多东西：**安全设置，宿主设置，上下文数据**。

每当初始线程使用辅助线程执行任务时，**初始线程的执行上下文会复制到辅助线程**，保证辅助线程使用相同的安全设置和宿主设置，但这一操作会影响性能。

System.Threading.ExecutionContext类可以对执行上下文的复制功能进行控制：

```c#
public sealed class ExecutionContext:IDisposable, ISerializabel
{
    [SecurityCritical]
    public static AsyncFlowControl SuppressFlow();
    public static void RestoreFlow();
    public static Boolean IsFlowSuppressed();
    // 未列出不常用的方法
}
```

**阻止执行上下文可以显著提升服务器应用程序，而客户端应用程序提升不明显**。

```c#
//调用
ExecutionContexts.Go();

internal static class ExecutionContexts 
{
   public static void Go() 
   {
      // Put some data into the Main thread’s logical call context
      CallContext.LogicalSetData(&#34;Name&#34;, &#34;Jeffrey&#34;);

      // Initiate some work to be done by a thread pool thread
      // The thread pool thread can access the logical call context data 
      ThreadPool.QueueUserWorkItem(
         state =&gt; Console.WriteLine(&#34;Name={0}&#34;, CallContext.LogicalGetData(&#34;Name&#34;)));


      // Suppress the flowing of the Main thread’s execution context
      ExecutionContext.SuppressFlow();

      // Initiate some work to be done by a thread pool thread
      // The thread pool thread can NOT access the logical call context data
      ThreadPool.QueueUserWorkItem(
         state =&gt; Console.WriteLine(&#34;Name={0}&#34;, CallContext.LogicalGetData(&#34;Name&#34;)));

      // Restore the flowing of the Main thread’s execution context in case 
      // it employs more thread pool threads in the future
      ExecutionContext.RestoreFlow();
      SecurityExample();
   }

   private static void SecurityExample() 
   {
      ProxyType highSecurityObject = new ProxyType();
      highSecurityObject.AttemptAccess(&#34;High&#34;);   // Works OK

      PermissionSet grantSet = new PermissionSet(PermissionState.None);
      grantSet.AddPermission(new SecurityPermission(SecurityPermissionFlag.Execution));
      AppDomain lowSecurityAppDomain = AppDomain.CreateDomain(&#34;LowSecurity&#34;, null, new AppDomainSetup() { ApplicationBase = AppDomain.CurrentDomain.BaseDirectory }, grantSet, null);
      ProxyType lowSecurityObject = (ProxyType)lowSecurityAppDomain.CreateInstanceAndUnwrap(typeof(ProxyType).Assembly.ToString(), typeof(ProxyType).FullName);
      lowSecurityObject.DoSomething(highSecurityObject);
      Console.ReadLine();
   }

   public sealed class ProxyType : MarshalByRefObject 
   {
      // This method executes in the low-security AppDomain
      public void DoSomething(ProxyType highSecurityObject) 
      {
         AttemptAccess(&#34;High-&gt;Low&#34;); // Throws

         // Attempt access from the high-security AppDomain via the low-security AppDomain: Throws
         highSecurityObject.AttemptAccess(&#34;High-&gt;Low-&gt;High&#34;);

         // Have the high-security AppDomain via the low-security AppDomain queue a work item to 
         // the thread pool normally (without suppressing the execution context): Throws
         highSecurityObject.AttemptAccessViaThreadPool(false, &#34;TP (with EC)-&gt;High&#34;);

         // Wait a bit for the work item to complete writing to the console before starting the next work item
         Thread.Sleep(1000);

         // Have the high-security AppDomain via the low-security AppDomain queue a work item to 
         // the thread pool suppressing the execution context: Works OK
         highSecurityObject.AttemptAccessViaThreadPool(true, &#34;TP (no EC)-&gt;High&#34;);
      }

      public void AttemptAccessViaThreadPool(Boolean suppressExecutionContext, String stack) 
      {
         // Since the work item is queued from the high-security AppDomain, the thread pool 
         // thread will start in the High-security AppDomain with the low-security AppDomain&#39;s 
         // ExecutionContext (unless it is suppressed when queuing the work item)
         using (suppressExecutionContext ? (IDisposable)ExecutionContext.SuppressFlow() : null) {
            ThreadPool.QueueUserWorkItem(AttemptAccess, stack);
         }
      }

      public void AttemptAccess(Object stack) 
      {
         String domain = AppDomain.CurrentDomain.IsDefaultAppDomain() ? &#34;HighSecurity&#34; : &#34;LowSecurity&#34;;
         Console.Write(&#34;Stack={0}, AppDomain={1}, Username=&#34;, stack, domain);
         try {
            Console.WriteLine(Environment.GetEnvironmentVariable(&#34;USERNAME&#34;));
         }
         catch (SecurityException) {
            Console.WriteLine(&#34;(SecurityException)&#34;);
         }
      }
   }
}

//结果
Name=Jeffrey
Name=
Stack=High, AppDomain=HighSecurity, Username=dujinfeng
Stack=High-&gt;Low, AppDomain=LowSecurity, Username=(SecurityException)
Stack=High-&gt;Low-&gt;High, AppDomain=HighSecurity, Username=(SecurityException)
Stack=TP (with EC)-&gt;High, AppDomain=HighSecurity, Username=(SecurityException)
Stack=TP (no EC)-&gt;High, AppDomain=HighSecurity, Username=dujinfeng

```

### 协作式取消和超时

协作式取消就是需要显式调用取消。

怎样显式调用取消：

&#43; System.Threading.CancellationTokenSource包含了取消有关的所有状态，**其为引用类型**，可以利用它的Token属性获得CancellationToken值类型来进行取消操作。

```c#
public sealed class CancellationTokenSource:IDisposable //引用类型
{
	public CancellationTokenSource();
	public void Dispose();//释放资源比如WaitHandle
	public Boolean IsCancellationRequested {get;}
	
	public void Cancel();
	public void Cancel(Boolean throwOnFirstException);
}
```

CancellationToken**轻量级值类型**，有一个字段包含对**CancellationTokenSource**对象的引用

```c#
public struct CancellationToken //一个值类型
{ 
	public static CancellationToken None {get;} //很好用
	public Boolean IsCancellationRequested {get;} //通过非Task调用的操作调用
	public void ThrowIfCancellationRequested()；//通过Task调用的操作调用
	// CancellationTokenSource取消时，WaitHandIe会收到信号
	public WaitHandle WaitHandle {get;}
	//GetHashCode,Equals,operator==和operaor!=成员未列出
	public Boolean CanBeCanceLed {get;}／／很少使用
	public CancellationTokenRegistration Register(Action&lt;Object&gt; callback, Object state, Boolean useSynchromzatronContext);//未列出更简单的重载版本
}
```

```c#
TaskDemo.Go();

internal static class CancellationDemo {
   public static void Go() {
      CancellingAWorkItem();
      Register();
      Linking();
   }

   private static void CancellingAWorkItem() {
      CancellationTokenSource cts = new CancellationTokenSource();

      // Pass the CancellationToken and the number-to-count-to into the operation
       //将CancellationToken和值转换为操作
      ThreadPool.QueueUserWorkItem(o =&gt; Count(cts.Token, 1000));

      Console.WriteLine(&#34;Press &lt;Enter&gt; to cancel the operation.&#34;);
      Console.ReadLine();
      cts.Cancel();  // If Count returned already, Cancel has no effect on it
       				//如果已经返回了Count，则Cancel对其无效
      // Cancel returns immediately, and the method continues running here...
       //取消立即返回，该方法在此处继续运行...

      Console.ReadLine();  // For testing purposes
   }

   private static void Count(CancellationToken token, Int32 countTo) {
      for (Int32 count = 0; count &lt; countTo; count&#43;&#43;) {
         if (token.IsCancellationRequested) {//必须手动在代码中监测取消，这个比较烦
            Console.WriteLine(&#34;Count is cancelled&#34;);
            break; // Exit the loop to stop the operation
         }

         Console.WriteLine(count);
         Thread.Sleep(200);   // For demo, waste some time
      }
      Console.WriteLine(&#34;Count is done&#34;);
   }

   private static void Register() {
      var cts = new CancellationTokenSource();
      cts.Token.Register(() =&gt; Console.WriteLine(&#34;Canceled 1&#34;));
      cts.Token.Register(() =&gt; Console.WriteLine(&#34;Canceled 2&#34;));

      // To test, let&#39;s just cancel it now and have the 2 callbacks execute
       //为了测试，让我们现在就取消它并执行2个回调
      cts.Cancel();
   }

   private static void Linking() {
      // Create a CancellationTokenSource
      var cts1 = new CancellationTokenSource();
      cts1.Token.Register(() =&gt; Console.WriteLine(&#34;cts1 canceled&#34;));

      // Create another CancellationTokenSource
      var cts2 = new CancellationTokenSource();
      cts2.Token.Register(() =&gt; Console.WriteLine(&#34;cts2 canceled&#34;));

      // Create a new CancellationTokenSource that is canceled when cts1 or ct2 is canceled
       //创建一个新的CancellationTokenSource，该对象在cts1或ct2被取消时被取消
      /* Basically, Constructs a new CTS and registers callbacks with all he passed-in tokens. Each callback calls Cancel(false) on the new CTS */
       /*基本上，构造一个新的CTS并使用他传入的所有令牌注册回调。 每个回调在新的CTS上调用Cancel（false）*/
      var ctsLinked = CancellationTokenSource.CreateLinkedTokenSource(cts1.Token, cts2.Token);
      ctsLinked.Token.Register(() =&gt; Console.WriteLine(&#34;linkedCts canceled&#34;));

      // Cancel one of the CancellationTokenSource objects (I chose cts2)
      cts2.Cancel();

      // Display which CancellationTokenSource objects are canceled
      Console.WriteLine(&#34;cts1 canceled={0}, cts2 canceled={1}, ctsLinked canceled={2}&#34;,
         cts1.IsCancellationRequested, cts2.IsCancellationRequested, ctsLinked.IsCancellationRequested);
   }
}

//结果

```

**不可取消操作可以使用CancellationToken的静态None属性。该属性返回一个特殊的CancellationToken对象，该对象的IsCancellationRequest总是返回false。**

**可调用CancellationTokenSource的Register方法注册一个在线程取消时调用的方法**。向该方法传递一个Action\&lt;Object\&gt;委托和一个bool值（名为useSynchronizationContext），**bool值用来控制是否使用调用线程（false:调用register的线程**）的SynchronizationContext（回调方法会被**send(同步调用)**给它，而不是post（异步调用））来调用委托，使用为true，不使用为false，如果不使用回调方法会被顺序执行。

Rigister中注册多个回调方法发生异常，要看一下CancellationTokenSource的Cancel方法，如果向该方法传递true，那么异常会阻塞回调方法的调用，如果是false，回调方法不会被阻塞，所有异常会被添加到集合中。该集合在Cancel抛出异常AggregateException的实例InnerExceptions属性中

Register方法返回一个CancellationTokenRegistration对象。

```c#
public struct CancellationTokenRegistration:
{
	IEquatable&lt;CancellationTokenRegistration&gt;, IDisposable
	{
		public void Dispose();
		// GetHashCode, Equals, operator==和operator!=成员未列出
	}
}
```

调用其Dispose可以取消回调函数调用。

可以使用CancellationTokenSource的**CreateLinkedTokenSource**方法连接一个CancellationTokenSource对象，返回一个CancellationTokenSource对象，**任何一个对象被取消其他对象就会被取消。**

```c#
CancellationDemo.Go();

internal static class CancellationDemo {
   public static void Go() {
      CancellingAWorkItem();
      Register();
      Linking();
   }

   private static void CancellingAWorkItem() {
      CancellationTokenSource cts = new CancellationTokenSource();

      // Pass the CancellationToken and the number-to-count-to into the operation
      ThreadPool.QueueUserWorkItem(o =&gt; Count(cts.Token, 1000));

      Console.WriteLine(&#34;Press &lt;Enter&gt; to cancel the operation.&#34;);
      Console.ReadLine();
      cts.Cancel();  // If Count returned already, Cancel has no effect on it
      // Cancel returns immediately, and the method continues running here...

      Console.ReadLine();  // For testing purposes
   }

   private static void Count(CancellationToken token, Int32 countTo) {
      for (Int32 count = 0; count &lt; countTo; count&#43;&#43;) {
         if (token.IsCancellationRequested) {
            Console.WriteLine(&#34;Count is cancelled&#34;);
            break; // Exit the loop to stop the operation
         }

         Console.WriteLine(count);
         Thread.Sleep(200);   // For demo, waste some time
      }
      Console.WriteLine(&#34;Count is done&#34;);
   }

   private static void Register() {
      var cts = new CancellationTokenSource();
      cts.Token.Register(() =&gt; Console.WriteLine(&#34;Canceled 1&#34;));
      cts.Token.Register(() =&gt; Console.WriteLine(&#34;Canceled 2&#34;));

      // To test, let&#39;s just cancel it now and have the 2 callbacks execute
      cts.Cancel();
   }

   private static void Linking() {
      // Create a CancellationTokenSource
      var cts1 = new CancellationTokenSource();
      cts1.Token.Register(() =&gt; Console.WriteLine(&#34;cts1 canceled&#34;));

      // Create another CancellationTokenSource
      var cts2 = new CancellationTokenSource();
      cts2.Token.Register(() =&gt; Console.WriteLine(&#34;cts2 canceled&#34;));

      // Create a new CancellationTokenSource that is canceled when cts1 or ct2 is canceled
      /* Basically, Constructs a new CTS and registers callbacks with all he passed-in tokens. Each callback calls Cancel(false) on the new CTS */
      var ctsLinked = CancellationTokenSource.CreateLinkedTokenSource(cts1.Token, cts2.Token);
      ctsLinked.Token.Register(() =&gt; Console.WriteLine(&#34;linkedCts canceled&#34;));

      // Cancel one of the CancellationTokenSource objects (I chose cts2)
      cts2.Cancel();

      // Display which CancellationTokenSource objects are canceled
      Console.WriteLine(&#34;cts1 canceled={0}, cts2 canceled={1}, ctsLinked canceled={2}&#34;,
         cts1.IsCancellationRequested, cts2.IsCancellationRequested, ctsLinked.IsCancellationRequested);
   }
}

//结果
Press &lt;Enter&gt; to cancel the operation.
0
1
2
3

Count is cancelled
Count is done

Canceled 2
Canceled 1
linkedCts canceled
cts2 canceled
cts1 canceled=False, cts2 canceled=True, ctsLinked canceled=True
请按任意键继续. . .

```

CancellationTokenSource提供了指定时间后自动取消的对象方法**CancelAfter**：

```c#
public sealed class CancellationTokenSource:IDisposab1e{//一个引用类型
public CancellationTokenSource（Int32 millisecondsDeiay)：
public CancellationTokenSource (TimeSpandelay)；
public void CancelAfter (Int32 millisecondsDeiay);
public Void CancelAfter (TimeSpandelay)；
}
```

### 任务

ThreadPool的QueueUserWorkItem方法的弊端

&#43; 不知道线程什么时候完成
&#43; 不能获得线程返回值

System.Threading.Tasks可以解决这些问题。

```c#
   private static void UsingTaskInsteadOfQueueUserWorkItem() {
      ThreadPool.QueueUserWorkItem(ComputeBoundOp, 5);
      new Task(ComputeBoundOp, 5).Start();
      Task.Run(() =&gt; ComputeBoundOp(5));
   }
```

Task构造函数接收一个Action或Action\&lt;Object\&gt;委托，静态方法Run需要传递一个Action或Func\&lt;TResult\&gt;委托。

还可以使用**TaskCreationOptions**标志来控制Task的执行。

![1582235272288](/dotnet/1582235272288.png)

#### 等待任务完成并获取结果

```c#
   private static void WaitForResult() {
      // Create and start a Task
      Task&lt;Int32&gt; t = new Task&lt;Int32&gt;(n =&gt; Sum((Int32)n), 10000);

      // You can start the task sometime later
      t.Start();

      // Optionally, you can explicitly wait for the task to complete
       //（可选）您可以显式等待任务完成
      t.Wait(); // FYI(for you information): Overloads exist accepting a timeout/CancellationToken
       //存在过载，接受超时/ CancellationToken

      // Get the result (the Result property internally calls Wait) 
      Console.WriteLine(&#34;The sum is: &#34; &#43; t.Result);   // An Int32 value
   }
```

Task\&lt;TResult\&gt;对象可以获得Task返回的结果。

Wait会使调用Wait的线程阻塞，等待Task结束，如果Task没有开始就用调用Wait线程执行Task。

如果Task任务抛出异常，不会被阻塞，直接存储到异常集合中，**在t.Wait或t.Result会抛出System.AggregateException异常**。该异常的**InnerExceptions**（返回ReadOnlyCollection\&lt;Exception\&gt;）和InnerException(从Exception集成)

System.AggregateException的一些功能：

&#43; GetBaseException方法，返回最内层AggregateException

&#43; Flatten属性，新建一个AggregateException，其InnerExceptions属性包含一个异常列表

&#43; Handle方法，使AggregateException中每个异常都调用一个回调方法

如果不调用Wait或Result方法或不查询task的Exception属性，就不会注意到异常，可以向**TaskSchedule的静态UnobservedTaskException事件登记一个回调方法**，如果在task被垃圾回收时发现有没有被注意的异常，会触发这个事件，并传递一个UnobservedTaskException对象，其包含一个AggregateException对象。

```c#
   private static void UnobservedException() {
      TaskScheduler.UnobservedTaskException &#43;= (sender, e) =&gt; {
         //e.SetObserved();
         Console.WriteLine(&#34;Unobserved exception {0}&#34;, e.Exception, e.Observed);
      };

      Task parent = Task.Factory.StartNew(() =&gt; {
         Task child = Task.Factory.StartNew(() =&gt; { throw new InvalidOperationException(); }, TaskCreationOptions.AttachedToParent);
         // Child’s exception is observed, but not from its parent
         child.ContinueWith((task) =&gt; { var _error = task.Exception; }, TaskContinuationOptions.OnlyOnFaulted);
      });

      // If we do not Wait, the finalizer thread will throw an unhandled exception terminating the process
      //parent.Wait(); // throws AggregateException(AggregateException(InvalidOperationException))

      parent = null;
      Console.ReadLine();  // Wait for the tasks to finish running

      GC.Collect();
      Console.ReadLine();
   }

//结果
/*
Unobserved exception System.AggregateException: 未通过等待任务或访问任务的 Exception 属性观察到任务的异常。因此，终结器 线程重新引发了未观察到的异常。 ---&gt; System.AggregateException: 发生一个或多个错误。 ---&gt; System.InvalidOperationException: 对象的当前状态使该操作无效。
   在 TaskDemo.&lt;&gt;c.&lt;UnobservedException&gt;b__8_2() 位置 C:\Users\dujinfeng\Desktop\NetCLRVia\CLR-via-C-4th-Edition-Code\Ch27-1-ComputeOps.cs:行号 355
   在 System.Threading.Tasks.Task.InnerInvoke()
   在 System.Threading.Tasks.Task.Execute()
   --- 内部异常堆栈跟踪的结尾 ---
   --- 内部异常堆栈跟踪的结尾 ---
---&gt; (内部异常 #0) System.AggregateException: 发生一个或多个错误。 ---&gt; System.InvalidOperationException: 对象的当前状态使该操作无效。
   在 TaskDemo.&lt;&gt;c.&lt;UnobservedException&gt;b__8_2() 位置 C:\Users\dujinfeng\Desktop\NetCLRVia\CLR-via-C-4th-Edition-Code\Ch27-1-ComputeOps.cs:行号 355
   在 System.Threading.Tasks.Task.InnerInvoke()
   在 System.Threading.Tasks.Task.Execute()
   --- 内部异常堆栈跟踪的结尾 ---
---&gt; (内部异常 #0) System.InvalidOperationException: 对象的当前状态使该操作无效。
   在 TaskDemo.&lt;&gt;c.&lt;UnobservedException&gt;b__8_2() 位置 C:\Users\dujinfeng\Desktop\NetCLRVia\CLR-via-C-4th-Edition-Code\Ch27-1-ComputeOps.cs:行号 355
   在 System.Threading.Tasks.Task.InnerInvoke()
   在 System.Threading.Tasks.Task.Execute()&lt;---
&lt;---
*/

请按任意键继续. . .
```

Task还提供了两个静态方法，允许等待一个Task对象数组

&#43; **WaitAny**，阻塞调用线程，等待任何一个Task完成，返回一个Int32的task数组索引，超时返回-1，通过CancellationToken取消会抛出Operation&#39;Cancel&#39;Exception异常。
&#43; **WaitAll**，阻塞调用线程，等待所有Task完成。其他和WaitAny一样。

#### 取消任务

```c#
   private static void Cancel() {
      CancellationTokenSource cts = new CancellationTokenSource();
      Task&lt;Int32&gt; t = Task.Run(() =&gt; Sum(cts.Token, 10000), cts.Token);

      // Sometime later, cancel the CancellationTokenSource to cancel the Task
      cts.Cancel();//这是一个异步请求

      try {
         // If the task got canceled, Result will throw an AggregateException
         Console.WriteLine(&#34;The sum is: &#34; &#43; t.Result);   // An Int32 value
      }
      catch (AggregateException ae) {
         // Consider any OperationCanceledException objects as handled. 
         // Any other exceptions cause a new AggregateException containing 
         // only the unhandled exceptions to be thrown          
         ae.Handle(e =&gt; e is OperationCanceledException);

         // If all the exceptions were handled, the following executes
         Console.WriteLine(&#34;Sum was canceled&#34;);
      }
   }
   private static Int32 Sum(CancellationToken ct, Int32 n) {
      Int32 sum = 0;
      for (; n &gt; 0; n--) {

         // The following line throws OperationCanceledException when Cancel 
         // is called on the CancellationTokenSource referred to by the token
         ct.ThrowIfCancellationRequested();

         //Thread.Sleep(0);   // Simulate taking a long time
         checked { sum &#43;= n; }
      }
      return sum;
   }
```

#### 任务完成启动新任务

**wait和result都可能会阻塞调用线程，可以使用ContinueWith来使得任务完成启动另一个Task，做Task完成做的事情。**

```c#
   private static void ContinueWith() {
      // Create and start a Task, continue with another task
      Task&lt;Int32&gt; t = Task.Run(() =&gt; Sum(10000));

      // ContinueWith returns a Task but you usually don&#39;t care
      Task cwt = t.ContinueWith(task =&gt; Console.WriteLine(&#34;The sum is: &#34; &#43; task.Result));
      cwt.Wait();  // For the testing only
   }
   private static Int32 Sum(Int32 n) {
      Int32 sum = 0;
      for (; n &gt; 0; n--) checked { sum &#43;= n; }
      return sum;
   }
```

如果在调用ContinueWith之前Task已经完成，直接显示结果。

Task对象内部包含ContinueWith任务的集合，Task可以多次调用ContinueWith方法，在调用ContunueWith时可以传递**TaskContinuationOptions**枚举来控制Task任务。

```c#
[Flags,Serializab1e]
public enum TaskContinuationOptions{
None            = 0x0000,//默认

//提议TaskSchedu1er你希望该任务尽快执行，
PreferFairness   = 0x0001,
//提议TaskSchedu1er应尽可能地创建线程池线程
LongRunning      = 0x0002,
//该提议总是被采纳：将一个Task和它的父Task关联(稍后讨论)
AttachedToParent  =0x0004,
//任务试图和这个父任务连接将抛出一个invalidOperationException
DenyChi1dAttach   = 0x0008,
//强迫了任务使用默认调度器而不是父仃务的调度器
HideScheduler     = 0x0010,
//除非前置任务(antecedent task)完成，否则禁正延续任务完成（取消）
LazyCance11ation  = 0x0020,
//这个标志指出你希望由执行第一个任务的线程执行
//ContinueWith任务。第一个任务完成后，调用
//continuewith的线程接着执行ContinueWith任务
ExecuteSynchronously  = 0x80000,
//这些标志指出在什么情况下运行ContinueWith任务
NotOnRanToComp1etion  = 0x10000,
NotOnFaulted   = 0x20000,
NotOnCance1ed  = 0x4000,
//这些标志是以上三个标志的便利组合
OnlyOnCance1ed = NotOnRanToCompletion|NotOnFaulted,
OnlyOnFau1ted  = NotOnRanToCompletion|NotOnCance1ed,
OnlyOnRanToComp1etion = NotOnFaulted|NotOnCanceled,
}
```



```c#
   private static void MultipleContinueWith() {
      // Create and start a Task, continue with multiple other tasks
      Task&lt;Int32&gt; t = Task.Run(() =&gt; Sum(10000));

      // Each ContinueWith returns a Task but you usually don&#39;t care
      t.ContinueWith(task =&gt; Console.WriteLine(&#34;The sum is: &#34; &#43; task.Result),
         TaskContinuationOptions.OnlyOnRanToCompletion);
      t.ContinueWith(task =&gt; Console.WriteLine(&#34;Sum threw: &#34; &#43; task.Exception),
         TaskContinuationOptions.OnlyOnFaulted);
      t.ContinueWith(task =&gt; Console.WriteLine(&#34;Sum was canceled&#34;),
         TaskContinuationOptions.OnlyOnCanceled);

      try {
         t.Wait();  // For the testing only
      }
      catch (AggregateException) {
      }
   }
```

#### 任务可以启动子任务

```c#
   private static void ParentChild() {
      Task&lt;Int32[]&gt; parent = new Task&lt;Int32[]&gt;(() =&gt; {
         var results = new Int32[3];   // Create an array for the results

         // This tasks creates and starts 3 child tasks
         new Task(() =&gt; results[0] = Sum(10000), TaskCreationOptions.AttachedToParent).Start();
         new Task(() =&gt; results[1] = Sum(20000), TaskCreationOptions.AttachedToParent).Start();
         new Task(() =&gt; results[2] = Sum(30000), TaskCreationOptions.AttachedToParent).Start();

         // Returns a reference to the array (even though the elements may not be initialized yet)
         return results;
      });

      // When the parent and its children have run to completion, display the results
      var cwt = parent.ContinueWith(parentTask =&gt; Array.ForEach(parentTask.Result, Console.WriteLine));

      // Start the parent Task so it can start its children
      parent.Start();

      cwt.Wait(); // For testing purposes
   }
   private static Int32 Sum(Int32 n) {
      Int32 sum = 0;
      for (; n &gt; 0; n--) checked { sum &#43;= n; }
      return sum;
   }
```

#### Task内部揭秘

Task对象是很消耗资源，**如果不需要一些额外的功能ThreadPool.QueueUserWorkItem的利用率更高**。

Task的组成:

&#43; Int32 ID:只读唯一Id属性，创建时初始化为0，首次查询时赋值，该值从1开始递增，VS调试器可以查看该值，Task.CurrentId可在即时窗口调用，并在并行任务查找该值找到自己的任务。
&#43; Int32 执行状态，status属性，返回TaskStatus枚举。首次构造Task状态是Created，Task启动状态WaitingToRun，Task在一个线程运行状态Running，还有几个Task属性可以提供状态的查询。

```c#
public enum TaskStatus
{
//这些标志指出一个Task在其生命期内的状态
Created,//任务己显式创建：可以手动start()这个任务
WaitingForActivation,//任务已隐式创建；会自动开始
WaitingToRun,//任务己调度，但尚未运行
Running,//任务正在运行
WaitingForChildrenToComplete,//任务正在等待它的子任务完成，子任务完成后它才完成
//任务的最终状态是以下三个之一：
RanToCompletion,
Canceled,
Faulted
}
```

&#43; 对父任务的引用
&#43; 创建时指定的TaskScheduler引用
&#43; 回调方法的引用
&#43; 传给回调方法对象的引用（通过AsyncState属性查询）
&#43; ExecutionContext的引用
&#43; ManualResetEventSlim的引用
&#43; 补充状态的引用，补充状态包括：CancellationToken、ContinueWithTask对象集合，为异常准备的Task集合

Task和Task\&lt;TResult\&gt;实现了IDisposable接口，Dispose主要内容是关闭ManualResetEventSlim,建以不要显示调用。

#### 任务工厂

Task工厂主要是启动一组相同配置和相同参数的Tasks设计的。TaskFactory\&lt;Int32\&gt;,创建Task\&lt;Int32\&gt;的工厂，共同享有CancellationTokenSource标记，创建的所有延续Task都以同步方式进行，所有Task对象都使用默认TaskScheduler。

```c#
   private static void TaskFactory() {
      Task parent = new Task(() =&gt; {
         var cts = new CancellationTokenSource();
         var tf = new TaskFactory&lt;Int32&gt;(cts.Token, TaskCreationOptions.AttachedToParent, TaskContinuationOptions.ExecuteSynchronously, TaskScheduler.Default);

         // 启动taskfactory创建3个子任务
         var childTasks = new[] {
            tf.StartNew(() =&gt; Sum(cts.Token, 10000)),
            tf.StartNew(() =&gt; Sum(cts.Token, 20000)),
            tf.StartNew(() =&gt; Sum(cts.Token, Int32.MaxValue))  // Too big, throws OverflowException
         };

         // 任何子任务抛出异常，就取消其余子任务
         for (Int32 task = 0; task &lt; childTasks.Length; task&#43;&#43;)
            childTasks[task].ContinueWith(t =&gt; cts.Cancel(), TaskContinuationOptions.OnlyOnFaulted);

         // 所有子任务完成后，从未出错/未取消的任务获取返回的最大值
         // 然后将最大值传给另一个task来显示最大结果
         tf.ContinueWhenAll(
            childTasks,
            completedTasks =&gt; completedTasks.Where(t =&gt; t.Status == TaskStatus.RanToCompletion).Max(t =&gt; t.Result),
            CancellationToken.None)
            .ContinueWith(t =&gt; Console.WriteLine(&#34;The maximum is: &#34; &#43; t.Result),
               TaskContinuationOptions.ExecuteSynchronously).Wait(); // 等待测试
      });

      // 子任务完成后，也显示任何未处理的异常
      parent.ContinueWith(p =&gt; {
         // 所有文本放到一个StringBuilder中，并只调用Console.WriteLine一次
         // 因为这个任务可能和上面的任务并行执行
         StringBuilder sb = new StringBuilder(&#34;The following exception(s) occurred:&#34; &#43; Environment.NewLine);
         foreach (var e in p.Exception.Flatten().InnerExceptions)
            sb.AppendLine(&#34;   &#34; &#43; e.GetType().ToString());
         Console.WriteLine(sb.ToString());
      }, TaskContinuationOptions.OnlyOnFaulted);

      // 启动父任务，使它能启动子任务
      parent.Start();

      try {
         parent.Wait(); // For testing purposes
      }
      catch (AggregateException) {
      }
   }
   private static Int32 Sum(CancellationToken ct, Int32 n) {
      Int32 sum = 0;
      for (; n &gt; 0; n--) {

         // The following line throws OperationCanceledException when Cancel 
         // is called on the CancellationTokenSource referred to by the token
         ct.ThrowIfCancellationRequested();

         //Thread.Sleep(0);   // Simulate taking a long time
         checked { sum &#43;= n; }
      }
      return sum;
   }
```

#### 任务调度器

FCL提供了两个从TaskScheduler类型派生的调度器：**线程调度器(thread pool task scheduler)**和**同步上下文调度器**(synchronization context task scheduler)。默认使用线程调度器。

**同步上下文调度器更适合GUI程序。该调度器不适用线程池，通过TaskScheduler.FormCurretSynchronizationContext方法获得该调度器。**

```c#
   private static void SynchronizationContextTaskScheduler() {
      var f = new MyForm();
      System.Windows.Forms.Application.Run();
   }
   
   private sealed class MyForm : System.Windows.Forms.Form {
      private readonly TaskScheduler m_syncContextTaskScheduler;
      public MyForm() {
         // Get a reference to a synchronization context task scheduler
         m_syncContextTaskScheduler = TaskScheduler.FromCurrentSynchronizationContext();

         Text = &#34;Synchronization Context Task Scheduler Demo&#34;;
         Visible = true; Width = 400; Height = 100;
      }

      private CancellationTokenSource m_cts;

      protected override void OnMouseClick(System.Windows.Forms.MouseEventArgs e) {
         if (m_cts != null) {    // An operation is in flight, cancel it
            m_cts.Cancel();
            m_cts = null;
         } else {                // An operation is not in flight, start it
            Text = &#34;Operation running&#34;;
            m_cts = new CancellationTokenSource();

            // This task uses the default task scheduler and executes on a thread pool thread
            Task&lt;Int32&gt; t = Task.Run(() =&gt; Sum(m_cts.Token, 20000), m_cts.Token);

            // These tasks use the synchronization context task scheduler and execute on the GUI thread
            t.ContinueWith(task =&gt; Text = &#34;Result: &#34; &#43; task.Result,
               CancellationToken.None, TaskContinuationOptions.OnlyOnRanToCompletion,
               m_syncContextTaskScheduler);

            t.ContinueWith(task =&gt; Text = &#34;Operation canceled&#34;,
               CancellationToken.None, TaskContinuationOptions.OnlyOnCanceled,
               m_syncContextTaskScheduler);

            t.ContinueWith(task =&gt; Text = &#34;Operation faulted&#34;,
               CancellationToken.None, TaskContinuationOptions.OnlyOnFaulted,
               m_syncContextTaskScheduler);
         }
         base.OnMouseClick(e);
      }
   }

   private static Int32 Sum(CancellationToken ct, Int32 n) {
      Int32 sum = 0;
      for (; n &gt; 0; n--) {

         // The following line throws OperationCanceledException when Cancel 
         // is called on the CancellationTokenSource referred to by the token
         ct.ThrowIfCancellationRequested();

         //Thread.Sleep(0);   // Simulate taking a long time
         checked { sum &#43;= n; }
      }
      return sum;
   }
```

Parallel Extensions Extras 提供了额外的调度器

### Parallel的静态For,ForEach,和Invoke方法

多个方法并行或顺序执行,使用线程池，内部使用Task。同时调用线程会将自己挂起，直到所有工作完成。任何一个Task出现异常，都会抛出AggregateException异常。多个线程需要添加线程同步锁保护共享数据。

```c#
   private static void SimpleUsage() {
      // One thread performs all this work sequentially
      for (Int32 i = 0; i &lt; 1000; i&#43;&#43;) DoWork(i);

      // The thread pool’s threads process the work in parallel
      Parallel.For(0, 1000, i =&gt; DoWork(i));

      var collection = new Int32[0];
      // One thread performs all this work sequentially
      foreach (var item in collection) DoWork(item);

      // The thread pool’s threads process the work in parallel
      Parallel.ForEach(collection, item =&gt; DoWork(item));

      // One thread executes all the methods sequentially
      Method1();
      Method2();
      Method3();

      // The thread pool’s threads execute the methods in parallel
      Parallel.Invoke(
         () =&gt; Method1(),
         () =&gt; Method2(),
         () =&gt; Method3());
   }
   private static void DoWork(Int32 i) { }
   private static void Method1() { }
   private static void Method2() { }
   private static void Method3() { }
```

ParallelOptions对象用于控制Parallel

```c#
public class ParallelOptions
{
	public ParallelOptions();
	//允许取消操作
	public CancellationToken CancellationToken{get;set;}//默认为CancellationToken.None
    //允许指定可以并发操作的最大工作项数目
    public Int32MaxDegreeOfParallelism {get;set;}//默认为-1(可用CPU数)
    //允许指定要使用哪个TaskScheduler
    public TaskSchedulerTaskScheduler {get;set;}//默认为TaskScheduler.Default
}
```

有一些For和Foreach的重载版本允许3个委托

&#43; Task初始化委托(localInit):每个任务被处理前调用
&#43; Task主体委托(body):每个任务处理时调用
&#43; Task总结委托(localFinally):每个任务结束的时候调用

计算一个目录中所有文件字节的长度.

```c#
      String path = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
      Console.WriteLine(&#34;The total bytes of all files in {0} is {1:N0}.&#34;,
         path, DirectoryBytes(@path, &#34;*.*&#34;, SearchOption.TopDirectoryOnly));
         

   private static Int64 DirectoryBytes(String path, String searchPattern, SearchOption searchOption) {
      var files = Directory.EnumerateFiles(path, searchPattern, searchOption);
      Int64 masterTotal = 0;

      ParallelLoopResult result = Parallel.ForEach&lt;String, Int64&gt;(files,
         () =&gt; { // localInit: 每个任务开始之前调用一次
            // 每个任务开始之前,总计值都初始化为0
            return 0;   // 将taskLocalTotal初始值为 0
         },

         (file, parallelLoopState, index, taskLocalTotal) =&gt; { // body: 每个工作项调用一次
            // 获得这个文件的大小,把它添加到这个任务的累加值上
            Int64 fileLength = 0;
            FileStream fs = null;
            try {
               fs = File.OpenRead(file);
               fileLength = fs.Length;
            }
            catch (IOException) { /* Ignore any files we can&#39;t access */ }
            finally { if (fs != null) fs.Dispose(); }
            return taskLocalTotal &#43; fileLength;
         },

         taskLocalTotal =&gt; { // 每个任务完成时调用
            //将这个任务的总计值taskLocalTotal添加到总的总计值mastertotal上
            Interlocked.Add(ref masterTotal, taskLocalTotal);
         });
      return masterTotal;
   }
   
//结果
The total bytes of all files in C:\Users\dujinfeng\Documents is 2,698.
请按任意键继续. . .
```

委托主体传递了一个ParallelLoopState对象,可以通过该对象与参与工作的其他任务进行交互,Stop方法使得循环停止,Break方法使得循环不再处理当前项之后的项.LoweatBreakIteration属性返回处理过程中调用过Break方法的最低项,如果没有break过,就返回null.

```c#
public calss ParallelLoopState
{
	public void Stop();
	public Boolean IsStopped{ get; }
	public void Break();
	public Int64? LowestBreakIteration{ get; }
	public Boolean IsExceptional{ get; }
	public Boolean ShouldExitCurrentIteration{ get; }
}
```

在处理任何一项任务造成了异常，IsException属性会返回true,如果调用过Stop,Break,CancellationTokenSource,查询ShouldExitCurrentIteration就会返回true。

For和ForEach会返回ParalleLoopResult实例，

```c#
public struct ParallelLoopResult
{
	//如果操作提提前终止，以下方法返回false
	public Boolean IsCompleted{get;}
	public INt64? LowestBreakIteration{get;}
}
```

如果正常运行完IsCompleted返回true。

### 并行语言集成查询

顺序查询和并行查询（Parallel LINQ），System.Linq.ParallelEnumerable类实现了PLINQ的所有功能。PatallelEnumerable的**AsParallel**扩展方法可以将顺序查询(基于IEnumerable或者IEnumerable\&lt;T\&gt;)转换成并行查询(基于ParallelQuery或ParallelQuery\&lt;T\&gt;)

```c#
public static ParallelQuery&lt;TSource&gt; AsParallel&lt;TSource&gt;(this IEnumerable&lt;TSource&gt; source);
public static ParallelQuery AsParallel(this IEnumerable source);
```

例子返回一个程序集所有过时(obsolete)方法

```c#
//调用
ObsoleteMethods(typeof(Object).Assembly);


   private static void ObsoleteMethods(Assembly assembly) {
      var query =
         from type in assembly.GetExportedTypes().AsParallel()
         from method in type.GetMethods(BindingFlags.Public |
            BindingFlags.Instance | BindingFlags.Static)
         let obsoleteAttrType = typeof(ObsoleteAttribute)
         where Attribute.IsDefined(method, obsoleteAttrType)
         orderby type.FullName
         let obsoleteAttrObj = (ObsoleteAttribute)
            Attribute.GetCustomAttribute(method, obsoleteAttrType)
         select String.Format(&#34;Type={0}\nMethod={1}\nMessage={2}\n&#34;,
            type.FullName, method.ToString(), obsoleteAttrObj.Message);

      // Display the results
      foreach (var result in query) Console.WriteLine(result);
      // Alternate (not as fast): query.ForAll(Console.WriteLine);
   }

//结果
Type=Microsoft.Win32.RegistryHive
Method=System.String ToString(System.IFormatProvider)
Message=The provider argument is not used. Please use ToString().

Type=Microsoft.Win32.RegistryHive
Method=System.String ToString(System.String, System.IFormatProvider)
Message=The provider argument is not used. Please use ToString(String).

Type=Microsoft.Win32.RegistryKeyPermissionCheck
Method=System.String ToString(System.String, System.IFormatProvider)
Message=The provider argument is not used. Please use ToString(String).

Type=Microsoft.Win32.RegistryKeyPermissionCheck
Method=System.String ToString(System.IFormatProvider)
Message=The provider argument is not used. Please use ToString().

Type=Microsoft.Win32.RegistryOptions
Method=System.String ToString(System.IFormatProvider)
Message=The provider argument is not used. Please use ToString().
```

ParallelEnumerable的AsSequential方法可以将并行查询转换成顺序查询。

```c#
public static IEnumerable&lt;TSource&gt; AsSequential&lt;TSource&gt; (this ParallelQuery&lt;TSource&gt; source)
```

顺序查询中处理结果方法foreach在并行中可以用ParallelEnumerable的ForAll方法处理查询：

```c#
static void ForAll&lt;TSource&gt; (this ParallelQuery&lt;TSource&gt; source,Action&lt;TSource&gt; action);

//上面例子的最后foreach可以被替换为：
query.ForAll(Console.WriteLine);
```

但调用Console.WriteLine并行执行没有意义，其内部让线程同步，同时只有一个线程能访问控制台。

如果需要并行执行并保持顺序可以使用ParallelEnumerable的AsOrdered方法，排序方法Orderby,OrderbyDescending,ThenBy,ThenByDescending。不排序的方法Distinct,Except,Intersect,Union,Join,GroupBy,GroupJoin,ToLookup。

并行用来控制查询的方法WithCancellation\&lt;TSource&gt;，WithDegreeOfParallelism\&lt;TSource\&gt;，WithExecutionMode\&lt;TSource\&gt;,WithMergeOptions\&lt;TSource\&gt;方法。

调用Concat,ElementAt,FIrst,Last,Skip，Take,Zip,Select等可以调用WithExecutionMode,向其传递ParallelExecutionMode标志，强迫以并行执行。

```c#
public enum ParallelExceptionMode
{
	Default=0;//让并行Linq决定处理查询的最佳方式
	ForceParallelism=1//强迫查询以其并行方式处理
}
```

并行处理，结果必须合并，可调用WithMergeOptions，向其传递ParallelMergeOptions标志，从而控制缓冲与合并方式

```c#
public enum ParallelMergeOptions
{
	Default=0,//目前和AutoBuffered一样
	NotBuffered=1,//结果一旦就绪就开始处理,最省内存，速度比较慢
	AutoBuffered=2,//每个线程在处理前缓冲一些结果，介于之间
	FullyBuffered=3//每个线程在处理前缓冲所有结果，运行速度最快，内存使用多
}
```

### 执行定时的计算限制操作

System.Threading.Timer定时器构造函数

```c#
puniic sealed class Timer：MarshalByRefObject,IDisposable{
	public Timer(TimerCa11back callback,0bject stater,Int32 dueTime,Int32 period);
	public Timer(TimerCallback callback,0bject state,UInt32 dueTime,UInt32 period);
	public Timer(TimerCallback callback,Object state,Int64 dueTime,Int64 period);
	public Timer(TimerCa1lback callback,0bject state,Timespan dueTime,TimeSpan Period);
}

//System.Threading.TimerCallback
delegate void TimerCallback(Object state);
```

在内部：所有Timer对象只使用一个线程，该线程知道Timer对象下一次什么时候到期，如果到期就会唤醒，调用ThreadPool的QueueUserWorkItem，

如果回调函数执行时间长，定时器有可能再次触发，造成多线程执行回调方法，针对这个问题，建以在构造Timer时，将period指定为Timeout.Infinite，这样只执行一次回调，在回调中使用Change指定新的dueTimer,并再次将Period执行Timeout.Infinite。

```c#
public sealed class Timer：MarshalByRefObject,IDisposabie
{
	public Boolean Change(Int32 dueTime,Int32 period);
	public Boolean Change(UInt32 dueTime,UInt32 period);
	public Boo1eam Change(Int64 dueTime,Int64 period);
	public Boolean Change(TimeSpan dueTime,Timespan period);
}
```

Timer类的Dispose方法完全取消计时器，在完成回调之后，向notifyObject参数标识的内核对象发出信号，

```c#
internal static class TimerDemo {
   private static Timer s_timer;

   public static void Go() {
      Console.WriteLine(&#34;Checking status every 2 seconds&#34;);

      // Create the Timer ensuring that it never fires. This ensures that
      // s_timer refers to it BEFORE Status is invoked by a thread pool thread
      s_timer = new Timer(Status, null, Timeout.Infinite, Timeout.Infinite);

      // Now that s_timer is assigned to, we can let the timer fire knowing
      // that calling Change in Status will not throw a NullReferenceException
      s_timer.Change(0, Timeout.Infinite);

      Console.ReadLine();   // Prevent the process from terminating
   }

   // This method&#39;s signature must match the TimerCallback delegate
   private static void Status(Object state) {
      // This method is executed by a thread pool thread
      Console.WriteLine(&#34;In Status at {0}&#34;, DateTime.Now);
      Thread.Sleep(1000);  // Simulates other work (1 second)

      // Just before returning, have the Timer fire again in 2 seconds
      s_timer.Change(2000, Timeout.Infinite);

      // When this method returns, the thread goes back
      // to the pool and waits for another work item
   }
}

//结果
Checking status every 2 seconds
In Status at 2020-02-22 06:57:32
In Status at 2020-02-22 06:57:35
In Status at 2020-02-22 06:57:38
In Status at 2020-02-22 06:57:41
In Status at 2020-02-22 06:57:44

```

还可以使用Task的Delay和async和await关键字来编码

```c#
internal static class DelayDemo {
   public static void Go() {
      Console.WriteLine(&#34;Checking status every 2 seconds&#34;);
      Status();

      Console.ReadLine();   // Prevent the process from terminating
   }

   // This method can take whatever parameters you desire
   private static async void Status() {
      while (true) {
         Console.WriteLine(&#34;Checking status at {0}&#34;, DateTime.Now);
         // Put code to check status here...

         // At end of loop, delay 2 seconds without blocking a thread
         await Task.Delay(2000); // await allows thread to return
         // After 2 seconds, some thread will continue after await to loop around
      }
   }
}
```

定时器类别

&#43; System.Threading.Timer类：最好的计时器，线程池上执行定时后台任务
&#43; System.Windows.Forms.Timer类：将计时器与调用线程关联，这个计时器触发，Windows将一条计时器消息(WM_TIMER)注入线程消息队列，线程执行消息泵提取消息进行回调函数，所有这些工作都是一个线程完成，不会并发执行
&#43; System.Windows.Threading.DispatcherTimer类：是Form.Timer类在Silverlight和WPF上的等价物
&#43; Windows.UI.Xaml.DispatcherTimer类，Form.Timer在WindowsStore应用中的等价物
&#43; SYstem.Timers.Timer类，本质上是Threading.Timer的包装类，是VS工具箱中的定时器允许放在设计器上，强烈建以不需要这个类，而改用System.Threading.Timer

### 线程池如何管理线程

微软的线程池技术相当的好，工作情况非常理想，很难搞出一个比CLR更好的线程池。

#### 设置线程池限制

设置最大线程数，最好不要设置上限，如果发生饥饿或死锁，那么这些线程都会被阻塞，如果设置上限，就会导致没有新的空闲线程执行任务。

System.Threading.ThreadPool类提供了一些静态方法：**GetMaxThreads**,**SetMaxThreads**,**GetMinThreads**,**SetMinThreads**和**GetAvailableThreads**方法，可以设置和查询线程数，强烈建以不要使用上诉的任何方法。

#### 管理工作者线程

![1582332645980](/dotnet/1582332645980.png)

线程池的线程都是工作者线程，主要讲解一下**ThreadPool.QueueUserWorkItem，Timer类**和**Task**异步执行时线程池操作的异同点：

&#43; ThreadPool.QueueUserWorkItem，Timer类执行异步时，**所有的工作项（回调函数）都被放到全局队列中，工作者线程采用先入先出(first in first out)算法将工作项从全局队列中取出**，**这使得所有工作者线程竞争一个线程同步锁**，在某些应用程序中成为瓶颈。
&#43; 使用TaskScheduler调度的Task对象。非工作者线程调度Task时，**该Task被添加到全局队列**。工作者线程调度一个Task时，**该Task被添加到调用线程的本地队列(每个工作者线程都有自己的本地队列)**。//**任务不代表线程**
&#43; 工作者线程准备工作项，先检查本地队列，如果存在Task，就取出Task采用后入先出（LIFO）算法取出Task，注意的是工作者线程是唯一允许访问它自己本地队列头的线程，所以无需同步锁。
&#43; 如果工作者线程本地队列为空，**会尝试从另一个工作者队列的本地队列尾取一个Task，并要求获取同步锁**。如果所有工作队列都空了，工作线程会**使用FIFO算法从全局队列中提取工作项**。如果全局队列也空了，工作者线程进入睡眠状态，如果睡眠时间太长，会自己醒来，销毁自身，并允许系统回收资源。

线程池默认创建CPU数量的工作线程，如果设置ThreadPool.SetMinThreads,则会创建设置值。

### 高速缓存区

cpu 一般拥有多个核心和一个cpu内的缓存(一般是L2)，缓存一般位于cpu芯片内, 他的速度远远高于主板上的内存,一般来说cpu会把数据从内存加载到缓存中 ,这样可以获得更好的性能(特别是频繁使用的数据)，高速缓存默认划分64 Byte为一个区域(这个数字可能在不同的平台上不一样, 可以通过 ﻿win32 api 函数 GetProcessorInformation 修改)，**一个区域在一个时间点只允许一个核心操作**，那么完全可能一个线程在操作field1 的时候 , 运行于另外一个cpu上的另外一个线程想操作field2,就必须等待线程1完成以后才能获取这个缓存区域的访问.

例子：显式布局，性能提高了80%

```c#
internal static class FalseSharing {
#if true
   private class Data {
      // These two fields are right next to each other in 
      // memory; most-likely in the same cache line
      public Int32 field1;
      public Int32 field2;
   }
#else
   [StructLayout(LayoutKind.Explicit)]
   private class Data {
      // These two fields are right next to each other in 
      // memory; most-likely in the same cache line
      [FieldOffset(0)]
      public Int32 field1;
      [FieldOffset(64)]
      public Int32 field2;
   }
#endif

   private const Int32 iterations = 100000000;
   private static Int32 s_operations = 2;
   private static Stopwatch s_stopwatch;

   public static void Go() {
      Data data = new Data();
      s_stopwatch = Stopwatch.StartNew();
      ThreadPool.QueueUserWorkItem(o =&gt; AccessData(data, 0));
      ThreadPool.QueueUserWorkItem(o =&gt; AccessData(data, 1));
      Console.ReadLine();
   }

   private static void AccessData(Data data, Int32 field) {
      for (Int32 x = 0; x &lt; iterations; x&#43;&#43;)
         if (field == 0) data.field1&#43;&#43;; else data.field2&#43;&#43;;

      if (Interlocked.Decrement(ref s_operations) == 0)
         Console.WriteLine(&#34;Access time: {0}&#34;, s_stopwatch.Elapsed);
   }
}
//结果，可能共享缓存
Access time: 00:00:01.0554650
//显式指定内存布局
Access time: 00:00:00.2595376

```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/02/dotnetbase7-netclrvia/  

