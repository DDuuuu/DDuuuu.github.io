# DotNet基础 Socket


## 概述

高性能的套接字编程围绕着两个方面：异步和复用。异步：高性能就是最大化计算机资源的利用，是不可能让线程有阻塞的，所以就有了各种异步模式。复用：计算机资源最好是能重复使用的，频繁的创建和销毁相同的对象也是对资源的浪费，所以就有了各种池和零拷贝；CPU在访问相邻资源的时候有特别的优势可以利用缓存区，所以池中对象尽量相邻创建。

Socket套接字编程历史悠久，发展出好几种方式，对应着DotNet异步编程的发展，分别：**异步编程模式(Asynchronous Programming Model ,APM)**、**基于事件的异步模式(Event-based Asynchronous Pattern ,EAP)**和**基于任务的异步模式(Task-based Asynchronous Pattern,TAP)**。

本文将简要介绍几种异步编程对应Socket的实现，每一种都写了一个简单的Socket服务端以供学习。



## 面向连接的套接字

套接字流程如下，在Accept，Read，Write，Connect和Disconnect方法均涉及到异步编程。为什么会异步，简单来说就是线程执行速度很快，网络传输的IO速度很慢，线程发出IO操作的指令后，不可能一直等待指令执行完。所以线程设置一个回调函数的入口地址，让IO执行完之后调用该入口地址，之后线程就去干其他事情了，等该IO调用该入口地址，线程再回来继续工作。

![image-20220914095432026](/dotnet/image-20220914095432026.png)

## 阻塞式套接字

[Socket接口](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socket?view=net-6.0)，下面是用阻塞方法创建的一个简单服务端。可以分析出该服务的性能是很差的，没有做任何的异步和复用。

```C#
//服务端
public static void Run(string m_ip, int m_port)
{
	var socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
	var ip = IPAddress.Parse(m_ip);
	var endpoint = new IPEndPoint(ip, m_port);
	socket.Bind(endpoint);
	socket.Listen(0);
	socket.ReceiveTimeout = -1;
	//线程池中后台线程执行
	Task.Run(() =&gt;
	{
		while (true)
		{
			var acceptSocket = socket.Accept();//线程阻塞等待连接请求队列
			if (acceptSocket != null &amp;&amp; acceptSocket.Connected)
			{
				//线程池中后台线程执行
				Task.Run(() =&gt;
				{
					byte[] receiveBuffer = new byte[1024];//每一个连接都在重新创建缓冲区
					int result = 0;
					do
					{
						if (acceptSocket.Connected)
						{
							result = acceptSocket.Receive(receiveBuffer, 0, receiveBuffer.Length,
								SocketFlags.None,
								out SocketError error);//线程阻塞等待缓冲区数据
							if (error == SocketError.Success &amp;&amp; result &gt; 0)
							{
								var recestr = Encoding.UTF8.GetString(receiveBuffer, 0, result);
								var Replaystr =
									$&#34;Server收到消息:{recestr};Server收到消息的时间:{DateTime.Now.ToString(&#34;yyyy-MM-dd HH:mm:ss:fff&#34;)}&#34;;
								var strbytes = Encoding.UTF8.GetBytes(Replaystr);
								acceptSocket.Send(strbytes, 0, strbytes.Length, SocketFlags.None);//线程阻塞等待发送完缓冲区数据
								if (recestr.Contains(&#34;stop&#34;))
								{
									break;
								}
							}
						}
						else
						{
							break;
						}
					} while (result &gt; 0);
				}).ContinueWith((t) =&gt;
				{
					System.Threading.Thread.Sleep(1000);
					acceptSocket.Disconnect(false);
					acceptSocket.Dispose();
				});
			}
		}
	}).Wait();
}
```

## 异步编程模式(Asynchronous Programming Model ,APM)

BeginXXX方法并不会阻塞线程，而EndXXX会，dotnet提供`Task&lt;T&gt;.Factory.FromAsync`可以将APM转成TAP模式异步模式以提高性能，下面提供一个示例，同时使用ArrayPool复用缓冲区，处理分包，粘包等问。

