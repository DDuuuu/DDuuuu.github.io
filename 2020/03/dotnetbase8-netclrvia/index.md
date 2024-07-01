# NetCLRVia 同步与竞争




## 基元线程同步构造

构建可伸缩的，响应灵敏的应用程序，关键在于不要阻塞线程，

多个线程同时访问共享数据，获取并释放一个线程同步锁。锁会损害性能，获取和释放锁是需要时间的。只允许一个线程访问共享资源，可以使用值类型，多个线程对共享数据进行只读访问是没有任何问题的。

### 类库和线程安全

**FCL保证所有静态方法都是线程安全的**。多个线程调用同一个静态方法，并不是说方法内部会获取一个同步锁，也可能内部数据是值类型。

FCL没有保证实例方法是线程安全的。

所有的静态方法都是线程安全的，所有实例方法都是非线程安全。

### 基元用户模式和内核模式构造

基元：primitive,

基元用户模式的速度要显著高于基元内核模式，因为其使用特殊的CPU指令来协调线程，**缺点是操作系统用还不会察觉线程在基元用户模式的构造上阻塞了，也就永远不会创建新线程来替换阻塞线程。**

基元用户模式中操作系统内核才能停止一个线程的运行，**用户模式中线程可能被系统抢占(preempted)，但会以最快的速度被调用**，如果暂时取不到的线程会一直处于自旋状态，这可能浪费大量CPU时间。

从上面可以看出基元用户模式跟CPU有关，是CPU提供的

基元内核模式是由操作系统提供的。在应用程序的线程中调用操作系统内核实现函数。用户模式切换为内核模式会造成巨大的损失。但内核模式的优点是：线程获取其他线程拥有的资源时，Windows会阻塞线程避免它浪费CPU时间。当资源可用时，Windows恢复线程，允许它访问资源。

在构造上等待的线程，如果是用户模式，线程将一直在CPU上运行，称为&#34;活锁“。如果是内核模式，线程将一直阻塞，称为&#34;死锁&#34;(deadlock)，但比较而言，死锁总是由于活锁，因为活锁既浪费CPU也浪费内存。

所以最好的构造方式应该是混合构造(hybrid construct)，在大多数时间运行的很快，偶尔运行的比较慢。

CLR中的线程同步构造实际是”Win32线程同步构造“的包装器，CLR线程就是Windows线程，Windows调度线程和控制线程同步。

### 用户模式构造

CLR可以保证对Boolean,Char,(S)Byte,(U)Int16,(U)Int32,(U)IntPtr,Single以及**引用类型**的读写是**原子性的，也就是一次性读写。**

```c#
internal static class SomeType
{
	public static Int32 x=0;
}

//线程执行这一行代码，
SomeType.x=0x01234567;
```

x变量会一次性从0x000000变成0x12345，另一个线程不可能看到中间状态的值，也不可能有别的线程访问。如果该类型是Int64，别的线程就有可能获得中间状态的值，其实是分为两个Move指令才能读完。

两种基元用户模式线程同步构造：

&#43; 易变构造(volatile construct):在特定时间，在一个简单数据类型的变量上执行原子读或写操作。
&#43; 互锁构造(interlocked construct)：和上面相同

易变和互锁构造都要求传递包含简单数据类型的内存地址。

#### 易变构造(Volatile Construct)

C#编译器将C#代码转换成IL，JIT将IL转换成CPU指令，C#编译器，JIT编译器，CPU都有可能优化代码。

```c#
   private static void OptimizedAway() {
      // An expression of constants is computed at compile time then put into a local variable that is never used
      Int32 value = 1 * 100 &#43; 0 / 1 % 2;
      if (value &gt;= 0) Console.WriteLine(&#34;Jeff&#34;);

      for (Int32 x = 0; x &lt; 1000; x&#43;&#43;) ;   // A loop that does nothing
   }
```

下面代码编译后会优化掉很多东西。

```c#
internal static class StrangeBehavior {
   // Compile with &#34;/platform:x86 /o&#34; and run it NOT under the debugger (Ctrl&#43;F5)
   private static Boolean s_stopWorker = false;

   public static void Go() {
      Console.WriteLine(&#34;Main: letting worker run for 5 seconds&#34;);
      Thread t = new Thread(Worker);
      t.Start();
      Thread.Sleep(5000);
      s_stopWorker = true;
      Console.WriteLine(&#34;Main: waiting for worker to stop&#34;);
      t.Join();
      Environment.Exit(0);
   }

   private static void Worker(Object o) {
      Int32 x = 0;
      while (!s_stopWorker) x&#43;&#43;;
      Console.WriteLine(&#34;Worker: stopped when x={0}&#34;, x);
   }
}
```

Main中创建一个新线程来执行Worker方法，在/platform:x86和/optimize&#43;开关编译(x86 JIT编译器比x64编译器更程数，所以它执行优化的时候更大胆)，会对代码进行优化，当编译器发现worker中s_stopWorker永远不发生变化，编译器会生成代码检查s_stopWorker。如果为false，就进入无线循环，一直递增x，**优化后，s_stopWorker检查只在循环前发生一次，不会每次迭代都检查。**

针对编译器和CPU的优化，FCL提供了两个静态方法，**禁止一些优化**，**System.Threading.Volatile**类提供了两个静态方法

```c#
internal static class ThreadsSharingData {
   internal sealed class ThreadsSharingDataV1 {
      private Int32 m_flag = 0;
      private Int32 m_value = 0;

      // 这个方法由一个线程执行
      public void Thread1() {
         // Note: 以下两行代码可以按相反的顺序执行
         m_value = 5;
         m_flag = 1;
      }

      // 这个方法由另一个线程执行 
      public void Thread2() {
         // Note: m_value 可能先于 m_flag 读取
         if (m_flag == 1) Console.WriteLine(m_value);
      }
   }
}
```

编译器和CPU解释优化代码的时候可能会反转Thread1()方法中的两行代码，在单线程顺序执行的时候，Thread1()，Thread2()优化并不会产生问题，但是多线程的时候，如果允许多线线程同时读写数据就会产生很多问题，**针对Thread1方法必须要保证m_value和m_flag，同时执行写入，在写入的时候，所有读取线程停止，写完再允许读**，**针对Thread2方法必须要保证先读取m_flag值在读取m_value值，在2个变量读取的时候禁止所有写入操作**。

System.Threading.Volatile是如何解决这些问题的，静态方法，原子操作：

```c#
public static class Volatile
{
	public static void Write(ref Int32 location,Int32 value);
	public static Int32 Read(ref Int32 location);
}
```

&#43; Volatile.Write方法强迫location中的值在调用时写入，按照编码顺序，之前加载和存储操作必须在调用Volatile.Write之前发生。
&#43; Volatile.Read方法强迫location中的值在调用时读取，按照编码顺序，之后加载和存储操作必须在调用Volatile.Read之后发生。

总结：**当线程通过共享内存相互通信时，调用Volatile.Write来写入最后一个值，调用Volatile.Read读取第一个值。**

```c#
   internal sealed class ThreadsSharingDataV2 {
      private Int32 m_flag = 0;
      private Int32 m_value = 0;

      // 这个方法由一个线程执行
      public void Thread1() {
         // Note: 在将1写入m_flag之前，必须先将5写入m_value.
         m_value = 5;
         Volatile.Write(ref m_flag, 1);
      }

      // 这个方法由另一个线程执行 
      public void Thread2() {
         // Note: m_value必然在读取 m_flag 之后读取
         if (Volatile.Read(ref m_flag) == 1)
            Console.WriteLine(m_value);
      }
   }
```

C#对Volatile的支持，为了简化编程，c#提供了volatile关键字，可应用于以下类型，Boolean,(S)Byte,(U)Int16，(U)Int32,(U)IntPtr,Single,Char和引用类型。Volatile关键字高速C#和JIT编译器不讲字段缓存到CPU寄存器，确保字段所有读写操作都是在RAM中执行。

```c#
   internal sealed class ThreadsSharingDataV3 {
       //非常重要
      private volatile Int32 m_flag = 0;
      private Int32 m_value = 0;

      // This method is executed by one thread 
      public void Thread1() {
         // Note: 5 must be written to m_value before 1 is written to m_flag
         m_value = 5;
         m_flag = 1;
      }

      // This method is executed by another thread 
      public void Thread2() {
         // Note: m_value must be read after m_flag is read 
         if (m_flag == 1)
            Console.WriteLine(m_value);
      }
   }
```

这种优化的方式也有一些缺陷。

```c#
m_amount = m_amout &#43; m_amout;//假定m_amout是类定义的一个volatile字段
```

如果优化，m_amout左移1位就可以了，如果不允许这个优化，会对性能造成很大影响，同时一旦使用了m_amount就不允许其值传递自己的引用。

