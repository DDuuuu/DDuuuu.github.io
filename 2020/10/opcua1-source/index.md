# OPCUA 源码解析


# 前言

一毕业第一个项目就做了OPC数据转发，面向API编程，调用固定接口，定时器轮询从OPC DA2.0 Server中获取数据写到数据库，定时器5分钟写一次，数据量比较大，有几千个点，当时是数据库有一个主表，通过主表的触发器，会为每个点生成一个子表，数据都往子表里写。可以说非常low了。有时候数据库和server需要走Network,DA是需要配置COM的，非常的麻烦。就开始研究OPCUA,也是面向API编程，调包程序员。再后来项目不需要就不研究了。在Github上找过其源码但是没有看过觉得真是一大遗憾，有一种只问其声，未见其人的感觉，非常的不爽。最近要写一个组件用到OPCUA，所以仔细看了它的源码。OPC的网上资料很少，有一本书写的太过理论，学组件还是直接看源码，源码看懂了再看概念理论就会非常清晰了。

# 底层通讯

ApplicationInstance:该类是服务的包装类，提供了加载参数，启动服务的方法。相当于服务入口。

Server:最重要的服务类：

![1603974747879](/opcua/1603974747879.png)

Server的接口非常多，但是扩展却非常容易，自己扩展只需要继承StandardServer就可以了。它是怎么实现各种订阅，浏览，事件，方法等功能的呢，这里面最重要的是用了一个Manager的思想。关注ServerInternal属性，其是ServerInternalData类，这个类非常像外观模式，包含了很多Manager。每个Manager是相当于一个组件管理器。比如NodeManager，所有的节点管理都在这个Manager中，还有SubscriptionManager,SessionManager,EventManager等。服务除了Manager，还有Certificate，OPCUA的安全这一块做的真好，TransportListeners管理所有的Host。

![1603975341434](/opcua/1603975341434.png)

Server中TransportListeners，底层通讯。

OPCUA默认实现了三种Host:UaHttpsChannelListener,UaTcpChannelListener,NullListener,这边用到了空对象模式，从名字就可以看出来Listener的职责就说监听连接，TCP监听实现是通过完成端口模型。

![1603976653100](/opcua/1603976653100.png)

![1604057776912](/opcua/1604057776912.png)

在OnAccept中：一旦有链接，就创建Channel来处理数据请求

![1604057874649](/opcua/1604057874649.png)



![1604057634344](/opcua/1604057634344.png)

channel内部对Socket完成端口模型进行了封装，TcpMessageSocket

![1604058221546](/opcua/1604058221546.png)

在TcpMessageSocket内部：

![1604058282130](/opcua/1604058282130.png)

接受数据完成端口模型，最终调用了Channel的OnMessageReceived方法

![1604058308648](/opcua/1604058308648.png)

![1604058510245](/opcua/1604058510245.png)

在Channel中处理消息，在ProcessRequestMessage方法内部调用了m_RequestReceived回调函数，其实是调用了Listener中的OnRequestReceived方法，该方法实际上是将请求给了专门ReceiveDataHandle类：SessionEndpoint类来处理请求，这个类其实对请求数据的一种管理，所有的请求都给了SessionEndpoint中BeginProcessRequest方法，这个SessionEndpoint管理器会创建一些具体解析请求的类，来处理请求ProcessRequestAsyncResult，

![1604058741455](/opcua/1604058741455.png)

![1604059150058](/opcua/1604059150058.png)

![1604059169202](/opcua/1604059169202.png)

![1604059187168](/opcua/1604059187168.png)

![1604059880031](/opcua/1604059880031.png)

在具体处理数据内部肯定是调用管理器类SessionEndpoint来具体请求数据，这边将找到的Service保存到m_Service中，只不过这边又做了一个请求处理队列，ServerForContext就是Server，server最后还是调用这个类的CallSynchronously方法，然后调用m_service的 

![1604060145717](/opcua/1604060145717.png)

![1604060447298](/opcua/1604060447298.png)

![1604060501554](/opcua/1604060501554.png)

ProcessRequestAsyncResult调用m_service

![1604061001059](/opcua/1604061001059.png)

这个m_service就是一开始在SessionEndpoint管理的服务，这些服务最后都指向Server中的一个方法。

比如Browse服务就是Server中的Browse方法。

![1604061123338](/opcua/1604061123338.png)

# 通讯框架总结

1. Server管理所有的Listener集合
2. Listener负责监听
3. Channel负责处理链接，解析消息结构
4. TcpMessageSocket负责封装Socket来收发消息
5. SessionEndpoint封装所有对消息体处理的服务，这些服务其实都是Server中方法委托。
6. TcpMessageSocket接受到消息最终调用SessionEndpoint中的BeginProcessRequest方法，然后生成了消息处理类ProcessRequestAsyncResult，该类将请求给了server中的RequestQueue,Server处理队列最终是调用ProcessRequestAsyncResult中的方法找到SessionEndpoint中对应的处理服务，调用server中对应的方法。
7. Server负责创建SessionEndpoint实例，Listen负责调用SessionEndpoint实例方法解析数据，channel负责解析数据类型调用Listen方法。

