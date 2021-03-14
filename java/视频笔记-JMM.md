### 视频笔记-JMM

####各个类加载器职责
* Bootstrap ClassLoader：JAVA_HOME/jre/lib/rt.jar、resources.jar、sun.boot.class.path路径下的包，用于提供jvm运行所需的包
* Extension ClassLoader：派生继承自java.lang.ClassLoader，父类加载器为启动类加载器。从系统属性：java.ext.dirs目录中加载类库，或者从JDK安装目录：jre/lib/ext目录下加载类库
* Application Classloader：它负责加载环境变量classpath或者系统属性java.class.path指定路径下的类库。它是程序中默认的类加载器，我们Java程序中的类，都是由它加载完成的。
* Custom Classloader:自定义类加载器

####类的加载过程
* 加载（loading）：查找并加载类的二进制数据（*.class文件）
* 准备（preparation）：为类的__静态变量__分配内存，并将其初始化为默认值，这里的默认值就是说引用对象为null，但是指对象就等于它们对应的默认值，这里的默认值如果是int就是为0，boolean就是为false，引用类型就是为null#这个class还没new，不会为实例变量分配内存
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
* 各个类加载器之间并不是继承关系，只是引用了parent属性而已
* 双亲委派是一个孩子向父亲方向，然后父亲向孩子方向的双亲委派过程。如果父亲能加载就让父亲加载，如果都不能就只能自己加载，如果我们自己写了一个String类，并且把这个类加载了，这样就会有安全问题。
* 自定义类加载器：实现jdk的ClassLoader抽象类，重写findClass方法，每次都会根据双亲委派原则加载Class。如果需要实现热部署，可以直接重写loadClass()，无需从父类需要class，重新生成ClassLoader并加载class即可


