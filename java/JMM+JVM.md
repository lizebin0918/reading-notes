### 视频笔记-JMM

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
* **解决上述问题：使用缓存行的对齐能够提高效率**，代码:

####CPU执行乱序问题
* CPU如果发现两条指令没有依赖关系，当第一条在执行等待，会在等待过程中执行第二条指令--https://www.cnblogs.com/liushaodong/p/4777308.html
* 解决乱序问题：CPU内存屏障
* sfence: store | 在sfence指令前的写操作必须在sfence指令后的写操作前完成
* lfence: load | 在lfence指令前的读操作必须在lfence指令后的读操作前完成
* mfence: mix | 在mfence指令前的读写操作必须在mfence后的读写操作前完成 

####volatile的实现细节
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
* 怎么定义垃圾？当对象没有被引用
* 如何找到垃圾？
    * 引用计数
    * 引用计数无法解决循环引用，采用根搜索算法
    > main线程的栈帧存储的对象都是根对象
    > 静态变量
    > 常量池
    > JNI 指针

####GC算法
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
* 组合二：Parallel Scavenge + Parallel Old
    * 多线程并行清理，会引起STW，JDK默认的垃圾回收算法
* 组合三：ParNew + CMS
    * ParNew 比 PS 只是加强而已，PN响应时间优先，PS吞吐量优先
    * CMS的四个阶段
        * initial mark（初始标记）：单线程，引起STW
        * concurrent mark（并发标记）：多线程，并发执行
        * remark（重新标记）：多线程标记
        * concurrent sweep（并发清理）：单线程，并发清理

####调优参数
* 查看参数：java -XX:+PrintFlagsFinal -version
* -XX:MaxTenuringThreshold:s0-s1之间的复制年龄超过限制时，进入old区


