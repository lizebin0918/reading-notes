#Java面试题（一）
来源：网上收集汇总

##**J2SE基础**

- 九种基本数据类型的大小，以及他们的封装类。

- Switch能否用string做参数？

- equals与==的区别。

- Object有哪些公用方法？

- Java的四种引用，强弱软虚，用到的场景。

- Hashcode的作用。

- ArrayList、LinkedList、Vector的区别。
> `ArrayList` 底层实现了`RandomAccess`接口，表示支持随机访问，推荐采用下标遍历。而且初始化的时候需要声明容量，防止数组拷贝。对于`add(i, element)`和`remove()`都会引起数组拷贝。
> `LinkedList`底层采用双向链表存储，对于`add(i, element)`和`remove()`等方法的执行效率较高，建议采用迭代器遍历而非下标方式。
> 以上两者共同的方法`add()`都是在末尾添加，如果`ArrayList`不发生数据拷贝，两者效率一样

- String、StringBuffer与StringBuilder的区别。

- Map、Set、List、Queue、Stack的特点与用法。

- HashMap和HashTable的区别。

- HashMap和ConcurrentHashMap的区别，HashMap的底层源码，解析ConcurrentHashMap的分段锁。

- TreeMap、HashMap、LindedHashMap的区别。<br/>
http://blog.csdn.net/ns_code/article/details/37867985

- Collection包结构，与Collections的区别。

- try catch finally，try里有return，finally还执行么？

> Excption与Error包结构。OOM你遇到过哪些情况，SOF你遇到过哪些情况。(SOF指StackOverFlowError-写一个无限递归，当线程栈的容量超过-Xss的大小就会抛出此异常)

- Java面向对象的三个特征与含义。

- Override和Overload的含义去区别。

>（这个题目还有其他问法：Java实现多态有哪些方式-重写（一个类的多态体现）与重载（多个类的多态体现））
>
>Overload:
>1.重写方法不能缩小访问权限
>2.参数列表必须与被重写方法相同（参数数量相同、类型相同、顺序相同）
>3.返回类型必须与被重写方法的相同或是其子类
>4.重写方法不能抛出新的异常，或者超出父类范围的异常。

- Interface与abstract类的区别。

- java多态的实现原理。（同38题）

- 实现多线程有哪几种方法？

> 线程同步的方法：sychronized、lock、reentrantLock等。

- 锁的等级

> 方法锁、对象锁、类锁。

- 写出生产者消费者模式。

- ThreadLocal的设计理念与作用。

- ThreadPool用法与优势。

>（可以把任务和执行任务分离，而且可以配置不同类型的线程池满足不同的情景，FixedPool会不断把处理不了的任务放到队列里面，CachedPool则不会放在任务队列，会一直创建线程来处理请求，有可能把服务器资源耗尽。如果你的应用是面向socket通信，而且是___异步双工长___连接，则适合采用FixedPool。）

- Concurrent包：ArrayBlockingQueue、ConcurrentLinkedQueue、CountDownLatch，以及应用场景。

- wait()和sleep()的区别。

- foreach与正常for循环效率对比。

- Java IO与NIO。

- 反射的作用及原理。

- 泛型常用特点，`List<String>`能否转为`List<Object>`。

- 解析XML的几种方式的原理与特点

> DOM、SAX、PULL(用于安卓)。

- 设计模式：单例、工厂、适配器、责任链、观察者等等。

- JNI的使用。

- `AtomicInteger`实现原理

> CAS(计算机的原语：compare and swap) + <font color=red>自旋</font>

> <font color=red>updated by lizb on 20151213
Atomic相关的类，并未用到锁，只是单纯的 CAS + 自旋。实际跟自旋锁没什么关系，自旋锁是用于线程访问同步块并获得锁才会用到。并不是用在此处</font>

- `synchronized`与`ReentrantLock`的区别

- 讲下java的锁原理

- 介绍`ConcurrentHashMap`原理，用的是哪种锁，segment有没可能增大，底层是怎么实现读共享，写互斥的？（不会，只会增大每个segment中的entry数组）

