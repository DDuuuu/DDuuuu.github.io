# TouchSocket 文件池


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

读写文件是应用中必不可少的操作，也是比较经典的问题。该问题一般要求：

&#43; 允许多个读者对文件执行读操作。
&#43; 只允许一个写者往文件中写信息。
&#43; 任一写者写完前不允许其他读者或写者工作。
&#43; 写者执行写操作前应让已有读者和写者全部退出。

实现要求的逻辑代码:

```C#
//声明
Semaphore rwlock=1;//读写锁
int rcount=0;//读者数量
Semaphore countlock=1;//读者数量锁
Semaphore firstwlock=1;//写优先锁

//写逻辑
Write()
{
	while(1)
	{
		firstwlock.WaitOne();
		rwlock.WaitOne();
		//Todo:写文件
		rwlock.Set();
		firstwlock.Set();
	}
}
//读逻辑
Reader()
{
	while(1)
	{
		firstwlock.WaitOne();
		countlock.WaitOne();
		if(rcount==0)
			rwlock.WaitOne();
		rcount&#43;&#43;;
		countlock.Set();
		firstwlock.Set();
		//Todo:读文件
		countlock.WaitOne();
		rcount--;
		if(rcount==0)
			rwlock.Set();
		countlock.Set();
	}
}
```

下面看一下TouchSocket文件池解析:

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index

## 文件池FilePool

池的作用是复用文件池中对于大文件缓存了文件句柄，对于小文件缓存了文件内容。这样多次对文件的操作就不需要频繁创建和销毁文件句柄。提高文件操作性能。

&#43; 对于缓存元数据的封装为FileStorage，也是真正对文件操作的实现类。FilePool保存其字典进行复用。
&#43; 为了便于对FileStorage的操作，将读写操作封装成FileStorageReader和FileStorageWriter
&#43; 在写入文件时为了保存的实时写入状态，封装了TouchRpcFileStream，并将状态信息封装成TouchRpcFileInfo。
&#43; FilePool和FileStorage类中相关操作全部加锁以保证线程安全，并通过原子操作保证FileStorage引用数量

相关类图：

![image-20220911083207553](/images/touchsocket/image-20220911083207553.png)

![image-20220911083219384](/images/touchsocket/image-20220911083219384.png)

![image-20220911083228691](/images/touchsocket/image-20220911083228691.png)

读写模式的互斥通过状态保证:

```C#
/// &lt;summary&gt;
/// 加载文件为读取流
/// &lt;/summary&gt;
/// &lt;param name=&#34;path&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static void LoadFileForRead(string path)
{
    lock (m_locker)
    {
        if (string.IsNullOrEmpty(path))
        {
            throw new System.ArgumentException($&#34;“{nameof(path)}”不能为 null 或空。&#34;, nameof(path));
        }

        path = Path.GetFullPath(path);
        if (pathStream.TryGetValue(path, out FileStorage storage))
        {
            if (storage.Access != FileAccess.Read)
            {
                throw new Exception(&#34;该路径的文件已经被加载为写入模式。&#34;);
            }
            return;
        }
        if (FileStorage.TryCreateFileStorage(path, FileAccess.Read, out FileStorage fileStorage, out string msg))
        {
            pathStream.TryAdd(path, fileStorage);
        }
        else
        {
            throw new Exception(msg);
        }
    }
}
```

读写操作：

```C#
/// &lt;summary&gt;
/// 从指定位置，读取数据到缓存区。线程安全。
/// &lt;/summary&gt;
/// &lt;param name=&#34;stratPos&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;buffer&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;offset&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;length&#34;&gt;&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public int Read(long stratPos, byte[] buffer, int offset, int length)
{
    using (WriteLock writeLock = new WriteLock(this.m_lockSlim))
    {
        if (this.m_disposedValue)
        {
            throw new ObjectDisposedException(this.GetType().FullName);
        }
        if (this.m_fileAccess != FileAccess.Read)
        {
            throw new Exception(&#34;该流不允许读取。&#34;);
        }
        if (this.m_cache)
        {
            int r = (int)Math.Min(this.m_fileData.Length - stratPos, length);
            Array.Copy(this.m_fileData, stratPos, buffer, offset, r);
            return r;
        }
        else
        {
            this.m_fileStream.Position = stratPos;
            return this.m_fileStream.Read(buffer, offset, length);
        }
    }
}

/// &lt;summary&gt;
/// 从指定位置，写入数据到存储区。线程安全。
/// &lt;/summary&gt;
/// &lt;param name=&#34;stratPos&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;buffer&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;offset&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;length&#34;&gt;&lt;/param&gt;
public void Write(long stratPos, byte[] buffer, int offset, int length)
{
    using (WriteLock writeLock = new WriteLock(this.m_lockSlim))
    {
        if (this.m_disposedValue)
        {
            throw new ObjectDisposedException(this.GetType().FullName);
        }
        if (this.m_fileAccess != FileAccess.Write)
        {
            throw new Exception(&#34;该流不允许写入。&#34;);
        }
        this.m_fileStream.Position = stratPos;
        this.m_fileStream.Write(buffer, offset, length);
        this.m_fileStream.Flush();
    }
}
```



## 总结

讲述了经典的读写问题，介绍了TouchSocket中FilePool的封装，该封装并不复杂，但我觉得一个简单且功能强大的设计才是一个优秀的设计。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket4-filepool/  

