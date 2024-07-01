# DotNet基础 Socket与完成端口模型


## 介绍

一次简单的Socket探索之旅，分别对Socket服务端的两种方式进行了测试和解析。

## CommonSocket

### 代码实现

实现一个简单的Socket服务，基本功能就是接收消息然后加上结束消息时间返回给客户端。

```c#
    /// &lt;summary&gt;
    /// 简单服务，收发消息
    /// &lt;/summary&gt;
    class FirstSimpleServer
    {
        public static void Run(string m_ip, int m_port)
        {
            var socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            var ip = IPAddress.Parse(m_ip);
            var endpoint = new IPEndPoint(ip, m_port);
            socket.Bind(endpoint);
            socket.Listen(0);
            socket.ReceiveTimeout = -1;
            Task.Run(() =&gt;
            {
                while (true)
                {
                    var acceptSocket = socket.Accept();
                    if (acceptSocket != null &amp;&amp; acceptSocket.Connected)
                    {
                        Task.Run(() =&gt;
                        {
                            byte[] receiveBuffer = new byte[256];

                            int result = 0;
                            do
                            {
                                if (acceptSocket.Connected)
                                {

                                    result = acceptSocket.Receive(receiveBuffer, 0, receiveBuffer.Length,
                                        SocketFlags.None,
                                        out SocketError error);
                                    if (error == SocketError.Success &amp;&amp; result &gt; 0)
                                    {
                                        var recestr = Encoding.UTF8.GetString(receiveBuffer, 0, result);
                                        var Replaystr =
                                            $&#34;Server收到消息:{recestr};Server收到消息的时间:{DateTime.Now.ToString(&#34;yyyy-MM-dd HH:mm:ss:fff&#34;)}&#34;;
                                        Console.WriteLine(Replaystr);
                                        var strbytes = Encoding.UTF8.GetBytes(Replaystr);
                                        acceptSocket.Send(strbytes, 0, strbytes.Length, SocketFlags.None);

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

### 简单测试

测试：一个客户端，发送10次数据，每次间隔50ms，

结果：客户端的显示如下，客户端发送消息，再接收到，十次中最长的耗时10ms。

```xaml
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:0;Client发送时间:2020-04-11 13:21:22:974};Server收到消息的时间:2020-04-11 13:21:22:981;ClientReceiceServer时间:2020-04-11 13:21:22:984}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:1;Client发送时间:2020-04-11 13:21:23:032};Server收到消息的时间:2020-04-11 13:21:23:032;ClientReceiceServer时间:2020-04-11 13:21:23:032}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:2;Client发送时间:2020-04-11 13:21:23:082};Server收到消息的时间:2020-04-11 13:21:23:082;ClientReceiceServer时间:2020-04-11 13:21:23:083}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:3;Client发送时间:2020-04-11 13:21:23:133};Server收到消息的时间:2020-04-11 13:21:23:133;ClientReceiceServer时间:2020-04-11 13:21:23:133}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:4;Client发送时间:2020-04-11 13:21:23:184};Server收到消息的时间:2020-04-11 13:21:23:184;ClientReceiceServer时间:2020-04-11 13:21:23:190}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:5;Client发送时间:2020-04-11 13:21:23:235};Server收到消息的时间:2020-04-11 13:21:23:235;ClientReceiceServer时间:2020-04-11 13:21:23:235}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:6;Client发送时间:2020-04-11 13:21:23:286};Server收到消息的时间:2020-04-11 13:21:23:286;ClientReceiceServer时间:2020-04-11 13:21:23:286}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:7;Client发送时间:2020-04-11 13:21:23:336};Server收到消息的时间:2020-04-11 13:21:23:336;ClientReceiceServer时间:2020-04-11 13:21:23:336}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:8;Client发送时间:2020-04-11 13:21:23:387};Server收到消息的时间:2020-04-11 13:21:23:387;ClientReceiceServer时间:2020-04-11 13:21:23:388}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:9;Client发送时间:2020-04-11 13:21:23:438};Server收到消息的时间:2020-04-11 13:21:23:438;ClientReceiceServer时间:2020-04-11 13:21:23:438}
```

假如客户端发送消息速度加快，对服务端会有什么影响？测试将客户端发送消息的间隔修改为1ms

System.Threading.Thread.Sleep(1);

结果如下,并没有发现问题。

```c#
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:0;Client发送时间:2020-04-11 13:48:57:193};Server收到消息的时间:2020-04-11 13:48:57:196;ClientReceiceServer时间:2020-04-11 13:48:57:197}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:1;Client发送时间:2020-04-11 13:48:57:198};Server收到消息的时间:2020-04-11 13:48:57:198;ClientReceiceServer时间:2020-04-11 13:48:57:201}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:2;Client发送时间:2020-04-11 13:48:57:200};Server收到消息的时间:2020-04-11 13:48:57:201;ClientReceiceServer时间:2020-04-11 13:48:57:202}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:3;Client发送时间:2020-04-11 13:48:57:202};Server收到消息的时间:2020-04-11 13:48:57:202;ClientReceiceServer时间:2020-04-11 13:48:57:203}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:4;Client发送时间:2020-04-11 13:48:57:204};Server收到消息的时间:2020-04-11 13:48:57:204;ClientReceiceServer时间:2020-04-11 13:48:57:204}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:5;Client发送时间:2020-04-11 13:48:57:206};Server收到消息的时间:2020-04-11 13:48:57:206;ClientReceiceServer时间:2020-04-11 13:48:57:207}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:6;Client发送时间:2020-04-11 13:48:57:208};Server收到消息的时间:2020-04-11 13:48:57:208;ClientReceiceServer时间:2020-04-11 13:48:57:208}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:7;Client发送时间:2020-04-11 13:48:57:209};Server收到消息的时间:2020-04-11 13:48:57:209;ClientReceiceServer时间:2020-04-11 13:48:57:211}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:8;Client发送时间:2020-04-11 13:48:57:211};Server收到消息的时间:2020-04-11 13:48:57:211;ClientReceiceServer时间:2020-04-11 13:48:57:212}
ClientReceiceServer:{Server收到消息:{Client:1:MessageID:9;Client发送时间:2020-04-11 13:48:57:213};Server收到消息的时间:2020-04-11 13:48:57:213;ClientReceiceServer时间:2020-04-11 13:48:57:213}
```

再极致一点，将客户端的发送间隔取消，循环发送。看到下面服务端接收消息的结果，可看到消息包很混乱。仔细分析一下，发现其实服务器其实就收到3次消息，前两次接受256个字节，最后一次接收138字节。这是由于设置服务端接收缓存的大小为256个字节。说明发送比较快或并行发送的时候，服务端会很快将接收的缓存块填满，一旦填满，Receive方法就会返回，不然就处于阻塞状态。

```xaml
Server收到消息:{Client:1:MessageID:1;Client发送时间:2020-04-11 13:51:18:723}{Client:1:MessageID:2;Client发送时间:2020-04-11 13:51:18:724}{Client:1:MessageID:3;Client发送时间:2020-04-11 13:51:18:724}{Client:1:MessageID:4;Client发送时间:2020-04-11 13:51:18:;Server收到消息的时间:2020-04-11 13:51:18:724
Server收到消息:724}{Client:1:MessageID:5;Client发送时间:2020-04-11 13:51:18:724}{Client:1:MessageID:6;Client发送时间:2020-04-11 13:51:18:724}{Client:1:MessageID:7;Client发送时间:2020-04-11 13:51:18:724}{Client:1:MessageID:8;Client发送时间:2020-04-11 13:51;Server收到消息的时间:2020-04-11 13:51:18:729
Server收到消息::18:724}{Client:1:MessageID:9;Client发送时间:2020-04-11 13:51:18:724};Server收到消息的时间:2020-04-11 13:51:18:732
```

通过一个简单的方法解决这个问题，每次客户端发送固定长度的消息，服务端接收固定长度的消息。现在客户端发送的消息是65个字节，设置服务端接收数据的缓存块为65字节。

```xaml
{Client:1:MessageID:1;Client发送时间:2020-04-11 13:51:18:723}
byte[] receiveBuffer = new byte[65];
```

再连续发送10条消息，下面为服务端测试的结果，结果显示正常：

```xaml
Server收到消息:{Client:1:MessageID:0;Client发送时间:2020-04-11 14:19:44:774};Server收到消息的时间:2020-04-11 14:19:44:778
Server收到消息:{Client:1:MessageID:1;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:781
Server收到消息:{Client:1:MessageID:2;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:781
Server收到消息:{Client:1:MessageID:3;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:782
Server收到消息:{Client:1:MessageID:4;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:782
Server收到消息:{Client:1:MessageID:5;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:782
Server收到消息:{Client:1:MessageID:6;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:783
Server收到消息:{Client:1:MessageID:7;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:783
Server收到消息:{Client:1:MessageID:8;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:784
Server收到消息:{Client:1:MessageID:9;Client发送时间:2020-04-11 14:19:44:776};Server收到消息的时间:2020-04-11 14:19:44:784

```

### 并发消息测试

如果并行发送消息，同时有两个消息到服务端，消息内容会混乱吗？客户端进行并行消息发送测试。下面为测试结果，发现并没有问题，说明一个消息可能没有被拆分，或则即使被拆分了在网络通讯底层也会恢复原来的消息结构。

```c#
Parallel.For(1, 10, (i) =&gt;
             {
                 var Replaystr =
                     $&#34;{{Client:1:MessageID:{i};Client发送时间:{DateTime.Now.ToString(&#34;yyyy-MM-dd HH:mm:ss:fff&#34;)}}}&#34;;
                 var strbytes = Encoding.UTF8.GetBytes(Replaystr);
                 socket.Send(strbytes, 0, strbytes.Length, SocketFlags.None);
             });
```

```xaml
Server收到消息:{Client:1:MessageID:2;Client发送时间:2020-04-11 17:11:44:568};Server收到消息的时间:2020-04-11 17:11:44:572
Server收到消息:{Client:1:MessageID:1;Client发送时间:2020-04-11 17:11:44:568};Server收到消息的时间:2020-04-11 17:11:44:575
Server收到消息:{Client:1:MessageID:4;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:576
Server收到消息:{Client:1:MessageID:5;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:576
Server收到消息:{Client:1:MessageID:6;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:576
Server收到消息:{Client:1:MessageID:7;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:576
Server收到消息:{Client:1:MessageID:8;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:577
Server收到消息:{Client:1:MessageID:9;Client发送时间:2020-04-11 17:11:44:572};Server收到消息的时间:2020-04-11 17:11:44:577
Server收到消息:{Client:1:MessageID:3;Client发送时间:2020-04-11 17:11:44:571};Server收到消息的时间:2020-04-11 17:11:44:577

```

### 并发客户端测试

再进一步测试，假设有多个客户端同时连接，并行发送消息。

```csharp
Parallel.For(0, 9, (Clienti) =&gt;
            {
                var socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
                var ip = IPAddress.Parse(m_ip);
                var endpoint = new IPEndPoint(ip, m_port);
                socket.ReceiveTimeout = -1;
                Task.Run(() =&gt;
                {
                    socket.Connect(endpoint);
                    ...
                    ...
                 }
             });
```

结果：这个测试是放在虚拟机中，使用的是NAT网络模式，同一个子网内客户端从发出消息接收服务端返回消息最长耗时有6秒，还是比较夸张的。

服务端结果：

![image-20240701215702363](/dotnet/image-20240701215702363.png)

客户端结果：

![image-20240701215716933](/dotnet/image-20240701215716933.png)

### 总结

这个Socket服务在少量客户端连接的时候好像没什么问题，它能抗住大量客户端的连接并发测试吗？我想答案肯定是否定的，为什么呢？因为每个客户端连接都需要消耗1个线程，线程是很昂贵的资源，每个线程自生就要消耗1M内存，100客户端连接什么都不做就消耗了100M，更不用说线程之间的上下文切换需要消耗更宝贵的CPU资源，所以这个服务端根本应对不了大量客户端的连接。

那么最理想的Socket服务端是什么样子的呢？在我看来就是只有与CPU核数相同的线程量在运行，如果4核那么就4个线程在运行，然后每个线程处理超级多的客户端，最好没有阻塞，不休息。怎样才能实现这个目标呢？微软给了一个简单的例子，已经极大程度的实现了这个想法，一起来看看吧

## SocketAsyncEventArgs

[MSDN中SocketAsyncEventArgs](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socketasynceventargs?view=netframework-4.8)

### 代码实现

我仿照微软提供的这个实例撸了个简单的Socket服务端

```c#
    public class SocketArgsServer
    {
        private static int m_numConnections;
        private static int m_receiveBufferSize;
        private static int m_sendBufferSize;
        private static byte[] m_receivebuffer;
        private static Stack&lt;int&gt; m_freeReceiveIndexPool;
        private static int m_currentReceiveIndex;
        private static byte[] m_sendbuffer;
        private static Stack&lt;int&gt; m_freeSendIndexPool;
        private static int m_currentSendIndex;
        private static Stack&lt;SocketAsyncEventArgs&gt; m_ReadPool;
        private static Stack&lt;SocketAsyncEventArgs&gt; m_WritePool;
        private static Semaphore m_maxNumberAcceptedClients;
        private static int m_numConnectedSockets;
        private static int m_totalBytesRead;
        private static Socket listenSocket;
        public static void Run(string m_ip, int m_port, int numConnections, int m_receiveBuffer, int m_sentBuffer)
        {
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

            listenSocket = new Socket(new IPEndPoint(IPAddress.Parse(m_ip), m_port).AddressFamily, SocketType.Stream, ProtocolType.Tcp);
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

        public static void ProcessAccept(SocketAsyncEventArgs e)
        {
            Interlocked.Increment(ref m_numConnectedSockets);
            SocketAsyncEventArgs readEventArgs = m_ReadPool.Pop();
            SocketAsyncEventArgs writeEventArgs = m_WritePool.Pop();
            ((AsyncUserToken)readEventArgs.UserToken).Socket = e.AcceptSocket;
            ((AsyncUserToken)readEventArgs.UserToken).readEventArgs = readEventArgs;
            ((AsyncUserToken)readEventArgs.UserToken).writeEventArgs = writeEventArgs;

            ((AsyncUserToken)writeEventArgs.UserToken).Socket = e.AcceptSocket;
            ((AsyncUserToken)writeEventArgs.UserToken).readEventArgs = readEventArgs;
            ((AsyncUserToken)writeEventArgs.UserToken).writeEventArgs = writeEventArgs;
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
                listenEventArg = new SocketAsyncEventArgs();
                listenEventArg.Completed &#43;= new EventHandler&lt;SocketAsyncEventArgs&gt;((sender, e) =&gt; ProcessAccept(e));
            }
            else
            {
                listenEventArg.AcceptSocket = null;
            }

            m_maxNumberAcceptedClients.WaitOne();
            bool willRaiseEvent = listenSocket.AcceptAsync(listenEventArg);

            if (!willRaiseEvent)
            {
                ProcessAccept(listenEventArg);
            }
        }

    }
    class AsyncUserToken
    {
 
        public Socket Socket { get; set; }

        public SocketAsyncEventArgs readEventArgs { set; get; }

        public SocketAsyncEventArgs writeEventArgs { set; get; }

    }
```

### 并发测试

先直接上测试结果，该测试环境还是在虚拟机中，忽略一下服务端收到消息时间，因为虚拟机时间和主机时间不是同步的。可以看到服务端发送消息到接收到消息最长耗时2s。

![image-20240701215736677](/dotnet/image-20240701215736677.png)

### 总结

这个Socket服务端直接丢弃了线程的概念，通过SocketAsyncEventArgs来实现了之前线程实现的所有功能。一个SocketAsyncEventArgs来监测连接，客户端连接的时候从SocketAsyncEventArgsPool中分配两个SocketAsyncEventArgs分别负责读写消息。读写消息的缓存块也进行了统一管理，共同组成一个大的缓存块进行重复使用。当客户端失去连接的时候将分配的读写SocketAsyncEventArgs返还给SocketAsyncEventArgsPool进行重复使用。

## 最后

在本文中探索了两种socket服务端的实现，并对这两种socket服务端进行了简单的剖析，我看了SuperSocket的底层实现思想采用的是第二种方式。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/04/dotnetbase9-socket/  

