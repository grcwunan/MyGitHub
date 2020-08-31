# <center>Netty核心源码剖析<center>

TCP/IP->原生JDK->NIO->Netty

异步、事件驱动

BIO、NIO、AIO

BIO流程：

1. 服务器端启动一个ServerSocket；
2. 客户端启动Socket对服务器进行通讯；
3. 客户端发出请求后，等待返回；

NIO三大组件：通道（Channel）、缓冲区（Buffer）、选择器（Selector）

BIO是针对流处理的，NIO是针对块处理的；

![](C:\Users\Administrator\Desktop\临时文件\NIO图示.png)

Buffer底层即是一个数组；

每个Channel都对接一个Buffer；

常用Buffer子类：7种（除了基础类型中的boolean）；

Buffer类的四个属性：capacity、position、limit、mark

![](C:\Users\Administrator\Desktop\临时文件\flip产生的影响.jpg)

Channel可以同时进行读写；常用通道：FileChannel、DatagramChannel、ServerSocketChannel（ServerSocketChannelImpl）、SocketChannel（SocketChannelImpl）

NIO与零拷贝（transforTo方法）

AIO(Proactor模式)

对比：

![](C:\Users\Administrator\Desktop\临时文件\image\BIO和NIO对比.jpg)

#### 有了NIO为什么还需要Netty？

![](C:\Users\Administrator\Desktop\临时文件\image\netty简述.jpg)

#### 线程模型

传统阻塞IO模型

Reactor模式：

- 单Reactor单线程
- 单Reactor多线程
- 主从Reactor多线程

Netty模型：









