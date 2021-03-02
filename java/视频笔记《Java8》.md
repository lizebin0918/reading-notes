# 视频笔记《Java8》

## `Lambda`表达式

* 函数式接口
    - 如果接口只有一个抽象方法，可以有多个默认方法，那么该接口就是一个函数式接口
    - 如果在接口上声明了`@FunctionalInterface`，那么编译器就会按照函数式接口的定义来要求接口
    - 如果接口只有一个抽象方法，但没有声明`@FunctionalInterface`，那么编译器依旧会将该接口看作是函数式接口
    - 如果接口声明中包含了`java.lang.Object`方法，不算一个新的抽象方法，因为实现该接口的类默认就是继承`Object`，所以就会有了默认的实现
    
* 函数式编程
    过去的编程模型都是定义好行为，提供调用。但是函数式编程只是提供一个行为的描述符，调用者只需把行为传递即可。函数里面不实现任何行为，只需调用者自己传递行为，提供了一种更高层次的抽象化
    
* 在很多语言中，`Lambda`表达式的类型是函数，但是在Java中，`Lambda`表达式是对象，它们必须依附于一类特别的对象类型--函数式接口（`@FunctionalInterface`）

* 传递行为，而不是传递值

    - `([type1] argument1, [type2] argument2...) -> {body}` 
    - `(int i, int j) -> {return i + j;}`
    - `() -> {System.out.println("hello world");}`
    - `() -> 42//等价于: () -> {return 42;}`
    - 一个`Lambda`表达式可以有零个或者多个参数
    - 参数和返回值类型可以通过上下文来判断，类似于`<>`语法

* 高阶函数:如果一个函数接受一个函数作为参数，或者返回一个函数作为返回值，那么该函数就叫做高阶函数

* 过去采用面向对象的思维定义四则运算，需要定义四个方法:`add/subtract/multipy/divide`分别写实现，通过函数式编程写法:

```java
    /**
     compute(1, 2, (a, b) -> a + b);
     compute(1, 2, (a, b) -> a - b);
     compute(1, 2, (a, b) -> a * b);
     compute(1, 2, (a, b) -> a / b);
     */
    public static int compute(int a, int b, BiFunction<Integer, Integer, Integer> function) {
   return function.apply(a, b);
}
```

* `Function`包括`compose(Function)`之前执行/`andThen(Function)`之后执行和`BiFunction`包括`andThen(Function)`之后执行，用于哪些实际情景？
* `Optional`用于方法返回值，强制判断元素是否为空（不能用于成员变量类型和方法参数类型），推荐写法:`optional.ifPresent(item -> System.out::println)`；`Optional`提供了`map()`/`flatMap()`方法，下面两种写法等价:

```java
//String storeCode = Objects.isNull(follower.getStoreId()) ? null : cmsCustomerStoreRepository.selectByPrimaryKey(follower.getStoreId()).orElseThrow(() -> new ServiceException("门店不存在")).getCode();
//类似于第一个如果不为空则取第一个值，反之取第二个值
String storeCode = Optional.ofNullable(follower.getStoreId()).map(id -> cmsCustomerStoreRepository.selectByPrimaryKey(id).orElseThrow(() -> new ServiceException("门店不存在")).getCode()).orElse(null);
```

* `Lambda`表达式四种方法引用，实际就是编写函数体，跟函数类型无关，参数有关

    - 类名::静态方法名
    - 实例变量名称::实例方法名
    - （重点注意）类名::实例方法名
    - 构造方法引用（有参或者无参）：类名::new

```java
public class Main {

    public static void main(String[] args) {
        Main main = new Main();
        //Person 集合
        List<Person> persons = Arrays.asList(new Person("p1"), new Person("p2"), new Person("p3"));
        //类名::静态方法名
        persons.sort(Person::staticNameComparator);
        //实例名::实例方法名
        persons.sort(main::nameComparator_1);
        //类名::实例方法名
        persons.sort(Person::nameComparator_2);//相当于把第一个变量作为调用者，后面的所有变量都作为该方法的参数传递
        //类名::new
        Person[] personArray = persons.stream().toArray(Person[]::new);
    }

    public int nameComparator_1(Person p1, Person p2) {
        return p1.name.compareToIgnoreCase(p2.name);
    }

    private static class Person {
        String name;

        public Person(String name) {
            this.name = name;
        }

        public static int staticNameComparator(Person p1, Person p2) {
            return p1.name.compareToIgnoreCase(p2.name);
        }

        public int nameComparator_2(Person p) {
            return this.name.compareTo(p.name);
        }
    }
}
```

