##《JVM视频学习》读书笔记

###类加载
* 类型（我们定义的类、接口各种类型）的加载、连接与初始化过程都是在程序运行期间完成的
* Java虚拟机与程序的声明周期
    * 执行了`System.exit()`
    * 程序正常执行结束
    * 程序在执行过程中遇到了异常或错误而异常终止
    * 由于操作系统出现错误而导致Java虚拟机进程终止
* 加载：查找并加载类的二进制数据（*.class文件）
* 连接：
    * 验证：确保被加载的类的正确性
    * 准备：为类的静态变量分配内存，并将其初始化为默认值，这里的默认值就是说引用对象为null，但是指对象就等于它们对应的默认值，这里的默认值如果是int就是为0，boolean就是为false，引用类型就是为null#这个class还没new，不会为实例变量分配内存
    * 解析：把类中的符号引用转换为直接引用（实际就是把引用变量，指向实际的地址）
* 初始化：为类的静态变量赋予正确的初始值
      
    ```java
    class A {
        private static int a;
        static {
            a = 3;//这个过程就是初始化过程
        }
    }
    ```
* 主动使用（六种）：     
     a.创建类的实例
     b.访问类或者接口的静态变量，或者对该静态变量赋值
     c.调用类的静态方法
     d.反射，如Class.forName(“");
     e.初始化一个类的子类
     f.Java虚拟机启动时被标明为启动类的类
     (反例)e.`ClassLoader.getSystemClassLoader().loadClass("");`并不会对类进行初始化
* 除了上述说的主动使用情况，其他情况并不会对类进行初始化（这里的初始化是表示第三部的初始化，之前已经对类进行了【加载】和【连接】）。    
* 类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内（方法区就是永久代，通过-XX：PermSize设置，但它并非堆内存的一部分，堆内存：Eden,s0,s1,old）。然后在堆区创建一个Class对象（这个Class对象在堆内存的哪个区域？堆的哪个区域？由JVM创建），用来封装在方法区内的数据结构。在装载类的时候，加入方法区中的所有信息，最后都会形成Class类的实例，代表这个被装载的类。方法区中的所有的信息，都是可以通过这个Class类对象反射得到。
* 扩展学习：`ClassNotFoundException`和`NoClassDefFoundError`的区别，前者是`Class.forName('')`动态加载类的时候，发现类在对应路径上不存在；后者是查找的类在编译的时候是存在的，但是在运行的时候却找不到了。
* `-XX:+TraceClassLoading`可以打印所有加载的类信息
* JVM参数写法:
    * `-XX:+<option>`:表示开启option选项（布尔值）
    * `-XX:+<option>`:表示关闭option选项（布尔值）
    * `-XX:<option>=<value>:表示将option选项的值设置的value`（赋值）
* 如果引用一个类的`static final`变量，将不算主动使用，不会导致类初始化。因为这个常量在编译阶段就已经确定下来了，它已经被存储在引用类方法区的常量池中
* 助记符
    * ldc表示将int,float或是String类型的常量值从常量池中推送至栈顶
    * bipush表示将单字节（-128~127）的常量值推送至栈顶
    * sipush表示将一个短整形常量值（-32768 ~ 32767）推送至栈顶
    * iconst_1表示将int类型1推送至栈顶（iconst_1 ~ iconst_5）
    * anewarray:创建一个引用类型的数组，并将其引用值押入栈顶
    * newarray:创建一个原生类型的数组，并将其引用值押入栈顶
* 反编译指令:`javap -c com.....`
* 对于数组实例来说，其类型是由JVM在运行期动态生成的，表示为[Lcom.shengsiyuan.jvm.classloader.MyParent4这种形式。动态生成的类型，其父类型就是Object(动态代理同理) 
* 接口初始化：当一个接口初始化时，并不要求父接口都完成了初始化；接口里面的变量都是在编译期就确定下来了
* 类加载器：类加载器并不需要等到某个类被“首次主动使用”时再加载它，这个时候还没初始化
    * Java虚拟机自带的加载器
        * 根类加载器（Bootstrap）
        * 扩展类加载器（Extension）
        * 系统（应用）类加载器（System）
    * 用户自定义的类加载器
        *  `java.lang.ClassLoader`的子类
        *  用户可以定制类的加载方式
* 接口初始化跟类初始化有区别：一个父接口并不会因为子接口的初始化而初始化，只有当程序首次使用特定接口的静态变量时，才会导致该接口的初始化。
    * 当初始化一个类的时候，要求它的所有父类都已经被初始化，但是这条规则不适用于接口
    * 在初始化一个类时，并不会先初始化它所实现的接口
    * 在初始化一个接口时，并不会先初始化它的父接口
    * 判断一个类是否有被加载
    
```java
interface A {
    public static Thread thread = new Thread() {
        {
            System.out.println("interface class instance");
        }
    };
}
```

* 若有一个类加载器能够成功加载Test类，那么这个类加载器被称为定义类加载器，所有能成功返回Class对象引用的类加载器（包括定义类加载器）都被称为初始类加载器。
* 双亲委托机制：在 Java 中，这种实现方式也称作 双亲委托。其实很简单，把 BootstrapClassLoader 想象为核心高层领导人， ExtClassLoader 想象为中层干部， AppClassLoader 想象为普通公务员。每次需要加载一个类，先获取一个系统加载器 AppClassLoader 的实例（ClassLoader.getSystemClassLoader()），然后向上级层层请求，由最上级优先去加载，如果上级觉得这些类不属于核心类，就可以下放到各子级负责人去自行加载。