```C#
public static Socket Run(string m_ip, int m_port)
{
	var socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
	var ip = IPAddress.Any;
	if (!string.IsNullOrEmpty(m_ip))
	{
		 ip = IPAddress.Parse(m_ip);
	}
	var endpoint = new IPEndPoint(ip, m_port);
	socket.Bind(endpoint);
	Console.WriteLine($&#34;[{DateTime.Now.GetFormString()}] Server Established localEndpoint:[{socket.LocalEndPoint.ToString()}]&#34;);
	socket.Listen(200);
	socket.ReceiveTimeout = -1;
	//后台线程执行
	Task.Run(async () =&gt;
	{
		while (true)
		{

			var acceptSocket = await Task&lt;Socket&gt;.Factory.FromAsync(
				socket.BeginAccept(null,null)
			,socket.EndAccept);//APM转TAP异步
			if (acceptSocket != null &amp;&amp; acceptSocket.Connected)
			{
				//后台线程来处理Receive逻辑
				var task = Task.Run(async () =&gt;
				  {
					  byte[] buffer = ArrayPool&lt;byte&gt;.Shared.Rent(1024);//从内存池中获取缓冲区
					  var bytesBuffered = 0;
					  var bytesConsumed = 0;
					  while (true)
					  {
						  if (acceptSocket != null &amp;&amp; acceptSocket.Connected)
						  {
							  var temremaining = bytesBuffered - bytesConsumed;
							  if (temremaining == 0)//缓存区全部解析完
							  {
								  bytesBuffered = 0;
								  bytesConsumed = 0;
							  }
							  else if (temremaining &lt; buffer.Length &amp;&amp; temremaining &gt; 0)//最后一个包不完整，部分数据未解析
							  {

								  var newbuffer = ArrayPool&lt;byte&gt;.Shared.Rent(buffer.Length);
								  Buffer.BlockCopy(buffer, bytesConsumed, newbuffer, 0, temremaining);
								  ArrayPool&lt;byte&gt;.Shared.Return(buffer);
								  buffer = newbuffer;
								  bytesBuffered = temremaining;
								  bytesConsumed = 0;
							  }
							  else //包不够大，分包了
							  {
								  var newbuffer = ArrayPool&lt;byte&gt;.Shared.Rent(buffer.Length * 2);
								  Buffer.BlockCopy(buffer, 0, newbuffer, 0, buffer.Length);
								  ArrayPool&lt;byte&gt;.Shared.Return(buffer);
								  buffer = newbuffer;
							  }
							  var bytesRemaining = buffer.Length - bytesBuffered;
							  
							  try
							  {
								  var bytesread = await Task&lt;int&gt;.Factory.FromAsync(
                                              acceptSocket.BeginReceive(buffer, bytesBuffered, bytesRemaining,
                                                  SocketFlags.None, null, null), acceptSocket.EndReceive);//APM转TAP异步
								  if (bytesread == 0)
								  {
									  break;
								  }
								  bytesbuffered &#43;= bytesread;
								  var lineposition = -1;
								  do
								  {
									  lineposition = array.indexof(buffer, (byte)0x23, bytesconsumed,bytesbuffered - bytesconsumed);
									  if (lineposition &gt;= 0)
									  {
										  var lineLength = linePosition - bytesConsumed;
										  ProcessLine(acceptSocket, buffer, bytesConsumed, bytesread);
										  bytesConsumed &#43;= bytesread;
									  }
								  } while (linePosition &gt;= 0);//包解析
							  }
							  catch (Exception e)
							  {
								  break;
							  }
						  }
						  else
						  {
							  break;
						  }
					  }
					  ArrayPool&lt;byte&gt;.Shared.Return(buffer);
				  }).ContinueWith((t) =&gt;
				  {
					  Console.WriteLine($&#34;[{DateTime.Now.GetFormString()}] ServerClient Disconnected localEndpoint:[{acceptSocket?.LocalEndPoint.ToString()}] remoteEndpoint:[{acceptSocket?.RemoteEndPoint.ToString()}]&#34;);
					  acceptSocket?.Shutdown(SocketShutdown.Both);
					  acceptSocket?.Close();
					  acceptSocket = null;
				  });
			}
		}
	});
	return socket;
}
```