* jdk为什么增加默认方法？为了保证新代码能向后兼容，例如:`List.sort()`

* `Stream`流元素由3个元素构成:源/零个或者多个中间操作/终止操作

    - 流操作分类:惰性求值(返回的都是`Stream<T>`)/及早求值
    - 惰性求值（也叫做中间操作）:`stream.xxx().yyy().zzz()`；只有执行`count()`才会执行中间操作的代码
    - 及早求值:`stream.xxxx().yyyy().zzz().count()`
    
* `Stream.collect()`收集器

    ```java
    List<String> list = Stream.of("a", "b", "c").collect(Collectors.toList());
            List<String> list1 = Stream.of("a", "b", "c").collect(Collectors.toCollection(LinkedList::new));
    
    ```
    
    等价于以下写法
    
    ```java
    List<String> list2 = Stream.of("a", "b", "c").collect(
          () -> new ArrayList<String>(),
          (pList, item) -> pList.add(item),
          (pList, newList) -> pList.addAll(newList)
    );
    
    ```
    
* `Stream.map()/Stream.flatMap()`一对一映射，把变量转换成其他对象/一对多映射，实际是扁平化操作

    ```java
    List<String> s = Stream.of("abb", "bcc", "cdd").flatMap(item -> Stream.of(item.split(""))).collect(Collectors.toList());
    System.out.println(s);//[a, b, b, b, c, c, c, d, d]
    ```
    
* `Stream-API`
    - `Stream`方式属于**内部迭代**，而且属于描述性表达，更通俗易懂，类似于`SQL`，而传统的jdk遍历方式`for() {}`属于**外部迭代**，偏向命令式表达。
    - 内部迭代是把`Stream`函数式接口的业务逻辑代码整合在一起，就类似英语考试的完形填空，而外部迭代则类似英语作文。所以`Stream`把业务逻辑整合在一起，还可以提供**并行化操作**(`Stream.parallelStream()`/操作是无状态的)
    - 集合关注的是数据存储本身，而流关注的是对数据的计算
    - `Collectors.groupingBy`(分组)/`Collectors.patitionBy`(分区是特殊的分组，只能分为两组)

* `Stream`深入解析
    - `Collector<T, A, R>`:**`T`表示实际元素类型/`A`每次收集元素的容器类型/`R`最终返回的类型**，它是一个可变的汇聚操作，将输入元素累计到一个可变的结果容器中；它会在所有元素都处理完毕后，将累计的结果转换为最终的展示，支持串行和并行两种方式执行
    - `Collectors`提供了常见的汇聚实现
    - `collect()`需要四个元素:
    
        > 一个新的容器承载结果元素:`Supplier<A> supplier()`
        > 将一个新的元素合并到新的容器中:`BiConsumer<A, T> accumulator()`
        > 合并两个结果容器（有可能是在原来的容器中合并，有可能是生成一个新的容器，只有在并发流的情况下才会有意义）:`BinaryOperator<A> combiner();`
        > 完成器:最终把结果结合成一个新的类型（可选）:`Function<A, R> finisher();`
    - 为了确保串行与并行操作结果的等价性，`Collector`函数需要门族两个条件:
        > identity:同一性-`a == combiner.apply(a, supplier.get())`相当于`(List<String> list1, List<String> list2) -> {list1.addAll(list2);return list1;}//实际就是元素积累`
        > associativity:结合性
        
        ```java
        //串行
        A a1 = supplier.get();
        accumulator.accept(a1, t1);
        accumulator.accept(a1, t2);
        R r1 = finisher.apply(a1);  // result without splitting
        //并行 
        A a2 = supplier.get();
        accumulator.accept(a2, t1);
        A a3 = supplier.get();
        accumulator.accept(a3, t2);
        R r2 = finisher.apply(combiner.apply(a2, a3));  // result with splitting        
        ```
    
    - `Collector`实现过程:
       
       ```java
        R container = collector.supplier().get();
        for (T t : data)
            collector.accumulator().accept(container, t);
        return collector.finisher().apply(container);       
        ``` 
        

    - `Collector.Characteristics`特性解析:
    > `CONCURRENT`:（并行流）表示多个线程操作一个结果容器（表示结果容器具有并发特性）；若不声明，则多个线程有多个容器，最终由`combiner()`进行合并
    > `UNORDERED`:表示结果容器是无序的，例如`Set`
    > `IDENTITY_FINISH`:表示中间容器类型与结果容器类型一致，则不会调用`finsher()`，直接通过强转返回
    
    - `Collectors`源码分析:

    > 计算流元素的汇总，为什么是采用数组？只有传递引用才能在`accumulator()`实现累加
    
    ```java
    public static <T> Collector<T, ?, Integer>
    summingInt(ToIntFunction<? super T> mapper) {
        return new CollectorImpl<>(
                () -> new int[1],
                (a, t) -> { a[0] += mapper.applyAsInt(t); },
                (a, b) -> { a[0] += b[0]; return a; },
                a -> a[0], CH_NOID);
    }
    
    //示例代码
    List<Integer> list7 = Arrays.asList(1, 2, 3, 4);
    System.out.println(Arrays.toString(list7.stream().collect(() -> new int[1], (a, b) -> a[0] += b, (a, b) -> {a[0] += b[0];})));//返回值为[10]
    System.out.println(list7.stream().collect(() -> 0,
    (a, b) -> a += b.intValue(),
    (a, b) -> a += b.intValue()));//返回值为0
    ```
    
    > `groupingBy()源码分析`:实际就是重新构造`Supplier`/`accumulator`/`combiner`
    
    ```java
    public static <T, K> Collector<T, ?, Map<K, List<T>>>
        groupingBy(Function<? super T, ? extends K> classifier) {//T表示分类器入参类型，K表示分类之后的类型
        return groupingBy(classifier, toList());
    }
    ```
    
