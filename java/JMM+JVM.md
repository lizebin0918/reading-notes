###视频笔记-JMM

####工具：https://console.perfma.com/

####各个类加载器职责

* Bootstrap ClassLoader：JAVA_HOME/jre/lib/rt.jar、resources.jar、sun.boot.class.path路径下的包，用于提供jvm运行所需的包
* Extension ClassLoader：派生继承自java.lang.ClassLoader，父类加载器为启动类加载器。从系统属性：java.ext.dirs目录中加载类库，或者从JDK安装目录：jre/lib/ext目录下加载类库
* Application Classloader：它负责加载环境变量classpath或者系统属性java.class.path指定路径下的类库。它是程序中默认的类加载器，我们Java程序中的类，都是由它加载完成的。
* Custom Classloader:自定义类加载器

####类的加载过程
* 加载（loading）：查找并加载类的二进制数据（*.class文件）
* 准备（preparation）：为类的**静态变量**分配内存，并将其初始化为默认值，这里的默认值就是说引用对象为null，但是指对象就等于它们对应的默认值，这里的默认值如果是int就是为0，boolean就是为false，引用类型就是为null#这个class还没new，不会为实例变量分配内存
* 解析（resolution）：解析：把类中的符号引用转换为直接引用（Object a = Object.class//指向这个class的实际内存地址）
* 初始化：为类的静态变量赋予正确的初始值（那初始化是在什么时候触发？-主动使用）

```java
class A {
    private static int a;//准备阶段默认值：0
    static {
        a = 3;//这个过程就是初始化过程
    }
}
```

