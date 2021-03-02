###Java Stream 源码理解

```java
List<String> list = Arrays.asList("1", "2", "3");//1
Stream<String> stream = list.stream();//2
Stream<Integer> intStream = stream.map(s -> Integer.parseInt(s));//3
List<Integer> newList = intStream.collect(Collectors.toList());//4
```

* 第一行代码不用多少，构造一个List列表
* 第二行生成一个Stream对象，跟踪源码：

    * 首先构造了一个 Spliterator 对象，代码步骤如下:
    * `StreamSupport.stream(spliterator(), false);`
    * `Spliterators.spliterator(this, 0);//this表示当前集合 List `
* `return new ArrayListSpliterator<>(this, 0, -1, 0);`（`ArrayList`重写了`Collection`的`spliterator()`，而不是默认返回`IteratorSpliterator`）
    * 最终返回实例化`ArrayListSpliterator`（实现了`Spliterator`分割迭代器接口）里面就是一些初始化成员变量，完成后则直接返回
    * `Spliterator`主要接口包括:
    > `forEachRemaining(Consumer) {do { } while (tryAdvance(action));}`外部逻辑对容器元素进行遍历，这里传递的**行为**，所有迭代的过程代码都在内部完成，区别于传统的`Iterator`迭代(必须有`hasNext()`和`next()`这样就耦合性太低，`tryAdvance`同时完成了这两个操作)    
    > `estimateSize()`:评估容器大小
    > `trySplit()`:对容器进行切割(`long targetBatchSize = s.estimateSize() / (ForkJoinPool.getCommonPoolParallelism() * 8);`)
    
    * 以上代码返回`ArrayListSpliterator`实例，并作为参数传递到如下代码，步骤如下:`return new ReferencePipeline.Head<>(spliterator, StreamOpFlag.fromCharacteristics(spliterator), parallel);`
    * 最终构造的是`ReferencePipeline`实例，核心构造代码:
    
    ```java
    AbstractPipeline(Spliterator<?> source, int sourceFlags, boolean parallel) {
        this.previousStage = null;//没有前节点
        this.sourceSpliterator = source;//ArrayListSpliterator实例
        this.sourceStage = this;//当前的Head节点
        //...
   }
    ```
    
    
* 第三行代码，可以关联多种中间操作，跟踪`ReferencePipeline.map()`的源码
    
    ```java
    public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
    	return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE, StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
    		Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
    			return new Sink.ChainedReference<P_OUT, R>(sink) {
    				@Override
    				public void accept(P_OUT u) {
    					downstream.accept(mapper.apply(u));
    				}
    			};
    		}
    	};
    }
    ```
    
    * 第三行代码里面的`map(Function)`采用了函数式编程，等价于如下代码:

    ```java
    Stream<Integer> intStream = stream.map(new Function<String, Integer> () {
		@Override
		public Integer apply(String s) {
			return Integer.parseInt(s);
		}
	});
    ```
    
    * 上述两两段代码对照，得知`P_OUT`就是`String`，`Integer`对应的是`R`
    * `new StatelessOp(this, ....)`这里的`this`是表示上一行代码的stream(`Stream<String>-upstream`)，构造函数主要做的事情是构造一个**双向链表**
        * 构造一个新的`PipelineReference`
        * 对成员变量`this.previousStage`(实际就是`Stream<String>`)进行赋值
        * 把`Stream<String>`的`nextStage`赋值为当前的stream(`Stream<Integer>`)
        * `this.sourceStage = previousStage.sourceStage;`表示`PipelineReference.Head`，实际就是`Stream<String>`
        
    * 所有实现`AbstractPipeline`的实例，都要实现抽象方法，如果是`ReferencePipeline.Head`则对外抛出异常:

    ```java
    abstract Sink<E_IN> opWrapSink(int flags, Sink<E_OUT> sink);
    ```
    
    * 而`map(Function<String, Integer>)`，这里包装的是一个`Sink`对象，而作为参数传递进来的`sink`等价于`downstream`（下一个中间操作包装的`Sink实例`）代码如下:
    
    ```java
    @Override
    Sink<String> opWrapSink(int flags, Sink<Integer> sink) {
        return new Sink.ChainedReference<String, Integer>(sink) {
            @Override
            public void accept(String u) {
                downstream.accept(mapper.apply(u));
            }
        };
    }
    ```
    
    * `opWrapSink(int flags, Sink<R> sink)`适用于多个中间操作包装在一起，只有遇到终止操作的时候，**根据双向链表的特性，从后往前遍历节点，并且对`ChainedReference.downstream`成员变量进行赋值，而当前的`opWrapSink()`返回的`Sink<T>`实例的T类型，实际是上一个中间操作`Sink.mapper(Function<I, T>)`的输出类型**，一直往后到前遍历。类似于实现**装饰者模式**（IO流的实现方式），装饰者是通过**继承**和**委托**实现的，而`stream`是通过链表和函数式编程实现的，对于多种操作可以自由组装，实现解耦
    
