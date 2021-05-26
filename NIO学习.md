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
* 非阻塞：开了线程轮询
    

