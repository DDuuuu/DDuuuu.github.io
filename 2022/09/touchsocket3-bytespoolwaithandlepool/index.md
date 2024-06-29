# TouchSocket 字节池和待处理池


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

池是高性能组件中必不可少的东西，池最主要的功能是复用，在软件中创建和销毁对象是有成本的，消耗的资源也特别多。在需要大量使用相同或相似对象的场景下使用池，需要使用对象时去池中取，用完之后再放回到池中，避免创建和销毁对象，从而提高软件的性能。

池在设计时要注意以下几点：

&#43; 池中对象的存储尽量利用高速缓冲区，这样可以更快速访问对象。

&#43; 池中对象可以用原型模式加以改造。

&#43; 池中对象的使用需要注意内存泄漏问题。因为对象使用完并没有销毁。

TouchSocket用到很多的池，字节池 (BytePool)、等待处理池(WaitHandlePool)、文件池(FilePool)、对象池(ObjectPool)，本文介绍字节池和等待处理池

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index

## 字节池BytePool

字节数组的复用是非常常见的做法，在高速IO中一定可以看见它的身影，微软对其也进行了各种封装。

TouchSocket字节池保存所有创建的字节数组，并根据数组的长度将其放在字典中等待复用，相同长度的数组通过队列进行缓存，最终字节此的样子

` private static readonly ConcurrentDictionary&lt;long, BytesQueue&gt; bytesDictionary = new ConcurrentDictionary&lt;long, BytesQueue&gt;();`

&#43; long:表示数组长度

&#43; BytesQueue:表示该长度数组队列。内部就是`private readonly ConcurrentQueue&lt;byte[]&gt; bytesQueue = new ConcurrentQueue&lt;byte[]&gt;();`

字节池并没有直接对外暴露字节数组，而是将其封装成ByteBlock，字节数组的装饰对象：ByteBlock，内部使用字节数组实现功能，并装饰Stream的对象。

&#43; 写入数据可以自动扩容，扩容基数1.5倍。
&#43; m_needDis控制Dispose时字节数组是否返回给池中

```C#
/// &lt;summary&gt;
///  构造函数
/// &lt;/summary&gt;
/// &lt;param name=&#34;byteSize&#34;&gt;默认64K&lt;/param&gt;
/// &lt;param name=&#34;equalSize&#34;&gt;默认false&lt;/param&gt;
public ByteBlock(int byteSize = 1024 * 64, bool equalSize = false)
{
    this.m_needDis = true;
    this.m_buffer = BytePool.GetByteCore(byteSize, equalSize);
    this.m_using = true;
}

/// &lt;summary&gt;
/// 构造函数
/// &lt;/summary&gt;
/// &lt;param name=&#34;bytes&#34;&gt;&lt;/param&gt;
public ByteBlock(byte[] bytes)
{
    this.m_buffer = bytes ?? throw new ArgumentNullException(nameof(bytes));
    this.m_length = bytes.Length;
    this.m_using = true;
}
/// &lt;summary&gt;
/// 扩容增长比，默认为1.5，
/// min：1.5
/// &lt;/summary&gt;
public static float Ratio
{
    get =&gt; m_ratio;
    set
    {
        if (value &lt; 1.5)
        {
            value = 1.5f;
        }
        m_ratio = value;
    }
}
/// &lt;summary&gt;
/// 读取数据，然后递增Pos
/// &lt;/summary&gt;
/// &lt;param name=&#34;buffer&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;offset&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;length&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
/// &lt;exception cref=&#34;ObjectDisposedException&#34;&gt;&lt;/exception&gt;
public override int Read(byte[] buffer, int offset, int length)
{
    if (!this.m_using)
    {
        throw new ObjectDisposedException(this.GetType().FullName);
    }
    int len = this.m_length - this.m_position &gt; length ? length : this.CanReadLen;
    Array.Copy(this.m_buffer, this.m_position, buffer, offset, len);
    this.m_position &#43;= len;
    return len;
}
/// &lt;summary&gt;
/// 写入
/// &lt;/summary&gt;
/// &lt;param name=&#34;buffer&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;offset&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;count&#34;&gt;&lt;/param&gt;
/// &lt;exception cref=&#34;ObjectDisposedException&#34;&gt;&lt;/exception&gt;
public override void Write(byte[] buffer, int offset, int count)
{
    if (count == 0)
    {
        return;
    }
    if (!this.m_using)
    {
        throw new ObjectDisposedException(this.GetType().FullName);
    }
    if (this.m_buffer.Length - this.m_position &lt; count)
    {
        int need = this.m_buffer.Length &#43; count - ((int)(this.m_buffer.Length - this.m_position));
        int lend = this.m_buffer.Length;
        while (need &gt; lend)
        {
            lend = (int)(lend * m_ratio);
        }
        this.SetCapacity(lend, true);
    }
    Array.Copy(buffer, offset, this.m_buffer, this.m_position, count);
    this.m_position &#43;= count;
    this.m_length = Math.Max(this.m_position, this.m_length);
}
/// &lt;summary&gt;
/// 重新设置容量
/// &lt;/summary&gt;
/// &lt;param name=&#34;size&#34;&gt;新尺寸&lt;/param&gt;
/// &lt;param name=&#34;retainedData&#34;&gt;是否保留元数据&lt;/param&gt;
/// &lt;exception cref=&#34;ObjectDisposedException&#34;&gt;&lt;/exception&gt;
public void SetCapacity(int size, bool retainedData = false)
{
    if (!this.m_using)
    {
        throw new ObjectDisposedException(this.GetType().FullName);
    }
    byte[] bytes = new byte[size];

    if (retainedData)
    {
        Array.Copy(this.m_buffer, 0, bytes, 0, this.m_buffer.Length);
    }
    BytePool.Recycle(this.m_buffer);
    this.m_buffer = bytes;
}
/// &lt;summary&gt;
/// &lt;inheritdoc/&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;disposing&#34;&gt;&lt;/param&gt;
protected sealed override void Dispose(bool disposing)
{
    if (this.m_holding)
    {
        return;
    }

    if (this.m_needDis)
    {
        if (Interlocked.Decrement(ref this.m_dis) == 0)
        {
            GC.SuppressFinalize(this);
            BytePool.Recycle(this.m_buffer);
            this.Dis();
        }
    }

    base.Dispose(disposing);
}
```