* 第四行代码就是整个`Stream<Integer>`的终止操作(`foreach()`)，跟踪代码:

    * 传递收集器`Collectors.toList()`，实际返回的是实现`Collector`接口的实例，主要包含如下方法:
    
        >1.`Supplier<A> supplier();`:容器的工厂方法，实际返回元素的容器(`new ArrayList<Integer>()`)
        >2.`BiConsumer<A, T> accumulator();`:容器收集器，实际执行`List<Integer>::add`
        >3.`BinaryOperator<A> combiner();`:结合器，用于并行流(`parallelStream`)都有各自的`supplier`容器，最终会经过`combiner()`进行合并。PS:如果`Collector`包含了`Characteristics.CONCURRENT`表示容器是支持并发的，就算是`parallelStream`只会同时对应并发容器，最终不会执行`combiner()`，后续会使用例子说明。而这个例子不会执行`combiner()`
        >4.`Function<A, R> finisher();`:作用于`Supplier<A>`返回的容器跟实际用户的目标类型不一致，则采用此方法进行转换，如果类型一致需要直接返回，可以加入特征值`Collector.Characteristics.IDENTITY_FINISH`，表示容器类型无需转换，直接强转类型返回即可
  
    * 第四行代码`intStream.collect()`接收`Collector`实例，而且是串行流，则执行`container = evaluate(ReduceOps.makeRef(collector));`返回的是一个`TerminalOp`实例，最终的代码逻辑都是`evaluate()`执行的`terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));`
    > 第一步`sourceSpliterator(terminalOp.getOpFlags())`是返回`Stream`的`Spliterator`对象，实际就是`ReferencePipeline.Head`(sourceStage.spliterator)
    > 第二步执行`evaluateSequential(this, spiliterator)`:`this`表示当前的`PipelineHelper`(因为`AbstractPipeline`继承了`PipelineHelper`)，调用`return helper.wrapAndCopyInto(makeSink(), spliterator).get();`，`makeSink()`实际调用的是构造`TerminalOp`时，在方法体内的已经声明子类，并实现了这个方法

    ```java
    public static <T, I> TerminalOp<T, I>
makeRef(Collector<? super T, I, ?> collector) {
        Supplier<I> supplier = Objects.requireNonNull(collector).supplier();
        BiConsumer<I, ? super T> accumulator = collector.accumulator();
        BinaryOperator<I> combiner = collector.combiner();
        class ReducingSink extends Box<I>
                implements AccumulatingSink<T, I, ReducingSink> {
            @Override
            public void begin(long size) {state = supplier.get();}
            @Override
            public void accept(T t) {accumulator.accept(state, t);}
            @Override
            public void combine(ReducingSink other) {state = combiner.apply(state, other.state);}
        }
        return new ReduceOp<T, I, ReducingSink>(StreamShape.REFERENCE) {
            @Override
            public ReducingSink makeSink() {return new ReducingSink();}
            @Override
            public int getOpFlags() {return collector.characteristics().contains(Collector.Characteristics.UNORDERED)? StreamOpFlag.NOT_ORDERED:0;}
        };
    }
    ```
    
    > 第三步:`helper.wrapAndCopyInto(makeSink(), spliterator).get();//等价于:copyInto(wrapSink(Objects.requireNonNull(sink)), spliterator).get();//最后的get()执行的是Box类的get方法，返回容器本身`，执行的逻辑是**根据双向链表的特征，从后往前遍历**，当前参数`sink`是通过第二步的`new ReducingSink()`返回，并执行`opWrapSink()`返回上一个中间操作的`Sink`对象，一直进行"包装"，最终返回一个`Sink`实例，`wrapSink()`代码如下：
    
    ```java
    for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
        sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
    }
    ```
    
    > 第四步:`copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator)`，详见代码:
    1.获取元素个数
    2.对源容器元素进行遍历（根据源容器的类型，会有对应的遍历方式，具体需要看`Spiliterator`实现），`wrappedSink()` 执行顶层`Sink`的`accept(Function<In, Out>)`，实际只会调用一次，实现链式调用
    3.无操作，元素已经add到container中
    
    ```java
    wrappedSink.begin(spliterator.getExactSizeIfKnown());//1
    ((ArrayListSpliterator)spliterator).forEachRemaining(wrappedSink);//2
    wrappedSink.end();//3
    ```
 
 * 最后根据是否要对容器进行类型转换，如果`Collector`包含特征值`Collector.Characteristics.IDENTITY_FINISH`表示中间容器与最终容器一致，直接类型强转返回

* 整个`Stream`的类结构:`BaseSteam` --> `AbstractPipeline` --> `ReferencePipeline` --> `StatelessOp`/`StatefulOp`/`Head`(中间操作)
* 整个终止操作的类结构:`TerminalOp` --> `FindOp`/`MatchOp`/`ReduceOp`/`ForEachOp`
    



