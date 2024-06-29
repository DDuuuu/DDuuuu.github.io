# TouchSocket Socket基础


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

TouchSocket的底层使用完成端口模型，基于事件的异步模式。关于完成端口模型的基础知识可以看[Socket基础](https://dduuuu.github.io/2022/09/dotnetbase1-socket/) 。结合上篇横切面扩展([TouchSocket 封装和扩展](https://dduuuu.github.io/2022/09/touchsocket5-encapsulationextension/))可以实现各种业务需求。

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index



## Accept

在TcpServer类中BeginListen方法，一个监听者用一个SocketAsyncEventArgs

```C#
foreach (var networkMonitor in this.m_monitors)
{
    SocketAsyncEventArgs e = new SocketAsyncEventArgs();
    e.UserToken = networkMonitor.Socket;
    e.Completed &#43;= this.Args_Completed;
    if (!networkMonitor.Socket.AcceptAsync(e))
    {
        this.OnAccepted(e);
    }
}
```

## Receive

在SocketClient的BeginReceive方法中，缓存区使用了内存池进行复用，该内存池的细节可以看[TouchSocket 字节池和待处理池](https://dduuuu.github.io/2022/09/touchsocket3-bytespoolwaithandlepool/)，注意在处理完缓冲区后再HandleBuffer的finally中调用byteBlock的Dispose方法，将缓存区返回内存池，如果m_holding被设置为true，则由GC自己回收。

```C#
internal void BeginReceive(ReceiveType receiveType)
{
    if (receiveType == ReceiveType.Auto)
    {
        SocketAsyncEventArgs eventArgs = new SocketAsyncEventArgs();
        eventArgs.Completed &#43;= this.EventArgs_Completed;
        ByteBlock byteBlock = BytePool.GetByteBlock(this.BufferLength);//内存池获取缓冲区
        eventArgs.UserToken = byteBlock;
        eventArgs.SetBuffer(byteBlock.Buffer, 0, byteBlock.Capacity);
        if (!this.m_mainSocket.ReceiveAsync(eventArgs))
        {
            this.ProcessReceived(eventArgs);
        }
    }
}

private void HandleBuffer(ByteBlock byteBlock)
{
    try
    {
        if (this.ClearType.HasFlag(ClearType.Receive))
        {
            this.m_lastTick = DateTime.Now.Ticks;
        }

        if (this.OnHandleRawBuffer?.Invoke(byteBlock) == false)
        {
            return;
        }
        if (this.UsePlugin &amp;&amp; this.PluginsManager.Raise&lt;ITcpPlugin&gt;(&#34;OnReceivingData&#34;, this, new ByteBlockEventArgs(byteBlock)))
        {
            return;
        }
        if (this.m_disposedValue)
        {
            return;
        }
        if (this.m_adapter == null)
        {
            this.Logger.Debug(LogType.Error, this, ResType.NullDataAdapter.GetDescription());
            return;
        }
        this.m_adapter.ReceivedInput(byteBlock);
    }
    catch (System.Exception ex)
    {
        this.Logger.Debug(LogType.Error, this, &#34;在处理数据时发生错误&#34;, ex);
    }
    finally
    {
        byteBlock.Dispose();//内存池回收
    }
}
```

## Send

SocketClient的SocketSend方法中使用Send同步方法发送，如果异步使用异步编程模式的BeginSend方法。

```C#
protected void SocketSend(byte[] buffer, int offset, int length, bool isAsync)
{
    if (!this.m_online)
    {
        throw new NotConnectedException(ResType.NotConnected.GetDescription());
    }
    if (this.HandleSendingData(buffer, offset, length))
    {
        lock (this.m_sendLocker)
        {
            if (this.UseSsl)
            {
                this.m_workStream.Write(buffer, offset, length);
            }
            else
            {
                if (isAsync)
                {
                    this.m_mainSocket.BeginSend(buffer, offset, length, SocketFlags.None, null, null);
                }
                else
                {
                    while (length &gt; 0)
                    {
                        int r = this.m_mainSocket.Send(buffer, offset, length, SocketFlags.None);
                        if (r == 0 &amp;&amp; length &gt; 0)
                        {
                            throw new Exception(&#34;发送数据不完全&#34;);
                        }
                        offset &#43;= r;
                        length -= r;
                    }
                }
            }
        }
        if (this.ClearType.HasFlag(ClearType.Send))
        {
            this.m_lastTick = DateTime.Now.Ticks;
        }
    }
}
```

## Connect

在TcpClient的Connect方法中，使用异步编程模式的BeginConnect和EndConnect方法

```C#
/// &lt;summary&gt;
/// 请求连接到服务器。
/// &lt;/summary&gt;
public virtual ITcpClient Connect(int timeout = 5000)
{
    if (this.m_online)
    {
        return this;
    }
    if (this.m_disposedValue)
    {
        throw new ObjectDisposedException(this.GetType().FullName);
    }
    if (this.m_config == null)
    {
        throw new ArgumentNullException(&#34;配置文件不能为空。&#34;);
    }
    IPHost iPHost = this.m_config.GetValue&lt;IPHost&gt;(TouchSocketConfigExtension.RemoteIPHostProperty);
    if (iPHost == null)
    {
        throw new ArgumentNullException(&#34;iPHost不能为空。&#34;);
    }
    if (this.m_mainSocket != null)
    {
        this.m_mainSocket.Dispose();
    }
    this.m_mainSocket = this.CreateSocket(iPHost);

    ClientConnectingEventArgs args = new ClientConnectingEventArgs(this.m_mainSocket);
    this.PrivateOnConnecting(args);

    var result = this.m_mainSocket.BeginConnect(iPHost.EndPoint, null, null);//APM
    if (result.AsyncWaitHandle.WaitOne(timeout))
    {
        if (this.m_mainSocket.Connected)
        {
            this.m_mainSocket.EndConnect(result);//APM
            this.LoadSocketAndReadIpPort();

            if (this.m_separateThreadSend)
            {
                this.m_asyncSender.SafeDispose();
                this.m_asyncSender = new AsyncSender(this.m_mainSocket, this.m_mainSocket.RemoteEndPoint, this.OnSeparateThreadSendError);
            }
            this.BeginReceive();
            this.m_online = true;
            this.PrivateOnConnected(new MsgEventArgs(&#34;连接成功&#34;));
            return this;
        }
    }
    this.m_mainSocket.Dispose();
    throw new TimeoutException();
}
```

## 总结

本篇讲述底层Socket实现，使用完成端口模型和内存池提高Socket性能。





---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket6-socket/  