## 基于事件异步的完成端口模型(Event-based Asynchronous Pattern ,EAP)

目前应用最广的Socket模型，完成端口模型还是按照&#34;回调函数&#34;的方式进行来实现异步，其本质是线程池，该线程池的核心工作就是去调用IO操作完成时的回调函数。另外因为IO操作毕竟是慢速的操作，所以几个线程就已经足可以应付成千上万的输入输出完成操作的请求(前提就是你的回调函数做的工作要足够少)，所以这个模型的性能是非常高的。也是现在Windows平台上性能最好的输入输出模型。自定义构造了内存池，将一大块内存切分成一定数据量的连续小内存，分别分配给不同的SocketAsyncEventArgs对象以提高服务性能，非常巴适；目前看到的FastSocket,SuperSocket,TouchSocket,NewLife等网络框架均采用这种模式，最主要的原因是应用范围广。

| 框架               | 版本                                                         |
| ------------------ | ------------------------------------------------------------ |
| **.NET**           | Core 1.0, Core 1.1, Core 2.0, Core 2.1, Core 2.2, Core 3.0, Core 3.1, 5, 6, 7 Preview 7 |
| **.NET Framework** | 2.0, 3.0, 3.5, 4.0, 4.5, 4.5.1, 4.5.2, 4.6, 4.6.1, 4.6.2, 4.7, 4.7.1, 4.7.2, 4.8 |
| **.NET Standard**  | 1.3, 1.4, 1.6, 2.0, 2.1                                      |
| **UWP**            | 10.0                                                         |
| **Xamarin.iOS**    | 10.8                                                         |
| **Xamarin.Mac**    | 3.0                                                          |

