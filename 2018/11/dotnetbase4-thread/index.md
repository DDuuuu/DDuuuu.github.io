# DotNet基础 线程


## 多线程和异步函数

&#43; **当异步线程在工作完成时如何通知调用线程**
&#43; **当异步线程出现异常的时候该如何处理**
&#43; **异步线程工作的进度如何实时的通知调用线程**
&#43; **如何在调用线程中取消正在工作的异步线程，并进行回滚操作**

### 异步函数模型

异步函数编程模式，**只要是使用委托对象封装的函数都可以实现该函数的异步调用**。因为委托类型有BeginInvoke和EndInvoke这两个方法来支持异步调用。

&#43; BeginInvoke无参数

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate void DoWork();
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        d.BeginInvoke(null, null);
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static void WorkPro()
    {
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
    }
}
}
 结果
Main Thread: 0
Asyn Thread: 0
Main Thread: 1
Asyn Thread: 1
Main Thread: 2
Asyn Thread: 2
Main Thread: 3
Asyn Thread: 3
Main Thread: 4
Main Thread Done
Asyn Thread: 4
Asyn Thread: 5
Asyn Thread: 6
Asyn Thread: 7
Asyn Thread: 8
Asyn Thread: 9
Asyn Thread Done
```

&#43; BeginInvoke有参数，
  &#43; BeginInvoke，IAsyncResult ，EndInvoke，使用这三个函数会等，**异步调用EndInvoke返回**再开启主线程，异步调用时间比主线程长，主线程会处于阻塞状态，阻塞位置在EndInvoke函数

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        IAsyncResult r = d.BeginInvoke(1000, null, null);
        int result = d.EndInvoke(r);
        Console.WriteLine($&#34;Asyn result:{result}&#34;);
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static int WorkPro(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
}
}
 结果：
Asyn Thread: 0
Asyn Thread: 1
Asyn Thread: 2
Asyn Thread: 3
Asyn Thread: 4
Asyn Thread: 5
Asyn Thread: 6
Asyn Thread: 7
Asyn Thread: 8
Asyn Thread: 9
Asyn Thread Done
Asyn result:45
Main Thread: 0
Main Thread: 1
Main Thread: 2
Main Thread: 3
Main Thread: 4
Main Thread Done
```

当调整EndInvoke的位置时，

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        IAsyncResult r = d.BeginInvoke(1000, null, null);
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        int result = d.EndInvoke(r);
        Console.WriteLine($&#34;Asyn result:{result}&#34;);
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static int WorkPro(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
}
}
结果：
Main Thread: 0
Asyn Thread: 0
Main Thread: 1
Asyn Thread: 1
Main Thread: 2
Asyn Thread: 2
Main Thread: 3
Asyn Thread: 3
Main Thread: 4
Asyn Thread: 4
Asyn Thread: 5
Asyn Thread: 6
Asyn Thread: 7
Asyn Thread: 8
Asyn Thread: 9
Asyn Thread Done
Asyn result:45
Main Thread Done
```

主线程执行时间比异步调用的长，并未看见阻塞

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        IAsyncResult r = d.BeginInvoke(1000, null, null);
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        int result = d.EndInvoke(r);
        Console.WriteLine($&#34;Asyn result:{result}&#34;);
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static int WorkPro(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
}
}
Main Thread: 0
Asyn Thread: 0
Main Thread: 1
Asyn Thread: 1
Main Thread: 2
Asyn Thread: 2
Main Thread: 3
Main Thread: 4
Asyn Thread: 3
Main Thread: 5
Asyn Thread: 4
Asyn Thread Done
Main Thread: 6
Main Thread: 7
Main Thread: 8
Main Thread: 9
Asyn result:10
Main Thread Done
```

### 自动通知主线程完成

上面两个例子这并不是理想的状态，理想状态是，异步调用完成后自动通知主线程完成，主线程调用。

BeginInvoke 方法启动异步调用。**该方法具有与你要异步执行的方法相同的参数，另加两个可选参数**。 第一个参数是一个 AsyncCallback 委托，此委托引用在异步调用完成时要调用的方法。 第二个参数是一个用户定义的对象object，该对象将信息传递到回调方法。

BeginInvoke 将立即返回，而不会等待异步调用完成。 BeginInvoke 返回可用于监视异步调用的进度的 IAsyncResult。

EndInvoke 方法用于检索异步调用的结果。 它可以在调用 BeginInvoke之后的任意时间调用。 如果异步调用尚未完成，那么 EndInvoke 将阻止调用线程，直到完成异步调用。 EndInvoke 的参数包括要异步执行的方法的 out 和 ref 参数，以及 BeginInvoke 返回的 IAsyncResult。

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        IAsyncResult r = d.BeginInvoke(1000, CallBack, d);
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static int WorkPro(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
    public static void CallBack(IAsyncResult r)
    {
        DoWork d = (DoWork)r.AsyncState;
        Console.WriteLine($&#34;Asyn result:{d.EndInvoke(r)}&#34;);
    }
}
}
Main Thread: 0
Asyn Thread: 0
Main Thread: 1
Asyn Thread: 1
Main Thread: 2
Asyn Thread: 2
Main Thread: 3
Asyn Thread: 3
Main Thread: 4
Asyn Thread: 4
Asyn Thread Done
Asyn result:10
Main Thread: 5
Main Thread: 6
Main Thread: 7
Main Thread: 8
Main Thread: 9
Main Thread Done
```

改进查看CallBack的执行情况

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        DoWork d = new DoWork(WorkPro);
        IAsyncResult r = d.BeginInvoke(1000, CallBack, d);
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread Done&#34;);
        Console.ReadKey();
    }
    private static int WorkPro(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread: {i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
    public static void CallBack(IAsyncResult r)
    {
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;CallBack: {i}&#34;);
        }
        DoWork d = (DoWork)r.AsyncState;
        Console.WriteLine($&#34;Asyn result:{d.EndInvoke(r)}&#34;);
    }
}
}
 
Main Thread: 0
Asyn Thread: 0
Main Thread: 1
Main Thread: 2
Asyn Thread: 1
Asyn Thread: 2
Main Thread: 3
Asyn Thread: 3
Main Thread: 4
Asyn Thread: 4
Asyn Thread Done
Main Thread: 5
Main Thread: 6
CallBack: 0
Main Thread: 7
CallBack: 1
Main Thread: 8
CallBack: 2
Main Thread: 9
Main Thread Done
CallBack: 3
CallBack: 4
Asyn result:10
```

回调函数也处于另一个线程中，与主线程并行执行，并且执行时间不等



## 正常的线程调用

.net在System.Threading和System.Threading.Tasks这两个命名空间中提供了Thread，ThreadPool，和Task三个类来处理多线程的问题，其中Thread是建立一个专用线程，ThreadPool是使用线程池中工作线程，而Task类是采用任务的方式，其内部也是使用线程池中的工作线程。

### Thread类

&#43; 使用方法很简单，它开辟的是一个专用线程，不是线程池中的工作线程，不由线程池去管理。该类提供4个重载版本，常见的使用前面两个就好了。
  &#43; `public Thread( ThreadStart start )`：其中ThreadStart是一个无参无返回值的委托类型。
  &#43; `public Thread( ParameterizedThreadStart start )`：其中ParameterizedThreadStart 是一个带有一个Object类型的参数，无返回值的委托类型。

从Thread类提供了两个构造函数可以看出，Thread类能够异步调用无参无返回值的函数，也能够异步调用带一个Object类型的无返回值的函数。下面就给出一个例子简单的演示一下如何使用Thread异步执行一个带参数的函数。


```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;  
namespace StartThread  
{  
	class Program  
	{  
		int interval = 200;  
		static void Main(string[] args)  
		{  
			Program p = new Program();  
			Thread nonParameterThread = new Thread(new ThreadStart(p.NonParameterRun));  
			nonParameterThread.Start();  
		}  
		/// &lt;summary&gt;  
		/// 不带参数的启动方法  
		/// &lt;/summary&gt;  
		public void NonParameterRun()  
		{  
			for (int i = 0; i &lt; 10; i&#43;&#43;)  
			{  
				Console.WriteLine(&#34;系统当前时间毫秒值：&#34;&#43;DateTime.Now.Millisecond.ToString());  
				Thread.Sleep(interval);//让线程暂停  
			}  
		}  
	}  
}
结果：
系统当前时间毫秒值：384
系统当前时间毫秒值：591
系统当前时间毫秒值：792
系统当前时间毫秒值：993
系统当前时间毫秒值：194
系统当前时间毫秒值：394
系统当前时间毫秒值：595
系统当前时间毫秒值：796
系统当前时间毫秒值：997
系统当前时间毫秒值：198
请按任意键继续. . .
```

## 带参数的线程

&#43; 线程输出10个值后就终止执行了，用ThreadStart委托作为构造函数来实例化thread是不带参数的，带参数的委托ParameterizedThreadStart，**其带有一个Object参数的方法**

```C#
using System;
using System.Threading;
namespace StartThread
{
    class Program
    {
        int interval = 200;
        static void Main(string[] args)
        {
            Program p = new Program();
            Thread parameterThread = new Thread(new ParameterizedThreadStart(p.ParameterRun));
            parameterThread.Name = &#34;Thread A:&#34;;
            parameterThread.Start(5);
        }
        /// &lt;summary&gt;  
        /// 带参数的启动方法  
        /// &lt;/summary&gt;  
        /// &lt;param name=&#34;ms&#34;&gt;让线程在运行过程中的休眠间隔&lt;/param&gt;  
        public void ParameterRun(object ms)
        {
            int j = 10;
            int.TryParse(ms.ToString(), out j);//这里采用了TryParse方法，避免不能转换时出现异常  
            for (int i = 0; i &lt; j; i&#43;&#43;)
            {
                Console.WriteLine(Thread.CurrentThread.Name &#43; &#34;系统当前时间毫秒值：&#34; &#43; DateTime.Now.Millisecond.ToString());
                Thread.Sleep(j);//让线程暂停  
            }
        }
    }
}
结果：
Thread A:系统当前时间毫秒值：127
Thread A:系统当前时间毫秒值：136
Thread A:系统当前时间毫秒值：142
Thread A:系统当前时间毫秒值：148
Thread A:系统当前时间毫秒值：154
请按任意键继续. . .  
```

## 两个线程参数不一样

&#43; 第一个线程启动后，线程实例就不需要存在了

```C#
两个线程间隔时间不一样
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;  
namespace StartThread  
{  
	class Program  
	{  
		int interval = 200;  
		static void Main(string[] args)  
		{  
			Program p = new Program();  
			Thread parameterThread = new Thread(new ParameterizedThreadStart(p.ParameterRun));  
			parameterThread.Name = &#34;Thread A:&#34;;  
			parameterThread.Start(30);  
			//启动第二个线程  
			parameterThread = new Thread(new ParameterizedThreadStart(p.ParameterRun));  
			parameterThread.Name = &#34;Thread B:&#34;;  
			parameterThread.Start(60);  
		}  
		/// &lt;summary&gt;  
		/// 带参数的启动方法  
		/// &lt;/summary&gt;  
		/// &lt;param name=&#34;ms&#34;&gt;让线程在运行过程中的休眠间隔&lt;/param&gt;  
		public void ParameterRun(object ms)  
		{  
			int j = 10;  
			int.TryParse(ms.ToString(), out j);//这里采用了TryParse方法，避免不能转换时出现异常  
			for (int i = 0; i &lt; 10; i&#43;&#43;)  
			{  
				Console.WriteLine(Thread.CurrentThread.Name&#43;&#34;系统当前时间毫秒值：&#34; &#43; DateTime.Now.Millisecond.ToString());  
				Thread.Sleep(j);//让线程暂停  
			}  
		}  
	}  
}  
结果：
Thread A:系统当前时间毫秒值：294
Thread B:系统当前时间毫秒值：294
Thread A:系统当前时间毫秒值：326
Thread B:系统当前时间毫秒值：357
Thread A:系统当前时间毫秒值：357
Thread A:系统当前时间毫秒值：388
Thread B:系统当前时间毫秒值：418
Thread A:系统当前时间毫秒值：419
Thread A:系统当前时间毫秒值：450
Thread B:系统当前时间毫秒值：479
Thread A:系统当前时间毫秒值：481
Thread A:系统当前时间毫秒值：511
Thread B:系统当前时间毫秒值：540
Thread A:系统当前时间毫秒值：543
Thread A:系统当前时间毫秒值：574
Thread B:系统当前时间毫秒值：600
Thread B:系统当前时间毫秒值：661
Thread B:系统当前时间毫秒值：723
Thread B:系统当前时间毫秒值：784
Thread B:系统当前时间毫秒值：845
请按任意键继续. . .
```