#### 互锁构造

Volatile的方法都是原子性操作。而System.Threading.Interlocked类提供的方法也是**执行原子操作**，同时建立了完整的**内存栅栏**(memory fence),Volatile有的都有。

简单来说就是：调用某个Interlocked方法之前的任何变量写入都在这个Interlocked方法调用前执行，调用之后的任何变量读取都在这个调用之后读取。**静态方法，原子操作**

```c#
public static class Interlocked
{
	//return (&#43;&#43;location)
	public static Int32 Increment(ref Int32 location);
	//return (--location)
	public static Int32 Decrement(ref Int32 location)；
	//return (location&#43;=value)
	//notes:value可能是一个负数，从而实现剑法运算
	public static Int32 Add(ref Int32 location1,Int32 value);
	//Int32 old = location; location=value;reurn old
	public static Int32 Exchange(retf Int32 location1,Int32 value);
	//Int32 old = location1;
	//if (location1==comparand) location1=value;
	//return old;
	public static Int32 CompareExchange(ref Int32 location1,Int32 value,Int32 comparand);
}
```

#### 使用互锁构造的WebClient例子。

```c#
//调用go

internal static class AsyncCoordinatorDemo {
   public static void Go() {
      const Int32 timeout = 50000;   // Change to desired timeout
      MultiWebRequests act = new MultiWebRequests(timeout);
      Console.WriteLine(&#34;All operations initiated (Timeout={0}). Hit &lt;Enter&gt; to cancel.&#34;,
         (timeout == Timeout.Infinite) ? &#34;Infinite&#34; : (timeout.ToString() &#43; &#34;ms&#34;));
      Console.ReadLine();
      act.Cancel();

      Console.WriteLine();
      Console.WriteLine(&#34;Hit enter to terminate.&#34;);
      Console.ReadLine();
   }

   private sealed class MultiWebRequests {
      // This helper class coordinates all the asynchronous operations
      private AsyncCoordinator m_ac = new AsyncCoordinator();

      // Set of Web servers we want to query &amp; their responses (Exception or Int32)
      private Dictionary&lt;String, Object&gt; m_servers = new Dictionary&lt;String, Object&gt; {
         { &#34;https://www.zhihu.com/&#34;, null },
         { &#34;https://www.cnblogs.com/&#34;,  null },
         { &#34;https://1.1.1.1&#34;,        null } 
      };

      public MultiWebRequests(Int32 timeout = Timeout.Infinite) {
         // Asynchronously initiate all the requests all at once
         var httpClient = new HttpClient();
         foreach (var server in m_servers.Keys) {
            m_ac.AboutToBegin(1);
            httpClient.GetByteArrayAsync(server).ContinueWith(task =&gt; ComputeResult(server, task));
         }

         // Tell AsyncCoordinator that all operations have been initiated and to call
         // AllDone when all operations complete, Cancel is called, or the timeout occurs
         m_ac.AllBegun(AllDone, timeout);
      }

      private void ComputeResult(String server, Task&lt;Byte[]&gt; task) {
         Object result;
         if (task.Exception != null) {
            result = task.Exception.InnerException;
         } else {
            // Process I/O completion here on thread pool thread(s)
            // Put your own compute-intensive algorithm here...
            result = task.Result.Length;   // This example just returns the length
         }

         // Save result (exception/sum) and indicate that 1 operation completed
         m_servers[server] = result;
         m_ac.JustEnded();
      }

      // Calling this method indicates that the results don&#39;t matter anymore
      public void Cancel() { m_ac.Cancel(); }

      // This method is called after all Web servers respond, 
      // Cancel is called, or the timeout occurs
      private void AllDone(CoordinationStatus status) {
         switch (status) {
            case CoordinationStatus.Cancel:
               Console.WriteLine(&#34;Operation canceled.&#34;);
               break;

            case CoordinationStatus.Timeout:
               Console.WriteLine(&#34;Operation timed-out.&#34;);
               break;

            case CoordinationStatus.AllDone:
               Console.WriteLine(&#34;Operation completed; results below:&#34;);
               foreach (var server in m_servers) {
                  Console.Write(&#34;{0} &#34;, server.Key);
                  Object result = server.Value;
                  if (result is Exception) {
                     Console.WriteLine(&#34;failed due to {0}.&#34;, result.GetType().Name);
                  } else {
                     Console.WriteLine(&#34;returned {0:N0} bytes.&#34;, result);
                  }
               }
               break;
         }
      }
   }

   private enum CoordinationStatus {
      AllDone,
      Timeout,
      Cancel
   };

   private sealed class AsyncCoordinator {
      private Int32 m_opCount = 1;        // Decremented when AllBegun calls JustEnded
      private Int32 m_statusReported = 0; // 0=false, 1=true
      private Action&lt;CoordinationStatus&gt; m_callback;
      private Timer m_timer;

      // This method MUST be called BEFORE initiating an operation
      public void AboutToBegin(Int32 opsToAdd = 1) {
         Interlocked.Add(ref m_opCount, opsToAdd);
      }

      // This method MUST be called AFTER an operations result has been processed
      public void JustEnded() {
         if (Interlocked.Decrement(ref m_opCount) == 0)
            ReportStatus(CoordinationStatus.AllDone);
      }

      // This method MUST be called AFTER initiating ALL operations
      public void AllBegun(Action&lt;CoordinationStatus&gt; callback, Int32 timeout = Timeout.Infinite) {
         m_callback = callback;
         if (timeout != Timeout.Infinite) {
            m_timer = new Timer(TimeExpired, null, timeout, Timeout.Infinite);
         }
         JustEnded();
      }

      private void TimeExpired(Object o) { ReportStatus(CoordinationStatus.Timeout); }

      public void Cancel() {
         if (m_callback == null)
            throw new InvalidOperationException(&#34;Cancel cannot be called before AllBegun&#34;);
         ReportStatus(CoordinationStatus.Cancel);
      }

      private void ReportStatus(CoordinationStatus status) {
         if (m_timer != null) {  // If timer is still in play, kill it
            Timer timer = Interlocked.Exchange(ref m_timer, null);
            if (timer != null) timer.Dispose();
         }

         // If status has never been reported, report it; else ignore it
         if (Interlocked.Exchange(ref m_statusReported, 1) == 0)
            m_callback(status);
      }
   }
}

//结果
All operations initiated (Timeout=50000ms). Hit &lt;Enter&gt; to cancel.
Operation completed; results below:
https://www.zhihu.com/ returned 52,010 bytes.
https://www.cnblogs.com/ returned 48,437 bytes.
https://1.1.1.1 returned 44,241 bytes.


Hit enter to terminate.

```

这个例子是通过异步的方式请求几个服务器，计算每个服务器返回的数据量。这个例子不阻塞任何线程，使用线程池来实现自动伸缩。

&#43; 初始化AsyncCoordinator和数据字典
&#43; 通过异步方式遍历所有字典，并使用ContinueWith,没遍历一项就调用AboutToBegin计数
&#43; 全部遍历完调用AllBegun，设置定时器时间，回调函数。
&#43; ContinueWith中，调用JustEnded减计数器。
&#43; 当计数器减到0或定时器时间到，调用回调函数。

&#43; m_opCount字段初始化1，非常重要，可以保证在调用AllBegun之前不会调用AllDone。

#### 简单自旋锁(spin lock)

```c#
//定义 
internal struct SimpleSpinLock {
      private Int32 m_ResourceInUse; // 0=false (default), 1=true

      public void Enter() {
         while (true) {
            // 总是将资源设为正在使用
            // 只有从未使用编程正在使用才返回
            if (Interlocked.Exchange(ref m_ResourceInUse, 1) == 0) return;
            // 这里添加黑科技
         }
      }

      public void Leave() {
         // 资源标记为未使用
         Volatile.Write(ref m_ResourceInUse, 0);
      }
   }


//使用
SimpleSpinLock ssl = new SimpleSpinLock();
for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) 
{
    ssl.Enter(); x&#43;&#43;; ssl.Leave();
}
```

简单解释一下：当两个线程一起调用Enter的时候，那么Interlocked.Exchange会确保一个线程将m_ResourceInUse从0变成1，并发现m_ResourceInUse原来是0，就直接return；然后才能执行业务代码。另一个线程m_ResourceInUse从1变成1，不是由0变成1，会不停调用Exchange进行自旋，直到第一个线程调用Leave。

自旋锁最大的问题在于：对锁竞争的前提下，自旋会浪费宝贵的CPU时间。在单CPU机器上使用，如果占有锁的线程优先级低于想要获取锁的优先级，会使得占有锁线程根本没有机会运行。可以在黑科技中取消优先级。