# 协议格式

在Channel的OnMessageReceived方法中，协议头中前4个字节是消息的类型：也就是消息ID，详情可以看TcpMessageType类

![1604109961342](/opcua/1604109961342.png)

![1604110030696](/opcua/1604110030696.png)

channel中的ReadSymmetricMessage方法，可以消息头：一共24个字节

MessageType;MessageSize;channelID;tokenId;sequenceNumber;requestId都是4个字节

![1604110800288](/opcua/1604110800288.png)channel解密TokenID

![1604111015281](/opcua/1604111015281.png)

通过Token解密消息，还可以看出消息的最后字节是Signature。

还是在Channel的ReadSymmetricMessage 方法中

![1604111191527](/opcua/1604111191527.png)

消息的最后还有一些没有用的填充消息，在签名的前一个字节为填充长度

![1604112388066](/opcua/1604112388066.png)

协议内容：

MessageType;MessageSize;channelID;tokenId;sequenceNumber;requestId;MessageBogy;Padding;Signature.

![1604111829721](/opcua/1604111829721.png)

总结

整个协议的格式都是在Channel的ReadSymmetricMessage方法中解析的，主要解析了协议头，签名和获取MessageBody并进行解密。

# 消息体

在channel的ProcessRequestMessage方法中看到对消息体的解析，将MessageBody解析成ServiceRequest

![1604113227288](/opcua/1604113227288.png)

整个解析职责是交给了BinaryDecoder类,Decoder实际是调用IEncodeable来进行解码，这边有点像迭代器模式，集合实现IEnumerable,而调用者使用IEnumator进行遍历。

![1604114314385](/opcua/1604114314385.png)

![1604115066158](/opcua/1604115066158.png)

MessageBody:NodeId&#43;NodeValue&#43;IEncodeable。，通过NodeId获取实际是IEncodeableType,调用其Decode方法，获取IEncodeable，然后显示转化为IServiceRequest接口

![1604113359651](/opcua/1604113359651.png)

![1604113459405](/opcua/1604113459405.png)

通过ID来获取解码类，进行解码：

解码类从哪里来呢？通过工厂，工厂通过反射获取程序集中所有IEncodeable

![1604113802227](/opcua/1604113802227.png)

![1604113822171](/opcua/1604113822171.png)

![1604113846559](/opcua/1604113846559.png)

![1604114209704](/opcua/1604114209704.png)

![1604116408322](/opcua/1604116408322.png)

![1604116611351](/opcua/1604116611351.png)

![1604116673012](/opcua/1604116673012.png)

至此解析出具体请求类进行回调，也就是将请求类交给SessionEndpoint进行处理。

总结：Channel调用Decoder，这边Decoder负责解析MessageBody,具体的解析操作职责给IEncodeable的子类。解析完后将IEncodeable显示转化为IServiceRequest交给SessionEndpoint处理。

# 感悟

软件中的架构大体和现实世界差不多，如果一个需求有多个不同的操作，就将其抽象为子类，一个管理器Manager或者XXXer负责管理所有的子类集合，子类可以由专门的工厂类创建也可以是由管理器创建。

现实生活中一个公司就相当于一个软件，有一个总经理（Master/Server/Instance）总经理负责协调各个部门，每个部门的部长就相当于(Manager/Presenter/Controller...)，它负责安排这个部门的职责，也就是管理部门里面的员工集合，员工就是具体干活的类，这些类可以由基类派生或实现统一的接口。派生类如何生成可以由部长负责，也可以由部长请求公司的HR相当于工厂生成派生类。工厂可以是抽象工厂也可以工厂模式，为每个部门生成一个工厂子类。

以OPCUA的通讯为例：

Server就相当于总经理，通讯组件(销售)就相当于一个通讯部门这边部长和总经理和二为一，可能这个部门非常重要，总经理想亲自管理，Listeners就相当于具体干活的员工，每个Listener(销售)负责一个通讯线的对接，每当有通讯连接就派Channel(具体负责哪个厂)去负责对接内容，Channel说对接内容还需要一个人去专门的地方去就派了MessageSocket通过完成端口模型去取通讯消息，MessageSocket每次取到消息就把消息给Channel解析，Channel先解析这是上面类型的消息，将其具体解析为一个请求文件，这个请求文件都汇总给了由Listener指派的SessionEndpoint这个人（有点像总经理秘书），这个人将负责管理所有的请求文件种类。它收到请求文件其实就把请求给了总经理，总经理根据不同的请求将请求转发给其他部门的人干。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/10/opcua1-source/  

