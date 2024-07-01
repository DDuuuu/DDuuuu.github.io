# DotNet基础 日志组件


## 介绍

近期一直在看开源社区的源码，看各种编程书籍，自己却没有实践，堪称身体力行了王者级输入，青铜级输出。这是一个非常不好的学习习惯，会导致知其然而不知其所以然，所以有一个声音一直在我脑袋里呐喊，你不能这样了，必须要进行实践了，所以我放下了书本，暂停学习新的东西，开始造我的第一个轮子日志组件。

## 0 性能对比

首先看一下性能对比，我选了两个Log4Net和EntLib的Log组件进行遍历和并行100W数据量的对比。

EntLib的日志组件，测试结果直接注释在代码下方，无线程竞争的情况下需要18S，并行线程竞争情况下写入需要14S。

![1588341616549](/dotnet/1588341616549.png)

![1588342002870](/dotnet/1588342002870.png)

Log4Net在无线程竞争的情况下需要9S，线程竞争并行写入的时候需要11S，多线程写入实践反而变长了

![1588342079886](/dotnet/1588342079886.png)

![1588342129138](/dotnet/1588342129138.png)

再来看看我造的轮子LogNet，遍历写入4S 不到，多线程并行写入只用了不到2S，在速度方面比Log4Net快了一倍都不止。但是在内存管理方面显然有很大差距，GC的1代2代3代的回收次数都显著多于其他组件。下面将讲一下，这个组件为什么块，内存为什么消耗这么多。

![1588342511402](/dotnet/1588342511402.png)

![1588342754480](/dotnet/1588342754480.png)

## 1 性能解析

### 1.1 高性能锁

在说锁之前先说说日志的大体实现思路，当有写入请求的时候先将写入的消息放到一个缓存队列中，不管多少的消息过来统一进入队列，后台有一个线程不停从线程队列中取得消息写入文件中。

所以在整个组件中最主要的锁就两个，一个是操作缓存队列的锁，还有一个是操作文件的锁。

操作缓存队列的锁线程只占用很短的时间，因为消息写入队列的数据很快，写入队列之前获取锁，写完立刻释放锁。

文件锁线程占用的时间相对来说长很多。写入文件毕竟的一个昂贵的操作。

对于缓存队列锁，我改造了一个带自旋的同步混合锁，看一下核心实现

![1588343806757](/dotnet/1588343806757.png)

在第一个线程的进入的时候会直接使用基元用户模式，快速获得锁，如果有线程竞争，第二个线程会直接进入自旋，在自旋的过程中如果第一个线程释放了锁，第二个线程可以直接通过基元用户模式获得锁。避免获得基元内核模式锁，从而损坏性能。但是如果线程在自旋的过程中，其他线程一直没有释放锁，那么线程不能一直自旋占用CPU片段，直接进入内核模式锁。这个有点像乐观锁的实现思想，只不过乐观锁是一直自旋，不进入基元内核锁，我觉得在这边使用乐观锁也可以，但毕竟是一种激进的做法。

![1588344300821](/dotnet/1588344300821.png)

释放锁，在线程竞争的情况下只有第一个线程是获得基元用户锁，其他线程都是基元内核模式构造的。

为什么这边使用这个自旋锁，我认为操作缓存队列的时间都比较短，如果短时间自旋一下，线程竞争的时候大多数线程都不用基元内核锁损坏性能。结果也看到了，确实在竞争模式块了很多。同时这边为什么不增加递归锁的功能，我觉得没有必要，在操作队列的过程中不会存在同一个线程同时获得两次锁的情况。假设如果真的出现，将同一个线程第二次获得锁视为另一个线程获得锁也未尝不可。

操作文件，直接使用不带自旋的混合锁就可以了。

![1588344747122](/dotnet/1588344747122.png)

![1588344759403](/dotnet/1588344759403.png)

### 1.2 自动扩容队列

日志组件的缓存队列非常简单，不需要很多复杂的操作，只需要基本的先进先出就可以了,一个简单的扩容队列，这个队列在满的时候会将队列容量扩展一倍，在队列长度是容量的四分之一的时候，会将容量减少一半。内存管理方面的问题，我认为就是这个自动扩容队列导致的，当大量消息写入缓存，又快速弹出大量消息，造成队列头和尾一直向下偏移，同时很多空的小内存片段得以释放。内存空间中充斥着很多小片段内存。后面我打算打造写一个循环缓存队列看看内存的问题是否可以得到改善，同时也会看一下Log4Net是如何实现的。将会进一步改进。