FCL提供了一个System.Threading.SpinWait结构，封装了黑科技的最新研究成果。还有System.Threading.SpinLock结构，内部使用了SpinWait结构，同时还提供了超时支持。

注意锁是**值类型**，不要传递SpinLock实例，会被复制，否则会失去同步。

#### 在线程中引入延迟

黑科技是自旋的线程暂停执行，也就是不占用CPU时间，使拥有资源的线程能执行代码，内部调用Sleep,Yield和SpinWait方法。

##### Sleep

```c#
public static void Sleep(Int32 millisecondsTimeout);
public static void Sleep(TimeSpan timeout);
```

时间到了后不会准时唤起线程。

传入System.Threading.Timeout.Infinite（-1）,永远不会调度线程（没意义最好退出线程，会输栈和内核对象），传入0，线程放弃当前时间片，强迫调用另一个线程，也可能会再一次调用自己。

##### Yield

调度另一个线程是Thread.Yield方法

```c#
public static Boolean Yield();
```

windows发现另一个线程准备好，Yield就返回true,在发现的线程上运行一个时间片，然后再回来。

#### Interlocked Anything

Interlocked没有提供Multiple，divide,Minimum,and,or,xor等方法,可以使用Interlocked.CompareExchange方法可以再Int32上执行任何操作。

```c#
public static Int32 Maximum(ref Int32 target,Int32 value)
{
    Int32 currentVal=target,startVal,desireVal;//
    //不要再循环中访问目标(target),除非是想要改变它时另一个线程也在动它
    do
    {
        //记录这一次循环迭代的起始值(startVal)
        startVal=currentVal;
        //可以进行更复杂的操作，只要将最终结果放到desireVal中
        //基于startVal和value计算desiredVal
        desireVal=Math.Max(startVal,value);
        //注意：线程在这里可能被&#34;抢占”，所以以下代码不是原子性的
        //if(target==startValue)target==desireVal
        //而应使用以下原子性的CompareExchange方法，它返回target在被方法修改前的值
        currentVale=Interlocked.CompareExchange(ref target,desireVal,startVal);
        //这个原子语句等于
        //currentValue=target;
        //if(target==startValue)target==desireVal
    }
    while(startVal!=currentVal)
    //在线程尝试设置它之前返回最大值
    return desireVal;
}
```

&#43; 进入方法currentValue初始化为target值

&#43; 在循环内部，startVal初始化为target值，可用startVal执行希望的任何操作，这个操作可以非常复杂，但最终需要将结果放到desiredValue中。在循环体中其他线程可能更改target值，如果其他线程更改，在执行原子语句的时候，currentValue=target，会再一次执行循环startvalue=currentvalue。
&#43; 第一个线程进入循环，如果这个时候有第二个线程执行进入循环，并执行完，target就会改变，第一个线程执行到Interlocked.ComareExchange，while时，会发现为true，重新执行一边，这就是乐观锁。

```c#
delegate Int32 Morpher&lt;TResult,TArgument&gt;(Int32 startValue,TArgument argument,out TResult morphResult);
static TResult Morph&lt;TResult,TArgument&gt;(ref Int32 target,TArgument argument,Morpher&lt;TResult,TArgument&gt; morpher)
{
	TResult morphResult;
	Int32 currentVal=target,startVal,desiredVal;
	do
	{
		startVal=currentVal;
		desiredVal=morpher(startVal,argument,out morphResult);
		currentVal=Interlocked.CmpareEcvhange(ref target,desiredVal,startVal);
	}
	while(startVal!=currentVal)
	return morphResult;
}
```

回调函数有一定性能损失，最好使用内联；

### 内核模式构造

内核模式构造比用户模式构造慢的多，有两个原因：1、需要Windows操作系统配合，2、内核对象上调用每个方法都会使线程从托管代码转换成本机用户模式代码，再转换成本机内核模式代码。

优点:

&#43; 资源竞争的时候阻塞掉新城，不会浪费CPU资源。
&#43; 实现本机(native)和托管(managed)线程之间的同步。
&#43; 可同步在同一台机器中不同进程的线程
&#43; 可应用安全性设置，防止未授权的账户访问
&#43; 线程阻塞，直到所有内核模式可用，或集合中内核模式可用
&#43; 阻塞的线程可以被时间值，在规定时间内获取不到资源就解除阻塞执行其他操作。

事件和信号量（Semaphores）是基元内核模式同步构造，其他都是在这两个构造上派生的

System.Threading.WaitHandle抽象基类，唯一的作用就是包装一个Windows内核对象句柄

```
WaitHandle
	EventWaitEvent
		AutoResetEvent
		ManualResetEvent
    Semaphore
    Mutex
```

WaitHandle内部有一个SafeWaitHandle字段，容纳一个Win32内核对象句柄。内核模式调用每个方法都代码一个完整的内存栅栏。

![1582869283171](/dotnet/1582869283171.png)

![1582869291300](/dotnet/1582869291300.png)

&#43; WaitOne：让调用线程等待底层内核对象接收信号，内部调用Win32 WaitForSignleObjectEx函数，如果对象收到信号返回true，超时返回false。内部元素不能超过64
&#43; WaitAll:让调用线程等待WaitHandle[]中指定所有内核对象收到信息，如果所有对象收到信号，返回true，超时返回false，内部调用Win32 WaitForMultipleObjectsEx函数，bWaitAll参数传递TRUE，内部元素不能超过64
&#43; WaitAny:让调用线程等待WaitHandle[]指定任何内核对象收到信号。返回Int32内核对象对应的数组索引，如果一直没有收到，返回WaitHandle.WaitTimeout,内部调用Win32 WaitForMultipleObjectsEx
&#43; Dispose:关闭底层内核对象句柄，内部调用Win32 CloseHandle，只有在确定没有别的线程使用内核线程对象才能显式调用

AutoResetEvent、ManualResetEvent、Semaphore和Mutex类都派生WaitHandle

&#43; 内部调用Win32 CreateEvent、CreateSemaphore或CreateMutex函数，函数句柄保存在私有的SafeWaitHandle字段中。
&#43; EventWaitHandle、Semaphore和Mutex都提供静态OpenExisting方法，内部调用win32 OpenEvent、OpenSemaphire或OpenMutex函数，

内核模式常用的方法是创建任何时刻只允许它运行的应用程序。

```c#
using System;
using System.Threading;
public static class Program
{
	public static void Main()
	{
		Boolean createNew;
		//尝试创建一个具有指定名称的内核对象
		using(new Semaphore(0,1,&#34;SomeUniqueStringIdentifyingMyApp&#34;,out createdNew))
		{
			if(createdNew)
			{
			//这个线程创建了内核对象，所以肯定没有这个应用程序的其他实例正在运行
			
			}
			else
			{
			有其他实例在运行
			}
		}
	}
}
```

将Semaphore替换成EventWaitHandle或Mutex一样的。两个线程同时创建Semaphore,windows内核确保只有一个能成功创建

#### Event

Event内部维护内核Boolean变量，false：事件在等待线程阻塞，true解除阻塞。

&#43; false，阻塞
&#43; true,不阻塞。

有两种事件，自动重置事件（AutoResetEvent）和手动重置事件(ManualResetEvent)。

AutoResetEvent:true的时候，唤起一个阻塞线程，然后自动为false，其余线程阻塞。

ManualResetEvent:true的时候，解除正在等待的所有线程阻塞，内核不会重置为false，必须手动进行。

```c#
public class EventWaitHandle:WaitHandle
{
	public Boolean Set();//将内核Boolean设置为true，总是返回true
	public Boolean Reset();//将内核Boolean设为false,总是返回true
}
public sealed class AutoResetEvent:EventWaitHandle
{
	public AutoResetEvent(Boolean initialState);
}
public sealed class ManualReserEvent:EventWaitHandle
{
	public ManualReserEvent(Boolean initialState);
}
```

事件锁

```c#
   private sealed class SimpleWaitLock : IDisposable {
      private readonly AutoResetEvent m_available;
      public SimpleWaitLock() {
         m_available = new AutoResetEvent(true); // Initially free
      }

      public void Enter() {
         // Block in kernel until resource available
         m_available.WaitOne();
      }

      public void Leave() {
         // Let another thread access the resource
         m_available.Set();
      }

      public void Dispose() { m_available.Dispose(); }
   }
```



```c#
   private sealed class SimpleWaitLock : IDisposable {
      private readonly AutoResetEvent m_available;
      public SimpleWaitLock() {
         m_available = new AutoResetEvent(true); // 最开始可以自由使用
      }

      public void Enter() {
         // 在内核中阻塞，直到资源可用，直到一个内核对象变为true，获取对象后自动变成false
         m_available.WaitOne();
      }

      public void Leave() {
         // 让另一个线程访问资源,置true
         m_available.Set();
      }

      public void Dispose() { m_available.Dispose(); }
   }
```

