# TouchSocket 插件


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

TouchSocket提供插件方式的扩展，这种方式对原框架的耦合较小。用插件基类封装了底层框架中所有的插件扩展接口，插件子类重写对应的接口就可注入相关业务。框架插件的注入和调用通过PluginsManager进行管理。

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index

## 插件架构

插件接口在框架中的位置可以在[TouchSocket 封装和扩展](https://dduuuu.github.io/2022/09/touchsocket5-encapsulationextension/)查看，插件基类封装了所有接口。

下图为插件基类。

![image-20220919085942919](/touchsocket/image-20220919085942919.png)

插件管理负责插件的注入和调用，通过一些扩展方法封装各种插件注入方式。

下图为插件管理：

![image-20220919090045496](/touchsocket/image-20220919090045496.png)

**插件的注入方式比较特别，通过反射方式找到类中所有插件接口并构造PluginMethod封装方法，该封装主要封装了异步方法，调用时会等待异步结果再进行返回。**

## 重连插件

当客户端连接断开时，提供自动重连。

通过扩展方法提供注入接口:

```C#
/// &lt;summary&gt;
/// 使用断线重连。
/// &lt;para&gt;该效果仅客户端在完成首次连接，且为被动断开时有效。&lt;/para&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;pluginsManager&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;successCallback&#34;&gt;成功回调函数&lt;/param&gt;
/// &lt;param name=&#34;tryCount&#34;&gt;尝试重连次数，设为-1时则永远尝试连接&lt;/param&gt;
/// &lt;param name=&#34;printLog&#34;&gt;是否输出日志。&lt;/param&gt;
/// &lt;param name=&#34;sleepTime&#34;&gt;失败时，停留时间&lt;/param&gt;
/// &lt;returns&gt;&lt;/returns&gt;
public static IPluginsManager UseReconnection(this IPluginsManager pluginsManager, int tryCount = 10, 
	bool printLog = false, int sleepTime = 1000, Action&lt;ITcpClient&gt; successCallback = null)
{
	var reconnectionPlugin = new ReconnectionPlugin&lt;ITcpClient&gt;(client=&gt; 
	{
		int tryT = tryCount;
		while (tryCount &lt; 0 || tryT-- &gt; 0)
		{
			try
			{
				if (client.Online)
				{
					return true;
				}
				else
				{
					client.Connect();
				}

				successCallback?.Invoke(client);
				return true;
			}
			catch (Exception ex)
			{
				if (printLog)
				{
					client.Logger.Debug(LogType.Error, client, &#34;断线重连失败。&#34;, ex);
				}
				Thread.Sleep(sleepTime);
			}
		}
		return true;
	});

	pluginsManager.Add(reconnectionPlugin);
	return pluginsManager;
}
```

ReconnectionPlugin继承TcpPluginBase，并重写OnDisconnected方法。通过Task.Run异步执行。

```C#
/// &lt;summary&gt;
/// &lt;inheritdoc/&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;client&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;e&#34;&gt;&lt;/param&gt;
protected override void OnDisconnected(ITcpClientBase client, ClientDisconnectedEventArgs e)
{
	base.OnDisconnected(client, e);

	if (client is ITcpClient tcpClient)
	{
		if (e.Manual)
		{
			return;
		}
		Task.Run(() =&gt;
		{
			while (true)
			{
				try
				{
					if (this.m_tryCon.Invoke((TClient)tcpClient))
					{
						break;
					}
				}
				catch
				{
				   
				}
			}
		});
	}
}
```

框架调用：

![image-20220919091214688](/touchsocket/image-20220919091214688.png)

通过PluginsManager调用插件接口，注意如果e.Handled在重写的方法里置为true，将不会调用Client.DisConnected和Client.OnDisConnected。

```c#
private void PrivateOnDisconnected(ClientDisconnectedEventArgs e)
{
	if (this.m_usePlugin)
	{
		this.PluginsManager.Raise&lt;ITcpPlugin&gt;(&#34;OnDisconnected&#34;, this, e);
		if (e.Handled)
		{
			return;
		}
	}
	this.OnDisconnected(e);
}
```

## 扩展插件横切面接口

HTTPPlugin插件扩展了HTTP协议的相关接口

![image-20220919092656409](/touchsocket/image-20220919092656409.png)

在HttpSocketClient中重写HandleReceivedData方法调用插件横切面。

```C#
/// &lt;summary&gt;
/// &lt;inheritdoc/&gt;
/// &lt;/summary&gt;
/// &lt;param name=&#34;byteBlock&#34;&gt;&lt;/param&gt;
/// &lt;param name=&#34;requestInfo&#34;&gt;&lt;/param&gt;
protected override void HandleReceivedData(ByteBlock byteBlock, IRequestInfo requestInfo)
{
    if (requestInfo is HttpRequest request)
    {
        this.OnReceivedHttpRequest(request);
    }
}

/// &lt;summary&gt;
/// 当收到到Http请求时。覆盖父类方法将不会触发插件。
/// &lt;/summary&gt;
protected virtual void OnReceivedHttpRequest(HttpRequest request)
{
    HttpContextEventArgs args = new HttpContextEventArgs(new HttpContext(request));

    switch (request.Method)
    {
        case TouchSocketHttpUtility.Get:
            {
                this.PluginsManager.Raise&lt;IHttpPlugin&gt;(&#34;OnGet&#34;, this, args);
                break;
            }
        case TouchSocketHttpUtility.Post:
            {
                this.PluginsManager.Raise&lt;IHttpPlugin&gt;(&#34;OnPost&#34;, this, args);
                break;
            }
        case TouchSocketHttpUtility.Put:
            {
                this.PluginsManager.Raise&lt;IHttpPlugin&gt;(&#34;OnPut&#34;, this, args);
                break;
            }
        case TouchSocketHttpUtility.Delete:
            {
                this.PluginsManager.Raise&lt;IHttpPlugin&gt;(&#34;OnDelete&#34;, this, args);
                break;
            }
        default:
            this.PluginsManager.Raise&lt;IHttpPlugin&gt;(&#34;OnReceivedOtherHttpRequest&#34;, this, args);
            break;
    }
}
```

## 总结

介绍了TouchSocket插件扩展，包括插件架构，利用重写默认的TcpPluginBase的方法实现业务注入；扩展插件横切面接口，这需要重写ClientBase的相关方法实现。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket7-plugs/  