```c#
    internal class ResizingArrayQueue&lt;Item&gt;:IEnumerable&lt;Item&gt;, IResizingArrayQueue&lt;Item&gt;
    {
        private Item[] q;       // queue elements
        private int n;          // number of elements on queue
        private int first;      // index of first element of queue
        private int last;       // index of next available slot

        /// &lt;summary&gt;
        /// Initializes an empty queue.
        /// &lt;/summary&gt;
        public ResizingArrayQueue()
        {
            q = new Item[2];
            n = 0;
            first = 0;
            last = 0;
        }
        /// 判断是否等于默认值
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        private bool IsEqlDefault&lt;T&gt;(T value)
        {
            //引用类型
            if (default(T) == null)
            {
                if (value == null) return true;
            }
            else//值类型
            {
                if (value.Equals(default(T))) return true;
            }
            return false;
        }
        /// &lt;summary&gt;
        /// Is this queue empty?
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public bool IsEmpty()
        {
            return n == 0;
        }

        /// &lt;summary&gt;
        /// Returns the number of items in this queue.
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public int Count =&gt; n;

        /// &lt;summary&gt;
        /// 清除数组中元素
        /// &lt;/summary&gt;
        public void Clear()
        {
            //弹出所有元素
            if (!IsEmpty())
            {
                Dequeue();
            }
        }
        /// &lt;summary&gt;
        /// resize the underlying array
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;capacity&#34;&gt;&lt;/param&gt;
        private void Resize(int capacity)
        {
            if (capacity &lt; n)
                throw new ArgumentException(&#34;capacity is less count&#34;);
            Item[] copy =new Item[capacity];
            for (int i = 0; i &lt; n; i&#43;&#43;)
            {
                copy[i] = q[(first &#43; i) % q.Length];
            }
            q = copy;
            first = 0;
            last = n;
        }
        /// &lt;summary&gt;
        /// Adds the item to this queue.
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;item&#34;&gt;&lt;/param&gt;
        public void Enqueue(Item item)
        {
            // double size of array if necessary and recopy to front of array
            if (n == q.Length) Resize(2 * q.Length);   // double size of array if necessary
            q[last&#43;&#43;] = item;                        // add item
            if (last == q.Length) last = 0;          // wrap-around
            n&#43;&#43;;
        }
        /// &lt;summary&gt;
        /// Removes and returns the item on this queue that was least recently added.
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public Item Dequeue()
        {
            if (IsEmpty()) throw new StackOverflowException(&#34;Queue underflow&#34;);
            Item item = q[first];
            q[first] = default(Item);                            // to avoid loitering
            n--;
            first&#43;&#43;;
            if (first == q.Length) first = 0;           // wrap-around
            // shrink size of array if necessary
            if (n &gt; 0 &amp;&amp; n == q.Length / 4) Resize(q.Length / 2);
            return item;
        }
        /// &lt;summary&gt;
        /// Returns the item least recently added to this queue.
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public Item Peek()
        {
            if (IsEmpty()) throw new StackOverflowException(&#34;Queue underflow&#34;);
            return q[first];
        }

        public override String ToString()
        {
            StringBuilder s = new StringBuilder();
            foreach (var value in this)
            {
                s.Append(value.ToString() &#43; &#34; &#34;);
            }

            return s.ToString();
        }

        public IEnumerator&lt;Item&gt; GetEnumerator()
        {
            return new Enumerator&lt;Item&gt;(q, first, n);
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }
        public struct Enumerator&lt;T&gt; : IEnumerator&lt;T&gt;
        {


            private int i;
            private T[] q;
            private int first;
            private int n;
            public Enumerator(T[] seq,int first,int n)
            {
                i = 0;
                q = seq;
                this.n = n;
                this.first = first;
            }
            public void Dispose()
            {
                
            }

            public bool MoveNext()
            {
                return i &lt; n;
            }

            public void Reset()
            {
                throw new NotImplementedException();
            }

            public T Current
            {
                get
                {
                    if (!MoveNext()) throw new OverflowException(&#34;value is overflow&#34;);
                    T item = q[(i &#43; first) % q.Length];
                    i&#43;&#43;;
                    return item;
                }
            }

            object IEnumerator.Current =&gt; Current;
        }
    }
```

## 2 代码实现

### 消息添加到缓存

![1588345104916](/dotnet/1588345104916.png)

在多线程消息添加换缓存的时候，会将消息添加到队列，然后调用保存到文件方法，但只有一个线程进行文件保存，其他线程直接返回。

### 写入文件

![1588345303919](/dotnet/1588345303919.png)

文件操作就先获得锁，然后不停从队列中获取消息写入文件，当队列中没有消息就释放锁。

## 3 总结

该日志组件功能相对简单，但也基本够用，胜在速度很快，内存管理方面有一些缺陷


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/05/dotnetbase11-log/  