性能比较

```c#
//调用go

internal static class LockComparison {
   public static void Go() {
      Int32 x = 0;
      const Int32 iterations = 10000000;  // 10 million

      // How long does it take to increment x 10 million times?
      Stopwatch sw = Stopwatch.StartNew();
      for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) {
         x&#43;&#43;;
      }
      Console.WriteLine(&#34;Incrementing x: {0:N0}&#34;, sw.ElapsedMilliseconds);

      // How long does it take to increment x 10 million times 
      // adding the overhead of calling a method that does nothing?
      sw.Restart();
      for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) {
         M(); x&#43;&#43;; M();
      }
      Console.WriteLine(&#34;Incrementing x in M: {0:N0}&#34;, sw.ElapsedMilliseconds);

      // How long does it take to increment x 10 million times 
      // adding the overhead of calling an uncontended SimpleSpinLock?
      SimpleSpinLock ssl = new SimpleSpinLock();
      sw.Restart();
      for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) {
         ssl.Enter(); x&#43;&#43;; ssl.Leave();
      }
      Console.WriteLine(&#34;Incrementing x in SimpleSpinLock: {0:N0}&#34;, sw.ElapsedMilliseconds);

      // How long does it take to increment x 10 million times 
      // adding the overhead of calling an uncontended SpinLock?
      SpinLock sl = new SpinLock(false);
      sw.Restart();
      for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) {
         Boolean taken = false;
         sl.Enter(ref taken); x&#43;&#43;; sl.Exit(false);
      }
      Console.WriteLine(&#34;Incrementing x in SpinLock: {0:N0}&#34;, sw.ElapsedMilliseconds);

      // How long does it take to increment x 10 million times 
      // adding the overhead of calling an uncontended SimpleWaitLock?
      using (SimpleWaitLock swl = new SimpleWaitLock()) {
         sw.Restart();
         for (Int32 i = 0; i &lt; iterations; i&#43;&#43;) {
            swl.Enter(); x&#43;&#43;; swl.Leave();
         }
         Console.WriteLine(&#34;Incrementing x in SimpleWaitLock: {0:N0}&#34;, sw.ElapsedMilliseconds);
      }
      Console.ReadLine();
   }

   [MethodImpl(MethodImplOptions.NoInlining)]
   private static void M() { }

   internal struct SimpleSpinLock {
      private Int32 m_ResourceInUse; // 0=false (default), 1=true

      public void Enter() {
         while (true) {
            // Always set resource to in-use
            // When this thread changes it from not in-use, return
            if (Interlocked.Exchange(ref m_ResourceInUse, 1) == 0) return;
            // Black magic goes here...
         }
      }

      public void Leave() {
         // Set resource to not in-use
         Volatile.Write(ref m_ResourceInUse, 0);
      }
   }

   private sealed class SimpleWaitLock : IDisposable {
      private readonly AutoResetEvent m_available;
      public SimpleWaitLock() {
         m_available = new AutoResetEvent(true); // Initially free
      }

      public void Enter() {
         // Block in kernel until resource available
         m_available.WaitOne();
      }

      public void Leave() {
         // Let another thread access the resource
         m_available.Set();
      }

      public void Dispose() { m_available.Dispose(); }
   }
}
//结果
Incrementing x: 24
Incrementing x in M: 57
Incrementing x in SimpleSpinLock: 166
Incrementing x in SpinLock: 182
Incrementing x in SimpleWaitLock: 22,526
```

可以看出仅仅加了什么都不做的函数性能都与影响，而内核事件锁的性能比自旋锁慢了近1000倍

&#43; 能避免线程同步就避免线程同步
&#43; 如果一定要进行线程同步请使用用户模式构造。内核模式尽量避免。

#### Semaphore构造

Semaphore在内部维护内核的Int32变量，Int32不同值代表的意思：

&#43; 0，在等待信号量的线程会阻塞，阻塞

&#43; 大于0，在等待信号量的线程解除阻塞。不阻塞，每有一个线程解除阻塞，int32值自动减1，
&#43; Int32值还关联了一个最大值，计数不允许超过最大值

```c#
public sealed class Semaphore:WaitHandle
{
	public Semaphore(Int32 initialCOunt,Int32 maximumCOunt);
	public Int32 Release();//调用Release(1)，返回上一个计数
	public Int32 Release(Int32 releaseCount);//返回上一个计数
}
```

Semaphore与AutoResetEvent,ManualResetEvent的异同

AutoResetEvent:不阻塞时，只有一个线程被解除阻塞

ManualResetEvent:不阻塞时，所有线程被解除阻塞

Semaphore:不阻塞时，可以使得releaseCount个线程被解除阻塞。

```c#
public sealed class SimpleWaitLovk:IDisposable
{
	private Semaphore m_available;
	public SimpleWaitLock(Int32 maxConcurrent)
	{
		m_available=new Semaphore(maxConcurrent,maxConcurrent);
	}
	public void Enter()
	{
		//在内核中阻塞直到资源可用
		m_available.WaitOne();
	}
	public void Leave()
	{
		m_available.Release(1);
	}
	public void Dispose(){m_available.Close();}
}
```

#### Mutex

Mutex和AutoaResetEvent的工作方式差不多，但是有很多额外的逻辑

&#43; 其会查询调用线程Int32 ID,在线程ReleaseMutex时，会对比释放线程和调用线程是否一致。不一致不会更改，而且释放线程会抛出System.ApplicaionException.
&#43; 如果拥有Mutex的线程异常终止，会抛出System.Threading.AbandoneMutexExeception异常，在等待Mutex的线程会被唤醒。
&#43; Mutex维护了一个递归计数，指出拥该Mutex的线程拥有它多少次。在调用ReleaseMutex会导致递减。

最后这个递归计数的功能是在递归锁中用到，**递归锁的使用场景是在一个获取锁的方法中调用另一个也需要锁的方法。这时就需要递归锁**

```c#
internal class SomeClass:IDisposable
{
	private readonly Mutex m_lock=new Mutex();
	public void Method1()
	{
		m_lock.WaitOne();
		//随便做什么事情。。。
		Method2();//Method2递归地获取锁
		m_lock.ReleaseMutex();
	}
	public void Method2()
	{
		m_lock.WaitOne();
		//随便做什么
		m_lock.ReleaseMutex();
	}
	public void Dispose()
	{
		m_lock.Dispose();
	}

}
```



使用AutoResetEvent也可用构造递归锁，从而摆脱Mutex的复杂操作。

```c#
internal sealed class RecursiveAutoResetEvent : IDisposable {
   private AutoResetEvent m_lock = new AutoResetEvent(true);
   private Int32 m_owningThreadId = 0;
   private Int32 m_recursionCount = 0;

   public void Enter() {
      // 获取调用线程的唯一 Int32 ID
      Int32 currentThreadId = Thread.CurrentThread.ManagedThreadId;

      // 如果调用线程拥有锁，就递增递归计数
      if (m_owningThreadId == currentThreadId) {
         m_recursionCount&#43;&#43;;
         return;
      }

      // 调用线程不拥有锁，等待它
      m_lock.WaitOne();

      // 调用线程现在拥有锁，初始化拥有线程的ID和计数器
      m_owningThreadId = currentThreadId;
      m_recursionCount--;
   }

   public void Leave() {
      // 如果调用的线程不拥有锁就出错
      if (m_owningThreadId != Thread.CurrentThread.ManagedThreadId)
         throw new InvalidOperationException();

      // 从递归计数中减1
      if (--m_recursionCount == 0) {
         // 如果递归计数为0，表明没有线程拥有锁
         m_owningThreadId = 0;
         m_lock.Set();   // 唤醒一个正在等待的线程(如果有的话)
      }
   }

   public void Dispose() { m_lock.Dispose(); }
}
```

## 混合线程同步构造

### 一个简单混合锁

```c#
  public sealed class SimpleHybridLock : IDisposable {
      // The Int32 is used by the primitive user-mode constructs (Interlocked mehtods)
      private Int32 m_waiters = 0;

      // The AutoResetEvent is the primitive kernel-mode construct
      private readonly AutoResetEvent m_waiterLock = new AutoResetEvent(false);

      public void Enter() {
         // 多个线程获得锁
         if (Interlocked.Increment(ref m_waiters) == 1)
            return; //无竞争直接返回

         //多个线程获得锁发生竞争，阻塞线程，
         m_waiterLock.WaitOne();  // 性能影响较大
         // WaitOnef返回一个线程获得锁。
      }

      public void Leave() {
         // 无竞争的时候释放原子锁
         if (Interlocked.Decrement(ref m_waiters) == 0)
            return; // 没有其他线程等待，直接返回

         // 唤醒一个阻塞的线程
         m_waiterLock.Set();  // 这里产生较大影响
      }

      public void Dispose() { m_waiterLock.Dispose(); }
   }
```

