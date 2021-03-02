# 读书笔记-《Effective Java 3rd》

##一、考虑用静态工厂方法代替构造方法，尽量少地暴露共有构造方法
* 静态工厂方法有名称（One advantage of static factory methods is that, unlike constructors,they have names）
* 不必在每次调用的时候，`new`一个新的对象（A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they’re invoked）
* 可以返回原返回类型的任何子类型对象（A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type）
* 根据不同的传参返回不同的对象（A fourth advantage of static factories is that the class of the returned object can vary from call to call as a function of the input parameters）

> `of` 为`valueOf()`更为完美的替代
> `getInstance`--返回单例
> `newInstance`--返回不同是实例

##二、遇到多个构造器参数时要考虑用构建器

>Notice:作者在此处有个前提：此实例一旦创建，就应该线程安全，字段不再对外提供set方法

* 多参数重叠构造器：可以采用多参数，多构造器方法。但是如果日后需要加参数，又要新增构造函数，这样会导致代码不好维护
* 采用最原始的方法，`new Instance()`，最后调用set方法，但是实例不再是线程安全，状体可变
* 作者推荐的做法，采用`Builder`模式，如果以后添加参数，无需改动构造函数，只需要添加字段和对应的setter方法
* 不足：性能会有一点损耗，毕竟构造多了一个对象，而且作者推荐参数多于4个才采用`Builder`设计模式

##三、用私有构造器或者枚举类型强化`Singleton`属性

* `Singleton`采用内部类写法

##四、通过私有构造器强化不可实例化的能力

* `XXXXUtils`类__无需__有共有构造方法，而且可以在私有构造方法中抛出异常，工具方法声明为`static`

##五、避免创建不必要的对象

* 适当使用内置对象

```java
    /*被注释的代码就是不推荐的写法*/
    
    //String _a = new String("a");
    String a = "a";

    //Boolean _b = new Boolean("false");
    Boolean b = Boolean.valueOf("false");
```

* 避免自动装箱（autoboxing），优先使用基本类型而不是装箱的基本类型，__在编码过程中应该避免无意识的装箱操作__
* 不要错误地认为创建对象是一种昂贵的操作，有时候创建小对象会有利于GC的回收，避免`大对象`迁移到`老年代`

##六、消除过期的对象引用

```java
//java.util.Vector
public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```
* 对于过期对象不进行回收，有以下常见情况:
 + 缓存，推荐使用`WeakHashMap`
 + 监听器和其他回调
 
##七、避免使用终结方法
 
> `finalizer`语义:当垃圾回收系统发现一个对象没有任何的引用的时候，也就是不可到达的时候，会把该对象标记成 finalizable , 并且把它放入到一个 Finalize Queue（F-Queue） 的特殊队列里。
> 当系统检测到这个 F-Queue 不为空的时候，就会从队列里弹出一个对象出来，调用其 finalize() 方法，把这个对象标记为 finalized ，这样，下次GC的时候，这些对象就可以被回收掉了。
 
* `finalizer`会导致不可预测的行为，例如
    + 当实例执行`finalized()`，实例只是进入回收队列，实际回收的时机是由JVM确认的，不同的JVM之间会存在差异
    + 对于实例添加`finalizer()`会导致创建和销毁对象的效率降低几倍

##八、覆盖equals时请遵守通用约定

* 遵守四条原则:自反性，对称性，传递性，一致性

##九、覆盖`equals()`时总要覆盖`hashCode()`

* 在应用程序的执行期间，只要对象的`equals`方法的比较操作所用到的信息没有被修改，那么对这同一个对象调用多次，`hashCode`方法都必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，每次执行返回的整数可以不一致。
* 如果两个对象根据`equals(Object)`方法比较是相等的，那么这两个对象的`hashCode()`返回的结果必须是相同的整数
* 如果两个对象的`equals(Object)`返回`false`，那么这两个对象的`hashCode()`返回的整数有可能相等。（为不相等的对象产生不相等的hashCode）