####主动使用情况
* 主动使用情况
* 创建类的实例
* 访问类或者接口的静态变量，或者对该静态变量赋值
* 调用类的静态方法
* 反射，如Class.forName(“");
* 初始化一个类的子类
* Java虚拟机启动时被标明为启动类的类

####双亲委托机制
* 父加载器不是"类加载器的加载器"
* 各个类加载器之间并不是继承关系，只是引用了parent属性而已，加载一个类会一直递归往上找对应的类加载器，如果没有找到就往下找到对应父类加载
* 双亲委派是一个孩子向父亲方向，然后父亲向孩子方向的双亲委派过程。如果父亲能加载就让父亲加载，如果都不能就只能自己加载，如果我们自己写了一个String类，并且把这个类加载了，这样就会有安全问题。
* 自定义类加载器：实现jdk的ClassLoader抽象类，重写findClass方法，每次都会根据双亲委派原则加载Class。如果需要实现热部署，可以直接重写loadClass()，无需从父类需要class，重新生成ClassLoader并加载class即可

####硬件测数据一致性
* intel的缓存一致性:MESI--https://www.cnblogs.com/z00377750/p/9180644.html
* 缓存行：读取缓存以cache line为基本单位，目前64bytes，位于同一缓存行的两个不同数据，被两个不同CPU锁定，产生互相影响的伪共享问题，引出总线风暴问题--https://cloud.tencent.com/developer/article/1707875
* **解决上述问题：使用缓存行的对齐能够提高效率**，代码:[](https://github.com/lizebin0918/test/blob/main/src/main/java/com/lzb/CacheLineTest.java)

####CPU执行乱序问题
* CPU如果发现两条指令没有依赖关系，当第一条在执行等待，会在等待过程中执行第二条指令--https://www.cnblogs.com/liushaodong/p/4777308.html
* 解决乱序问题：CPU内存屏障
* sfence: store | 在sfence指令前的写操作必须在sfence指令后的写操作前完成
* lfence: load | 在lfence指令前的读操作必须在lfence指令后的读操作前完成
* mfence: mix | 在mfence指令前的读写操作必须在mfence后的读写操作前完成 

####volatile的实现细节
* volatile除了修饰变量，也可以修饰对象。**而被volatile关键字修饰的对象作为类变量或实例变量时，其对象中携带的类变量和实例变量也相当于被volatile关键字修饰了**
* 保证了可见性和有序性（实际就是在指令前后添加屏障）
* 字节码层面：class字节码文件只是添加了一个关键字：volatile
* JVM层面：
    * 写操作:StoreStoreBarrier/volatile 写操作/StoreLoadBarrier
    * 读操作:LocalLoadBarrier/volatile 读操作/LoadStoreBarrier
* 

####synchronized实现细节
* 字节码层面
    * monitorenter/monitorexit
* C/C++调用了操作系统提供的同步机制
* OS和硬件层面
    * X86: lock

####Java的内存模型
* 每个线程都有自己的线程栈和工作内存，每次读取和存储变量都是在工作内存和主内存之间交互
* JVM会有主内存
* 一个线程栈里面装着一个个栈帧，每个栈帧包括：Local variable/Operand Stacks/Dynamic linking/return address

####GC垃圾回收
* 安全点（safe point）：程序执行时并非在所有地方都能停顿下来开始GC，只有在达到安全点时才能暂停，因为每条指令执行的时间非常短暂，程序不太可能因为指令流长度太长而长时间运行，“长时间执行”的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具有这些功能的指令才会产生安全点
    * 抢占式中断（少用）：GC时，首先把所有线程全部中断，如果有线程中断的地方不在安全点上，就回复线程，让它“跑”到安全点上
    * 主动式中断：GC不直接对线程操作，仅仅简单滴设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起，轮询标志的地方和安全点是重合的，另外再加上创建对象需要分配内存的地方
* 安全区域（safe region）：对于部分线程出于Sleep或者Blocked状态的线程，这时候线程无法响应JVM的中断请求，对于这种情况，就需要**安全区域**来解决
* 怎么定义垃圾？当对象没有被引用
* 如何找到垃圾？
    * 引用计数
    * 引用计数无法解决循环引用，采用根搜索算法
    > main线程的栈帧存储的对象都是根对象
    > 静态变量
    > 常量池
    > JNI 指针

####GC算法（具体垃圾回收算法的实现）
* Mark-Sweep（标记清除）
    * 太多空间碎片，无法存放大对象
    * 适合存货对象比较多，回收对象比较少的情况
    * 两边扫描：第一遍标记，第二遍清除
* Mark-Compact（标记整理（压缩））
    * 扫描两次：第一遍标记对象，第二遍需要移动对象
* Coping（复制）：适合Eden区
    * 对象的拷贝移动，引用需要调整
    * 需要更多的空间
    
####堆内存逻辑分区
* Young（new） 区：eden(8):survivor0(1):survivor1(1)，采用copying算法
* Old 区：tenured，采用Mark Compact 或者 Mark Sweep
* 方法区：jdk1.7 永久代/jdk1.8 metaspace，存放class对象，1.8之后，字符串常量池存储在堆区，metaspace 是放在单独的空间
* 新生代大量死去，少量存货，采用复制算法
* 老年代存活率高，回收较少，采用MC或MS算法
* GC按照回收区域分为两大种类型
    * 一种是部分收集（Partial GC）
    > 新生代收集（Minor GC / Young GC）：只是新生代的垃圾收集
    > 老年代收集（Major GC / Old GC）：只是老年代的收集。 注意很多时候Major GC 会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收
    
    * 一种是整堆收集（Full GC）
    > 收集整个Java堆和方法区的垃圾回收
    
* GC触发条件
    * Minor GC 只在Eden区满的时候触发，Survivor区满并不会触发Minor GC，但是并不是说Survivor区不会被垃圾回收，而是说在Eden区满时触发Minor GC然后Eden区和Survivor区一起被垃圾回收，可以说Survivor区时被动垃圾回收的
    * 在老年代空间不足的时候，会先尝试触发Minor GC，如果之后空间还不足，就会执行Major GC
    * Full GC触发机制
        * 调用System.gc()时，系统建议执行Full GC，但是不必然执行
        * 老年代不足的时候
        * 方法区不足的时候
        * 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
        * 由Eden区，survivor0（From Space）区先survivor1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

####垃圾收集器
* 组合一：Serial + Serial Old
    * 所有应用线程都要停止，单CPU效率最高，Client模式默认垃圾回收器
    * 在safe point之后才能进行回收 
* 组合二：Parallel Scavenge（基于复制算法） + Parallel Old（基于mark-compact-sweep）
    * 多线程并行清理，会引起STW，JDK默认的垃圾回收算法
* 组合三：ParNew + CMS
    * ParNew 比 PS 只是加强而已，PN响应时间优先，PS吞吐量优先
    * CMS的四个阶段
        * initial mark（初始标记）：单线程，引起STW
        * concurrent mark（并发标记）：多线程，并发执行
        * remark（重新标记）：多线程标记
        * concurrent sweep（并发清理）：单线程，并发清理
    * 并发标记：三色标记
    * CMS缺点
        * CMS会由于碎片太多，无法放置大对象，会退化成Serial Old
        * Memnory fragmentation
            * -XX:UseCMSCompactAtFullCollection
            * -XX:CmsFullGCsBeforeCompaction 默认为0指的是经过多少次FGC才进行压缩
        * Floating Garbage
            * GC日志：Concurrent Mode Failure / PromotionFailed
            * 解决方案：降低出发CMS的阈值-CMSInitiatingOccupancyFraction 从默认的92%降低到68%，会浪费一些内存，换来更早地释放垃圾

####常用指令
* dump出堆内存快照，解析工具：MAT/jhat
* 在线分析工具：https://console.perfma.com/

####日志详解
* 生产环境的gc log 需要按大小切分，不然没法看

```
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/xxxx-service-gc-%t.log 日志文件的输出路径，按照系统时间
-XX:+PrintReferenceGC 打印年轻代各个引用的数量以及时长
-XX:+UseGCLogFileRotation：是否启用 GC 日志文件自动转储, GC 日志文件按大小切割时需要设置，文件会自动滚动清除
-XX:GCLogFileSize：控制 GC 日志文件的大小, 用于 GC 日志的切割, 不宜设置太大, 太大由于写日志需要进行 I/O 操作, I/O 操作需要在用户态和和心态之间切换, 会直接影响 user time 大小和 sys time 大小, 最终影响到 real time 大小, real time 就是 GC 耗费的时间。例如：-XX:GCLogFileSize=50M
-XX:NumberOfGCLogFiles：GC 日志文件最多保存的个数。例如：-XX:NumberOfGCLogFiles=10
-XX:+PrintGCApplicationStopedTime ：查看GC造成的应用暂停时间
-XX:+HeapDumpOnOutOfMemoryError:内存溢出时输出 dump 文件
-XX:+printGCCause GC原因
```

* Parallel Scavenge + Parallel Old
    * 2021-03-10T17:00:48.600+0800:gc时间
    * GC：触发GC，这里指的是Young GC，Full GC就是整堆回收
    * (Allocation Failure)：触发GC的原因
    * PSYoungGen：年轻代收集器
    * 2045356K->13285K(2067456K)：回收前的年轻代的空间2045356K，清理之后为 13285K，这里的清理是**指回收的对象+升迁到老年代的对象**，2067456K 为**整个年轻代大小**
    * 2460578K->428516K(3116032K)：回收前的总堆空间2460578K，清理之后为 428516K，3116032K 为**整个堆的大小**
    * 0.0214478 secs：实际回收时间

```
2021-03-10T17:00:48.600+0800: 1446.902: [GC (Allocation Failure) [PSYoungGen: 2045356K->13285K(2067456K)] 2460578K->428516K(3116032K), 0.0214478 secs] [Times: user=0.20 sys=0.01, real=0.02 secs]
```

* ParNew + CMS

```
2021-03-18T15:20:16.458+0800: 5090.260: [GC (CMS Initial Mark) [1 CMS-initial-mark: 525226K(1048576K)] 615702K(5767168K), 0.0086121 secs] [Times: user=0.02 sys=0.01, real=0.00 secs]
> 525226K(1048576K)：老年代使用（最大值）
> 615702K(5767168K)：整个堆使用（最大值）

2021-03-18T15:20:21.595+0800: 5095.397: [GC (CMS Final Remark) [YG occupancy: 1566656 K (4718592 K)]2021-03-18T15:20:21.595+0800: 5095.397: [Rescan (parallel) , 0.3725245 secs]2021-03-18T15:20:21.967+0800: 5095.769: [weak refs processing, 0.0005480 secs]2021-03-18T15:20:21.968+0800: 5095.770: [class unloading, 0.0536364 secs]2021-03-18T15:20:22.022+0800: 5095.823: [scrub symbol table, 0.0130051 secs]2021-03-18T15:20:22.035+0800: 5095.836: [scrub string table, 0.0015001 secs][1 CMS-remark: 525226K(1048576K)] 2091883K(5767168K), 0.4418468 secs] [Times: user=1.56 sys=0.00, real=0.44 secs]

> STW阶段，YG occupancy:年轻代占用及容量
> [rescan(parallel):STW下的存活对象标记
> weak refs processing:弱引用处理
> class unloading:卸载不用的class
> scrub string table:clean up symbol and string tables which hold class-level metadata and internalized string respectively
> CMS-remark：525226K(1048576K)：阶段过后的老年代占用及容量

```

* 查看gc情况：./jstat -gc [pid] 1000

```
S0C：第一个幸存区的大小
S1C：第二个幸存区的大小
S0U：第一个幸存区的使用大小
S1U：第二个幸存区的使用大小
EC：伊甸园区的大小
EU：伊甸园区的使用大小
OC：老年代大小
OU：老年代使用大小
MC：方法区大小
MU：方法区使用大小
CCSC:压缩类空间大小
CCSU:压缩类空间使用大小
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

####JVM案例
* 吞吐量：用户代码执行时间 /（用户代码执行时间 + 垃圾回收时间）--允许较长时间的STW换区总吞吐量最大化，离线处理 PS + PO，单位时间内，最大化一个应用的工作量
* 响应时间：STW越短，响应时间越高--在线处理并立即响应，关注实时交互 PN + CMS
* 所谓调优，首先确定，吞吐量优先？还是响应时间优先？
* 并发量：对于某一个业务的接口，1秒钟可以处理多少个请求（这里的操作可以算做一个事务）
* 如何根据并发量预估内存？
    * 前提：要求响应时间再100ms以内
    * 假设：一个请求需要1M的内存
    * 评估：1秒有1000个并发，需要1G的内存，其实也可以512M，在0.5秒前500个请求处理完并且返回释放内存，后500个再来也就可以了。但是如果处理太慢，导致前面的没处理完，后面的又来，这个时候就有可能发生Full GC
* 系统CPU经常100%，如何调优？
    * 找出哪个进程CPU高（top）
    * 该进程中的哪个线程CPU高（top -Hp）
    * 导出该线程的堆栈（jstack）
    * 查找哪个栈帧的消耗时间（jstack，要转成16进制）
    * 工作线程占比高 | 垃圾回收线程占比高
* 查看当前堆中最多的对象排名：./jmap -histo [pid] | head -20
* 线上环境开启jmx需要运维开通端口，还要配置账号密码，**而且对程序运行有一定影响**，一般不会采用，最好还是用arthas，图形界面是在压测阶段用的

#####CMS详解
* 存在问题
    * 内存碎片，产生浮动垃圾，添加参数：-XX:CMSFullGCsBeforeCompaction，这个操作会导致GC时间变长
* 第一阶段（初始标记-单线程）：找到根对象（GC Roots）-- This is initial Marking phase of CMS where all the objects directly reachable from roots are marked and this is done with all the mutator threads stopped.
* 第二阶段（并发标记）：从第一阶段的根对象进行扫描并标记，用户线程和扫描线程并发执行，遍历老年代，标记所有存货的对象-- Start of concurrent marking phase.In Concurrent Marking phase, threads stopped in the first phase are started again and all the objects transitively reachable from the objects marked in first phase are marked here
* 第三阶段（重新标记-多线程并行）：在上阶段的标记过程中，可能会存在多标记或者漏标记的情况，需要STW，重新整理标记即可
* 第四阶段（并发清理）

#####G1详解[](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)
* 视频：[](https://www.bilibili.com/video/BV1D741177rV?p=2&share_source=copy_web)
* 与应用线程同时工作，几乎不需要STW
* 整理剩余空间，不产生内存碎片
* GC停顿更加可控
* 不牺牲系统的吞吐量
* gc不要求额外的内存空间（CMS需要预留空间存储浮动垃圾）
* G1在某些方面弥补了CMS的不足，比如，CMS使用的是mark-sweep算法，会产生内存碎片；**而G1基于copying算法**，高效整理剩余内存，不需要管理内存碎片
* 内存区域分成（不同角色，实际是把整块内存分成小块-region）：Eden、Survivor、Old、Humongous
* G1使用了gc停顿可预测的模型，来满足用户设定的gc停顿时间，根据用户设定的目标时间，G1会自动地选择哪些region要清除，一次清除多少个region
* 在物理上不连续，带来了额外的好处--有的分区垃圾对象特别多，有的分区垃圾对象很少，G1会优先回收垃圾对象特别多的分区，即首先手机垃圾最多的分区
* Young GC : 会进行STW，基于复制算法进行多线程并行回收，CSet是所有年轻代里面的Region
* Mixed GC：进行全局标记，中间有两个步骤需要STW，只能回收部分老年代的Region，如果回收不过来，只能serial old GC，CSet是所有年轻代里的Region加上全局并发标记阶段标记出来的收益高的Region
    * 初始标记：STW，哪些引用是被新生代引用
    * 并发标记
    * 重新标记
    * 复制清除：新生代和老年代同时收集清除
    * 什么时候触发mixed GC？
        * G1HeapWastePercent:在global concurrent marking结束之后，可以知道O区有多少regions需要被回收，在每次YGC之后和再次Mixed GC之前会检查垃圾占比是否达到此参数
        * G1MixedGCLiveThresholdPercent:O区中的存回对象的占比，只有在此参数之下，才会被选入CSet


* Card Table：把一个Region分成多个card table
* 收集集合（CSet）：一组可被回收的分区集合，存活的对象会被移动到另一个可用的分区
* 已记忆集合（RSet）：记录了其他Region中的对象引用本Region中对象的关系，使得垃圾收集器不需要扫描整个堆找到谁引用了当前分区中的对象，只需扫描RSet即可
> 比如Region1的CardTable1的对象和Region3的CardTable3的对应引用了Region2的对象，而Region2的RSet就会记录CardTable1和CardTable3表示有外部引用。
> 一旦对进行Region2进行扫描，只需要对RSet对应的CardTable进行扫描即可
> 当对应的内存空间发生改变时，标记为dirty

* 算法概念：做YGC的时候，根对象可能已经挪到old区，这样扫描的效率非常低，JVM设置了Card Table，如果一个old区的Card Table中有对象指向Young区，就将它设置为dirty，下次再扫描时，只需要扫描Dirty Card Table即可，在结构上，Card Table 用BitMap来实现
* 并发收集，也是采用三色标记算法
* 压缩空闲空间不会延长GC的暂停时间
* 更容易预测的GC暂停时间
* 适用不需要事先很高的吞吐量的场景，**吞吐量也很重要，但是往往通过升级硬件即可**    

####三色标记算法
* 白色：未标记的对象
* 灰色：自身被标记，成员变量未标记完
* 黑色：自身和成员变量都已标记
* 标记规则：从根（GC Root）开始标记，一直往子节点（成员变量）遍历，遍历的时候先标记灰色（**维护一个队列，灰色的入队**），灰色节点出队，当前节点标记为黑色，子节点标记为灰色继续入队，一直这样层序遍历所有节点
* 漏标的情况
    * **当变量进行赋值的时候，对象的关系就会发生变化**
    * 本来是黑色->灰色->白色，但是用户线程也在进行，有可能黑色指向了白色，而且灰色指向白色的断掉了
    * 并发标记过程中，Mutator删除了所有从灰色到白色的引用，会产生漏标
* 解决漏标--SATB snapshot at the beginning
    * 只要引用改变，通过Write Barrier对引用字段进行赋值做了额外的处理，**发生变化的对象都要改成非白色的**，当做浮动垃圾下次再回收 
    * post Write Barrier--增量更新，关注引用的增加，把黑色重新标记为灰色，下次重新扫描
    * pre Write Barrier -- 关注引用的删除，当灰色指向白色消失时，要把这个引用推到GC的堆栈，保证白色还能被GC扫描到

#####常见晋升老年代机制
* 担保机制
> 在Serial+Serial Old的情况下，发现放不下就直接启动担保机制；在Parallel Scavenge+Serial Old的情况下，却是先要去判断一下要分配的内存是不是>=Eden区大小的一半，如果是那么直接把该对象放入老生代，否则才会启动担保机制
> 当Survivor区的的内存大小不足以装下下一次Minor GC所有存活对象时，就会启动担保机制，把Survivor区放不下的对象放到老年代；

* 大对象直接放入老年代
> 大对象（大小大于-XX:PretenureSizeThreshold的对象）直接在老年代分配内存；（只对Serial和ParNew收集器有效，对于Parallel Scavenge收集器无效）
> 大对象在Eden区放不下，直接放到O区

* 长期存活的对象进入老年代
> 把age大于-XX:MaxTenuringThreshold的对象晋升到老年代；（对象每在Survivor区熬过一次，其age就增加一岁）

####分析手段
* gc日志分析:GCViewer [](https://github.com/chewiebug/GCViewer/releases)
* dump文件分析：MAT [](https://www.cnblogs.com/trust-freedom/p/6744948.html)
* 记录好日志
* 对程序做好性能监控；
* 根据日志和性能监控数据修改程序；
* 使用专业工具通过不同的JVM参数进行压测并获得最佳配置。
####JVM参数
* 查询默认参数：java -XX:+PrintFlagsInitial | grep 
* 打印开启参数：java -XX:+PrintCommandLineFlags -version
* Parallel常用参数
    * -XX:PreTenureSizeThreshold=大对象直接放到O区
    * -XX:MaxTenuringThreshold=升代年龄
    * -XX:+PrintTenuringDistribution:打印对象年龄
* CMS常用参数
    * -XX:+UseConcMarkSweepGC
    * -XX:ParallelCMSThreads:CMS线程数量（机器核数/2）
    * -XX:CMSInitiatingOccupancyFraction:使用多少比例的老年代后开始CMS收集
    * -XX:+UseCMSCompactAtFullCollection:在FGC时进行压缩
    * -XX:CMSFullGCsBeforeCompaction:多少次FGC之后进行压缩
* G1常用参数
    * -XX:MaxGCPauseMillis:控制回收时长，设置的时间越短表示每次收集的CSet越小，导致垃圾逐步积累变多，最终不得不退化成Serial GC；停顿时间设置过长，导致每次都会产生长时间的停顿