### 自旋、线程所有权和递归

使用内核模式非常影响性能，所以就有了自旋锁，即使线程什么也不干，自旋一定时间也不使用内核模式。

```c#
  public sealed class AnotherHybridLock : IDisposable {
      // The Int32 is used by the primitive user-mode constructs (Interlocked methods)
      private Int32 m_waiters = 0;

      // The AutoResetEvent is the primitive kernel-mode construct
      private AutoResetEvent m_waiterLock = new AutoResetEvent(false);

      // This field controls spinning in an effort to improve performance
      private Int32 m_spincount = 4000;   // Arbitrarily chosen count

      // 字段指出哪个线程拥有锁，以及拥有了多少次，以支持递归锁
      private Int32 m_owningThreadId = 0, m_recursion = 0;

      public void Enter() {
         // If the calling thread already owns this lock, increment the recursion count and return
         Int32 threadId = Thread.CurrentThread.ManagedThreadId;
         if (threadId == m_owningThreadId) { m_recursion&#43;&#43;; return; }

         // SpinWait值类型
         SpinWait spinwait = new SpinWait();
         for (Int32 spinCount = 0; spinCount &lt; m_spincount; spinCount&#43;&#43;) {
            // If the lock was free, this thread got it; set some state and return
            if (Interlocked.CompareExchange(ref m_waiters, 1, 0) == 0) goto GotLock;

            // 发生线程竞争后线程等待
            spinwait.SpinOnce();
         }

         // 自旋结束，锁还没获得，再试一次
         if (Interlocked.Increment(ref m_waiters) &gt; 1) {
            // 如果仍在竞争进入内核模式
            m_waiterLock.WaitOne(); // Wait for the lock; performance hit
            // When this thread wakes, it owns the lock; set some state and return
         }

      GotLock:
         // When a thread gets the lock, we record its ID and 
         // indicate that the thread owns the lock once
         m_owningThreadId = threadId; m_recursion = 1;
      }

      public void Leave() {
         // 如果调用线程不拥有锁，抛出异常
         Int32 threadId = Thread.CurrentThread.ManagedThreadId;
         if (threadId != m_owningThreadId)
            throw new SynchronizationLockException(&#34;Lock not owned by calling thread&#34;);

         // 递归锁减去次数
         if (--m_recursion &gt; 0) return;

         m_owningThreadId = 0;   // No thread owns the lock now

         // 没有其他线程在等待
         if (Interlocked.Decrement(ref m_waiters) == 0)
            return;

         // 有其他线程等待，唤起一个
         m_waiterLock.Set();	// Bad performance hit here
      }

      public void Dispose() { m_waiterLock.Dispose(); }
   }
```

### FCL中的混合构造

FCL也提供了很多的混合构造锁，一起研究以下。

#### ManualResetEventSlim和SemaphoreSlim

ManualResetEventSlim和SemaphoreSlim类，这两个构造的工作方式和内核模式构造都是瓦全一样的。都在用户模式自旋，在第一次竞争使才创建内核模式，wait方法允许传递一个超时值和一个CancellationToken。

#### Monitor和同步块

最常用的混合同步构造就是Monitor，它提供自旋、线程所有权和递归的互斥锁。

堆中每个对象都可以关联一个**同步块**的数据结构，该数据结构的字段：内核对象字段，拥有线程的ID字段，递归计数字段，等待线程计数字段。

Monitor是静态类，拥有的方法可以接受任何堆对象的引用，然后操作对象的同步块。

```c#
public static class Monitor
{
	public static void Enter(Object obj);
	public static void Exit(Object obj);
	//尝试进入锁时超时值
	public static Boolean TryEnter(Object obj,Int32 millisecondsTimeOut);
	public static void Enter(Object obj,ref Boolean lockTaken);
	public static void TryEnter(Object obj,Int32 millisecondsTimeout,ref Boolean lockTaken)；
}
```

如果每个对象都关联一个同步块数据就非常浪费内存，所以CLR进行了改进:CLR初始化时在堆中分配一个同步块数组。在堆中创建对象的实例，都有两个额外的开销：类型对象指针(指向类型对象的内存地址)，同步块索引：同步块数组中的整数索引。

对象实例在构造时，同步块索引初始化为-1，调用Monitor.Enter时，CLR找到一个空白同步块，并设置对象实例的同步块，在调用Exit时，检查是否有其他线程等待对象实例的同步块，如果没有同步块就自由了，对象实例同步块被设为-1.

![1582964468619](/dotnet/1582964468619.png)

三个对象实例通过类型对象指针，指向类型T，说明他们都是同一个类型,可以看出每个对象的同步块索引都隐式为公共的。这会造成问题。先看一下正常调用的情况。

```c#
internal sealed class Transaction
{
	private DateTime m_timeOfLastTrans;
	public void PerformTransaction()
	{
		Monitor.Enter(this);
		//以下代码拥有对数据的独占访问权
		m_timeOfLastTrans=DateTime.Now;
		Monitor.Exit(this);
	}
	public DateTime LastTransaction
	{
		get{
			Monitor.Enter(this);
			//以下线程拥有对数据的独占访问
			DateTime temp=m_timeOfLastTrans;
			Monitor.Exit(this);
			return temp;
		}
	}
}
```

有问题调用的情况

```
public static void SomeMethod()
{
	var t=new Transaction();
	Monitor.Enter(t);//获取锁
	ThreadPool.QueueUserWorkItem(o=&gt;Console.WriteLine(t.LastTransaction))；//如果在LastTransaction内部也是用Monitor.Enter(this)，线程会阻塞，因为在外部以及调用了Monitor.Enter(t)，获取了t的同步锁。
}
```

所以在使用Monitro的时候最好构造一个字段，也称私有锁。

```c#
internal sealed class Transaction
{
	private readonlu Object m_lock=new Object();
	private DateTIe m_timeOfLastTrans;
	public void PerformTransaction()
	{
		Monitor.Enter(m_lock);
		m_timeOfLastTrans=DateTime.Now;
		Monitor.Exit(m_lock);
	
	}
	public DateTime LastTransaction
	{
		get
		{
			Monitor.Enter(m_lock);
			DateTime temp=m_timeOfLastTrans;
			Monitor.Exit(m_lock);
			return temp
		}
	}
}
```

Monitor被设置为静态的还有一些其他的问题。所以尽量不要使用Monitor和lock

#### ReadWriteLockSlim类

读写锁：

&#43; 一个线程写入，所有访问线程阻塞
&#43; 读取时，所有写入线程阻塞
&#43; 写入线程结束，要么解除一个写入线程，要么解除所有读取线程阻塞。

```c#
   private sealed class Transactions : IDisposable {
      private readonly ReaderWriterLockSlim m_lock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);
      private DateTime m_timeOfLastTrans;

      public void PerformTransaction() {
         m_lock.EnterWriteLock();
         // This code has exclusive access to the data...
         m_timeOfLastTrans = DateTime.Now;
         m_lock.ExitWriteLock();
      }

      public DateTime LastTransaction {
         get {
            m_lock.EnterReadLock();
            // This code has shared access to the data...
            DateTime temp = m_timeOfLastTrans;
            m_lock.ExitReadLock();
            return temp;
         }
      }
      public void Dispose() { m_lock.Dispose(); }
   }
```

ReadWriteLockSlim构造器允许传递一个LockRecurionsPolicy标志，

```c#
public enum LockRecursionPolicy {NoRecursion,SupportsRecursion}
```

SupportsRecursion标志，允许传递线程所有权和递归。建议使用NoRecursion，因为SupportsRecursion代价很大，在内部使用了一个互斥的自旋锁来实现线程所有权和递归。

ReadWriteLockSlim运行权限升级，读取锁升级为写入锁。

存在的问题

&#43; 即使不存在线程竞争，速度也非常慢。
&#43; 线程所有权和递归行为是加强的取消不了。

#### OneManyLock类

JeffreyRicher提供的读写锁，