```C#
public class MyIOCPSocket
{
	private static int m_numConnections;//最大连接数
	private static int m_receiveBufferSize;//接收缓存区数量
	private static int m_sendBufferSize;//发送缓存区大小
	private static byte[] m_receivebuffer;//接收缓存区
	private static Stack&lt;int&gt; m_freeReceiveIndexPool;//可用的接收缓存索引栈
	private static int m_currentReceiveIndex;//当前的接收缓存区索引
	private static byte[] m_sendbuffer;//发送缓存区
	private static Stack&lt;int&gt; m_freeSendIndexPool;//可用的发送缓存索引栈
	private static int m_currentSendIndex;//当前的发送缓存区索引
	private static Stack&lt;SocketAsyncEventArgs&gt; m_ReadPool;//接收SocketAsyncEventArgs池
	private static Stack&lt;SocketAsyncEventArgs&gt; m_WritePool;//发送SocketAsyncEventArgs池
	private static Semaphore m_maxNumberAcceptedClients;//最大连接锁
	private static int m_numConnectedSockets;//连接的Socket数量
	private static int m_totalBytesRead;//总的接收字节数
	private static Socket listenSocket;//监听Socket
	public static void Run(string m_ip, int m_port, int numConnections, int m_receiveBuffer, int m_sentBuffer)
	{
		//初始化
		m_numConnections = numConnections;
		m_receiveBufferSize = m_receiveBuffer;
		m_sendBufferSize = m_sentBuffer;
		m_receivebuffer = new byte[m_receiveBufferSize * m_numConnections];
		m_freeReceiveIndexPool = new Stack&lt;int&gt;();
		m_currentReceiveIndex = 0;
		m_sendbuffer = new byte[m_sendBufferSize * m_numConnections];
		m_freeSendIndexPool = new Stack&lt;int&gt;();
		m_currentSendIndex = 0;
		m_ReadPool = new Stack&lt;SocketAsyncEventArgs&gt;(m_numConnections);
		m_WritePool = new Stack&lt;SocketAsyncEventArgs&gt;(m_numConnections);
		m_maxNumberAcceptedClients = new Semaphore(m_numConnections, m_numConnections);
		m_numConnectedSockets = 0;
		m_totalBytesRead = 0;

		//接收缓存分配
		for (int i = 0; i &lt; m_numConnections; i&#43;&#43;)
		{
			var readEventArg = new SocketAsyncEventArgs();
			readEventArg.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;(ReadWriteIOComleted);
			readEventArg.UserToken = new AsyncUserToken();
			if (m_freeReceiveIndexPool.Count &gt; 0)
			{
				readEventArg.SetBuffer(m_receivebuffer, m_freeReceiveIndexPool.Pop(), m_receiveBufferSize);
			}
			else
			{
				if ((m_receiveBufferSize * m_numConnections - m_receiveBufferSize) &lt; m_currentReceiveIndex)
				{
					new ArgumentException(&#34;接收缓存设置异常&#34;);
				}
				readEventArg.SetBuffer(m_receivebuffer, m_currentReceiveIndex, m_receiveBufferSize);
				m_currentReceiveIndex &#43;= m_receiveBufferSize;
			}
			m_ReadPool.Push(readEventArg);


			//发送缓存分配
			var writeEventArg = new SocketAsyncEventArgs();
			writeEventArg.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;(ReadWriteIOComleted);
			writeEventArg.UserToken = new AsyncUserToken();
			if (m_freeSendIndexPool.Count &gt; 0)
			{
				writeEventArg.SetBuffer(m_sendbuffer, m_freeSendIndexPool.Pop(), m_sendBufferSize);
			}
			else
			{
				if ((m_sendBufferSize * m_numConnections - m_sendBufferSize) &lt; m_currentSendIndex)
				{
					new ArgumentException(&#34;发送缓存设置异常&#34;);
				}
				writeEventArg.SetBuffer(m_sendbuffer, m_currentSendIndex, m_sendBufferSize);
				m_currentSendIndex &#43;= m_sendBufferSize;
			}
			m_WritePool.Push(writeEventArg);

		}

		//设置监听socket
		listenSocket = new Socket(new IPEndPoint(IPAddress.Parse(m_ip), m_port).AddressFamily, SocketType.Stream, ProtocolType.Tcp);
		//绑定端口
		listenSocket.Bind(new IPEndPoint(IPAddress.Parse(m_ip), m_port));
		listenSocket.Listen(100);

		StartAccept(null);
		Console.WriteLine(&#34;Press any key to terminate the server process....&#34;);
		Console.ReadKey();
	}

	public static void ReadWriteIOComleted(object sender, SocketAsyncEventArgs e)
	{
		switch (e.LastOperation)
		{
			case SocketAsyncOperation.Receive:
				ProcessReceive(e);
				break;
			case SocketAsyncOperation.Send:
				ProcessSend(e);
				break;
			default:
				throw new ArgumentException(&#34;The last operation completed on the socket was not a receive or send&#34;);
		}
	}
	//发送消息回调
	public static void ProcessSend(SocketAsyncEventArgs e)
	{
		if (e.SocketError == SocketError.Success)
		{
			AsyncUserToken token = (AsyncUserToken)e.UserToken;
			bool willRaiseEvent = token.Socket.ReceiveAsync(token.readEventArgs);
			if (!willRaiseEvent)
			{
				ProcessReceive(token.readEventArgs);
			}
		}
		else
		{
			CloseClientSocket(e);
		}


	}
	//关闭客户端
	public static void CloseClientSocket(SocketAsyncEventArgs e)
	{
		AsyncUserToken token = e.UserToken as AsyncUserToken;
		try
		{
			token.Socket.Shutdown(SocketShutdown.Send);
		}
		catch (Exception exception)
		{
			Console.WriteLine(exception);
		}
		token.Socket.Close();
		Interlocked.Decrement(ref m_numConnectedSockets);
		//将资源返回池中进行复用
		m_ReadPool.Push(token.readEventArgs);
		m_WritePool.Push(token.writeEventArgs);
		token.Socket = null;
		token.readEventArgs = null;
		token.writeEventArgs = null;
		m_maxNumberAcceptedClients.Release();
	}
	
	public static void ProcessReceive(SocketAsyncEventArgs e)
	{
		
		AsyncUserToken token = (AsyncUserToken)e.UserToken;
		//接收到消息
		if (e.BytesTransferred &gt; 0 &amp;&amp; e.SocketError == SocketError.Success)
		{
		
			Interlocked.Add(ref m_totalBytesRead, e.BytesTransferred);
			byte[] data = new byte[e.BytesTransferred];
			Array.Copy(e.Buffer, e.Offset, data, 0, e.BytesTransferred);
			var recestr = Encoding.UTF8.GetString(data);
			var Replaystr =
				$&#34;Server收到消息:{recestr};Server收到消息的时间:{DateTime.Now.ToString(&#34;yyyy-MM-dd HH:mm:ss:fff&#34;)}&#34;;
			Console.WriteLine(Replaystr);
			var strbytes = Encoding.UTF8.GetBytes(Replaystr);
			Array.Copy(strbytes, 0, token.writeEventArgs.Buffer, token.writeEventArgs.Offset,
				strbytes.Length);
			//完成端口模型处理发送
			bool willRaiseEvent = token.Socket.SendAsync(token.writeEventArgs);
			if (!willRaiseEvent)
			{
				ProcessSend(token.writeEventArgs);
			}
		}
		else
		{
			CloseClientSocket(e);
		}
	}



	//如果异步完成，有线程池中线程执行
	public static void ProcessAccept(SocketAsyncEventArgs e)
	{
		Interlocked.Increment(ref m_numConnectedSockets);
		//从池中获取数据
		SocketAsyncEventArgs readEventArgs = m_ReadPool.Pop();
		SocketAsyncEventArgs writeEventArgs = m_WritePool.Pop();
		((AsyncUserToken)readEventArgs.UserToken).Socket = e.AcceptSocket;
		((AsyncUserToken)readEventArgs.UserToken).readEventArgs = readEventArgs;
		((AsyncUserToken)readEventArgs.UserToken).writeEventArgs = writeEventArgs;

		((AsyncUserToken)writeEventArgs.UserToken).Socket = e.AcceptSocket;
		((AsyncUserToken)writeEventArgs.UserToken).readEventArgs = readEventArgs;
		((AsyncUserToken)writeEventArgs.UserToken).writeEventArgs = writeEventArgs;
		//使用完成端口模型接收数据
		bool willRaiseEvent = e.AcceptSocket.ReceiveAsync(readEventArgs);
		if (!willRaiseEvent)
		{
			ProcessReceive(readEventArgs);
		}
		StartAccept(e);
	}

	public static void StartAccept(SocketAsyncEventArgs listenEventArg)
	{
		if (listenEventArg == null)
		{
			//完成端口模型需要借助SocketAsyncEventArgs
			listenEventArg = new SocketAsyncEventArgs();
			//设置完成端口的回调
			listenEventArg.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;((sender, e) =&gt; ProcessAccept(e));
		}
		else
		{
			listenEventArg.AcceptSocket = null;
		}

		m_maxNumberAcceptedClients.WaitOne();
		bool willRaiseEvent = listenSocket.AcceptAsync(listenEventArg);
		//如果同步完成返回False，异步完成返回True，触发Completed事件
		if (!willRaiseEvent)
		{
			ProcessAccept(listenEventArg);
		}
	}

}
class AsyncUserToken
{
	/// &lt;summary&gt;  
	/// 通信SOKET  
	/// &lt;/summary&gt;  
	public Socket Socket { get; set; }
	/// &lt;summary&gt;
	/// 读SocketAsyncEventArgs
	/// &lt;/summary&gt;
	public SocketAsyncEventArgs readEventArgs { set; get; }
	/// &lt;summary&gt;
	/// 写SocketAsyncEventArgs
	/// &lt;/summary&gt;
	public SocketAsyncEventArgs writeEventArgs { set; get; }

}
```

