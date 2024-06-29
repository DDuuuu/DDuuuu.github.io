# TouchSocket 封装和扩展


## 概述

注意：该版本为TouchSocket 0.3.5,最新版已有较大改动。

Socket模型中需要考虑对各种方法的封装，需要考虑对各种业务场景的扩展，在实现业务流的同时，针对业务流的各个横切面做扩展，甚至业务流本身可以被替换。常用的扩展方式有下面几种，并按耦合从高到低的顺序：继承/泛型，接口/委托/事件，插件/扩展方法。

框架地址：https://github.com/RRQM/TouchSocket

文档地址：https://www.yuque.com/rrqm/touchsocket/index


## Socket

Socket模型如下图：

![image-20220913092159745](/touchsocket/image-20220913092159745.png)



TouchSocket封装。客户端所有接口封装成TCPClient；服务端将通讯部分桥接给SocketClient，外部接口封装成TCPServer，并通过泛型将SocketClient的类型传入；数据包封装成XXHandlingAdapter；参数设置TouchSocketConfig

![image-20220913095108670](/touchsocket/image-20220913095108670.png)

可扩展的横切面，横切面主要有：虚方法用于继承重写，事件委托用于订阅，插件扩展方法用于插件扩展。

服务端：

![image-20220914082746671](/touchsocket/image-20220914082746671.png)

客户端：

![image-20220913113515819](/touchsocket/image-20220913113515819.png)

## 总结

服务端封装成TCPServer,SocketClient；客户端封装成TCPClient，将通讯包封装成XXHandlingAdapter，参数设置封装TouchSocketConfig。

通过扩展方法/插件，事件/委托，虚方法提供横切面的扩展。





---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/09/touchsocket5-encapsulationextension/  