##十、始终要覆盖`toString()`
* 当对象被传递给`println`/`printf`/字符串连接操作/assert/被调试器打印出来，`toString()`方法会被自动调用

##十一、谨慎地覆盖`clone()`
* 从1.6发行版开始，`Cloneable`接口并没有清楚地指明，一个类在实现这个接口时应该承担哪些责任。实际上，对于实现了`Cloneable`的类，我们总是期望它也提供一个功能适当的公有的`clone`方法。通常情况下，除非该类的所有超类都提供了行为良好的`clone`实现，无论是公有的还是受保护的，否则，都不可能这么做

##十二、考虑实现`Comparable`接口

##十三、使类和成员的可访问性最小化

##十四、在公有类中使用访问方法而非共有域

##十五、使可变性最小化
* 不要提供任何会修改对象状态的方法
* 保证类不会被扩展
* 使所有的域都是`final`的
* 使所有的域都成为私有的
* 确保对于任何可变组件的互斥访问

> 坚决不要为每个get方法编写一个响应的set方法。除非有很好的理由要让类成为可变的类，否则就应该是不可变的。
> 构造器应该创建完全初始化的对象，并建立起所有的约束关系。__不要在构造器或者静态工厂之外__再提供公有的初始化方法，除非有令人信服的理由必须这么做。

##十六、复合优先于继承
* 继承关系：对于两个类A和B，只有当两者之间确实存在`is-a`关系时，类B才应该扩展类A。

##十七、要么为继承而设计，并提供文档说明，要么就禁止继承
* 构造器局不能调用可被覆盖的方法（有可能被子类覆盖后运行）

##十八、接口优于抽象类

##十九、接口只用于定义类型
* 接口不应该定义常量，常量属于实现细节

##二十、类层次由于标签类
* 标签类就是指枚举类型的声明，但是标签类可以作为一个展现就可以

##二十一、用函数对象表示策略
* 函数指针的主要用途就是实现策略模式

##二十二、优先考虑静态成员类
* 嵌套类存在的目的应该只是为了给自己的外部类提供服务

##二十三、请不要在新代码中使用原生类型（P101）

```java
public void test(List list1, List list2) {};
//这种写法更优，因为有了泛型限制，而上面方法却没有，可能会引起类型不一致报错
public void test(List<?> list1, List<?> list2) {};
```
##二十四、消除非受检警告

##二十五、列表优先于数组
* 数组是具体化的，因此数组会在运行时才知道并检查元素的类型约束

```java
    Object[] oa = new Long[10];//编译通过
    oa[0] = "aaaa";
    List<Object> ol = new ArrayList<Long>();//编译不通过
```

##二十六、优先考虑泛型

##二十七、优先考虑泛型方法
* 类型限制`<T extends Comparable<T>>`:针对可以与自身进行比较的每个类型T(T这个类型可以对自身进行比较)

##二十八、利用有限通配符来提升API的灵活性
* 为了获得最大限度的灵活性，要在表示生产者或消费者的输入参数上使用通配符类型。如果某个输入参数既是生产者，又是消费者，那么通配符类型对你就没什么好处了：因为你需要的是严格的类型匹配，这不是任何通配符能够实现的。
* 通配符使用原则：PECS-producer extends, consumer super
* 换句话说，如果参数化类型表示一个T生产者，就使用`<? extends T>`;如果它表示一个T消费者，就使用`<? super T>`,Stack.pushAll的src参数产生E实例供Stack使用，因此src相应的类型为`Iterable<? extends E>`(producer);popAll的dst参数通过Stack.pop消费E实例,因此dst相应的类型为`Collection<? super E>`(consumer)

##二十九、优先考虑类型安全的异构容器(P128)
```java
class Favourites {

        private Map<Class<?>, Object> favourites = new HashMap<>();

        public <T> void put(Class<T> type, T instance) {
            favourites.put(type, instance);
        }

        public <T> T get(Class<T> type) {
            return type.cast(favourites.get(type));
        }
}
```
##三十、用`enum`代替`Int`常量
* Java枚举类型背后的基本思想：通过公有的静态final域为每个枚举常量导出实例的类，枚举类的类型就是类的`final`/`static`实例