## 基于任务的异步模式(Task-based Asynchronous Pattern,TAP)

相对于前几个模型，基于任务的网络模型是比较新的模型，但是性能是最好的，最主要的原因是微软提供了**System.Net.Sockets.SocketTaskExtensions**封装TAP的异步方法；**System.IO.Pipelines**管道模型，在 .NET 中执行高性能 I/O 更加容易。该管道可以实现流量控制和反压。PipeScheduler可以进行回调线程控制。PipeReader和PipeWriter封装了对内存数据的直接操作，实现零拷贝得以大大提供业务流的性能。可惜的是应用范围比较小，目前框架只支持`2.1, 2.2, 3.0, 3.1, 5, 6, 7 Preview 7`，Framework不支持。

```C#
private static Pipe pipe;
public static Socket Run(string m_ip, int m_port)
{
	//监听Socket
	var socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
	var ip = IPAddress.Parse(m_ip);
	var endpoint = new IPEndPoint(ip, m_port);
	socket.Bind(endpoint); //绑定端口和IP
	socket.Listen(200); //允许同时监听的队列
	socket.ReceiveTimeout = -1;
	Task.Run(async () =&gt;
	{
		while (true)
		{
			var acceptSocket = await socket.AcceptAsync(); //TAP异步接收
			if (acceptSocket != null &amp;&amp; acceptSocket.Connected) 
			{
				pipe = new Pipe();
				var writer = pipe.Writer;
				var reader = pipe.Reader;

				var writetaskr = Task.Run(async () =&gt;
				{
					while (true)
					{
						var memory = writer.GetMemory(1024);
						try
						{
							//TAP 异步读取数据
							int bytesRead = await acceptSocket.ReceiveAsync(memory, SocketFlags.None);
							if (bytesRead == 0)
							{
								break;
							}
							//告诉 PipeWriter 写入多少数据。
							writer.Advance(bytesRead);
						}
						catch (Exception e)
						{
							break;
						}
						//刷新写入
						FlushResult result = await writer.FlushAsync();

						if (result.IsCompleted)
						{
							break;
						}
					}

					// 完成写入
					await writer.CompleteAsync();

				}).ContinueWith((t) =&gt;
				{
					acceptSocket?.Shutdown(SocketShutdown.Both);
					//acceptSocket?.Disconnect(true);
					acceptSocket?.Dispose();
					acceptSocket = null;
				});

				var readingtask= Task.Run(async() =&gt;
				{
					while (true)
					{
						try
						{
							//从管道中读取
							ReadResult result = await reader.ReadAsync();
							ReadOnlySequence&lt;byte&gt; buffer = result.Buffer;
							while (TryReadLine(ref buffer, out ReadOnlySequence&lt;byte&gt; line))//解析
							{
								ProcessLine(acceptSocket, line);
							}
							//实际读了多少
							reader.AdvanceTo(buffer.Start, buffer.End);
							//是否写已经结束
							if (result.IsCompleted)
							{
								break;
							}
						}
						catch (Exception e)
						{
						   break;
						}
					}
					await reader.CompleteAsync();
				}).ContinueWith((t) =&gt;
					{
						acceptSocket?.Shutdown(SocketShutdown.Both);
						acceptSocket?.Dispose();
						acceptSocket = null;
					}
				);
			}
		}
	});
	return socket;
}
```

## 总结

主要讲述在套接字编程中，如何实现异步和复用以提高性能。讲述了异步编程(APM)、基于事件的异步模型(EAP)和基于任务的异步模型(TAP)；复用方面从内存池(ArrayPool)，到自定义构建内存池(利用高速缓存)和完成端口池，再到最新的管道模型，实现零拷贝。





---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/10/dotnetbase1-socket/  