该字节池具有如下特性

&#43; 每隔1小时自动清理所有缓存的字节数组
&#43; 最大缓存的不同字节数组的数量为100

&#43; 回收的数组可以设置是否清零
&#43; 缓存的最大字节数512M
&#43; 缓存字节数组的范围1KB~20M

自动清理功能，每1个小时会自动清理池中所有的字节数组

```C#
static BytePool()
{
    m_timer = new Timer((o) =&gt;
    {
        BytePool.Clear();
    }, null, 1000 * 60 * 60, 1000 * 60 * 60);//1小时
    m_keyCapacity = 100;
    m_autoZero = false;
    m_maxSize = 1024 * 1024 * 512;//512M
    SetBlockSize(1024, 1024 * 1024 * 20);//1KB~ 20M
    AddSizeKey(10240);//10KB
}
/// &lt;summary&gt;
/// 清理
/// &lt;/summary&gt;
public static void Clear()
{
    bytesDictionary.Clear();
    GC.Collect();
}
/// &lt;summary&gt;
/// 获取内存核心。获取的核心可以不用归还。
/// &lt;/summary&gt;
/// &lt;param name=&#34;byteSize&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;equalSize&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static byte[] GetByteCore(int byteSize, bool equalSize = false)
{
    BytesQueue bytesCollection;
    if (equalSize)
    {
        //等长
        if (bytesDictionary.TryGetValue(byteSize, out bytesCollection))
        {
            if (bytesCollection.TryGet(out byte[] bytes))
            {
                m_fullSize -= byteSize;
                return bytes;
            }
        }
        else
        {
            CheckKeyCapacity(byteSize);
        }
        return new byte[byteSize];
    }
    else
    {
        byteSize = HitSize(byteSize);//取整，最小10K
        //搜索已创建集合
        if (bytesDictionary.TryGetValue(byteSize, out bytesCollection))
        {
            if (bytesCollection.TryGet(out byte[] bytes))
            {
                m_fullSize -= byteSize;
                return bytes;
            }
        }
        else
        {
            CheckKeyCapacity(byteSize);
        }
        return new byte[byteSize];
    }
}
/// &lt;summary&gt;
/// 回收内存核心
/// &lt;/summary&gt;
/// &lt;param name=&#34;bytes&#34;&gt;&lt;/param&gt;
public static void Recycle(byte[] bytes)
{
    if (bytes == null)
    {
        return;
    }
    if (m_maxSize &gt; m_fullSize)
    {
        if (bytesDictionary.TryGetValue(bytes.Length, out BytesQueue bytesQueue))
        {
            if (m_autoZero)
            {
                Array.Clear(bytes, 0, bytes.Length);
            }
            m_fullSize &#43;= bytes.Length;
            bytesQueue.Add(bytes);
        }
    }
    else
    {
        long size = 0;
        foreach (var collection in bytesDictionary.Values)
        {
            size &#43;= collection.FullSize;
        }
        m_fullSize = size;
    }
}
/// &lt;summary&gt;
/// 获取ByteBlock
/// &lt;/summary&gt;
/// &lt;param name=&#34;byteSize&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static ByteBlock GetByteBlock(int byteSize)
{
    if (byteSize &lt; m_minBlockSize)
    {
        byteSize = m_minBlockSize;
    }
    return GetByteBlock(byteSize, false);
}

/// &lt;summary&gt;
/// 获取内存池容量
/// &lt;/summary&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static long GetPoolSize()
{
    long size = 0;
    foreach (var item in bytesDictionary.Values)
    {
        size &#43;= item.FullSize;
    }
    return size;
}
```

