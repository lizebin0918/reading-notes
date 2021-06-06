# NIO学习

## java.io/java.nio
* java.io:面向流的编程
* java.nio:Selector/Channel/Buffer，面向块（block）或是缓冲期（buffer）进行编程
    * 一个Thread对应一个Selector，一个Selector对应多个Channel，每个Channel必须有Buffer（byte数组），Selector可以在多个Channel之间来回切换
    * 数据从Channel读到Buffer，程序再从Buffer读取数据，这个Buffer支持读写双向的

* Buffer
    * 对应8中的原生类型
* Channel
    * 可以向其写入或者读取数据，但是读写数据都是通过Buffer进行的

## IO模型
* 最终是解决两个问题，一共4步
    * 等待数据准备 (Waiting for the data to be ready)
    * 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)
* 阻塞：read/write是否阻塞
* 同步：线程判断是否可读可写
* 异步：无需判断是否可读可写，直接由对方回调告知
* BIO(同步阻塞IO)
    * 传统IO-一个进一个出，每个线程开一个socket依然是BIO模型（只是提高了吞吐量，多线程切换、新建、销毁都是耗费资源的）
    * 当用户进程调用了recvfrom这个系统调用，内核就开始了IO的第一个阶段：等待数据准备。对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候内核就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当内核一直等到数据准备好了，它就会将数据从内核中拷贝到用户内存，然后内核返回结果，用户进程才解除block的状态，重新运行起来。所以，blocking IO的特点就是在IO执行的两个阶段都被block了。
* NIO(同步非阻塞IO)
    * 当用户进程调用recvfrom时，系统不会阻塞用户进程，而是立刻返回一个ewouldblock错误，从用户进程角度讲 ，并不需要等待，而是马上就得到了一个结果。用户进程判断标志是ewouldblock时，就知道数据还没准备好，于是它就可以去做其他的事了，于是它可以再次发送recvfrom，一旦内核中的数据准备好了。并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。当一个应用程序在一个循环里对一个非阻塞调用recvfrom，我们称为轮询。应用程序不断轮询内核，看看是否已经准备好了某些操作。这通常是浪费CPU时间，但这种模式偶尔会遇到。
* I/O复用（I/O Multiplexing)
    * select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就**通知**用户进程
    * select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接
    * 整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。
    * 在I/O复用模型中，会用到select，这个函数也会使进程阻塞，但是和阻塞I/O所不同的的，这两个函数可以同时阻塞多个I/O操作，而且可以同时对多个读操作，多个写操作的I/O函数进行检测，直到有数据可读或可写时，才真正调用I/O操作函数
    * **Netty的非阻塞I/O的实现关键是基于I/O复用模型，这里用Selector对象表示**
* 信号驱动的I/O (Signal Driven I/O)
    * 用户进程不是阻塞的。首先用户进程建立SIGIO信号处理程序，并通过系统调用sigaction执行一个信号处理函数，这时用户进程便可以做其他的事了，一旦数据准备好，系统便为该进程生成一个SIGIO信号，去通知它数据已经准备好了，于是用户进程便调用recvfrom把数据从内核拷贝出来，并返回结果。
* AIO(异步非阻塞)
    * 事件驱动，直接把**业务逻辑**传到内核
    * 这个模型和前面的信号驱动I/O模型的主要区别是，在信号驱动的I/O中，内核告诉我们何时可以启动I/O操作，但是异步I/O时，内核告诉我们何时I/O操作完成。当用户进程向内核发起某个操作后，会立刻得到返回，并把所有的任务都交给内核去完成（包括将数据从内核拷贝到用户自己的缓冲区），内核完成之后，只需返回一个信号告诉用户进程已经完成就可以了。

| 阻塞 | 非阻塞 |
| --- | --- |
| 阻塞式I/O模型（同步）:单线程read/wirte阻塞，阻塞的是 Socket IO | 非阻塞式I/O模型（同步）：read/write立即返回（通过轮询判断Socket是否有值可读或者可写） |
| I/O多路复用模型（同步）：用一个线程来检查多个文件描述符（Socket）的就绪状态，阻塞的是select遍历Socket事件 | 信号驱动I/O模型（异步）：直接传递Handler，内核返回执行即可 |
|  | 异步IO模型（异步） |

## 目前流行的多路复用IO模型

| IO模型 | 相对性能 | 关键思路 | 操作系统 | Java支持 |
| --- | --- | --- | --- | --- |
| select | 较高 | Reactor | windows/linux | 支持，Reactor模式。linux操作系统的kernels2.4内核版本之前，默认使用select；而目前window下对同步IO的支持，都是select模型 |
| poll | 较高 | Reactor | linux | linux下的Java Nio框架，linux kernels 2.6 内核版本之前使用poll进行支持，也是使用Reactor模式 |
| epoll | 高 | Reactor/Proactor | linux | linux kernels 2.6内核版本及以后使用epoll进行支持；2.6之前使用poll |
| kqueue | 高 | Proactor | linux | Java暂不支持 |

## ByteBuffer（数组实现的回环队列，只是limit不能动而已....）

> 0<=mark<=position<=limit<=capacity

