#DCL(Double Check Lock)单例模式写法
部分内容转载：
[http://lifethinker.iteye.com/blog/260515](http://lifethinker.iteye.com/blog/260515)
[http://www.race604.com/java-double-checked-singleton/](http://www.race604.com/java-double-checked-singleton/)
##`happen-before` 原则

> - 我们一般说一个操作`happen-before`另一个操作，这到底是什么意思呢？
> 	- 当说操作A `happen-before`操作B时，我们其实是在说在发生操作B之前，操作A对内存施加的影响能够被观测到。所谓“对内存施加的影响”就是指对变量的写入，“被观测到”指当读取这个变量时能够得到刚才写入的值（如果中间没有发生其它的写入）。听起来很绕口？这就对了，请你保持耐心。
> 	- 举个例子来说明一下。线程Ⅰ执行了操作A：x=3，线程Ⅱ执行了操作Ｂ：y=x。如果操作Ａ`happen-before`操作B，线程Ⅱ在执行操作B之前就确定操作"x=3"被执行了，它能够确定，是因为如果这两个操作之间没有任何对x的写入的话，它读取x的值将得到3，这意味着线程Ⅱ执行操作B会写入y的值为3。如果两个操作之间还有对x的写入会怎样呢？假设线程Ⅲ在操作A和B之间执行了操作C: x=5，并且操作C和操作B之前并没有`happen-before`关系 **(后面我会说明时间上的先后并不一定导致`happen-before`关系)** 。这时线程Ⅱ执行操作B会讲到x的什么值呢？3还是5?答案是两者皆有可能。
> - 这是因为 **`happen-before`关系保证一定 能够观测到前一个操作施加的内存影响，只有时间上的先后关系而并没有`happen-before`关系可能但并不保证能观测前一个操作施加的内存影响** 。如果读到了值3，我们就说读到了“陈旧”的数据。正是多种可能性导致了多线程的不确定性和复杂性，但是要分析多线程的安全性，我们只能分析确定性部分，这就要求找出`happen-before`关系，这又得利用`happen-before`规则。

> - 下面是我列出的三条非常重要的`happen-before`规则，利用它们可以确定两个操作之间是否存在`happen-before`关系。
> 	- 同一个线程中，书写在前面的操作`happen-before`书写在后面的操作。这条规则是说，在单线程中操作间`happen-before`关系完全是由源代码的顺序决定的，这里的前提“在同一个线程中”是很重要的，这条规则也称为单线程规则 。这个规则多少说得有些简单了，考虑到控制结构和循环结构，书写在后面的操作可能`happen-before`书写在前面的操作，不过我想读者应该明白我的意思。
> 	- 对锁的unlock操作`happen-before`后续的对同一个锁的lock操作。这里的“后续”指的是时间上的先后关系，unlock操作发生在退出同步块之后，lock操作发生在进入同步块之前。这是条最关键性的规则，线程安全性主要依赖于这条规则。但是仅仅是这条规则仍然不起任何作用，它必须和下面这条规则联合起来使用才显得意义重大。这里关键条件是必须对“同一个锁”的lock和unlock（`synchronized`和`ReentrantLock`）。
> 	- 如果操作A `happen-before`操作B，操作B `happen-before`操作C，那么操作A `happen-before`操作C。这条规则也称为传递规则。

##DCL Demo

###代码片段

```java
import java.util.Random;

public class LazySingleton {
    private int someField;
    
    private static LazySingleton instance;
    
    private LazySingleton() {
        this.someField = new Random().nextInt(200)+1;         // (1)
    }
    
    public static LazySingleton getInstance() {
        if (instance == null) {                               // (2)
            synchronized(LazySingleton.class) {               // (3)
                if (instance == null) {                       // (4)
                    instance = new LazySingleton();           // (5)
                }
            }
        }
        return instance;                                      // (6)
    }
    
    public int getSomeField() {
        return this.someField;                                // (7)
    }
}

```

###代码分析

> - 分析一：我们要先明白`new LazySingleton()`（即代码（5））实际编译器做了什么事情？这句代码实际会产生三条指令:
	- 1. 为`LazySingleton`实例开辟内存空间；
	- 2. 实例化`LazySingleton`
	- 3. 把步骤2的内存空间引用返回给`instance`
	- 在编译器看来，`new LazySingleton()`是线程安全（在同步块内）的，所以它会进行优化（__指令重排序__），步骤3比步骤2先执行，__提早返回引用，但是实例化过程还没完成，所以就有了分析二__
> - 分析二：这里的关键是尽管得到了LazySingleton的正确引用，但是却有可能访问到其成员变量的不正确值。**比如LazySingleton.getInstance().getSomeField()有可能返回someField的默认值0**。


##结论

> - __对DCL的分析也告诉我们一条经验原则，对引用（包括对象引用和数组引用）的非同步访问，即使得到该对象引用的最新值，却并不能保证也能得到其成员变量（对数组而言就是每个数组元素）的最新值。__
> - 解决办法一：jdk1.5之后增强了`volatile`的语义，禁止了指令重排序（代码一）
> - 解决办法二：利用了 Java 的语言特性，内部类只有在使用的时候，才会去加载，从而初始化内部静态变量。关于线程安全，这是 Java 运行环境自动给你保证的，在加载的时候，会自动隐形的同步。在访问对象的时候，不需要同步 ，完成之后Java 虚拟机又会自动给你取消同步，所以效率非常高。（代码二）


###代码一：

```java
import java.util.Random;

public class LazySingleton {
    private int someField;
    
    private volatile static LazySingleton instance;//instance 加上 volatile 修饰
    
    private LazySingleton() {
        this.someField = new Random().nextInt(200)+1;          // (1)
    }
    
    public static LazySingleton getInstance() {
    	//volatile 变量的读写操作是一个比较重的操作,创建临时变量优化
    	LazySingleton _instance = instance;
        if (_instance == null) {                               // (2)
            synchronized(LazySingleton.class) {
                _instance = instance;						     // (3)
                if (_instance == null) {                       // (4)
                    instance = _instance = new LazySingleton();// (5)
                }
            }
        }
        return _instance;                                      // (6)
    }
    
    public int getSomeField() {
        return this.someField;                                 // (7)
    }
}
```

###分析
> - 通过这样修改以后，在运行过程中，除了第一次实例化以外，其他的调用只要访问 `volatile` 变量  `instance`  一次，这样能提高 25% 的性能，详见 [维基百科](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)

> - 这里为什么需要再定义一个临时变量`_instance`？通过前面的对`volatile`关键字作用解释可知，访问`volatile`变量，需要保证一些执行顺序，所以的开销比较大。这里定义一个临时变量，在 `instance` 不为空的时候（这是绝大部分的情况），只要在开始访问一次 `volatile`变量，返回的是临时变量。如果没有此临时变量，则需要访问两次，而降低了效率。

###代码二
```java
public class LazySingleton {

	private LazySingleton() {}
	
	public static LazySingleton getInstance() {
		return InnerSingletonClass.instance;
	}
	
	private static class InnerSingletonClass {
		public static final LazySingleton instance = new LazySingleton();
	}
}
```
###分析
> - `InnerSingletonClass.instance` 为什么不加`volatile`修饰?
> - 自从jdk1.5之后，final语义增强，具体如下：只要对象是正确构造的，那么就不需要使用同步(lock)或者`volatile`变量修饰，就可以保证任意线程都能看到`instance`被初始化之后的值（禁止了指令重排序和解决可见性问题），添加`volatile`只是画蛇添足而已