* `Stream`源码分析

    >`Stream`与`Collection`的区别在于前者是对元素进行计算，而且这种计算是无状态的（先算第一个和先算第100个）结果都是一样的，而后者是对元素进行存储和便于访问
     
* `Spliterator解析`
    
    >`Spliterator.trySplit()`:通过对容器进行切割成若干份，直到粒度足够小，返回一个新的`Spliterator`，返回`null`则表示分割完成 
    --P38 1.什么时候才会用到 `trySplit()` 方法
    --2.如果集合是无穷个的话，`trySplit()` 返回的是`null`，
    
    >`Spliterator.tryAdvance()`:通过传递行为`Consumer`，实现了传统的`Iterator`的`hasNext()`和`next()`方法的合并
    
    >P45
    >所有中间操作，返回的都是类似于`StatelessOp`的实例，最终都是由终止操作执行，通过`TerminalOp`的实例。
    >`abstract<P_IN, S extends Sink<P_OUT>> S wrapAndCopyInto(S sink, Spliterator<P_IN> spliterator);`:所有操作应用到`spliterator`（相当于数据来源）对象上，之后把结果输入到`Sink`里面
    >`wrapSink(Sink<E_OUT> sink)`完成所有中间操作的串联，包装成一个`Sink`
    >通过链表对中间操作进行串联，详细看代码:
    
```java
    
//代码片段一(AbstractPipeline.java):通过链表从后往前遍历，对中间操作进行串联
@Override
@SuppressWarnings("unchecked")
final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
   Objects.requireNonNull(sink);

   for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
        //opWrapSink定义:Sink<P_OUT> opWrapSink(int flags, Sink<R> sink)，表示上一个操作的结果就是下一操作的参数 
        sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
   }
   return (Sink<P_IN>) sink;
}
    
//代码片段二(ReferencePipeline.java):最终包装的中间操作，类似于实现装饰者模式（IO流的实现方式）
return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
       @Override
       Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
           //sink 实际就是 downstream
           return new Sink.ChainedReference<P_OUT, R>(sink) {
               @Override
               public void accept(P_OUT u) {
                   downstream.accept(mapper.apply(u));
               }
           };
       }
   };
```

>串联包装完中间操作之后，执行`copyInto()`

```java
@Override
final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
   Objects.requireNonNull(wrappedSink);

   if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
       wrappedSink.begin(spliterator.getExactSizeIfKnown());
       spliterator.forEachRemaining(wrappedSink);
       wrappedSink.end();
   }
   else {
       copyIntoWithCancel(wrappedSink, spliterator);
   }
}
```

>短路操作(`short-circuiting`)：表示只要查到符合条件的元素，后续的遍历就不再执行
    
    
        
        
        
        