## 等待处理池`WaitHandlePool&lt;T&gt;`

该池的主要是为了复用`WaitData&lt;T&gt;`对象，该对象的功能是：交由外部系统处理对象并返回数据。当给外部系统发送命令，等待外部系统执行命令，并在规定时间内返回数据。

该功能的传统实现方式是：

&#43; 创建命令队列，将所有已发送外部系统的命令装进命令队列中，同时记录命令的发送时间；
&#43; 当外部系统返回数据时，去队列中找到对应的命令执行命令成功的方法；如果命令队列中没有对应的命令，调用未知数据处理方法；
&#43; 设置定时器定期扫描命令队列，将超期没有接收到返回数据的命令移除，并调用对应命令异常执行方法；

该池的实现方式是构造`WaitData&lt;T&gt;`对象集合，发送命令前构建WaitData对象，发送完命令后该对象使用信号量阻塞发送命令的线程，当收到返回数据时，信号量置位继续执行，处理对应接收数据；信号量阻塞的时候可以设置阻塞时间，时间到则执行对应异常方法。

由于`WaitData`对象需要构建信号量，创建和销毁的代价比较大。WaitHandlePool构建` ConcurrentQueue&lt;WaitData&lt;T&gt;&gt; waitQueue;`队列保存空闲WaitData。` ConcurrentQueue&lt;WaitData&lt;T&gt;&gt; waitQueue;`保存所有已经使用WaitData。

