#JVM监控常用指令

> 当系统出现OOM（OutOfMemory）或者应用的“STW”（Stop-The-World）时间很长，最终导致应用所有线程挂起，不再响应任何请求，可从以下步骤获取有用信息，便于后续问题排查：
>
>__PS:不要使用view等命令编辑*.log文件，因为linux会把整个*.log文件加载到内存，如果内存不足，linux会自动根据进程优先级来kill进程，如果是kill了应用的JVM进程，那就悲剧了~__

##1.获取当前应用的jvm进程号(pid)和相关启动信息：

`jdk/bin>./jps -lvm`

##2.获取当前jvm的堆内存占用情况，GC的相关信息
`jdk/bin>./jstat -gcutil [pid] 1000 100 > [yyyyMMdd]_[appname]_state.txt`

>一秒钟执行一次，一共执行100次，并写到文件：
>
> - S0:Suvivor0的使用率
> - S1:Suvivor1的使用率
> - E:Eden的空间使用率
> - O:Old空间使用率
> - P:永久代空间使用率
> - YGC:表示jvm启动到现在的young gc times(次数)
> - YGCT:表示jvm启动到现在的young gc time(时间)
> - FGC:表示jvm启动到现在的full gc times
> - FGCT:表示jvm启动到现在的full gc time
> - GCT:表示jvm启动到现在的gc time

__PS:如果应用一直持续做FULL GC，耗时会很长，导致"STW"，需要追踪原因了__

##3.获取线程栈快照(TDA分析)：

`jdk/bin>./jstack -l [pid] > /[yyyyMMdd]_[appname].tdump`

>  tdump 文件里，值得关注的线程状态有：
> 
> - __死锁:Deadlock（重点关注）__
> - 执行中:Runnable   
> - __等待资源:Waiting on condition（重点关注）__
> - __等待获取监视器:Waiting on monitor entry（重点关注）__
> - 暂停:Suspended
> - 对象等待中:Object.wait() 或 TIMED_WAITING
> - __阻塞:Blocked（重点关注）__
> - 停止:Parked

----------------------------------------------------------------------
>1. Deadlock：死锁线程，一般指多个线程调用间，进入相互资源占用，导致一直等待无法释放的情况。
>2. Runnable：一般指该线程正在执行状态中，该线程占用了资源，正在处理某个请求，有可能正在传递SQL到数据库执行，有可能在对某个文件操作，有可能进行数据类型等转换。
>3. Waiting on condition：等待资源，或等待某个条件的发生。具体原因需结合 stacktrace来分析。如果堆栈信息明确是应用代码，则证明该线程正在等待资源。一般是大量读取某资源，且该资源采用了资源锁的情况下，线程进入等待状态，等待资源的读取。又或者，正在等待其他线程的执行等。如果发现有大量的线程都在处在 Wait on condition，从线程 stack看，正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几乎消耗了所有的带宽，仍然有大量数据等待网络读写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。另外一种出现 Wait on condition的常见情况是该线程在 sleep，等待 sleep的时间到了时候，将被唤醒。
>4. Blocked：线程阻塞，是指当前线程执行过程中，所需要的资源长时间等待却一直未能获取到，被容器的线程管理器标识为阻塞状态，可以理解为等待资源超时的线程。
>5. Waiting for monitor entry 和 in Object.wait()：Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。从下图1中可以看出，每个 Monitor在某个时刻，只能被一个线程拥有，该线程就是 “Active Thread”，而其它线程都是 “Waiting Thread”，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。在 “Entry Set”中等待的线程状态是 “Waiting for monitor entry”，而在 “Wait Set”中等待的线程状态是 “in Object.wait()”。

##4.获取堆内存快照(MAT分析)：

> __Notice__: dump 内存快照之前先思考几个问题：
>
> 1. dump 内存快照，会把整个应用的线程挂起，导致`stop-the-world`,应用会无法响应请求，而且时间会很长（堆内存可能会很大）
> 2. dump 出来的快照文件可能会很大，磁盘空间是否足够大？

`jdk/bin>./jmap -dump:live,format=b,file=[yyyyMMdd]_[appname].hprof [pid]`