```c#
   /// &lt;summary&gt;
   /// Implements a ResourceLock by way of a high-speed reader/writer lock.
   /// &lt;/summary&gt;
   public sealed class OneManyLock : IDisposable {
      #region Lock State Management
#if false
      private struct BitField {
         private Int32 m_mask, m_1, m_startBit;
         public BitField(Int32 startBit, Int32 numBits) {
            m_startBit = startBit;
            m_mask = unchecked((Int32)((1 &lt;&lt; numBits) - 1) &lt;&lt; startBit);
            m_1 = unchecked((Int32)1 &lt;&lt; startBit);
         }
         public void Increment(ref Int32 value) { value &#43;= m_1; }
         public void Decrement(ref Int32 value) { value -= m_1; }
         public void Decrement(ref Int32 value, Int32 amount) { value -= m_1 * amount; }
         public Int32 Get(Int32 value) { return (value &amp; m_mask) &gt;&gt; m_startBit; }
         public Int32 Set(Int32 value, Int32 fieldValue) { return (value &amp; ~m_mask) | (fieldValue &lt;&lt; m_startBit); }
      }

      private static BitField s_state = new BitField(0, 3);
      private static BitField s_readersReading = new BitField(3, 9);
      private static BitField s_readersWaiting = new BitField(12, 9);
      private static BitField s_writersWaiting = new BitField(21, 9);
      private static OneManyLockStates State(Int32 value) { return (OneManyLockStates)s_state.Get(value); }
      private static void State(ref Int32 ls, OneManyLockStates newState) {
         ls = s_state.Set(ls, (Int32)newState);
      }
#endif
      private enum OneManyLockStates {
         Free = 0x00000000,
         OwnedByWriter = 0x00000001,
         OwnedByReaders = 0x00000002,
         OwnedByReadersAndWriterPending = 0x00000003,
         ReservedForWriter = 0x00000004,
      }

      private const Int32 c_lsStateStartBit = 0;
      private const Int32 c_lsReadersReadingStartBit = 3;
      private const Int32 c_lsReadersWaitingStartBit = 12;
      private const Int32 c_lsWritersWaitingStartBit = 21;

      // Mask = unchecked((Int32) ((1 &lt;&lt; numBits) - 1) &lt;&lt; startBit);
      private const Int32 c_lsStateMask = unchecked((Int32)((1 &lt;&lt; 3) - 1) &lt;&lt; c_lsStateStartBit);
      private const Int32 c_lsReadersReadingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsReadersReadingStartBit);
      private const Int32 c_lsReadersWaitingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsReadersWaitingStartBit);
      private const Int32 c_lsWritersWaitingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsWritersWaitingStartBit);
      private const Int32 c_lsAnyWaitingMask = c_lsReadersWaitingMask | c_lsWritersWaitingMask;

      // FirstBit = unchecked((Int32) 1 &lt;&lt; startBit);
      private const Int32 c_ls1ReaderReading = unchecked((Int32)1 &lt;&lt; c_lsReadersReadingStartBit);
      private const Int32 c_ls1ReaderWaiting = unchecked((Int32)1 &lt;&lt; c_lsReadersWaitingStartBit);
      private const Int32 c_ls1WriterWaiting = unchecked((Int32)1 &lt;&lt; c_lsWritersWaitingStartBit);

      private static OneManyLockStates State(Int32 ls) { return (OneManyLockStates)(ls &amp; c_lsStateMask); }
      private static void SetState(ref Int32 ls, OneManyLockStates newState) {
         ls = (ls &amp; ~c_lsStateMask) | ((Int32)newState);
      }

      private static Int32 NumReadersReading(Int32 ls) { return (ls &amp; c_lsReadersReadingMask) &gt;&gt; c_lsReadersReadingStartBit; }
      private static void AddReadersReading(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1ReaderReading * amount); }

      private static Int32 NumReadersWaiting(Int32 ls) { return (ls &amp; c_lsReadersWaitingMask) &gt;&gt; c_lsReadersWaitingStartBit; }
      private static void AddReadersWaiting(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1ReaderWaiting * amount); }

      private static Int32 NumWritersWaiting(Int32 ls) { return (ls &amp; c_lsWritersWaitingMask) &gt;&gt; c_lsWritersWaitingStartBit; }
      private static void AddWritersWaiting(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1WriterWaiting * amount); }

      private static Boolean AnyWaiters(Int32 ls) { return (ls &amp; c_lsAnyWaitingMask) != 0; }

      private static String DebugState(Int32 ls) {
         return String.Format(CultureInfo.InvariantCulture,
            &#34;State={0}, RR={1}, RW={2}, WW={3}&#34;, State(ls),
            NumReadersReading(ls), NumReadersWaiting(ls), NumWritersWaiting(ls));
      }

      /// &lt;summary&gt;
      /// Returns a string representing the state of the object.
      /// &lt;/summary&gt;
      /// &lt;returns&gt;The string representing the state of the object.&lt;/returns&gt;
      public override String ToString() { return DebugState(m_LockState); }
      #endregion

      #region State Fields
      private Int32 m_LockState = (Int32)OneManyLockStates.Free;

      // Readers wait on this if a writer owns the lock
      private Semaphore m_ReadersLock = new Semaphore(0, Int32.MaxValue);

      // Writers wait on this if a reader owns the lock
      private Semaphore m_WritersLock = new Semaphore(0, Int32.MaxValue);
      #endregion

      #region Construction and Dispose
      /// &lt;summary&gt;Constructs a OneManyLock object.&lt;/summary&gt;
      public OneManyLock() : base() { }

      public void Dispose() {
         m_WritersLock.Close(); m_WritersLock = null;
         m_ReadersLock.Close(); m_ReadersLock = null;
      }
      #endregion

      #region Writer members
      private Boolean m_exclusive;

      /// &lt;summary&gt;Acquires the lock.&lt;/summary&gt;
      public void Enter(Boolean exclusive) {
         if (exclusive) {
            while (WaitToWrite(ref m_LockState)) m_WritersLock.WaitOne();
         } else {
            while (WaitToRead(ref m_LockState)) m_ReadersLock.WaitOne();
         }
         m_exclusive = exclusive;
      }

      private static Boolean WaitToWrite(ref Int32 target) {
         Int32 start, current = target;
         Boolean wait;
         do {
            start = current;
            Int32 desired = start;
            wait = false;

            switch (State(desired)) {
               case OneManyLockStates.Free:  // If Free -&gt; OBW, return
               case OneManyLockStates.ReservedForWriter: // If RFW -&gt; OBW, return
                  SetState(ref desired, OneManyLockStates.OwnedByWriter);
                  break;

               case OneManyLockStates.OwnedByWriter:  // If OBW -&gt; WW&#43;&#43;, wait &amp; loop around
                  AddWritersWaiting(ref desired, 1);
                  wait = true;
                  break;

               case OneManyLockStates.OwnedByReaders: // If OBR or OBRAWP -&gt; OBRAWP, WW&#43;&#43;, wait, loop around
               case OneManyLockStates.OwnedByReadersAndWriterPending:
                  SetState(ref desired, OneManyLockStates.OwnedByReadersAndWriterPending);
                  AddWritersWaiting(ref desired, 1);
                  wait = true;
                  break;
               default:
                  Debug.Assert(false, &#34;Invalid Lock state&#34;);
                  break;
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wait;
      }

      /// &lt;summary&gt;Releases the lock.&lt;/summary&gt;
      public void Leave() {
         Int32 wakeup;
         if (m_exclusive) {
            Debug.Assert((State(m_LockState) == OneManyLockStates.OwnedByWriter) &amp;&amp; (NumReadersReading(m_LockState) == 0));
            // Pre-condition:  Lock&#39;s state must be OBW (not Free/OBR/OBRAWP/RFW)
            // Post-condition: Lock&#39;s state must become Free or RFW (the lock is never passed)

            // Phase 1: Release the lock
            wakeup = DoneWriting(ref m_LockState);
         } else {
            var s = State(m_LockState);
            Debug.Assert((State(m_LockState) == OneManyLockStates.OwnedByReaders) || (State(m_LockState) == OneManyLockStates.OwnedByReadersAndWriterPending));
            // Pre-condition:  Lock&#39;s state must be OBR/OBRAWP (not Free/OBW/RFW)
            // Post-condition: Lock&#39;s state must become unchanged, Free or RFW (the lock is never passed)

            // Phase 1: Release the lock
            wakeup = DoneReading(ref m_LockState);
         }

         // Phase 2: Possibly wake waiters
         if (wakeup == -1) m_WritersLock.Release();
         else if (wakeup &gt; 0) m_ReadersLock.Release(wakeup);
      }

      // Returns -1 to wake a writer, &#43;# to wake # readers, or 0 to wake no one
      private static Int32 DoneWriting(ref Int32 target) {
         Int32 start, current = target;
         Int32 wakeup = 0;
         do {
            Int32 desired = (start = current);

            // We do this test first because it is commonly true &amp; 
            // we avoid the other tests improving performance
            if (!AnyWaiters(desired)) {
               SetState(ref desired, OneManyLockStates.Free);
               wakeup = 0;
            } else if (NumWritersWaiting(desired) &gt; 0) {
               SetState(ref desired, OneManyLockStates.ReservedForWriter);
               AddWritersWaiting(ref desired, -1);
               wakeup = -1;
            } else {
               wakeup = NumReadersWaiting(desired);
               Debug.Assert(wakeup &gt; 0);
               SetState(ref desired, OneManyLockStates.OwnedByReaders);
               AddReadersWaiting(ref desired, -wakeup);
               // RW=0, RR=0 (incremented as readers enter)
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wakeup;
      }
      #endregion

      #region Reader members
      private static Boolean WaitToRead(ref Int32 target) {
         Int32 start, current = target;
         Boolean wait;
         do {
            Int32 desired = (start = current);
            wait = false;

            switch (State(desired)) {
               case OneManyLockStates.Free:  // If Free-&gt;OBR, RR=1, return
                  SetState(ref desired, OneManyLockStates.OwnedByReaders);
                  AddReadersReading(ref desired, 1);
                  break;

               case OneManyLockStates.OwnedByReaders: // If OBR -&gt; RR&#43;&#43;, return
                  AddReadersReading(ref desired, 1);
                  break;

               case OneManyLockStates.OwnedByWriter:  // If OBW/OBRAWP/RFW -&gt; RW&#43;&#43;, wait, loop around
               case OneManyLockStates.OwnedByReadersAndWriterPending:
               case OneManyLockStates.ReservedForWriter:
                  AddReadersWaiting(ref desired, 1);
                  wait = true;
                  break;

               default:
                  Debug.Assert(false, &#34;Invalid Lock state&#34;);
                  break;
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wait;
      }

      // Returns -1 to wake a writer, &#43;# to wake # readers, or 0 to wake no one
      private static Int32 DoneReading(ref Int32 target) {
         Int32 start, current = target;
         Int32 wakeup;
         do {
            Int32 desired = (start = current);
            AddReadersReading(ref desired, -1);  // RR--
            if (NumReadersReading(desired) &gt; 0) {
               // RR&gt;0, no state change &amp; no threads to wake
               wakeup = 0;
            } else if (!AnyWaiters(desired)) {
               SetState(ref desired, OneManyLockStates.Free);
               wakeup = 0;
            } else {
               Debug.Assert(NumWritersWaiting(desired) &gt; 0);
               SetState(ref desired, OneManyLockStates.ReservedForWriter);
               AddWritersWaiting(ref desired, -1);
               wakeup = -1;   // Wake 1 writer
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wakeup;
      }
      #endregion
   }
```