```C#
/// &lt;summary&gt;
/// 等待数据对象
/// &lt;/summary&gt;
/// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
public class WaitData&lt;T&gt; : DisposableObject
{
    private WaitDataStatus m_status;

    private readonly AutoResetEvent m_waitHandle;

    private T m_waitResult;

    /// &lt;summary&gt;
    /// 构造函数
    /// &lt;/summary&gt;
    public WaitData()
    {
        this.m_waitHandle = new AutoResetEvent(false);
    }

    /// &lt;summary&gt;
    /// 状态
    /// &lt;/summary&gt;
    public WaitDataStatus Status =&gt; this.m_status;

    /// &lt;summary&gt;
    /// 等待数据结果
    /// &lt;/summary&gt;
    public T WaitResult =&gt; this.m_waitResult;

    /// &lt;summary&gt;
    /// 取消任务
    /// &lt;/summary&gt;
    public void Cancel()
    {
        this.m_status = WaitDataStatus.Canceled;
        this.m_waitHandle.Set();
    }

    /// &lt;summary&gt;
    /// 释放
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;disposing&#34;&gt;&lt;/param&gt;
    protected override void Dispose(bool disposing)
    {
        this.m_status = WaitDataStatus.Disposed;
        this.m_waitResult = default;
        this.m_waitHandle.Dispose();
        base.Dispose(disposing);
    }

    /// &lt;summary&gt;
    /// Reset。
    /// 设置&lt;see cref=&#34;WaitResult&#34;/&gt;为null。然后重置状态为&lt;see cref=&#34;WaitDataStatus.Default&#34;/&gt;，waitHandle.Reset()
    /// &lt;/summary&gt;
    public bool Reset()
    {
        this.m_status = WaitDataStatus.Default;
        this.m_waitResult = default;
        return this.m_waitHandle.Reset();
    }

    /// &lt;summary&gt;
    /// 使等待的线程继续执行
    /// &lt;/summary&gt;
    public bool Set()
    {
        this.m_status = WaitDataStatus.SetRunning;
        return this.m_waitHandle.Set();
    }

    /// &lt;summary&gt;
    /// 使等待的线程继续执行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;waitResult&#34;&gt;等待结果&lt;/param&gt;
    public bool Set(T waitResult)
    {
        this.m_waitResult = waitResult;
        this.m_status = WaitDataStatus.SetRunning;
        return this.m_waitHandle.Set();
    }

    /// &lt;summary&gt;
    /// 加载取消令箭
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;cancellationToken&#34;&gt;&lt;/param&gt;
    public void SetCancellationToken(CancellationToken cancellationToken)
    {
        if (cancellationToken.CanBeCanceled)
        {
            cancellationToken.Register(this.Cancel);
        }
    }

    /// &lt;summary&gt;
    /// 载入结果
    /// &lt;/summary&gt;
    public void SetResult(T result)
    {
        this.m_waitResult = result;
    }

    /// &lt;summary&gt;
    /// 等待指定毫秒
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;millisecond&#34;&gt;&lt;/param&gt;
    public WaitDataStatus Wait(int millisecond)
    {
        if (!this.m_waitHandle.WaitOne(millisecond))
        {
            this.m_status = WaitDataStatus.Overtime;
        }
        return this.m_status;
    }
}
```

```C#
/// &lt;summary&gt;
/// 等待处理数据
/// &lt;/summary&gt;
/// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
public class WaitHandlePool&lt;T&gt; : IDisposable where T : IWaitResult
{
    private readonly SnowflakeIDGenerator idGenerator;
    private readonly ConcurrentDictionary&lt;long, WaitData&lt;T&gt;&gt; waitDic;
    private readonly ConcurrentQueue&lt;WaitData&lt;T&gt;&gt; waitQueue;

    /// &lt;summary&gt;
    /// 构造函数
    /// &lt;/summary&gt;
    public WaitHandlePool()
    {
        this.waitDic = new ConcurrentDictionary&lt;long, WaitData&lt;T&gt;&gt;();
        this.waitQueue = new ConcurrentQueue&lt;WaitData&lt;T&gt;&gt;();
        this.idGenerator = new SnowflakeIDGenerator(4);
    }

    /// &lt;summary&gt;
    /// 销毁
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;waitData&#34;&gt;&lt;/param&gt;
    public void Destroy(WaitData&lt;T&gt; waitData)
    {
        if (waitData.DisposedValue)
        {
            throw new ObjectDisposedException(nameof(waitData));
        }
        if (this.waitDic.TryRemove(waitData.WaitResult.Sign, out _))
        {
            waitData.Reset();
            this.waitQueue.Enqueue(waitData);
        }
    }

    /// &lt;summary&gt;
    /// 释放
    /// &lt;/summary&gt;
    public void Dispose()
    {
        foreach (var item in this.waitDic.Values)
        {
            item.Dispose();
        }
        foreach (var item in this.waitQueue)
        {
            item.Dispose();
        }
        this.waitDic.Clear();

        this.waitQueue.Clear();
    }

    /// &lt;summary&gt;
    ///  获取一个可等待对象
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;result&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;autoSign&#34;&gt;设置为false时，不会生成sign&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public WaitData&lt;T&gt; GetWaitData(T result, bool autoSign = true)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitQueue.TryDequeue(out waitData))
        {
            if (autoSign)
            {
                result.Sign = this.idGenerator.NextID();
            }
            waitData.SetResult(result);
            this.waitDic.TryAdd(result.Sign, waitData);
            return waitData;
        }

        waitData = new WaitData&lt;T&gt;();
        if (autoSign)
        {
            result.Sign = this.idGenerator.NextID();
        }
        waitData.SetResult(result);
        this.waitDic.TryAdd(result.Sign, waitData);
        return waitData;
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;sign&#34;&gt;&lt;/param&gt;
    public void SetRun(long sign)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(sign, out waitData))
        {
            waitData.Set();
        }
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;sign&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;waitResult&#34;&gt;&lt;/param&gt;
    public void SetRun(long sign, T waitResult)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(sign, out waitData))
        {
            waitData.Set(waitResult);
        }
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;waitResult&#34;&gt;&lt;/param&gt;
    public void SetRun(T waitResult)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(waitResult.Sign, out waitData))
        {
            waitData.Set(waitResult);
        }
    }
}
```