```java
/*计算器功能*/
public enum Operation {

    PLUS("+") { double apply(double x, double y) {return x + y;}},
    MINUS("-") { double apply(double x, double y) {return x - y;}},
    TIMES("*") { double apply(double x, double y) {return x * y;}},
    DIVIDE("/") { double apply(double x, double y) {return x / y;}};

    private final String operation;
    Operation(String operation) {
        this.operation = operation;
    }

    abstract double apply(double x, double y);
}
```

* 枚举提供了编译时的类型安全，如：平时更多的是用`int`,`String`类型的静态`final`常量声明枚举，这种做法无法保证值的准确性，如果声明为枚举类型，就可以保证被传到该参数上的任何非`Null`的对象引用一定属于枚举值的有效值
* 枚举类型的toString()方法该枚举值的声明名称
* 枚举构造器不可以访问枚举的静态域，因为构造器运行的时候，这些静态域还没有被初始化

##三十一、用实例域代替序数

##三十二、用`EnumSet`代替位域 

##三十三、用`EnumMap`代替序数索引

##三十四、用接口模式模拟可伸缩的枚举
```java
public interface Operation {
    double apply(double x, double y);
}
public enum BaseOperation implements Operation {
    PLUS("+") { public double apply(double x, double y) {return x + y;}},
    MINUS("-") { public double apply(double x, double y) {return x - y;}},
    TIMES("*") { public double apply(double x, double y) {return x * y;}},
    DIVIDE("/") { public double apply(double x, double y) {return x / y;}};
    private final String operation;
    BaseOperation(String operation) {
        this.operation = operation;
    }
}
public enum ExtendOperation implements Operation {
    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    };
    private final String operation;
    ExtendOperation(String operation) {
        this.operation = operation;
    }
}
```
##三十五、注解优于命名模式
* spring的声明式事务，优于xml的命名配置？case by case

##三十六、坚持使用`Override`注解
* 要覆盖超类声明的每个方法声明中使用`Override`注解

##三十七、用标记接口定义类型
* 标记接口定义的类型是由被标记类的实例实现的，标记注解则没有定义这样的类型
* 如果标记是应该用到任何程序元素而不是类或者接口，就必须 使用注解
* 如果标记只应用给类和接口，应该思考编写一个还是多个值接受有标记的方法？如果是，就应该使用标记接口非注解
* 例如:
```java
//标记接口:Serializable, AccessRandom
//标记注解:@Service, @Controller
```
##三十八、检查参数的有效性
* 对于公有方法可以抛出`Exception`，然后通过`@throws`声明
* 对于私有方法，通过`assert`断言来校验参数（断言是为了方便调试程序，并不是发布程序的组成部分。理解这一点是很关键的。）

##三十九、必要时进行保护性拷贝
* 对于参数类型可以被不可信任方子类化的参数，请不要使用`clone`方法尽心保护性拷贝
* 客户对象提供的对象是否可能是可变的，如果是，则要考虑你的类是否能够容忍对象的修改。

##四十、谨慎设计方法签名

##四十一、慎用重载
* 安全而保守的策略是，永远不要导出两个具有相同参数数目的重载方法

##四十二、慎用可变参数

##四十三、返回零长度的数组或者集合，而不是为`null`
* 返回空的`Colleciton`，推荐采用`Collections.emptyXXXX();`

##四十四、为所有导出的API元素编写文档注释

##四十五、将局部变量的作用域最小化
* 要使局部变量的作用域最小化，最有利的方法就是在第一次使用它的地方声明
* 有利于增强代码可读性

##四十六、`for-each`循环优先于传统的for循环
* 基于数组或者实现`RandomAccess`接口的集合都需要用`for(int i=0,size=list.size(); i<size; i++)`的方式遍历，因为

##四十七、了解和使用类库