- `volatie`的作用（使变量对所有线程可见，同时禁止指令重排序，但是不能对变量加锁实现同步(可联想double-check的单例模式)）

- clone接口，深复制和浅复制分别怎么实现
浅复制：直接调用super.clone()
深克隆：采用`ObjectOutputStream`流和`ByteArrayOutputStream`进行

- 如何让线程A等待线程B结束后再执行（join，单线程池）

> updated on 20151105 by lizebin
模拟情景，A线程先执行，之后B开始指向，
之后等A要等待B执行完了，A再执行。
可采用Thread.join()和LockSupport类实现。
单线程池的实现方案暂未想到。

- 如何保证线程1、2、3的执行顺序（）

- 单例模式最优实现

- HashMap的底层实现是：数组 + 链表；但是如果Hash冲突严重，就会导致链表很长，效率很低。怎么解决？（通过比较jdk8的HashMap实现）


##**spring**

- 介绍spring的IOC和AOP分别如何实现
> classloader(本质就是applicationContext管理了classloader)和动态代理

- spring中bean加载机制，bean生成的具体步骤，IOC注入的方式

- 什么是控制反转(IOC)？什么是依赖注入？
> 控制反转是应用于软件工程领域中的，在运行时被装配器对象来绑定耦合对象的一种编程技巧，对象之间耦合关系在编译时通常是未知的。在传统的编程方式中，业务逻辑的流程是由应用程序中的早已被设定好关联关系的对象来决定的。在使用控制反转的情况下，业务逻辑的流程是由对象关系图来决定的，该对象关系图由装配器负责实例化，这种实现方式还可以将对象之间的关联关系的定义抽象化。而绑定的过程是通过“依赖注入”实现的。
控制反转是一种以给予应用程序中目标组件更多控制为目的设计范式，并在我们的实际工作中起到了有效的作用。
依赖注入是在编译阶段尚未知所需的功能是来自哪个的类的情况下，将其他对象所依赖的功能对象实例化的模式。这就需要一种机制用来激活相应的组件以提供特定的功能，所以依赖注入是控制反转的基础。否则如果在组件不受框架控制的情况下，框架又怎么知道要创建哪个组件？

- `BeanFactory`和ApplicationContext有什么区别？
> `BeanFactory` 可以理解为含有bean集合的工厂类。`BeanFactory` 包含了种bean的定义，以便在接收到客户端请求时将对应的bean实例化。
> BeanFactory还能在实例化对象的时生成协作类之间的关系。此举将bean自身与bean客户端的配置中解放出来。`BeanFactory`还包含了bean生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。
> 从表面上看，`ApplicationContext`如同`BeanFactory`一样具有bean定义、bean关联关系的设置，根据请求分发bean的功能，但是`AplicationContext`会在容器启动之后，并实例化单例的bean。而且`ApplicationContext`在此基础上还提供了其他的功能。
> 1.	提供了支持国际化的文本消息
> 2.	统一的资源文件读取方式
> 3.	已在监听器中注册的bean的事件

##**JVM**
- 内存模型以及分区，需要详细到每个区放什么。

> 内存模型：
> Java内存模型中规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存（可以与前面将的处理器的高速缓存类比），线程的工作内存中保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要在主内存来完成

- 堆里面的分区：Eden，survival from to，老年代，各自的特点。
	
> 年轻代：
> 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。年轻代分三个区。一个Eden区，两个Survivor区(一般而言)。大部分对象在Eden区中生成。当Eden区满时，还存活的对象将被复制到Survivor区（两个中的一个），当这个Survivor区满时，此区的存活对象将被复制到另外一个Survivor区，当这个Survivor去也满了的时候，从第一个Survivor区复制过来的并且此时还存活的对象，将被复制“年老区(Tenured)”。__需要注意，Survivor的两个区是对称的，没先后关系，所以同一个区中可能同时存在从Eden复制过来 对象，和从前一个Survivor复制过来的对象，而复制到年老区的只有从第一个Survivor去过来的对象。而且，Survivor区总有一个是空的。同时，根据程序需要，Survivor区是可以配置为多个的（多于两个），这样可以增加对象在年轻代中的存在时间，减少被放到年老代的可能。
> 年老代：
> 在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
> 持久代：
> 用于存放静态文件，如今Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。持久代大小通过`-XX:MaxPermSize=<N>`进行设置。