* mark 是一個被标记的位置，调用 mark() 時, 會把 position 的值設定給 mark, 调用 reset() 方法時, 會把 mark 的值设置成 position
* capacity:容量（不可变），**比如长度=6，capacity=6（指向最后一个预留位）**，用来区分“空”和“满”的情况。数组实现的回环队列同理）
* limit:最后一个可读写的下一位索引（也是第一个不可读写的索引）
* position:第一个（将要）读或者写的索引
* flip():读写模式互换，position 恒小于或等于 limit
    * 写->读：position指向第一个可读的位置，limit指向可读的最后一位的下一位
    * 读->写：position指向第一个可写的位置，limit指向可写的最后一位的下一位
    
    ```java
        /*
         * buf.put(magic);// Prepend header-写头部
         * in.read(buf);// Read data into rest of buffer-读取data写到buffer
         * buf.flip();// Flip buffer-写->读
         * out.write(buf);// Write header + data to channel-读取buffer写到out
        */
        //等价于
        buffer.limit(buffer.position()).position(0);
    ```

* clear():position=0，limit=capacity
* compact():比如一共四个字节，只读了两个，调用compact()继续写，把前两个元素往前挪，把position=3，，limit=capacity
* remaining():limit-position-表示可读或者可写的大小
* 实现重复读，比如position=2，调用mark()，postion继续往前走，调用reset()，position返回上次mark的位置
    * mark():mark = position
    * reset():position = mark
* 相对方法：limit与position相互配合作用读取
* 绝对方法：完全忽略limit与position，通过索引下标读取

## DirectByteBuffer extends MappedByteBuffer
* 堆外内存，实现“零”拷贝,**属于JVM进程内存**
* 发生默认的垃圾回收器和G1会导致对象压缩，在内存空间移动，这样会导致io读取数据有误，所以有了对外内存，不被垃圾回收
* 用完就会被回收掉
* [](https://www.bilibili.com/video/BV1Bp4y1B7zc?from=search&seid=3312293717851805791)
* 内核内存映射：`MappedByteBuffer`

## Selector(总部调度者，所有Channal都要注册到上面)
* `SelectionKey`:监听事件，并返回对应监听的Channel（ServerSocketChannal.accept()，SocketChannel.read()/write()）
* `selector.keys()`:返回所有事件（连接、可读、可写、断开）
* `selector.selectedKeys()`:返回指定事件（keys的子集）
* `selectionKeys.iterator().next().cancel()`:返回从指定事件取消的事件（keys的子集）
* 轮询所有socket，把可写可读的socket返回给Channel
* 监听ACCEPT事件，再转成WRITE事件，这样Selector的性能及其低下

## 编码
* ASCII(American Standard Code for Information Interchange, 美国信息交换标准代码)
	* 7bit表示一个字符，共计128种字符
* IOS-8859-1  
	* 8bit表示一个字符，一个字节来表示一个字符，可以表示256个字符
* GB2312
	* 两个字节表示一个汉字
* GBK(包括生僻字)
* GB18030(包含所有汉字)
* big5(针对繁体中文)
* unicode(全世界的所有字符)：采用两个字节表示一个字符-\u(十六进制)，存储太大
* UTF-8/UTF-16(unicode translation format):unicode 是编码方式，UTF则是一种存储方式，UTF-8是Unicode的实现方式之一
	* 变长字节表示形式
	* BOM(Byte Order Mark):字节序标记

## Netty核心流程
* Server端

```java
//定义事件轮询线程组
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();

ServerBootstrap server = new ServerBootstrap();
server.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class);
//定义初始化器
server.childHandler(new ChannelInitializer<SocketChannel>() {
  @Override
  protected void initChannel(SocketChannel socketChannel) throws Exception {
      //所有业务逻辑处理全部归总到了一个队列中
      //这个队列中包含了各种各样的处理逻辑，对这些处理逻辑在Netty中有一个封装
      //封装成一个对象，无锁化串行任务队列
      ChannelPipeline p = socketChannel.pipeline();

      //添加编解码
      //p.addLast();

      //执行业务逻辑:XXXXXXHandler extends ChannelInboundHandlerAdapter
      //p.addLast(XXXXXXHandler);
  }
});

//绑定端口
ChannelFuture sync = server.bind(8080).sync();
sync.channel().closeFuture().sync();
``` 

 * Client 端

```java
//发起网络请求
EventLoopGroup worker = new NioEventLoopGroup();
Bootstrap client = new Bootstrap();
client.group(worker).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true);
client.handler(new ChannelInitializer<SocketChannel>() {
@Override
protected void initChannel(SocketChannel socketChannel) throws Exception {
   ChannelPipeline p = socketChannel.pipeline();

   //还原 InvokerProtocol
   //对于自定义协议，需要对内容编解码
   p.addLast(new LengthFieldBasedFrameDecoder(Integer.MAX_VALUE,
                                              0, 4, 0, 4));
   //自定义编码
  
   //执行业务逻辑:XXXXXXXHandler extends ChannelInboundHandlerAdapter
   p.addLast(XXXXXXXHandler);
}
});
ChannelFuture future = client.connect("localhost", 8080).sync();
future.channel().writeAndFlush(msg).sync();
future.channel().closeFuture().sync();
return XXXXXXXHandler.getResult();
``` 
	