该锁维护了哪些内容：锁状态，正在读取reader线程数量(RR)，等待reader线程数量(RW，这些线程在AutoResetEvent阻塞)，正在等待写入write锁的数量(WW，在Semaphore上阻塞)

作责还提供了一个OneManyResourceLock可以死锁检测，锁的所有权和递归行为，锁的运行时行为，线程等待获取一个锁的最长时间和一个锁被占有的最短和最长时间。

```c#
   /// &lt;summary&gt;
   /// Implements a ResourceLock by way of a high-speed reader/writer lock.
   /// &lt;/summary&gt;
   public sealed class OneManyResourceLock : ResourceLock {
      #region Lock State Management
#if false
      private struct BitField {
         private Int32 m_mask, m_1, m_startBit;
         public BitField(Int32 startBit, Int32 numBits) {
            m_startBit = startBit;
            m_mask = unchecked((Int32)((1 &lt;&lt; numBits) - 1) &lt;&lt; startBit);
            m_1 = unchecked((Int32)1 &lt;&lt; startBit);
         }
         public void Increment(ref Int32 value) { value &#43;= m_1; }
         public void Decrement(ref Int32 value) { value -= m_1; }
         public void Decrement(ref Int32 value, Int32 amount) { value -= m_1 * amount; }
         public Int32 Get(Int32 value) { return (value &amp; m_mask) &gt;&gt; m_startBit; }
         public Int32 Set(Int32 value, Int32 fieldValue) { return (value &amp; ~m_mask) | (fieldValue &lt;&lt; m_startBit); }
      }

      private static BitField s_state = new BitField(0, 3);
      private static BitField s_readersReading = new BitField(3, 9);
      private static BitField s_readersWaiting = new BitField(12, 9);
      private static BitField s_writersWaiting = new BitField(21, 9);
      private static OneManyLockStates State(Int32 value) { return (OneManyLockStates)s_state.Get(value); }
      private static void State(ref Int32 ls, OneManyLockStates newState) {
         ls = s_state.Set(ls, (Int32)newState);
      }
#endif
      private enum OneManyLockStates {
         Free = 0x00000000,
         OwnedByWriter = 0x00000001,
         OwnedByReaders = 0x00000002,
         OwnedByReadersAndWriterPending = 0x00000003,
         ReservedForWriter = 0x00000004,
      }

      private const Int32 c_lsStateStartBit = 0;
      private const Int32 c_lsReadersReadingStartBit = 3;
      private const Int32 c_lsReadersWaitingStartBit = 12;
      private const Int32 c_lsWritersWaitingStartBit = 21;

      // Mask = unchecked((Int32) ((1 &lt;&lt; numBits) - 1) &lt;&lt; startBit);
      private const Int32 c_lsStateMask = unchecked((Int32)((1 &lt;&lt; 3) - 1) &lt;&lt; c_lsStateStartBit);
      private const Int32 c_lsReadersReadingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsReadersReadingStartBit);
      private const Int32 c_lsReadersWaitingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsReadersWaitingStartBit);
      private const Int32 c_lsWritersWaitingMask = unchecked((Int32)((1 &lt;&lt; 9) - 1) &lt;&lt; c_lsWritersWaitingStartBit);
      private const Int32 c_lsAnyWaitingMask = c_lsReadersWaitingMask | c_lsWritersWaitingMask;

      // FirstBit = unchecked((Int32) 1 &lt;&lt; startBit);
      private const Int32 c_ls1ReaderReading = unchecked((Int32)1 &lt;&lt; c_lsReadersReadingStartBit);
      private const Int32 c_ls1ReaderWaiting = unchecked((Int32)1 &lt;&lt; c_lsReadersWaitingStartBit);
      private const Int32 c_ls1WriterWaiting = unchecked((Int32)1 &lt;&lt; c_lsWritersWaitingStartBit);

      private static OneManyLockStates State(Int32 ls) { return (OneManyLockStates)(ls &amp; c_lsStateMask); }
      private static void SetState(ref Int32 ls, OneManyLockStates newState) {
         ls = (ls &amp; ~c_lsStateMask) | ((Int32)newState);
      }

      private static Int32 NumReadersReading(Int32 ls) { return (ls &amp; c_lsReadersReadingMask) &gt;&gt; c_lsReadersReadingStartBit; }
      private static void AddReadersReading(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1ReaderReading * amount); }

      private static Int32 NumReadersWaiting(Int32 ls) { return (ls &amp; c_lsReadersWaitingMask) &gt;&gt; c_lsReadersWaitingStartBit; }
      private static void AddReadersWaiting(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1ReaderWaiting * amount); }

      private static Int32 NumWritersWaiting(Int32 ls) { return (ls &amp; c_lsWritersWaitingMask) &gt;&gt; c_lsWritersWaitingStartBit; }
      private static void AddWritersWaiting(ref Int32 ls, Int32 amount) { ls &#43;= (c_ls1WriterWaiting * amount); }

      private static Boolean AnyWaiters(Int32 ls) { return (ls &amp; c_lsAnyWaitingMask) != 0; }

      private static String DebugState(Int32 ls) {
         return String.Format(CultureInfo.InvariantCulture,
            &#34;State={0}, RR={1}, RW={2}, WW={3}&#34;, State(ls),
            NumReadersReading(ls), NumReadersWaiting(ls),
            NumWritersWaiting(ls));
      }

      /// &lt;summary&gt;
      /// Returns a string representing the state of the object.
      /// &lt;/summary&gt;
      /// &lt;returns&gt;The string representing the state of the object.&lt;/returns&gt;
      public override String ToString() { return DebugState(m_LockState); }
      #endregion

      #region State Fields
      private Int32 m_LockState = (Int32)OneManyLockStates.Free;

      // Readers wait on this if a writer owns the lock
      private Semaphore m_ReadersLock = new Semaphore(0, Int32.MaxValue);

      // Writers wait on this if a reader owns the lock
      private Semaphore m_WritersLock = new Semaphore(0, Int32.MaxValue);
      #endregion

      #region Construction and Dispose
      /// &lt;summary&gt;Constructs a OneManyLock object.&lt;/summary&gt;
      public OneManyResourceLock() : base(ResourceLockOptions.None) { }

      ///&lt;summary&gt;Releases all resources used by the lock.&lt;/summary&gt;
      protected override void Dispose(Boolean disposing) {
         m_WritersLock.Close(); m_WritersLock = null;
         m_ReadersLock.Close(); m_ReadersLock = null;
         base.Dispose(disposing);
      }
      #endregion

      #region Writer members
      /// &lt;summary&gt;Acquires the lock.&lt;/summary&gt;
      protected override void OnEnter(Boolean exclusive) {
         if (exclusive) {
            while (WaitToWrite(ref m_LockState)) m_WritersLock.WaitOne();
         } else {
            while (WaitToRead(ref m_LockState)) m_ReadersLock.WaitOne();
         }
      }

      private static Boolean WaitToWrite(ref Int32 target) {
         Int32 start, current = target;
         Boolean wait;
         do {
            start = current;
            Int32 desired = start;
            wait = false;

            switch (State(desired)) {
               case OneManyLockStates.Free:  // If Free -&gt; OBW, return
               case OneManyLockStates.ReservedForWriter: // If RFW -&gt; OBW, return
                  SetState(ref desired, OneManyLockStates.OwnedByWriter);
                  break;

               case OneManyLockStates.OwnedByWriter:  // If OBW -&gt; WW&#43;&#43;, wait &amp; loop around
                  AddWritersWaiting(ref desired, 1);
                  wait = true;
                  break;

               case OneManyLockStates.OwnedByReaders: // If OBR or OBRAWP -&gt; OBRAWP, WW&#43;&#43;, wait, loop around
               case OneManyLockStates.OwnedByReadersAndWriterPending:
                  SetState(ref desired, OneManyLockStates.OwnedByReadersAndWriterPending);
                  AddWritersWaiting(ref desired, 1);
                  wait = true;
                  break;
               default:
                  Debug.Assert(false, &#34;Invalid Lock state&#34;);
                  break;
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wait;
      }

      /// &lt;summary&gt;Releases the lock.&lt;/summary&gt;
      protected override void OnLeave(Boolean write) {
         Int32 wakeup;
         if (write) {
            Debug.Assert((State(m_LockState) == OneManyLockStates.OwnedByWriter) &amp;&amp; (NumReadersReading(m_LockState) == 0));
            // Pre-condition:  Lock&#39;s state must be OBW (not Free/OBR/OBRAWP/RFW)
            // Post-condition: Lock&#39;s state must become Free or RFW (the lock is never passed)

            // Phase 1: Release the lock
            wakeup = DoneWriting(ref m_LockState);
         } else {
            var s = State(m_LockState);
            Debug.Assert((State(m_LockState) == OneManyLockStates.OwnedByReaders) || (State(m_LockState) == OneManyLockStates.OwnedByReadersAndWriterPending));
            // Pre-condition:  Lock&#39;s state must be OBR/OBRAWP (not Free/OBW/RFW)
            // Post-condition: Lock&#39;s state must become unchanged, Free or RFW (the lock is never passed)

            // Phase 1: Release the lock
            wakeup = DoneReading(ref m_LockState);
         }

         // Phase 2: Possibly wake waiters
         if (wakeup == -1) m_WritersLock.Release();
         else if (wakeup &gt; 0) m_ReadersLock.Release(wakeup);
      }

      // Returns -1 to wake a writer, &#43;# to wake # readers, or 0 to wake no one
      private static Int32 DoneWriting(ref Int32 target) {
         Int32 start, current = target;
         Int32 wakeup = 0;
         do {
            Int32 desired = (start = current);

            // We do this test first because it is commonly true &amp; 
            // we avoid the other tests improving performance
            if (!AnyWaiters(desired)) {
               SetState(ref desired, OneManyLockStates.Free);
               wakeup = 0;
            } else if (NumWritersWaiting(desired) &gt; 0) {
               SetState(ref desired, OneManyLockStates.ReservedForWriter);
               AddWritersWaiting(ref desired, -1);
               wakeup = -1;
            } else {
               wakeup = NumReadersWaiting(desired);
               Debug.Assert(wakeup &gt; 0);
               SetState(ref desired, OneManyLockStates.OwnedByReaders);
               AddReadersWaiting(ref desired, -wakeup);
               // RW=0, RR=0 (incremented as readers enter)
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wakeup;
      }
      #endregion

      #region Reader members
      private static Boolean WaitToRead(ref Int32 target) {
         Int32 start, current = target;
         Boolean wait;
         do {
            Int32 desired = (start = current);
            wait = false;

            switch (State(desired)) {
               case OneManyLockStates.Free:  // If Free-&gt;OBR, RR=1, return
                  SetState(ref desired, OneManyLockStates.OwnedByReaders);
                  AddReadersReading(ref desired, 1);
                  break;

               case OneManyLockStates.OwnedByReaders: // If OBR -&gt; RR&#43;&#43;, return
                  AddReadersReading(ref desired, 1);
                  break;

               case OneManyLockStates.OwnedByWriter:  // If OBW/OBRAWP/RFW -&gt; RW&#43;&#43;, wait, loop around
               case OneManyLockStates.OwnedByReadersAndWriterPending:
               case OneManyLockStates.ReservedForWriter:
                  AddReadersWaiting(ref desired, 1);
                  wait = true;
                  break;

               default:
                  Debug.Assert(false, &#34;Invalid Lock state&#34;);
                  break;
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wait;
      }

      // Returns -1 to wake a writer, &#43;# to wake # readers, or 0 to wake no one
      private static Int32 DoneReading(ref Int32 target) {
         Int32 start, current = target;
         Int32 wakeup;
         do {
            Int32 desired = (start = current);
            AddReadersReading(ref desired, -1);  // RR--
            if (NumReadersReading(desired) &gt; 0) {
               // RR&gt;0, no state change &amp; no threads to wake
               wakeup = 0;
            } else if (!AnyWaiters(desired)) {
               SetState(ref desired, OneManyLockStates.Free);
               wakeup = 0;
            } else {
               Debug.Assert(NumWritersWaiting(desired) &gt; 0);
               SetState(ref desired, OneManyLockStates.ReservedForWriter);
               AddWritersWaiting(ref desired, -1);
               wakeup = -1;   // Wake 1 writer
            }
            current = Interlocked.CompareExchange(ref target, desired, start);
         } while (start != current);
         return wakeup;
      }
      #endregion
   }
```