## 传递多参数

&#43; 如果需要传递两个参数怎么办呢，有两种方法
  &#43; 调用ParameterizedThreadStart，将参数封装成类，或者结构进行调用
  &#43; 构造线程类，将自己的线程和参数封装在一起

```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;  
namespace StartThread  
{  
	class MyThreadParameter  
	{  
		private int interval;  
		private int loopCount;  
		/// &lt;summary&gt;  
		/// 循环次数  
		/// &lt;/summary&gt;  
		public int LoopCount  
		{  
			get { return loopCount; }  
		}  
		/// &lt;summary&gt;  
		/// 线程的暂停间隔  
		/// &lt;/summary&gt;  
		public int Interval  
		{  
			get { return interval; }  
		}  
		/// &lt;summary&gt;  
		/// 构造函数  
		/// &lt;/summary&gt;  
		/// &lt;param name=&#34;interval&#34;&gt;线程的暂停间隔&lt;/param&gt;  
		/// &lt;param name=&#34;loopCount&#34;&gt;循环次数&lt;/param&gt;  
		public MyThreadParameter(int interval,int loopCount)  
		{  
			this.interval = interval;  
			this.loopCount = loopCount;  
		}  
	}  
	class Program  
	{  
		int interval = 200;  
		static void Main(string[] args)  
		{  
			Program p = new Program();  
  
			Thread parameterThread = new Thread(new ParameterizedThreadStart(p.MyParameterRun));  
			parameterThread.Name = &#34;Thread A:&#34;;  
			MyThreadParameter paramter = new MyThreadParameter(50, 5);  
			parameterThread.Start(paramter);  
		}  
		/// &lt;summary&gt;  
		/// 带多个参数的启动方法  
		/// &lt;/summary&gt;  
		/// &lt;param name=&#34;ms&#34;&gt;方法参数&lt;/param&gt;  
		public void MyParameterRun(object ms)  
		{  
			MyThreadParameter parameter = ms as MyThreadParameter;//类型转换  
			if (parameter != null)  
			{  
				for (int i = 0; i &lt; parameter.LoopCount; i&#43;&#43;)  
				{  
					Console.WriteLine(Thread.CurrentThread.Name &#43; &#34;系统当前时间毫秒值：&#34; &#43; DateTime.Now.Millisecond.ToString());  
					Thread.Sleep(parameter.Interval);//让线程暂停  
				}  
			}  
		}  
	}  
}  
结果：
Thread A:系统当前时间毫秒值：215
Thread A:系统当前时间毫秒值：270
Thread A:系统当前时间毫秒值：321
Thread A:系统当前时间毫秒值：372
Thread A:系统当前时间毫秒值：423
请按任意键继续. . .
```

```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;   
namespace StartThread  
{  
	class MyThreadParameter  
	{  
		private int interval;  
		private int loopCount;  
		private Thread thread;  	  
	/// &lt;summary&gt;  
	/// 构造函数  
	/// &lt;/summary&gt;  
		/// &lt;param name=&#34;interval&#34;&gt;线程的暂停间隔&lt;/param&gt;  
		/// &lt;param name=&#34;loopCount&#34;&gt;循环次数&lt;/param&gt;  
		public MyThreadParameter(int interval,int loopCount)  
		{  
			this.interval = interval;  
			this.loopCount = loopCount;  
			thread = new Thread(new ThreadStart(Run));  
		}  
		public void Start()  
		{  
			if (thread != null)  
			{  
				thread.Start();  
			}  
		}  
		private void Run()  
		{  
			for (int i = 0; i &lt; loopCount; i&#43;&#43;)  
			{  
				Console.WriteLine(&#34;系统当前时间毫秒值：&#34; &#43; DateTime.Now.Millisecond.ToString());  
				Thread.Sleep(interval);//让线程暂停  
			}  
		}  
	}  
	class Program  
	{  
		static void Main(string[] args)  
		{  
			MyThreadParameter parameterThread = new MyThreadParameter(30, 5);  
			parameterThread.Start();  
		}  
	}  
}  
结果
系统当前时间毫秒值：438
系统当前时间毫秒值：471
系统当前时间毫秒值：502
系统当前时间毫秒值：533
系统当前时间毫秒值：563
请按任意键继续. . .
```

## 多线程同干一件事

多线程同干一件事的时候对发生一些问题

```C#
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading;
namespace StartThread
{
    public class ThreadLock
    {
        private Thread threadOne;
        private Thread threadTwo;
        private List&lt;string&gt; ticketList;
        private object objLock = new object();
        public ThreadLock()
        {
            threadOne = new Thread(new ThreadStart(Run));
            threadOne.Name = &#34;Thread_1&#34;;
            threadTwo = new Thread(new ThreadStart(Run));
            threadTwo.Name = &#34;Thread_2&#34;;
        }
        static void Main(string[] args)
        {
            ThreadLock th = new ThreadLock();
            th.Start();
        }
        public void Start()
        {
            ticketList = new List&lt;string&gt;(10);
            for (int i = 1; i &lt;= 10; i&#43;&#43;)
            {
                ticketList.Add(i.ToString().PadLeft(3, &#39;0&#39;));//实现3位的票号，如果不足3位数，则以0补足3位  
            }
            threadOne.Start();
            threadTwo.Start();
        }
        private void Run()
        {
            while (ticketList.Count &gt; 0)//①  
            {
                string ticketNo = ticketList[0];//②  
                Console.WriteLine(&#34;{0}:售出一张票，票号：{1}&#34;, Thread.CurrentThread.Name, ticketNo);
                ticketList.RemoveAt(0);//③  
                Thread.Sleep(1);
            }
        }
    }
} 
结果：
Thread_1:售出一张票，票号：001
Thread_2:售出一张票，票号：001
Thread_1:售出一张票，票号：003
Thread_2:售出一张票，票号：004
Thread_1:售出一张票，票号：005
Thread_2:售出一张票，票号：006
Thread_1:售出一张票，票号：007
Thread_2:售出一张票，票号：008
Thread_1:售出一张票，票号：008
Thread_2:售出一张票，票号：010
Thread_1:售出一张票，票号：010
未经处理的异常:  System.ArgumentOutOfRangeException: 索引超出范围。必须为非负值并小于集合大小。
参数名: index
   在 System.ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument argument, ExceptionResource resource)
   在 System.Collections.Generic.List`1.RemoveAt(Int32 index)
   在 StartThread.ThreadLock.Run() 位置 C:\Users\DuJinfeng\Desktop\C#源码\test\test\练习\test\Program.cs:行号 79
   在 System.Threading.ThreadHelper.ThreadStart_Context(Object state)
   在 System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
   在 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
   在 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
   在 System.Threading.ThreadHelper.ThreadStart()