```C#
/// &lt;summary&gt;
/// 等待处理数据
/// &lt;/summary&gt;
/// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
public class WaitHandlePool&lt;T&gt; : IDisposable where T : IWaitResult
{
    private readonly SnowflakeIDGenerator idGenerator;
    private readonly ConcurrentDictionary&lt;long, WaitData&lt;T&gt;&gt; waitDic;
    private readonly ConcurrentQueue&lt;WaitData&lt;T&gt;&gt; waitQueue;

    /// &lt;summary&gt;
    /// 构造函数
    /// &lt;/summary&gt;
    public WaitHandlePool()
    {
        this.waitDic = new ConcurrentDictionary&lt;long, WaitData&lt;T&gt;&gt;();
        this.waitQueue = new ConcurrentQueue&lt;WaitData&lt;T&gt;&gt;();
        this.idGenerator = new SnowflakeIDGenerator(4);
    }

    /// &lt;summary&gt;
    /// 销毁
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;waitData&#34;&gt;&lt;/param&gt;
    public void Destroy(WaitData&lt;T&gt; waitData)
    {
        if (waitData.DisposedValue)
        {
            throw new ObjectDisposedException(nameof(waitData));
        }
        if (this.waitDic.TryRemove(waitData.WaitResult.Sign, out _))
        {
            waitData.Reset();
            this.waitQueue.Enqueue(waitData);
        }
    }

    /// &lt;summary&gt;
    /// 释放
    /// &lt;/summary&gt;
    public void Dispose()
    {
        foreach (var item in this.waitDic.Values)
        {
            item.Dispose();
        }
        foreach (var item in this.waitQueue)
        {
            item.Dispose();
        }
        this.waitDic.Clear();

        this.waitQueue.Clear();
    }

    /// &lt;summary&gt;
    ///  获取一个可等待对象
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;result&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;autoSign&#34;&gt;设置为false时，不会生成sign&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public WaitData&lt;T&gt; GetWaitData(T result, bool autoSign = true)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitQueue.TryDequeue(out waitData))
        {
            if (autoSign)
            {
                result.Sign = this.idGenerator.NextID();
            }
            waitData.SetResult(result);
            this.waitDic.TryAdd(result.Sign, waitData);
            return waitData;
        }

        waitData = new WaitData&lt;T&gt;();
        if (autoSign)
        {
            result.Sign = this.idGenerator.NextID();
        }
        waitData.SetResult(result);
        this.waitDic.TryAdd(result.Sign, waitData);
        return waitData;
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;sign&#34;&gt;&lt;/param&gt;
    public void SetRun(long sign)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(sign, out waitData))
        {
            waitData.Set();
        }
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;sign&#34;&gt;&lt;/param&gt;
    /// &lt;param name=&#34;waitResult&#34;&gt;&lt;/param&gt;
    public void SetRun(long sign, T waitResult)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(sign, out waitData))
        {
            waitData.Set(waitResult);
        }
    }

    /// &lt;summary&gt;
    /// 让等待对象恢复运行
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;waitResult&#34;&gt;&lt;/param&gt;
    public void SetRun(T waitResult)
    {
        WaitData&lt;T&gt; waitData;
        if (this.waitDic.TryGetValue(waitResult.Sign, out waitData))
        {
            waitData.Set(waitResult);
        }
    }
}
```

## 总结

主要介绍了字节池和等待处理池，详细介绍了相关特性和业务功能的实现。对于等待处理池我觉得还可以使用`TaskCompletionSource&lt;TResult&gt;`来实现，消耗比信号量更小，同时是异步执行，不会阻塞线程。

---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket3-bytespoolwaithandlepool/  