- 对象创建方法，对象的内存分配，对象的访问定位。

- GC的两种判定方法

> 引用计数与引用链（GC ROOTS 不可到达的对象将会被垃圾收集器回收）。

- GC的三种收集方法：标记清除、标记整理、复制算法的原理与特点，分别用在什么地方，如果让你优化收集方法，有什么思路？

- GC收集器有哪些？CMS收集器与G1收集器的特点。

- Minor GC与Full GC分别在什么时候发生？

> Parrel Scavenge GC
> 一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。
> Full GC
> 对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个对进行回收(FULL GC触发有可能是年老代或者持久代满了)，所以比Scavenge GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于FullGC的调节。
> 有如下原因可能导致Full GC：
	a.年老代（Tenured）被写满
	b.持久代（Perm）被写满
	c.System.gc()被显示调用

补充：http://colobu.com/2015/04/07/minor-gc-vs-major-gc-vs-full-gc/

- 几种常用的内存调试工具：jmap、jstack、jconsole。

- 类加载的五个过程

> 加载、验证、准备、解析、初始化。

-  双亲委派模型：Bootstrap ClassLoader、Extension ClassLoader、ApplicationClassLoader。

- 垃圾回收算法，为什么要分代处理

> 分代的垃圾回收策略，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的收集方式（年轻代采用的算法是复制算法，年老代采用的是“标记-清除”或者“标记-整理”），以便提高回收效率。

> 在Java程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关，比如Http请求中的Session对象、线程、Socket连接，这类对象跟业务直接挂钩，因此生命周期比较长。但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如：String对象，由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收。
试想，在不进行对象存活时间区分的情况下，每次垃圾回收都是对整个堆空间进行回收，花费时间相对会长，同时，因为每次回收都需要遍历所有存活对象，但实际上，对于生命周期长的对象而言，这种遍历是没有效果的，因为可能进行了很多次遍历，但是他们依旧存在。因此，分代垃圾回收采用分治的思想，进行代的划分，把不同生命周期的对象放在不同代上，不同代上采用最适合它的垃圾回收方式进行回收。
  
- 如何用工具分析jvm状态

- jvm中类加载过程，解释父亲委派加载，及类是在哪个加载器加载的

- 介绍jvm内存机制（把各个内存区域作用、回收算法，收集器分类统统说一遍）

- 如何优化jvm参数(堆大小、xmx一般和xms设成一样大（ 避免每次垃圾回收完成后JVM重新分配内存.），永久代大小，收集器选择，收集器参数，新生代对象年龄阀值)

##**mysql**

- 解释mysql索引，b树，为啥要用平衡二叉树

> 磁盘和内存存储方式不同，而且mysql的BTREE实际就是按照索引值排序，而且每一个叶子节点到根节点距离都一样

- 各个join的区别

##**实际项目**
- 计算一个int里面二进制有几个1

- 心跳包是怎么设计的

- json传输数据有什么不好

- http和socket的区别， 两个协议哪个更高效一点

- java四种引用

- java GC算法与GC方法

- 锁的几种等级

- GC判定算法与方法，哪个区域用什么GC算法，怎么改进复制算法。

- HashMap删除元素的方法，for each和正常for的用在不同数据结构（ArrayList、set、hashmap）上的效率区别

- static class和non-static class的区别

- OOM(out of memory)怎么处理的？

- java反射机制？主要用在什么地方？

##**SQL语句**
- 1.找出各科目最高分的学生名字。
- 2.统计出各科目，80分到90分的人数。


> 表数据如下：

| id      | subject_name| student_name |subject_score|
|---------|-------------| -------------| ------------|
| 1       | A           |a             |98
| 2       | B           |a             |97
| 3       | C           |a             |96
| 4       | A           |b             |95
| 5       | B           |b             |99
| 6       | C           |b             |93
| 7       | A           |c             |100
| 8       | B           |c             |100
| 9       | C           |c             |90
| 10      | D           |a             |99


