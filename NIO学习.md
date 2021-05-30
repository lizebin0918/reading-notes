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

## IO模型（同步阻塞、异步非阻塞）
* 同步：正常调用
* 异步：基于回调
* 阻塞：没开线程
* 非阻塞：开了线程轮询（NIO-）
* BIO(同步阻塞IO):传统IO-一个进一个出，每个线程开一个socket依然是BIO模型（只是提高了吞吐量，多线程切换、新建、销毁都是耗费资源的）
* NIO(同步非阻塞IO):多路复用，一条路可以进出，并且加入了缓冲区，不用再阻塞，线程池实现异步（Selector类似叫号员）
* AIO(异步非阻塞):事件驱动，回调

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
* flip():写模式切换成读模式，position 恒小于或等于 limit
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

## Selector
* Selector.Key:监听事件
* `selector.keys()`:返回所有事件（连接、可读、可写、断开）
* `selector.selectedKeys()`:返回指定事件（keys的子集）
* `selectionKeys.iterator().next().cancel()`:返回从指定事件取消的事件（keys的子集）
* 
* 轮询所有socket，把可写可读的socket返回给Channel
* 监听ACCEPT事件，再转成WRITE事件，这样Selector的性能及其低下



