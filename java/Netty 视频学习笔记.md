# Netty 视频学习笔记

* 框架定义:Netty is an asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.（`Netty`是基于异步事件驱动的框架，用于可维护、高性能协议的客户端或者服务端的快速开发）
* 名词解析
    * `BIO`:Blocking-IO 同步阻塞IO
    * `AIO`:Asynchronous-IO 异步阻塞IO
    * `NIO`:non-Blocking-IO 同步非阻塞IO
* `https://netty.io/`解读
    * Highly customizable thread model - single thread, one or more thread pools such as SEDA--高度定制化的线程模型，包括单线程或者线程池，SEDA(**Staged Event Driven Architecture--阶段性事件驱动架构，主要理念是把一个请求处理过程，分成若干个stage，由不同线程处理，stage之间是通过event来驱动**)
    * True connectionless datagram socket support (since 3.1)--支持无连接的socket报文传输(实际就是UDP) 
    * Better throughput, lower latency--搞吞吐，低延迟
    * `RTSP`协议:Real Time Streaming Protocol 流媒体播放，用于视频直播
    * `RPC`:调用逻辑及原理，自己去查--protocol buffer/thrift

* `Netty`充当HTTP服务器，示例代码:
    * 在命令行执行:`curl 'http://localhost:8899'`
    * `HttpServerHandler`(自定义处理器)会处理两个请求
    
    > DefaultHttpRequest(decodeResult: success, version: HTTP/1.1)
    > GET / HTTP/1.1
    > Host: localhost:8899
    > User-Agent: curl/7.54.0
    > Accept: */* : 
    > {"decoderResult":     {"failure":false,"finished":true,"success":true},"method":{},"protocolVersion":{"keepAliveDefault":true},"uri":"/"}
    > ------------分割线----------------
    > EmptyLastHttpContent : {"decoderResult":{"failure":false,"finished":true,"success":true}}

* `SimpleChannelInboundHandler`源码分析
    * `curl 'http://localhost:8899'`执行流程:`handlerAdded --> channel registered --> channel active --> channelRead0 --> channelInactive --> channel unregistered`
    * 浏览器执行流程:`handlerAdded --> channel registered --> channel active --> channelRead0`，最后两步没有执行，因为我们设置了HTTP版本为1.1，服务器端会把连接缓存，只有`keepAlive`超时或者浏览器关闭，才会释放资源

* `bootstrap.group(bossGroup, workerGroup).channel(...).childHandler(null)/handler(null);`:`childHandler()`是用于`workGroup`的处理器，`handler()`用于`bossGroup`的处理器
* `Netty`构建通讯服务
    * `Server`端定义
    
    > 定义`EventLoopGroup`类型的父子线程组
    > 实例化引导对象--`ServerBootstrap serverBootstrap = new ServerBootstrap();`
    > `serverBootstrap`程序组装父子线程池，并监听服务端口，阻塞读取消息，最后定义消息处理器`ChannelHandler`，代码如下:

    ```java
    serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).childHandler(new ServerInitializer());
    ChannelFuture channelFuture = serverBootstrap.bind(8899).sync();
    channelFuture.channel().closeFuture().sync();
    ```
    > 定义`ChannelInitializer`在`initChannel`方法中消息处理`ChannelHandler`以及消息处理器可以监听各个事件，包括`channelRegistered`/`channelUnregistered`/`channelActive`/`channelInactive`/`channelRead0`
    
    * `Client`端定义
    
    > 定义`EventLoopGroup`类型的线程组（一个即可）
    > 定义客户端的引导实例--`Bootstrap bootstrap = new Bootstrap();`，连接对应端口
    > 定义`ChannelInitializer`在`initChannel`方法中添加消息处理`ChannelHandler`，原理同上

* `Netty`对于`RPC`支持
    
    * `RMI`(只能针对Java)以及所有`RPC`(跨语言)都是自动生成对应代码，`client`生成的代码叫`stub`，`server`生成的代码叫`skeleton`
    * 序列化与反序列化，实际叫做:编码与解码
    * 定义一个说明文件：描述了对象（结构体）、对象成员、接口方法等一系列信息
    * 通过RPC框架锁提供的编译器，将接口说明文件编译成对应编程语言的文件
    * 在客户端与服务端分别引入RPC编译器锁生成的文件，即可像调用本地方法一样调用远程方法

* `gRPC`学习
    * 官网:https://developers.google.com/protocol-buffers/
    * 定义:ProtoBuf是由Google开发的一种数据序列化协议（类似于XML、JSON、hessian）--一套接口描述语言。ProtoBuf能够将数据进行序列化，并广泛应用在数据存储、通信协议等方面
    * 定义`proto`文件，以及对应的`message`实体，实现序列化与反序列化例子
    * 生成代码命令:`protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto`
    * 利用`Netty`对RPC进行通信，主要声明`XXXInitializer`，**但是这种方式只能针对固定Message类型，因为RPC对外只是一个地址，而不像HTTP那样可以通过URL执行对应的逻辑，如何解决？**，解决方案是通过外层`Message`嵌套这各种具体业务的`Message`，并且通过枚举类型和`oneof`指定对应类型。具体代码如下
    
    ```protoc
    syntax = "proto2";
    package lesson14.protobuf;
    option optimize_for = SPEED;
    option java_package = "lesson14.protobuf";
    option java_outer_classname = "Sinoxk";
    message Message {
        enum DataType {
            PersonType = 1;
            DogType = 2;
            CatType = 3;
        }
        required DataType data_type = 1;
        oneof dataBody {
            Person person = 2;
            Dog dog = 3;
            Cat cat = 4;
        }
    }
    message Person {
        optional string name = 1;
        optional string age = 2;
        optional string address = 3;
    }
    message Dog {
        optional string name = 1;
        optional int32 age = 2;
    }
    message Cat {
        optional string name = 1;
        optional string city = 2;
    }
    ```
    
    * 对于公共模块化的代码可以采用`git`的`submodule`或者`subtree`进行管理，也可以采用maven打包成jar依赖的方式，但是后者会存在一个`deploy`jar和手动修改pom文件的操作

* `Thrift`学习
    
    * `list`:一系列由T类型的数据组成的有序列表，元素可以重复
    * `set`:一系列由T类型的数据组成的无需集合，元素不可重复
    * `map`:同理
    * 定义thrift的文件，由thrift文件(IDL)生成双方语言的接口、model,在生成的model以及接口中会有解码编码的代码
    * 结构体(struct):等价于JavaBean
    * 枚举(enum)
    * 异常(exception)
    * 服务(service):若干个方法的集合，等价于Java的Interface一样
    * 类型定义:`typedef i32 int`--表示用`int`表示整形
    * 常量(const)
    * 命名空间:等价于Java中的package意思:`namespace java com.XXX.XX`
    * 可选与必填:`required`/`optional`，分别用于表示字段
    * 作业：编写示例代码
    * IDL:Interface Description Language
    * RPC解决的问题：传输的效率问题/异构平台的调用
    * **Thrift:提供了服务端和客户端的载体**，但是protobuf只是一个消息定义，没有实现消息传输的载体

`GRPC 学习`:
    * GRPC:基于protocol buffer3(传统的protocol只是一种描述语言，定义了对应的规则，但是没有传输层)，可以生成服务端和客户端的传输载体代码
    * gradlew => gradle wrapper:使得使用者，在本地未安装gradle运行环境的情况下，依然可以通过一条命令实现项目构建。
    * 在实际开发中，当我们使用gradle的时候，项目管理者创建好项目，通过执行 gradle wrapper，就可以自动生成gradlew（实际就是gradle的快捷工具）。当其他开发者clone项目的时候，只要执行gradlew的时候，就会生成对应的gradle执行环境，本地再安装，因为版本实在太多，这样方便其他开发者的环境搭建
    * GRPC 是基于protobuf进行数据传输，通过proto文件生成相关的代码（Message文件），GRPC在此基础上增加了RPC service的功能，实现通信