##四十八、如果需要精确的答案，请避免使用`float`和`double`
* 金额以分的形式存在
* 金额运算需要用`BigDecimal`
* 如果数值范围没有超过9位十进制数字，用Int；不超过18位，用long，超过18位只能用`BigDecimal`

##四十九、基本类型优先于装箱基本类型
```java
Comparator<Integer> naturalOrder = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                //o1 > o2 自动拆箱，o1 == o2 比较对象同一性
                return o1 > o2 ? 1 : (o1 == o2 ? 0 : -1);
            }
        };        
System.out.println(naturalOrder.compare(new Integer(42), new Integer(42)));//-1
```
* 当在一项操作中混合使用基本类型和装箱基本类型时，装箱基本类就会自动拆箱

##五十、如果其他类型更适合，则尽量避免使用字符串

##五十一、当心字符串连接的性能

##五十二、通过接口引用对象
* 如果有合适的接口类型存在，那么对于参数、返回值、变量和域来说，都应该使用接口类型进行声明

##五十三、接口优先于反射机制（P201）

##五十四、谨慎地使用本地方法

##五十五、谨慎地进行优化
* 努力避免那些限制性能的设计决策，当一个系统设计完成之后，其中最难以更改的组件是那些指定了模块之间交互关系以及模块与外界交互关系的组件。
* 要考虑API设计决策的性能后果
* 在每次试图做优化之前和之后，要对性能进行测量

##五十六、遵守普遍接受的命名惯例

##五十七、只针对异常的情况才使用异常
* 基于异常模式的代码比标准模式代码会慢2倍
* 异常只用于异常的情况下，永远不应该用于正常的控制流

##五十八、对可恢复的情况使用受检异常，对编程错误使用运行时异常
* 如果期望调用者能够适当地恢复，对于这种情况就应该使用受检的异常
* 错误：用运行时异常来表明编程错误

##五十九、避免不必要地使用受检的异常

##六十、优先使用标准的异常

##六十一、抛出与抽象相对应的异常

##六十二、每个方法抛出的异常都要写有文档

##六十三、在细节消息中包含能捕获失败的信息
* 异常的细节消息不应该与“用户层次的错误消息”混为一谈

##六十四、努力使失败保持原子性
* 一般而言，失败的方法调用应该使对象保持在被调用之前的状态，如`Collections.sort()`执行排序之前，首先把它的输入列转到一个数组中，以便降低在排序的内循环中访问元素所需要的开销，而且这样做，即使排序失败，它也能保证输入列表保持原样。

##六十五、不要忽略异常

##六十六、同步访问共享的可变数据

##六十七、避免过度同步
* 在同步区域内，不要调用设计成被覆盖的方法，因为这种方法是外来的的，不知道会产生怎样的效果，最浅显的例子就是被覆盖的方法和外部同步块（新建了线程）而且需要获取同一把对象锁，就会产生死锁
* 通常，在同步区域内应该做尽可能少的工作

##六十八、`executor`和`task`优先于线程

##六十九、并发工具优先于`wait`和`notify`
* `wait`正确用法:

```java
synchronized(obj) {
    while(<condition does not hold>) {
        //必须写在循环之内
        obj.wait();
    }
}
```

##七十、线程安全性的文档化
> 线程安全的四个级别:
> 不可变的
> 无条件的线程安全
> 有条件的线程安全
> 非线程安全

##七十一、慎用延迟初始化
* 单例写法:`double-checked`和`内部类`

##七十二、不要依赖于线程调度器

##七十三、避免使用线程组

##七十四、谨慎地实现`Serializable`接口
* 一旦一个类被发布，就大大降低了“改变这个类的实现”的灵活性
* 增加了出现Bug和安全漏洞的可能性
* 随着类发行新的版本，相关测试负担也增加了
* 内部类不应该实现`Serializable`

##七十五、考虑使用自定义的序列化形式

##七十六、保护性地编写`readObject`方法

##七十七、对于实例控制，枚举类型优先于`readResolve`

##七十八、考虑用序列化代理代替序列化实例