请按任意键继续. . .
```

&#43; 该程序在③处会出现问题，如果第一个线程在③处的时间片段正好用完第二个线程调用就会出现一张票被卖两次的问题
&#43; 同步问题的解决方法：
  &#43; lock、
  &#43; Mutex、
  &#43; Monitor、
  &#43; Semaphore、
  &#43; Interlocked
  &#43; ReaderWriterLock等
  &#43; 同步策略可以有同步上下文、同步代码区、手动同步

### 上下文同步策略

&#43; 上下文同步：
&#43; 同步上下文的策略主要是依靠SynchronizationAttribute类来实现

```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
//需要添加对System.EnterpriseServices.dll这个类库的引用采用使用这个dll  
using System.EnterpriseServices;  
namespace StartThread  
{  
    [Synchronization(SynchronizationOption.Required)]//确保创建的对象已经同步  
    public class SynchronizationAttributeClass  
    {  
        public void Run()  
        {  
        }  
    }  
} 
```

&#43; 所有在同一个上下文域的对象共享同一个锁。这样创建的对象实例属性、方法和字段就具有线程安全性，需要注意的是类的静态字段、属性和方法是不具有线程安全性的。


### 同步代码区

同步代码区是另外一种策略，它是针对特定部分代码进行同步的一种方法

&#43; lock同步

```C#
private void Run()  
{  
    while (ticketList.Count &gt; 0)//①  
        {  
            lock (objLock)  
            {  
                if (ticketList.Count &gt; 0)//必须要再一次判断在1之后可能进入其他线程
                {  
                    string ticketNo = ticketList[0];//②  
                    Console.WriteLine(&#34;{0}:售出一张票，票号：{1}&#34;, Thread.CurrentThread.Name, ticketNo);  
                    ticketList.RemoveAt(0);//③  
                    Thread.Sleep(1);  
                }  
            }  
    }  
}  
```

&#43; Monitor类同步

```C#
private void Run()  
{  
    while (ticketList.Count &gt; 0)//①  
        {  
            Monitor.Enter(objLock);  
                if (ticketList.Count &gt; 0)  
                {  
                    string ticketNo = ticketList[0];//②  
                    Console.WriteLine(&#34;{0}:售出一张票，票号：{1}&#34;, Thread.CurrentThread.Name, ticketNo);  
                    ticketList.RemoveAt(0);//③  
                    Thread.Sleep(1);  
                }  
            Monitor.Exit(objLock);  
    }  
}  
```

&#43; 使用lock关键字的代码实际上是用Monitor来实现的。

```C#
lock (objLock){  
//同步代码  
}  
//等价于
try{  
Monitor.Enter(objLock);  
//同步代码  
}  
finally  
{  
Monitor.Exit(objLock);  
}  
```

&#43; Monitor类除了Enter()和Exit()方法之外，还有Wait()和Pulse()方法。
&#43; Wait()方法是**临时释放当前活得的锁**，并使当前对象处于阻塞状态
&#43; Pulse()方法是通知处于等待状态的对象可以准备就绪了，它一会就会释放锁

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Collections;
using System.Threading;
public class ThreadDemo
{
    private Thread threadOne;
    private Thread threadTwo;
    private ArrayList stringList;
    private event EventHandler OnNumberClear;//数据删除完成引发的事件
    public static void Main()
    {
        ThreadDemo demo = new ThreadDemo(10);
        demo.Action();
        Console.ReadKey();
    }
    public ThreadDemo(int number)
    {
        Random random = new Random(100);
        stringList = new ArrayList(number);
        for (int i = 0; i &lt; number; i&#43;&#43;)
        {
            stringList.Add(random.Next().ToString());
        }
        threadOne = new Thread(new ThreadStart(Run));//两个线程共同做一件事情
        threadTwo = new Thread(new ThreadStart(Run));//两个线程共同做一件事情

        threadOne.Name = &#34;线程1&#34;;
        threadTwo.Name = &#34;线程2&#34;;
        for (int i = 0; i &lt; number; i&#43;&#43;)
        {
            Console.WriteLine($&#34;主线程1：{i.ToString()}&#34;);
        }
        OnNumberClear &#43;= new EventHandler(ThreadDemo_OnNumberClear);
    }
    /// &lt;summary&gt;
    /// 开始工作
    /// &lt;/summary&gt;
    public void Action()
    {
        threadOne.Start();
        threadTwo.Start();
    }
    /// &lt;summary&gt;
    /// 共同做的工作
    /// &lt;/summary&gt;
    private void Run()
    {
        string stringValue = null;
        while (true)
        {
            Monitor.Enter(this);//锁定，保持同步
            stringValue = (string)stringList[0];
            Console.WriteLine(Thread.CurrentThread.Name &#43; &#34;删除了&#34; &#43; stringValue);
            stringList.RemoveAt(0);//删除ArrayList中的元素
            if (stringList.Count == 0)
            {
                OnNumberClear(this, new EventArgs());//引发完成事件
            }
            Monitor.Exit(this);//取消锁定
            Thread.Sleep(5);
        }
    }
    //执行完成之后，停止所有线程
    void ThreadDemo_OnNumberClear(object sender, EventArgs e)
    {
        Console.WriteLine(&#34;执行完了，停止了所有线程的执行。&#34;);
        threadTwo.Abort();
        threadOne.Abort();//终止线程调用
    }
}
结果：
主线程1：0
主线程1：1
主线程1：2
主线程1：3
主线程1：4
主线程1：5
主线程1：6
主线程1：7
主线程1：8
主线程1：9
线程1删除了2080427802
线程2删除了341851734
线程1删除了1431988776
线程2删除了1938005744
线程1删除了761513014
线程2删除了2037243568
线程1删除了1528357293
线程2删除了1311292502
线程1删除了749943798
线程2删除了319576108
执行完了，停止了所有线程的执行。
```

```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;  
namespace StartThread  
{  
    public class ThreadWaitAndPluse  
    {  
        private object lockObject;  
        private int number;  
        private Random random;  
        public ThreadWaitAndPluse()  
        {  
            lockObject = new object();  
            random = new Random();  
        }  
        //显示生成数据的线程要执行的方法  
        public void ThreadMethodOne()  
        {  
            Monitor.Enter(lockObject);//获取对象锁  
            Console.WriteLine(&#34;当前进入的线程：&#34; &#43; Thread.CurrentThread.GetHashCode());  
            for (int i = 0; i &lt; 5; i&#43;&#43;)  
            {  
                Monitor.Wait(lockObject);//释放对象锁，并阻止当前线程  
                Console.WriteLine(&#34;WaitAndPluse1:工作&#34;);  
                Console.WriteLine(&#34;WaitAndPluse1:得到了数据，number=&#34; &#43; number &#43; &#34;,Thread ID=&#34; &#43; Thread.CurrentThread.GetHashCode());  
                //通知其它等待锁的对象状态已经发生改变,当这个对象释放锁之后等待锁的对象将会活得锁  
                Monitor.Pulse(lockObject);  
            }  
            Console.WriteLine(&#34;退出当前线程：&#34; &#43; Thread.CurrentThread.GetHashCode());  
            Monitor.Exit(lockObject);//释放对象锁  
        }  
        //生成随机数据线程要执行的方法  
        public void ThreadMethodTwo()  
        {  
            Monitor.Enter(lockObject);//获取对象锁  
            Console.WriteLine(&#34;当前进入的线程：&#34; &#43; Thread.CurrentThread.GetHashCode());  
            for (int i = 0; i &lt; 5; i&#43;&#43;)  
            {  
                //通知其它等待锁的对象状态已经发生改变,当这个对象释放锁之后等待锁的对象将会活得锁  
                Monitor.Pulse(lockObject);  
                Console.WriteLine(&#34;WaitAndPluse2:工作&#34;);  
                number =random.Next(DateTime.Now.Millisecond);//生成随机数  
                Console.WriteLine(&#34;WaitAndPluse2:生成了数据，number=&#34; &#43; number &#43; &#34;,Thread ID=&#34; &#43; Thread.CurrentThread.GetHashCode());  
                Monitor.Wait(lockObject);//释放对象锁，并阻止当前线程  
            }  
            Console.WriteLine(&#34;退出当前线程：&#34; &#43; Thread.CurrentThread.GetHashCode());  
            Monitor.Exit(lockObject);//释放对象锁  
        }  
        public static void Main()  
        {  
            ThreadWaitAndPluse demo=new ThreadWaitAndPluse();  
            Thread t1 = new Thread(new ThreadStart(demo.ThreadMethodOne));  
            t1.Start();  
            Thread t2 = new Thread(new ThreadStart(demo.ThreadMethodTwo));  
            t2.Start();  
            Console.ReadLine();  
        }  
    }  
}  
结果：
当前进入的线程：3
当前进入的线程：4
WaitAndPluse2:工作
WaitAndPluse2:生成了数据，number=128,Thread ID=4
WaitAndPluse1:工作
WaitAndPluse1:得到了数据，number=128,Thread ID=3
WaitAndPluse2:工作
WaitAndPluse2:生成了数据，number=34,Thread ID=4
WaitAndPluse1:工作
WaitAndPluse1:得到了数据，number=34,Thread ID=3
WaitAndPluse2:工作
WaitAndPluse2:生成了数据，number=615,Thread ID=4
WaitAndPluse1:工作
WaitAndPluse1:得到了数据，number=615,Thread ID=3
WaitAndPluse2:工作
WaitAndPluse2:生成了数据，number=759,Thread ID=4
WaitAndPluse1:工作
WaitAndPluse1:得到了数据，number=759,Thread ID=3
WaitAndPluse2:工作
WaitAndPluse2:生成了数据，number=894,Thread ID=4
WaitAndPluse1:工作
WaitAndPluse1:得到了数据，number=894,Thread ID=3
退出当前线程：3
退出当前线程：4
```

&#43; 这个一般情况下都是正常的，但是不能保证线程1先执行,为了避免这种情况可以让线程2延迟一个合适的时间

### 手动同步

&#43; ReaderWriterLock：ReaderWriterLock支持**单个写线程和多个读线程的锁**，使用ReaderWriterLock来进行读写同步比使用监视的方式（如Monitor）效率要高
&#43; .NET Framework 具有两个读取器 / 编写器锁， ReaderWriterLockSlim 和 ReaderWriterLock。 ReaderWriterLockSlim 建议将所有新的开发的。 ReaderWriterLockSlim 类似于 ReaderWriterLock, ，只是简化了递归、 升级和降级锁定状态的规则。 ReaderWriterLockSlim 可避免潜在的死锁的很多情况。 此外，性能的 ReaderWriterLockSlim 明显优于 ReaderWriterLock。
&#43; ReaderWriterLock 用于同步对资源的访问。 在任何给定时间，它允许多个线程的并发读访问权限，或者一个单独的线程的写访问权限。 在某个资源，很少更改的情况下 ReaderWriterLock 提供了更好的吞吐量比简单的一次锁，如 Monitor。

&#43; ReaderWriterLock 其中大多数的访问权限是读取，而写入是很少和持续时间较短的效果最佳。 多个读取器交替使用单个编写器，以便读取器和编写器都不被阻止较长时间

下面的示例演示如何使用 ReaderWriterLock 若要保护的共享的资源，一个整数值，名为 resource, ，即并发读取和写入以独占方式由多个线程。 请注意， ReaderWriterLock 以便对所有线程可见的类级别声明。

```C#
using System;
using System.Threading;
public class Example
{
    static ReaderWriterLock rwl = new ReaderWriterLock();
    // 定义受ReaderWriterLock保护的共享资源。
    static int resource = 0;
    /// &lt;summary&gt;
    /// 线程数量
    /// &lt;/summary&gt;
    const int numThreads = 10;
    /// &lt;summary&gt;
    /// 线程启动开关
    /// &lt;/summary&gt;
    static bool running = true;
    /// &lt;summary&gt;
    /// 随机
    /// &lt;/summary&gt;
    static Random rnd = new Random();
    // 读取超时时间
    static int readerTimeouts = 0;
    //写入超时时间
    static int writerTimeouts = 0;
    //读取次数
    static int reads = 0;
    //写入次数
    static int writes = 0;
    public static void Main()
    {
        // 启动一系列线程以随机读取和写入共享资源。
        Thread[] t = new Thread[numThreads];
        for (int i = 0; i &lt; numThreads; i&#43;&#43;)
        {
            t[i] = new Thread(new ThreadStart(ThreadProc));
            t[i].Name = new String(Convert.ToChar(i &#43; 65), 1);
            t[i].Start();
            Console.WriteLine($&#34;线程：{t[i].Name }启动&#34;);
            if (i &gt; 10)
                Thread.Sleep(300);
        }
        // 告诉线程关闭并等待它们全部完成。
        running = false;
        for (int i = 0; i &lt; numThreads; i&#43;&#43;)
        {
            t[i].Join();//阻止调用线程，直到某个线程终止时为止。
            Console.WriteLine($&#34;线程：{t[i].Name }阻止&#34;);
        }
        // 显示统计信息
        Console.WriteLine($&#34;\n{reads} 次读, {writes} 次写, 读取请求超时时间{readerTimeouts} , 写请求超时时间{writerTimeouts} .&#34;);
        Console.Write(&#34;退出 &#34;);
        Console.ReadLine();
    }
    static void ThreadProc()
    {
        // 随机选择线程从共享资源中读取和写入的方式
        while (running)
        {
            double action = rnd.NextDouble();//返回一个0-1之间的随机数
            if (action &lt; .7)
                ReadFromResource(10);//80%读
            else if (action &lt; .81)
                ReleaseRestore(50);
            else if (action &lt; .90)
                UpgradeDowngrade(100);
            else
                WriteToResource(100);
        }
    }
    // 请求并释放读卡器锁，并处理超时。
    static void ReadFromResource(int timeOut)
    {
        try
        {
            rwl.AcquireReaderLock(timeOut);//获取读线程锁。//使用一个int超时值获取读线程
            try
            {
                // 此线程可以安全地从共享资源中读取。
                Display(&#34;读取资源值:&#34; &#43; resource);
                Interlocked.Increment(ref reads);//对资源操作
            }
            finally
            {
                // 确保已释放锁定。
                rwl.ReleaseReaderLock();//减少锁计数。
            }
        }
        catch (ApplicationException)
        {
            //读卡器锁定请求超时。
            Interlocked.Increment(ref readerTimeouts);//时间增加
        }
    }
    // 请求并释放写入程序锁定，并处理超时。
    static void WriteToResource(int timeOut)
    {
        try
        {
            rwl.AcquireWriterLock(timeOut);//获取写线程锁。//使用一个int超时值获取写线程
            try
            {
                // 此线程可以安全地从共享资源进行访问。
                resource = rnd.Next(100);
                Display(&#34;写资源值: &#34; &#43; resource);
                Interlocked.Increment(ref writes);//写入次数
            }
            finally
            {
                // 确保已释放锁定。
                rwl.ReleaseWriterLock();//减少写线程上锁的计数
            }
        }
        catch (ApplicationException)
        {
            // The writer lock request timed out.
            Interlocked.Increment(ref writerTimeouts);
        }
    }
    // 请求读取器锁定，将读取器锁定升级到写入器锁定，并再次将其降级为读取器锁定。
    static void UpgradeDowngrade(int timeOut)
    {
        try
        {
            rwl.AcquireReaderLock(timeOut);//通过设置的读时间int值获取读线程锁
            try
            {
                // 这个线程从共享资源中读取是安全的。
                Display(&#34;读资源值: &#34; &#43; resource);
                Interlocked.Increment(ref reads);

                // 要写入资源，要么释放读取器锁定并请求写入程序锁定，要么升级读取器锁定。
                //升级读取器锁将线程置于写入队列中，位于可能正在等待写入器锁定的任何其他线程之后。
                try
                {
                    LockCookie lc = rwl.UpgradeToWriterLock(timeOut);//将一个设置超时int值的读线程锁升级为写线程锁
                    try
                    {
                        // 此线程可以安全地从共享资源读取或写入。
                        resource = rnd.Next(100);
                        Display(&#34;从读锁变成写锁写资源值: &#34; &#43; resource);
                        Interlocked.Increment(ref writes);//写值
                    }
                    finally
                    {
                        // 确保已释放锁定。
                        rwl.DowngradeFromWriterLock(ref lc);//将线程的锁状态还原为调用 UpgradeToWriterLock 前的状态。
                    }
                }
                catch (ApplicationException)
                {
                    // 更新写线程时间
                    Interlocked.Increment(ref writerTimeouts);
                }

                // 如果锁被降级，从资源中读取仍然是安全的。
                Display(&#34;从读锁变成写锁再变读锁读资源值: &#34; &#43; resource);
                Interlocked.Increment(ref reads);
            }
            finally
            {
                // 确保已释放锁定。
                rwl.ReleaseReaderLock();
            }
        }
        catch (ApplicationException)
        {
            // 读卡器锁定请求超时。
            Interlocked.Increment(ref readerTimeouts);
        }
    }
    //释放所有锁，然后恢复锁定状态。
    //使用序列号来确定另一个线程是否已获得写入器锁定，因为此线程上次访问该资源。
    static void ReleaseRestore(int timeOut)
    {
        int lastWriter;
        try
        {
            rwl.AcquireReaderLock(timeOut);//使用一个 Int32 超时值获取读线程锁。
            try
            {
                // 此线程可以安全地从共享资源中读取，因此读取并缓存资源值。
                int resourceValue = resource;     // 缓存资源值。
                Display(&#34;读取资源值: &#34; &#43; resourceValue);
                Interlocked.Increment(ref reads);
                // 保存当前的编写器序列号。
                lastWriter = rwl.WriterSeqNum;
                // 释放锁并保存cookie，以便稍后恢复锁定。
                LockCookie lc = rwl.ReleaseLock();//释放锁，不管线程获取锁的次数如何。
                // 等待一个随机间隔，然后恢复以前的锁定状态。
                Thread.Sleep(rnd.Next(250));
                rwl.RestoreLock(ref lc);
                // 检查其他线程是否在间隔中获得了写入器锁定。 如果不是，则资源的缓存值仍然有效。
                if (rwl.AnyWritersSince(lastWriter))//指示获取序列号之后是否已将写线程锁授予某个线程。
                {
                    resourceValue = resource;
                    Interlocked.Increment(ref reads);
                    Display(&#34;读锁毁灭再恢复，资源值再次读:&#34; &#43; resourceValue);
                }
                else
                {
                    Display(&#34;读锁没有恢复：资源值没有改变:&#34; &#43; resourceValue);
                }
            }
            finally
            {
                // 确保锁释放
                rwl.ReleaseReaderLock();
            }
        }
        catch (ApplicationException)
        {
            // 读线程锁超时
            Interlocked.Increment(ref readerTimeouts);
        }
    }
    // Helper方法简要显示最近的线程操作。
    static void Display(string msg)
    {
        Console.WriteLine($&#34;线程 {Thread.CurrentThread.Name}: {msg}.&#34;);
    }
}
结果：
线程：A启动
线程 A: 读取资源值: 0.
线程：B启动
线程 B: 读取资源值:0.
线程 B: 读取资源值:0.
线程：C启动
线程 C: 读取资源值:0.
线程：D启动
线程 B: 写资源值: 12.
线程 B: 读取资源值: 12.
线程 C: 读取资源值:12.
线程 C: 读取资源值:12.
线程 C: 读取资源值:12.
线程 C: 读取资源值:12.
线程 D: 读取资源值:12.
线程 E: 读取资源值:12.
线程 C: 写资源值: 8.
线程 C: 读取资源值:8.
线程 D: 读取资源值:8.
线程 D: 读取资源值:8.
线程 D: 读取资源值:8.
线程 E: 读取资源值:8.
线程 E: 读取资源值:8.
线程 E: 读取资源值:8.
线程 E: 读取资源值:8.
线程 D: 读资源值: 8.
线程 E: 读取资源值:8.
线程 C: 读取资源值: 8.
线程 D: 从读锁变成写锁写资源值: 33.
线程 D: 从读锁变成写锁再变读锁读资源值: 33.
线程 D: 读取资源值:33.
线程 D: 读取资源值:33.
线程 D: 读取资源值:33.
线程 D: 读资源值: 33.
线程 E: 读取资源值:33.
线程 D: 从读锁变成写锁写资源值: 84.
线程：E启动
线程 D: 从读锁变成写锁再变读锁读资源值: 84.
线程 D: 读取资源值:84.
线程 E: 读取资源值:84.
线程 F: 写资源值: 0.
线程 F: 读取资源值: 0.
线程：F启动
线程 E: 读取资源值:0.
线程 E: 读取资源值:0.
线程 E: 读取资源值:0.
线程 E: 读取资源值:0.
线程 E: 读取资源值: 0.
线程：G启动
线程：H启动
线程 H: 读取资源值:0.
线程 H: 读取资源值:0.
线程：I启动
线程 I: 读取资源值:0.
线程：J启动
线程 D: 读取资源值: 0.
线程 G: 读资源值: 0.
线程 H: 写资源值: 18.
线程 I: 读取资源值:18.
线程 J: 写资源值: 44.
线程 G: 从读锁变成写锁写资源值: 18.
线程 G: 从读锁变成写锁再变读锁读资源值: 18.
线程 C: 读锁毁灭再恢复，资源值再次读:18.
线程 F: 读锁毁灭再恢复，资源值再次读:18.
线程 B: 读锁毁灭再恢复，资源值再次读:18.
线程 A: 读锁毁灭再恢复，资源值再次读:18.
线程：A阻止
线程：B阻止
线程：C阻止
线程 E: 读锁毁灭再恢复，资源值再次读:18.
线程 D: 读锁毁灭再恢复，资源值再次读:18.
线程：D阻止
线程：E阻止
线程：F阻止
线程：G阻止
线程：H阻止
线程：I阻止
线程：J阻止
50 次读, 8 次写, 读取请求超时时间0 , 写请求超时时间0 .
退出
```

### AutoResetEvent 类

通知正在等待的线程已发生事件。MSCN介绍中，**终止状态就是有信号**

&#43; AutoResetEvent（bool isover）：构造函数指示是否指示是否将初始状态设置为终止的
&#43; Reset:将事件状态设置为非终止状态，导致线程阻止。
&#43; Set:将事件状态设置为终止状态，允许一个或多个等待线程继续。**将事件状态设置为有信号**
&#43; WaitOne():阻止当前线程，直到当前 WaitHandle 收到信号。**会自动改变信号，讲有信号自动变成无信号，需要自己调用set**
&#43; WaitOne(Int32):阻止当前线程，直到当前 WaitHandle 收到信号，同时使用 32 位带符号整数指定时间间隔

``` C#
using System;
using System.Threading;
class Example
{
    //mscn中终止状态就是有信号
    //全局
    private static AutoResetEvent event_1 = new AutoResetEvent(true);//初始化为终止状态，有信号
    private static AutoResetEvent event_2 = new AutoResetEvent(false);//初始状态为非终止状态，没有信号
    static void Main()
    {
        Console.WriteLine(&#34;按Enter键创建三个线程并启动它们。\r\n&#34; &#43;
                          &#34;线程在创建的AutoResetEvent1上等待\r\n&#34; &#43;
                          &#34;在信号状态，所以第一个线程被释放。\r\n&#34; &#43;
                          &#34;这使AutoResetEvent1进入无信号状态.&#34;);
        Console.ReadLine();
        for (int i = 1; i &lt; 4; i&#43;&#43;)
        {
            Thread t = new Thread(ThreadProc);
            t.Name =  i&#43; &#34;号线程&#34;;
            Console.WriteLine(&#34;{0}号线程开始&#34;,t.Name);
            t.Start();
        }
        Thread.Sleep(250);
        for (int i = 0; i &lt; 2; i&#43;&#43;)
        {
            Console.WriteLine(&#34;按Enter键以释放另一个线程。&#34;);
            Console.ReadLine();
            event_1.Set();
            Thread.Sleep(250);
        }
        Console.WriteLine(&#34;\r\n所有线程现在都在等待AutoResetEvent＃2。&#34;);
        for (int i = 0; i &lt; 3; i&#43;&#43;)
        {
            Console.WriteLine(&#34;按Enter键以释放线程。&#34;);
            Console.ReadLine();
            event_2.Set();
            Thread.Sleep(250);
        }
    }
    static void ThreadProc()
    {
        string name = Thread.CurrentThread.Name;
        Console.WriteLine(&#34;{0} 等待AutoResetEvent # 1。&#34;, name);
        event_1.WaitOne();
        Console.WriteLine(&#34;{0} 从AutoResetEvent # 1中释放。&#34;, name);
        Console.WriteLine(&#34;{0} 等待AutoResetEvent # 2。&#34;, name);
        event_2.WaitOne();
        Console.WriteLine(&#34;{0} 从AutoResetEvent # 2中释放。&#34;, name);
        Console.WriteLine(&#34;{0} 结束。&#34;, name);
    }
}
结果
按Enter键创建三个线程并启动它们。
线程在创建的AutoResetEvent1上等待
在信号状态，所以第一个线程被释放。
这使AutoResetEvent1进入无信号状态.

1号线程号线程开始
2号线程号线程开始
1号线程 等待AutoResetEvent # 1。
1号线程 从AutoResetEvent # 1中释放。
1号线程 等待AutoResetEvent # 2。
2号线程 等待AutoResetEvent # 1。
3号线程号线程开始
3号线程 等待AutoResetEvent # 1。
按Enter键以释放另一个线程。

2号线程 从AutoResetEvent # 1中释放。
2号线程 等待AutoResetEvent # 2。
按Enter键以释放另一个线程。

3号线程 从AutoResetEvent # 1中释放。
3号线程 等待AutoResetEvent # 2。

所有线程现在都在等待AutoResetEvent＃2。
按Enter键以释放线程。

1号线程 从AutoResetEvent # 2中释放。
1号线程 结束。
按Enter键以释放线程。

2号线程 从AutoResetEvent # 2中释放。
2号线程 结束。
按Enter键以释放线程。

3号线程 从AutoResetEvent # 2中释放。
3号线程 结束。
请按任意键继续. . .
```

### ManualResetEvent 类

通知一个或多个正在等待的线程已发生事件。 

&#43; ManualResetEvent：用一个指示是否将初始状态设置为终止的布尔值初始化，初始状态是否有信号
&#43; set:将事件状态设置为终止状态，允许一个或多个等待线程继续。,**设置有信号,设为有信号后就一直有信号，即使waitone后也信号，直到reset将其设为无信号**
&#43; WaitOne()：阻止当前线程，直到当前 WaitHandle 收到信号。
&#43; WaitOne(Int32)：阻止当前线程，直到当前 WaitHandle 收到信号，同时使用 32 位带符号整数指定时间间隔（以毫秒为单位）。
&#43; Reset：将事件状态设置为非终止状态，导致线程阻止。

```C#
using System;
using System.Threading;
public class Example
{
    private static ManualResetEvent mre = new ManualResetEvent(false);//初始状态没有信号
    static void Main()
    {
        Console.WriteLine(&#34;\n启动3个在ManualResetEvent上阻塞的命名线程：\n&#34;);
        for (int i = 0; i &lt;= 2; i&#43;&#43;)
        {
            Thread t = new Thread(ThreadProc);
            t.Name = i&#43;&#34;号线程&#34;;
            Console.WriteLine(&#34;{0}开始&#34;, t.Name);
            t.Start();
        }
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n当所有三个线程都已启动时，按Enter键调用Set（）&#34; &#43;
                          &#34;\n释放所有线程。\n&#34;);
        Console.ReadLine();
        mre.Set();
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n发出ManualResetEvent信号时，调用WaitOne（）的线程&#34; &#43;
                          &#34;\n不要阻止。 按Enter键显示此信息。\n&#34;);
        Console.ReadLine();
        for (int i = 3; i &lt;= 4; i&#43;&#43;)
        {
            Thread t = new Thread(ThreadProc);
            t.Name = i &#43; &#34;号线程&#34;;
            t.Start();
        }
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n按Enter键调用Reset（），以便线程再次阻止&#34; &#43;
                          &#34;\n当他们调用WaitOne（）时.\n&#34;);
        Console.ReadLine();
        mre.Reset();
        // Start a thread that waits on the ManualResetEvent.
        Thread t5 = new Thread(ThreadProc);
        t5.Name = &#34;Thread_5&#34;;
        t5.Start();
        Thread.Sleep(500);
        Console.WriteLine(&#34;\nPress Enter to call Set() and conclude the demo.&#34;);
        Console.ReadLine();
        mre.Set();
    }
    private static void ThreadProc()
    {
        string name = Thread.CurrentThread.Name;
        Console.WriteLine(name &#43; &#34; 启动并调用mre.WaitOne（）&#34;);
        mre.WaitOne();
        Console.WriteLine(name &#43; &#34; ends.&#34;);
    }
}
结果：

启动3个在ManualResetEvent上阻塞的命名线程：

0号线程开始
1号线程开始
0号线程 启动并调用mre.WaitOne（）
2号线程开始
1号线程 启动并调用mre.WaitOne（）
2号线程 启动并调用mre.WaitOne（）

当所有三个线程都已启动时，按Enter键调用Set（）
释放所有线程。


2号线程 ends.
0号线程 ends.
1号线程 ends.

发出ManualResetEvent信号时，调用WaitOne（）的线程
不要阻止。 按Enter键显示此信息。


3号线程 启动并调用mre.WaitOne（）
3号线程 ends.
4号线程 启动并调用mre.WaitOne（）
4号线程 ends.

按Enter键调用Reset（），以便线程再次阻止
当他们调用WaitOne（）时.


Thread_5 启动并调用mre.WaitOne（）

Press Enter to call Set() and conclude the demo.

Thread_5 ends.
请按任意键继续. . .
```

### Interlocked 类

为多个线程共享的变量提供原子操作。

&#43; Increment(Int32)：以原子操作的形式递增指定变量的值并存储结果。
&#43; Add(Int32, Int32)：对两个 32 位整数进行求和并用和替换第一个整数，上述操作作为一个原子操作完成。
&#43; CompareExchange(Double, Double, Double)：比较两个双精度浮点数是否相等，如果相等，则替换第一个值。







### waithandle

&#43; WaitHandle类是一个抽象类，有多个类直接或者间接继承自WaitHandle类
&#43; 在WaitHandle类中SignalAndWait、WaitAll、WaitAny及WaitOne这几个方法都有重载形式，其中除WaitOne之外都是静态的。
&#43; WaitHandle方法常用作同步对象的基类。WaitHandle对象通知其他的线程它需要对资源排他性的访问，其他的线程必须等待，直到WaitHandle不再使用资源和等待句柄没有被使用。
&#43; WaitHandle方法有多个Wait的方法，这些方法的区别如下：
  &#43; WaitAll：等待指定数组中的所有元素收到信号。
  &#43; WaitAny：等待指定数组中的任一元素收到信号。
  &#43; WaitOne：当在派生类中重写时，阻塞当前线程，直到当前的 WaitHandle 收到信号。

MSCN上面的代码

```C#
using System;
using System.Threading;
namespace AutoResetEvent_Examples
{
class MyMainClass
{
    //Initially not signaled.
    const int numIterations = 5;
    static AutoResetEvent myResetEvent = new AutoResetEvent(false);
    static int number;
    static void Main()
    {
        //Create and start the reader thread.
        Thread myReaderThread = new Thread(new ThreadStart(MyReadThreadProc));
        myReaderThread.Name = &#34;ReaderThread&#34;;
        myReaderThread.Start();
        for (int i = 1; i &lt;= numIterations; i&#43;&#43;)
        {
            Console.WriteLine(&#34;Writer thread writing value: {0}&#34;, i);
            number = i;
            //Signal that a value has been written.
            myResetEvent.Set();
            //Give the Reader thread an opportunity to act.
            Thread.Sleep(1);
        }
        //Terminate the reader thread.
        myReaderThread.Abort();
    }
    static void MyReadThreadProc()
    {
        while (true)
        {
            //The value will not be read until the writer has written
            // at least once since the last read.
            myResetEvent.WaitOne();//等待set
            Console.WriteLine(&#34;{0} reading value: {1}&#34;, Thread.CurrentThread.Name, number);
        }
    }
}
}
 结果
Writer thread writing value: 1
ReaderThread reading value: 1
Writer thread writing value: 2
ReaderThread reading value: 2
Writer thread writing value: 3
ReaderThread reading value: 3
Writer thread writing value: 4
ReaderThread reading value: 4
Writer thread writing value: 5
ReaderThread reading value: 5
请按任意键继续. . .
```

```C#
using System;
using System.Threading;
public class Example
{
    // mre is used to block and release threads manually. It is
    // created in the unsignaled state.
    private static ManualResetEvent mre = new ManualResetEvent(false);//设为非终止状态，线程会被阻塞
    static void Main()
    {
        Console.WriteLine(&#34;\n启动在MalualReSeTebug上阻止的3个命名线程:\n&#34;);
        for (int i = 0; i &lt;= 2; i&#43;&#43;)
        {
            Thread t = new Thread(ThreadProc);
            t.Name =  i&#43;&#34;线程&#34;;
            t.Start();
        }
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n当所有三个线程都已启动时，按Enter调用SET（）&#34; &#43;
                          &#34;\n释放所有线程。\n&#34;);
        Console.ReadLine();
        mre.Set();//事件状态被设置为有信号，允许线程执行，3个线程执行，同时ManualResetEvent 被设置为终止状态，线程不会阻塞
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n当一个MavaReSeTeEvices被发出信号时，调用WAOTIONE（）的线程&#34; &#43;
                          &#34;\n不要阻塞。按Enter显示这一点。\n&#34;);
        Console.ReadLine();
        for (int i = 3; i &lt;= 4; i&#43;&#43;)
        {
            Thread t = new Thread(ThreadProc);
            t.Name = i &#43; &#34;线程&#34;;
            t.Start();
        }
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n按Enter调用REST（），使线程再次阻塞&#34; &#43;
                          &#34;\n当他们调用WAOTIFEL（）时。\n&#34;);
        Console.ReadLine();
        mre.Reset();//将ManualResetEvent 重新设为非终止状态
        // Start a thread that waits on the ManualResetEvent.
        Thread t5 = new Thread(ThreadProc);
        t5.Name = &#34;5线程&#34;;
        t5.Start();
        Thread.Sleep(500);
        Console.WriteLine(&#34;\n按Enter调用SET（）并结束演示。&#34;);
        Console.ReadLine();
        mre.Set();
        // If you run this example in Visual Studio, uncomment the following line:
        //Console.ReadLine();
    }
    private static void ThreadProc()
    {
        string name = Thread.CurrentThread.Name;

        Console.WriteLine(name &#43; &#34; starts and calls mre.WaitOne()&#34;);

        mre.WaitOne();//当ManualResetEvent 设为非终止状态时，线程会被阻塞

        Console.WriteLine(name &#43; &#34; ends.&#34;);
    }
}
启动在MalualReSeTebug上阻止的3个命名线程:
0线程 starts and calls mre.WaitOne()
2线程 starts and calls mre.WaitOne()
1线程 starts and calls mre.WaitOne()
当所有三个线程都已启动时，按Enter调用SET（）
释放所有线程。
1线程 ends.
0线程 ends.
2线程 ends.
当一个MavaReSeTeEvices被发出信号时，调用WAOTIONE（）的线程
不要阻塞。按Enter显示这一点。
3线程 starts and calls mre.WaitOne()
3线程 ends.
4线程 starts and calls mre.WaitOne()
4线程 ends.
按Enter调用REST（），使线程再次阻塞
当他们调用WAOTIFEL（）时。
5线程 starts and calls mre.WaitOne()
按Enter调用SET（）并结束演示。
```


&#43; 这个讲的是一个计算过程，最终的计算结果为第一项＋第二项＋第三项，在计算第一、二、三项时需要使用基数来进行计算。在代码中使用了线程池也就是ThreadPool来操作 

```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Threading;  
namespace StartThread  
{  
    //下面的代码摘自MSDN，笔者做了中文代码注释  
    //周公  
    public class EventWaitHandleDemo  
    {  
        double baseNumber, firstTerm, secondTerm, thirdTerm;  
        AutoResetEvent[] autoEvents;  
        ManualResetEvent manualEvent;  
        //产生随机数的类.  
        Random random;  
        static void Main()  
        {  
            EventWaitHandleDemo ewhd = new EventWaitHandleDemo();  
            Console.WriteLine(&#34;Result = {0}.&#34;,  
                ewhd.Result(234).ToString());  
            Console.WriteLine(&#34;Result = {0}.&#34;,  
                ewhd.Result(55).ToString());  
            Console.ReadLine();  
        }  
        //构造函数  
        public EventWaitHandleDemo()  
        {  
            autoEvents = new AutoResetEvent[]  
            {  
                new AutoResetEvent(false),  
                new AutoResetEvent(false),  
                new AutoResetEvent(false)  
            };  
            manualEvent = new ManualResetEvent(false);  
        }  
        //计算基数  
        void CalculateBase(object stateInfo)  
        {  
            baseNumber = random.NextDouble();  
            //指示基数已经算好.  
            manualEvent.Set();  
        }  
  
        //计算第一项  
        void CalculateFirstTerm(object stateInfo)  
        {  
            //生成随机数  
            double preCalc = random.NextDouble();  
            //等待基数以便计算.  
            manualEvent.WaitOne();  
            //通过preCalc和baseNumber计算第一项.  
            firstTerm = preCalc * baseNumber *random.NextDouble();  
            //发出信号指示计算完成.  
            autoEvents[0].Set();  
        }  
        //计算第二项  
        void CalculateSecondTerm(object stateInfo)  
        {  
            double preCalc = random.NextDouble();  
            manualEvent.WaitOne();  
            secondTerm = preCalc * baseNumber *random.NextDouble();  
            autoEvents[1].Set();  
        }  
        //计算第三项  
        void CalculateThirdTerm(object stateInfo)  
        {  
            double preCalc = random.NextDouble();  
            manualEvent.WaitOne();  
            thirdTerm = preCalc * baseNumber *random.NextDouble();  
            autoEvents[2].Set();  
        }  
        //计算结果  
        public double Result(int seed)  
        {  
            random = new Random(seed);  
            //同时计算  
            ThreadPool.QueueUserWorkItem(new WaitCallback(CalculateFirstTerm));  
            ThreadPool.QueueUserWorkItem(new WaitCallback(CalculateSecondTerm));  
            ThreadPool.QueueUserWorkItem(new WaitCallback(CalculateThirdTerm));  
            ThreadPool.QueueUserWorkItem(new WaitCallback(CalculateBase));  
            //等待所有的信号.  
            WaitHandle.WaitAll(autoEvents);  
            //重置信号，以便等待下一次计算.  
            manualEvent.Reset();  
            //返回计算结果  
            return firstTerm &#43; secondTerm &#43; thirdTerm;  
        }  
    }  
}  
程序的运行结果如下：
Result = 0.355650523270459.
Result = 0.125205692112756.
```

### 线程池

为了处理短时间内大量创建对象，简单处理下，又要销毁损耗性能的行为，线程过多后会增加操作系统资源的占用，多线程资源竞争变得复杂
线程池的优点：

&#43; 缩短程序的响应时间
&#43; 不必维护管理生存周期短暂的问题
&#43; 线程池会根据当前系统特点对池内的线程进行优化处理

在.NET中有一个线程的类ThreadPool，它提供了线程池的管理

ThreadPool是一个静态类，它没有构造函数，对外提供的函数也全部是静态的。其中有一个QueueUserWorkItem方法，它有两种重载形式，如下：

&#43; `public static bool QueueUserWorkItem(WaitCallback callBack)`:将方法排入队列以便执行。此方法在有线程池线程变得可用时执行。
&#43; `public static bool QueueUserWorkItem(WaitCallback callBack,Object state)`:将方法排入队列以便执行，并指定包含该方法所用数据的对象。此方法在有线程池线程变得可用时执行。

QueueUserWorkItem方法中使用的的WaitCallback参数表示一个delegate，它的声明如下：

&#43; public delegate void WaitCallback(Object state)
&#43; 如果需要传递任务信息可以利用WaitCallback中的state参数，类似于ParameterizedThreadStart委托

```C#
using System.Threading;  
using System.Collections;  
using System.Diagnostics;  
using System;  
using System.ComponentModel;  
namespace ThreadPoolDemo  
{  
    class ThreadPoolDemo1  
    {  
        public ThreadPoolDemo1()  
        {  
        }  
        public void Work()  
        {  
            ThreadPool.QueueUserWorkItem(new WaitCallback(CountProcess));  
            ThreadPool.QueueUserWorkItem(new WaitCallback(GetEnvironmentVariables));  
        }  
        /// &lt;summary&gt;  
        /// 统计当前正在运行的系统进程信息  
        /// &lt;/summary&gt;  
        /// &lt;param name=&#34;state&#34;&gt;&lt;/param&gt;  
        private void CountProcess(object state)  
        {  
            Process[] processes = Process.GetProcesses();  
            foreach (Process p in processes)  
            {  
                try  
                {  
                    Console.WriteLine(&#34;Id:{0},ProcessName:{1},StartTime:{2}&#34;, p.Id, p.ProcessName, p.StartTime);  
                }  
                catch (Win32Exception e)  
                {  
                    Console.WriteLine(&#34;ProcessName:{0}&#34;, p.ProcessName);  
                }  
                finally  
                {  
                }  
            }  
            Console.WriteLine(&#34;获取进程信息完毕。&#34;);  
        }  
        /// &lt;summary&gt;  
        /// 获取当前机器系统变量设置  
        /// &lt;/summary&gt;  
        /// &lt;param name=&#34;state&#34;&gt;&lt;/param&gt;  
        public void GetEnvironmentVariables(object state)  
        {  
            IDictionary list=System.Environment.GetEnvironmentVariables();  
            foreach (DictionaryEntry item in list)  
            {  
                Console.WriteLine(&#34;key={0},value={1}&#34;, item.Key, item.Value);  
            }  
            Console.WriteLine(&#34;获取系统变量信息完毕。&#34;);  
        }  
    static void Main(string[] args)  
        {  
            ThreadPoolDemo1 tpd1 = new ThreadPoolDemo1();  
            tpd1.Work();  
            Thread.Sleep(5000);  
            Console.WriteLine(&#34;OK&#34;);  
            Console.ReadLine();  
        }  
    }  
}  
```

在上面的代码中我们使用了线程池，并让它执行了两个任务，一个是列出系统当前所有环境变量的值，一个是列出系统当前运行的进程名和它们的启动时间。

当然，优点和缺点总是同时存在的，使用ThreadPool也有一些缺点，使用线程池有如下缺点：

&#43; 1、一旦加入到线程池中就没有办法让它停止，除非任务执行完毕自动停止；

&#43; 2、一个进程共享一个线程池；

&#43; 3、要执行的任务不能有返回值（当然，线程中要执行的方法也是不能有返回值，如果确实需要返回值必须采用其它技巧来解决）；

&#43; 4、在线程池中所有任务的优先级都是一样的，无法设置任务的优先级；

&#43; 5、不太适合需要长期执行的任务（比如在Windows服务中执行），也不适合大的任务；

&#43; 6、不能为线程设置稳定的关联标识，比如为线程池中执行某个特定任务的线程指定名称或者其它属性。

如果我们要面临的情况正好是线程池的缺点，那么我们只好继续使用线程而不是线程池。不过在某些情况下使用线程池确实可以带来很多方便的，比如在WEB服务器中，可以使用线程池来处理来自客户端的请求，可以以比较高的性能运行。


## UI界面卡顿的问题

在开发Windows应用程序时经常会使用到线程。对于耗时的操作如果不使用线程将会是UI界面长时间处于停滞状态，这种情况是用户非常不愿意看到的，在这种情况下我们希望使用线程来解决这个问题。

```C#
using System;  
using System.Collections.Generic;  
using System.ComponentModel;  
using System.Data;  
using System.Drawing;  
using System.Text;  
using System.Windows.Forms;  
using System.Threading;  
namespace ThreadPoolDemo  
{  
    public partial class ThreadForm : Form  
    {  
        public ThreadForm()  
        {  
            InitializeComponent();  
        }  
        private void btnThread_Click(object sender, EventArgs e)  
        {  
            Thread thread = new Thread(new ThreadStart(Run));  
            thread.Start();  
        }  
        private void Run()  
        {  
            while (progressBar.Value &lt; progressBar.Maximum)  
            {  
                progressBar.PerformStep();  
            }  
        }  
    }  
}  
```

本意是点击“启动”按钮来启动模拟一个操作，在进度条中显示操作的总体进度。不过如果我们真的点击“启动”按钮会很失望，因为它会抛出一个`System.InvalidOperationException`异常，异常描述就是“线程间操作无效: **从不是创建控件‘progressBar’的线程访问它**。”

解决方案

&#43; CheckForIllegalCrossThreadCalls属性

因为在.NET中做了限制，不允许在调试环境下使用线程访问并非它自己创建的UI控件，这么做可能是怕在多线程环境下对界面控件进行操作会出现不可预知的情况，

```C#
using System;  
using System.Collections.Generic;  
using System.ComponentModel;  
using System.Data;  
using System.Drawing;  
using System.Linq;  
using System.Text;  
using System.Windows.Forms;  
using System.Threading;   
namespace ThreadPoolDemo  
{  
    public partial class ThreadForm : Form  
    {  
        public ThreadForm()  
        {  
            InitializeComponent();  
        }  
        private void btnThread_Click(object sender, EventArgs e)  
        {  
            //指示是否对错误线程的调用，即是否允许在创建UI的线程之外访问线程  
            CheckForIllegalCrossThreadCalls = false;  
            Thread thread = new Thread(new ThreadStart(Run));  
            thread.Start();  
        }  
        private void Run()  
        {  
            while (progressBar.Value &lt; progressBar.Maximum)  
            {  
                progressBar.PerformStep();  
            }  
        }  
    }  
} 
```

不过使用上面的代码我们可能还有些犯嘀咕，毕竟是不允许直接在线程中直接操作界面的，那么我们还可以用Invoke方法。

```C#
using System;  
using System.Collections.Generic;  
using System.ComponentModel;  
using System.Data;  
using System.Drawing;  
using System.Linq;  
using System.Text;  
using System.Windows.Forms;  
using System.Threading;   
namespace ThreadPoolDemo  
{  
    public partial class ThreadForm : Form  
    {  
        //定义delegate以便Invoke时使用  
        private delegate void SetProgressBarValue(int value);  
        public ThreadForm()  
        {  
            InitializeComponent();  
        }  
        private void btnThread_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            //指示是否对错误线程的调用，即是否允许在创建UI的线程之外访问线程  
            //CheckForIllegalCrossThreadCalls = false;  
            Thread thread = new Thread(new ThreadStart(Run));  
            thread.Start();  
        }  
        //使用线程来直接设置进度条  
        private void Run()  
        {  
            while (progressBar.Value &lt; progressBar.Maximum)  
            {  
                progressBar.PerformStep();  
            }  
        }  
        private void btnInvoke_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            Thread thread = new Thread(new ThreadStart(RunWithInvoke));  
            thread.Start();  
        }  
        //使用Invoke方法来设置进度条  
        private void RunWithInvoke()  
        {  
            int value = progressBar.Value;  
            while (value&lt; progressBar.Maximum)  
            {  
                //如果是跨线程调用  
                if (InvokeRequired)  
                {  
                    this.Invoke(new SetProgressBarValue(SetProgressValue), value&#43;&#43;);  
                }  
                else  
                {  
                    progressBar.Value = &#43;&#43;value;  
                }  
            }  
        }  
        //跟SetProgressBarValue委托相匹配的方法  
        private void SetProgressValue(int value)  
        {  
            progressBar.Value = value;  
        }  
    }  
}  
```

还可以使用BackgroundWorker类来完成同样的功能。

```C#
using System;  
using System.Collections.Generic;  
using System.ComponentModel;  
using System.Data;  
using System.Drawing;  
using System.Linq;  
using System.Text;  
using System.Windows.Forms;  
using System.Threading;  
namespace ThreadPoolDemo  
{  
    public partial class ThreadForm : Form  
    {  
        //定义delegate以便Invoke时使用  
        private delegate void SetProgressBarValue(int value);  
        private BackgroundWorker worker;  
        public ThreadForm()  
        {  
            InitializeComponent();  
        }  
        private void btnThread_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            //指示是否对错误线程的调用，即是否允许在创建UI的线程之外访问线程  
            //CheckForIllegalCrossThreadCalls = false;  
            Thread thread = new Thread(new ThreadStart(Run));  
            thread.Start();  
        }  
        //使用线程来直接设置进度条  
        private void Run()  
        {  
            while (progressBar.Value &lt; progressBar.Maximum)  
            {  
                progressBar.PerformStep();  
            }  
        }  
        private void btnInvoke_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            Thread thread = new Thread(new ThreadStart(RunWithInvoke));  
            thread.Start();  
        }  
        //使用Invoke方法来设置进度条  
        private void RunWithInvoke()  
        {  
            int value = progressBar.Value;  
            while (value&lt; progressBar.Maximum)  
            {  
                //如果是跨线程调用  
                if (InvokeRequired)  
                {  
                    this.Invoke(new SetProgressBarValue(SetProgressValue), value&#43;&#43;);  
                }  
                else  
                {  
                    progressBar.Value = &#43;&#43;value;  
                }  
            }  
        }  
        //跟SetProgressBarValue委托相匹配的方法  
        private void SetProgressValue(int value)  
        {  
            progressBar.Value = value;  
        }  
        private void btnBackgroundWorker_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            worker = new BackgroundWorker();  
            worker.DoWork &#43;= new DoWorkEventHandler(worker_DoWork);  
            //当工作进度发生变化时执行的事件处理方法  
            worker.ProgressChanged &#43;= new ProgressChangedEventHandler(worker_ProgressChanged);  
            //当事件处理完毕后执行的方法  
            worker.RunWorkerCompleted &#43;= new RunWorkerCompletedEventHandler(worker_RunWorkerCompleted);  
            worker.WorkerReportsProgress = true;//支持报告进度更新  
            worker.WorkerSupportsCancellation = false;//不支持异步取消  
            worker.RunWorkerAsync();//启动执行  
            btnBackgroundWorker.Enabled = false;  
        }  
        //当事件处理完毕后执行的方法  
        void worker_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)  
        {  
            btnBackgroundWorker.Enabled=true;  
        }  
        //当工作进度发生变化时执行的事件处理方法  
        void worker_ProgressChanged(object sender, ProgressChangedEventArgs e)  
        {  
            //可以在这个方法中与界面进行通讯  
            progressBar.Value = e.ProgressPercentage;  
        }  
        //开始启动工作时执行的事件处理方法  
        void worker_DoWork(object sender, DoWorkEventArgs e)  
        {  
            int value = progressBar.Value;  
            while (value &lt; progressBar.Maximum)  
            {  
                worker.ReportProgress(&#43;&#43;value);//汇报进度  
            }  
        }  
    }  
}  
```

System.Windows.Forms.Timer类也能完场上面的功能

```C#
using System;  
using System.Collections.Generic;  
using System.ComponentModel;  
using System.Data;  
using System.Drawing;  
using System.Linq;  
using System.Text;  
using System.Windows.Forms;  
using System.Threading;  
namespace ThreadPoolDemo  
{  
    public partial class ThreadForm : Form  
    {  
        //定义delegate以便Invoke时使用  
        private delegate void SetProgressBarValue(int value);  
        private BackgroundWorker worker;  
        public ThreadForm()  
        {  
            InitializeComponent();  
        }  
        private void btnThread_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            //指示是否对错误线程的调用，即是否允许在创建UI的线程之外访问线程  
            //CheckForIllegalCrossThreadCalls = false;  
            Thread thread = new Thread(new ThreadStart(Run));  
            thread.Start();  
        }  
        //使用线程来直接设置进度条  
        private void Run()  
        {  
            while (progressBar.Value &lt; progressBar.Maximum)  
            {  
                progressBar.PerformStep();  
            }  
        }  
        private void btnInvoke_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            Thread thread = new Thread(new ThreadStart(RunWithInvoke));  
            thread.Start();  
        }  
        //使用Invoke方法来设置进度条  
        private void RunWithInvoke()  
        {  
            int value = progressBar.Value;  
            while (value&lt; progressBar.Maximum)  
            {  
                //如果是跨线程调用  
                if (InvokeRequired)  
                {  
                    this.Invoke(new SetProgressBarValue(SetProgressValue), value&#43;&#43;);  
                }  
                else  
                {  
                    progressBar.Value = &#43;&#43;value;  
                }  
            }  
        }  
        //跟SetProgressBarValue委托相匹配的方法  
        private void SetProgressValue(int value)  
        {  
            progressBar.Value = value;  
        }  
        private void btnBackgroundWorker_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            worker = new BackgroundWorker();  
            worker.DoWork &#43;= new DoWorkEventHandler(worker_DoWork);  
            //当工作进度发生变化时执行的事件处理方法  
            worker.ProgressChanged &#43;= new ProgressChangedEventHandler(worker_ProgressChanged);  
            //当事件处理完毕后执行的方法  
            worker.RunWorkerCompleted &#43;= new RunWorkerCompletedEventHandler(worker_RunWorkerCompleted);  
            worker.WorkerReportsProgress = true;//支持报告进度更新  
            worker.WorkerSupportsCancellation = false;//不支持异步取消  
            worker.RunWorkerAsync();//启动执行  
            btnBackgroundWorker.Enabled = false;  
        }  
        //当事件处理完毕后执行的方法  
        void worker_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)  
        {  
            btnBackgroundWorker.Enabled=true;  
        }  
        //当工作进度发生变化时执行的事件处理方法  
        void worker_ProgressChanged(object sender, ProgressChangedEventArgs e)  
        {  
            //可以在这个方法中与界面进行通讯  
            progressBar.Value = e.ProgressPercentage;  
        }  
        //开始启动工作时执行的事件处理方法  
        void worker_DoWork(object sender, DoWorkEventArgs e)  
        {  
            int value = progressBar.Value;  
            while (value &lt; progressBar.Maximum)  
            {  
                worker.ReportProgress(&#43;&#43;value);//汇报进度  
            }  
        }  
        //使用System.Windows.Forms.Timer来操作界面能  
        private void btnTimer_Click(object sender, EventArgs e)  
        {  
            progressBar.Value = 0;  
            //注意在.net中有多个命名空间下存在Timer类，为了便于区别，使用了带命名空间形式  
            System.Windows.Forms.Timer timer = new System.Windows.Forms.Timer();  
            timer.Interval = 1;  
            timer.Tick &#43;= new EventHandler(timer_Tick);  
            timer.Enabled = true;  
        }  
        //Timer中要定期执行的方法  
        void timer_Tick(object sender, EventArgs e)  
        {  
            int value = progressBar.Value;  
            if (value &lt; progressBar.Maximum)  
            {  
                progressBar.Value = value&#43;100;  
            }  
        }  
    }  
} 
```


## Task类

Task类是封装的一个任务类，内部使用的是ThreadPool类，提供了内建机制，让你知道什么时候异步完成以及如何获取异步执行的结果，并且还能取消异步执行的任务。下面看一个例子是如何使用Task类来执行异步操作的。

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        Task t = new Task((c) =&gt;
        {
            int count = (int)c;
            for (int i = 0; i &lt; count; i&#43;&#43;)
            {
                Thread.Sleep(10);
                Console.WriteLine($&#34;Asyn Thread:{i}&#34;);
            }
            Console.WriteLine(&#34;Asyn Thread Done&#34;);
        }, 10);//10为传递的参数C      N0.1
        t.Start();
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread:{i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread done&#34;);
        Console.ReadKey();
    }
}
}
Main Thread:0
Asyn Thread:0
Asyn Thread:1
Main Thread:1
Asyn Thread:2
Main Thread:2
Asyn Thread:3
Main Thread:3
Asyn Thread:4
Main Thread:4
Main Thread done
Asyn Thread:5
Asyn Thread:6
Asyn Thread:7
Asyn Thread:8
Asyn Thread:9
Asyn Thread Done
```

no.1处使用Task的构造函数为：

`public Task( Action&lt;Object&gt; action, Object state )一个Action&lt;Object&gt;`类型的委托（即异步调用函数具有一个Object类型的参数），和一个Object类型的参数，也就是传递给异步函数的参数，

Task类还有几种方式的重载，我们还可以传递一些TaskCreationOptions标志来控制Task的执行方式。在这里我使用的是lambda表达去写委托的，这样使得程序的结构更加的清晰，使用Start()来启动异步函数的调用。

&#43; 有返回值，主线程阻塞

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        Task&lt;int&gt; t = new Task&lt;int&gt;((c) =&gt;
        {
            int count = (int)c;
            int sum = 0;
            for (int i = 0; i &lt; count; i&#43;&#43;)
            {
                Thread.Sleep(10);
                sum &#43;= i;
                Console.WriteLine($&#34;Asyn Thread:{i}&#34;);
            }
            Console.WriteLine(&#34;Asyn Thread Done&#34;);
            return sum;
        }, 10);//no.1
        t.Start();
        t.Wait();
        Console.WriteLine($&#34;Asyn Result:{t.Result}&#34;);
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread:{i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread done&#34;);
        Console.ReadKey();
    }
}
}
Asyn Thread:0
Asyn Thread:1
Asyn Thread:2
Asyn Thread:3
Asyn Thread:4
Asyn Thread:5
Asyn Thread:6
Asyn Thread:7
Asyn Thread:8
Asyn Thread:9
Asyn Thread Done
Asyn Result:45
Main Thread:0
Main Thread:1
Main Thread:2
Main Thread:3
Main Thread:4
Main Thread done
```

如果任务中出现了异常，那么异常会被吞噬掉，并存储到一个集合中去，而线程可以返回到线程池中去。但是如果在代码中调用了Wait方法或者是Result属性，任务有异常发生就会被引发，不会被吞噬掉。其中Result属性内部本身也调用了Wati方法。Wait方法和上一节中的委托的EndInvoke方法类似，会使得调用线程阻塞直到异步任务完成。


&#43; 取消正在运行的任务
  &#43; 取消任务要引用一个CancellationTokenSource 对象。在需要异步执行的方法中增加一个CancellationToken类型的形参。然后在异步函数的for循环代码中用一个if语句判断CancellationToken的CanBeCanceled属性，这个属性可以用来判断在调用线程是否取消任务的执行，
  &#43; 除CanBeCanceled属性之外，还可以使用ThrowIfCancellationRequested方法，该方法的作用是如果在调用线程调用CancellationTokenSource对象的Cancel方法，那么就会引发一个异常，然后在调用线程进行捕捉就好了，这是在异步函数中的处理方式。
  &#43; no.1在构建任务之前需要建立一个CancellationTokenSource ，
  &#43; no2.并且把CancellationTokenSource传递给异步调用函数，传递的是CancellationTokenSource对象的Toke属性，该属性是一个CancellationToken类型的对象。这样就完成任务的取消模式，如果想在调用线程中取消任务的执行，只需要调用CancellationTokenSource 的Cancel方法就行啦。

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        CancellationTokenSource cts = new CancellationTokenSource();//NO.1
        Task&lt;int&gt; t = new Task&lt;int&gt;((c) =&gt; Sum(cts.Token, (int)c), 10);//NO.2
        t.Start();
        //cts.Cancel();//NO.3如果任务没有完成，但是Task有可能完成了
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread:{i}&#34;);
        }
        cts.Cancel();
        Console.WriteLine(&#34;Main Thread done&#34;);
        Console.ReadKey();
    }
    static int Sum(CancellationToken ct, int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; count; i&#43;&#43;)
        {
            //if (!ct.CanBeCanceled)
            if (!ct.IsCancellationRequested)
            {
                Thread.Sleep(10);
                sum &#43;= i;
                Console.WriteLine($&#34;Asyn Thread:{i}&#34;);
            }
            else
            {
                Console.WriteLine(&#34;任务取消&#34;);
                //return -1;
            }
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
}
}
 结果
Main Thread:0
Asyn Thread:0
Main Thread:1
Asyn Thread:1
Main Thread:2
Asyn Thread:2
Main Thread:3
Asyn Thread:3
Main Thread:4
Main Thread done
Asyn Thread:4
任务取消
任务取消
任务取消
任务取消
任务取消
Asyn Thread Done
```

&#43; 主线程不会阻塞
  &#43; 前面就说过了，获取任务结果调用Wait方法和Result属性导致调用线程阻塞，那么如何处理这种情况呢，这就使用了`Task&lt;TResult&gt;`类提供的ContinueWith方法。该方法的作用是当任务完成时，启动一个新的任务，不仅仅是如此，该方法还有可以在任务只出现异常或者取消等情况的时候才执行，只需要给该方法传递TaskContinuationOptions枚举类型就可以了。下面就演示一下如何使用ContinueWith方法。
  &#43; 首先看下ContinueWith方法的原型。
  &#43; `public Task ContinueWith( Action&lt;Task&gt; continuationAction )`采用一个Action&lt;Task&gt;类型的委托。该方法提供了多种重载的版本，这只是最简单的一种。
  &#43; `public Task ContinueWith( Action&lt;Task&gt; continuationAction, TaskContinuationOptions continuationOptions )`第二个参数代表新任务的执行条件，当任务满足这个枚举条件才执行 Action&lt;Task&gt;类型的回调函数。

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
namespace testasy
{
class Program
{
    public delegate int DoWork(int count);
    static void Main(string[] args)
    {
        Task&lt;int&gt; t = new Task&lt;int&gt;((c) =&gt; Sum((int)c), 10);
        t.Start();
        t.ContinueWith(task =&gt; Console.WriteLine(&#34;&#34;), TaskContinuationOptions.OnlyOnFaulted);//当任务出现异常时才执行
        for (int i = 0; i &lt; 5; i&#43;&#43;)
        {
            Thread.Sleep(10);
            Console.WriteLine($&#34;Main Thread:{i}&#34;);
        }
        Console.WriteLine(&#34;Main Thread done&#34;);
        Console.ReadKey();
    }
    static int Sum( int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; count; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
            Console.WriteLine($&#34;Asyn Thread:{i}&#34;);
        }
        Console.WriteLine(&#34;Asyn Thread Done&#34;);
        return sum;
    }
}
}
Main Thread:0
Asyn Thread:0
Main Thread:1
Main Thread:2
Asyn Thread:1
Main Thread:3
Asyn Thread:2
Main Thread:4
Main Thread done
Asyn Thread:3
Asyn Thread:4
Asyn Thread:5
Asyn Thread:6
Asyn Thread:7
Asyn Thread:8
Asyn Thread:9
Asyn Thread Done
```

	&#43; t.Start()之后调用第一个ContinueWith方法，该方法第一参数就是一个`Action&lt;Task&gt;`的委托类型，相当于是一个回调函数，在这里我也用lambda表达式，当任务完成就会启用一个新任务去执行这个回调函数。而第二个ContinueWith里面的回调方法却不会执行，因为我们的任务也就是Sum方法不会发生异常，不能满足TaskContinuationOptions.OnlyOnFaulted这个枚举条件。**这种用法比委托的异步函数编程看起来要简单些。最关键的是ContinueWith的还有一个重载版本可以带一个TaskScheduler对象参数，该对象负责执行被调度的任务**。
	&#43; FCL中提供两种任务调度器，均派生自TaskScheduler类型：线程池调度器，和同步上下文任务调用器。而在Winform窗体程序设计中TaskScheduler尤为有用，为什么这么说呢？因为在窗体程序中的控件都是有ui线程去创建，而我们所执行的后台任务使用线程都是线程池中的工作线程，所以当我们的任务完成之后需要反馈到Winform控件上，但是控件创建的线程和任务执行的线程不是同一个线程，如果在任务线程中去更新控件就会导致控件对象安全问题会出现异常。所以操作控件，**就必须要使用ui线程去操作。**因此在ContinueWith获取任务执行的结果的并反馈到控件的任务调度上不能使用线程池任务调用器，而要使用同步上下文任务调度器去调度，即采用ui这个线程去调用ContinueWith方法所绑定的回调用函数即`Action&lt;Task&gt;`类型的委托。下面将使用任务调度器来把异步执行的Sum计算结果反馈到Winform界面的TextBox控件中。

```c#
public partial class Form1 : Form
{
    private readonly TaskScheduler contextTaskScheduler;//声明一个任务调度器
    public Form1()
    {
        InitializeComponent();
        contextTaskScheduler = TaskScheduler.FromCurrentSynchronizationContext();//no.1获得一个上下文任务调度器
    }
    private void button1_Click(object sender, EventArgs e)
    {
        Task&lt;int&gt; t = new Task&lt;int&gt;((n) =&gt; Sum((int)n),10);
        t.Start();
        t.ContinueWith(task =&gt;this.textBox1 .Text =task.Result.ToString(),contextTaskScheduler);//当任务执行完之后执行
        t.ContinueWith(task=&gt;MessageBox .Show (&#34;任务出现异常&#34;),CancellationToken.None,TaskContinuationOptions.OnlyOnFaulted,contextTaskScheduler ); //当任务出现异常时才执行
    }
    int Sum(int count)
    {
        int sum = 0;
        for (int i = 0; i &lt; count; i&#43;&#43;)
        {
            Thread.Sleep(10);
            sum &#43;= i;
        }
        Console.WriteLine(&#34;任务处理完成&#34;);
        return sum;
    }
}
```

在no.1窗体的构造函数获取该UI线程的同步上下文调度器。在按钮的事件接受异步执行的结果时候，都传递了contextTaskScheduler同步上下文的调度器，目的是，当异步任务完成之后，调度UI线程去执行任务完成之后的回调函数。

### 多线程UI的例子

```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Threading;
namespace testthreadui
{
public partial class Form1 : Form
{
    private readonly TaskScheduler contextTaskScheduler;//声明一个任务调度器
    public Form1()
    {
        InitializeComponent();
        contextTaskScheduler = TaskScheduler.FromCurrentSynchronizationContext();//no.1获得一个上下文任务调度器
    }
    private void button1_Click(object sender, EventArgs e)
    {
        Task&lt;int&gt; t = new Task&lt;int&gt;((n) =&gt; Sum((int)n), 100);
        t.Start();
        t.ContinueWith(task =&gt; this.textBox1.Text = task.Result.ToString(), contextTaskScheduler);//当任务执行完之后执行
        t.ContinueWith(task =&gt; MessageBox.Show(&#34;任务出现异常&#34;), CancellationToken.None, TaskContinuationOptions.OnlyOnFaulted, contextTaskScheduler);//当任务出现异常时才执行
        for (int i = 0; i &lt; 10; i&#43;&#43;)
        {

            textBox1.Text = i.ToString();
            textBox2.Text = i.ToString();
            Thread.Sleep(100);
        }
    }
    int Sum(int count)
    {
        try
        {
            int sum = 0;
            for (int i = 0; i &lt; count; i&#43;&#43;)
            {
                Thread.Sleep(10);
                sum &#43;= i;
                //throw new Exception(&#34;错误&#34;);
            }
            Console.WriteLine(&#34;任务处理完成&#34;);
            return sum;
        }
        catch (Exception e)
        {
            MessageBox.Show(&#34;任务出现异常&#34;);
        }
        return -1;
    }
    private void button2_Click(object sender, EventArgs e)
    {
        textBox1.Text = 100.ToString();
    }
}
} 
//未发现异常
```


&#43; 实现实时更新UI

首先建立一个winform项目，在主窗体上拖入一个button，一个progressbar，一个lable。如下图所示。


![image-20240701151908591](/dotnet/image-20240701151908591.png)

编写一个处理数据的类（WriteDate），源代码如下

```c#
public class DataWrite
{
    public delegate void UpdateUI(int step);//声明一个更新主线程的委托
    public UpdateUI UpdateUIDelegate;
    public delegate void AccomplishTask();//声明一个在完成任务时通知主线程的委托
    public AccomplishTask TaskCallBack;
    public void Write(object lineCount)
    {
        StreamWriter writeIO = new StreamWriter(&#34;text.txt&#34;, false, Encoding.GetEncoding(&#34;gb2312&#34;));
        string head = &#34;编号,省,市&#34;;
        writeIO.Write(head);
        for (int i = 0; i &lt; (int)lineCount; i&#43;&#43;)
        {
            writeIO.WriteLine(i.ToString() &#43; &#34;,湖南,衡阳&#34;);
            //写入一条数据，调用更新主线程ui状态的委托
            UpdateUIDelegate(1);
        }
        //任务完成时通知主线程作出相应的处理
        TaskCallBack();
        writeIO.Close();
    }
} 
```

主界面中的代码如下：

首先要建立一个委托来实现非创建控件的线程更新控件。 

`delegate void AsynUpdateUI(int step); `

然后编写多线程去启动写入数据的方法以及回调的函数。

```c#
private void btnWrite_Click(object sender, EventArgs e)
{
    int taskCount = 10000; //任务量为10000
    this.pgbWrite.Maximum = taskCount;
    this.pgbWrite.Value = 0;

    DataWrite dataWrite = new DataWrite();//实例化一个写入数据的类
    dataWrite.UpdateUIDelegate &#43;= UpdataUIStatus;//绑定更新任务状态的委托
    dataWrite.TaskCallBack &#43;= Accomplish;//绑定完成任务要调用的委托

    Thread thread = new Thread(new ParameterizedThreadStart(dataWrite.Write));
    thread.IsBackground = true;
    thread.Start(taskCount);
}
//更新UI
private void UpdataUIStatus(int step)
{
    if (InvokeRequired)
    {
	        this.Invoke(new AsynUpdateUI(delegate(int s)
        {
            this.pgbWrite.Value &#43;= s;
            this.lblWriteStatus.Text = this.pgbWrite.Value.ToString() &#43; &#34;/&#34; &#43; this.pgbWrite.Maximum.ToString();
        }), step);
    }
    else
    {
        this.pgbWrite.Value &#43;= step;
        this.lblWriteStatus.Text = this.pgbWrite.Value.ToString() &#43; &#34;/&#34; &#43; this.pgbWrite.Maximum.ToString();
    }
}
//完成任务时需要调用
private void Accomplish()
{
    //还可以进行其他的一些完任务完成之后的逻辑处理
    MessageBox.Show(&#34;任务完成&#34;);
} 
```


完整代码

```C#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Threading;
using System.IO;
namespace testthreadui
{
public partial class Form1 : Form
{
    delegate void AsynUpdateUI(int step);
    public Form1()
    {
        InitializeComponent();
    }
    private void button1_Click(object sender, EventArgs e)
    {
        int taskCount = 10000; //任务量为10000
        this.pgbWrite.Maximum = taskCount;
        this.pgbWrite.Value = 0;
        DataWrite dataWrite = new DataWrite();//实例化一个写入数据的类
        dataWrite.UpdateUIDelegate &#43;= UpdataUIStatus;//绑定更新任务状态的委托
        dataWrite.TaskCallBack &#43;= Accomplish;//绑定完成任务要调用的委托
        Thread thread = new Thread(new ParameterizedThreadStart(dataWrite.Write));
        thread.IsBackground = true;
        thread.Start(taskCount);
    }
    private void UpdataUIStatus(int step)
    {
        if (InvokeRequired)
        {
            this.Invoke(new AsynUpdateUI(delegate (int s)
            {
                this.pgbWrite.Value &#43;= s;
                this.lblWriteStatus.Text = this.pgbWrite.Value.ToString() &#43; &#34;/&#34; &#43; this.pgbWrite.Maximum.ToString();
            }), step);
        }
        else
        {
            this.pgbWrite.Value &#43;= step;
            this.lblWriteStatus.Text = this.pgbWrite.Value.ToString() &#43; &#34;/&#34; &#43; this.pgbWrite.Maximum.ToString();
        }
    }
    //完成任务时需要调用
    private void Accomplish()
    {
        //还可以进行其他的一些完任务完成之后的逻辑处理
        MessageBox.Show(&#34;任务完成&#34;);
    }
}
public class DataWrite
{
    public delegate void UpdateUI(int step);//声明一个更新主线程的委托
    public UpdateUI UpdateUIDelegate;
    public delegate void AccomplishTask();//声明一个在完成任务时通知主线程的委托
    public AccomplishTask TaskCallBack;
    public void Write(object lineCount)
    {
        StreamWriter writeIO = new StreamWriter(&#34;text.txt&#34;, false, Encoding.GetEncoding(&#34;gb2312&#34;));
        string head = &#34;编号,省,市&#34;;
        writeIO.Write(head);
        for (int i = 0; i &lt; (int)lineCount; i&#43;&#43;)
        {
            writeIO.WriteLine(i.ToString() &#43; &#34;,湖南,衡阳&#34;);
            //写入一条数据，调用更新主线程ui状态的委托
            UpdateUIDelegate(1);
        }
        //任务完成时通知主线程作出相应的处理
        TaskCallBack();
        writeIO.Close();
    }
}
}
```

































---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/11/dotnetbase4-thread/  