#### CountdownEvent类

内部使用MuanualResetEventSlim对象。

#### Barrier类

该类型可以控制一系列线程需要并行工作。

这儿有一个例子将GC回收线程操作完整说了一遍。

#### 线程同步构造小结

以下两种情况阻塞线程

&#43; 线程模型很简单
&#43; 线程有专门用途

### 著名的双检锁技术

单实例，

```c#
      public sealed class Singleton {
         // s_lock is required for thread safety and having this object assumes that creating  
         // the singleton object is more expensive than creating a System.Object object and that 
         // creating the singleton object may not be necessary at all. Otherwise, it is more  
         // efficient and easier to just create the singleton object in a class constructor
         private static readonly Object s_lock = new Object();

         // This field will refer to the one Singleton object
         private static Singleton s_value = null;

         // Private constructor prevents any code outside this class from creating an instance 
         private Singleton() { /* ... */ }

         // Public, static method that returns the Singleton object (creating it if necessary) 
         public static Singleton GetSingleton() {
            // If the Singleton was already created, just return it (this is fast)
            if (s_value != null) return s_value;

            Monitor.Enter(s_lock);  // Not created, let 1 thread create it
            if (s_value == null) {
               // Still not created, create it
               Singleton temp = new Singleton();

               // Save the reference in s_value (see discussion for details)
               Volatile.Write(ref s_value, temp);
            }
            Monitor.Exit(s_lock);

            // Return a reference to the one Singleton object 
            return s_value;
         }
      }
```

简单版本相同效果，缺点在于访问任何成员都会调用类型构造器

```c#
  public sealed class Singleton {
     private static Singleton s_value = new Singleton();

     // Private constructor prevents any code outside this class from creating an instance 
     private Singleton() { }

     // Public, static method that returns the Singleton object (creating it if necessary) 
     public static Singleton GetSingleton() { return s_value; }
  }
```

第三种方法，这个版本有可能创建多个s_value；很小的几率

```c#
      public sealed class Singleton {
         private static Singleton s_value = null;

         // Private constructor prevents any code outside this class from creating an instance 
         private Singleton() { }

         // Public, static method that returns the Singleton object (creating it if necessary) 
         public static Singleton GetSingleton() {
            if (s_value != null) return s_value;

            // Create a new Singleton and root it if another thread didn’t do it first
            Singleton temp = new Singleton();
            Interlocked.CompareExchange(ref s_value, temp, null);

            // If this thread lost, then the second Singleton object gets GC’d

            return s_value; // Return reference to the single object
         }
      }
```




---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/03/dotnetbase8-netclrvia/  

