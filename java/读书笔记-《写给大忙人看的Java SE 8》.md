# 读书笔记:《写给大忙人看的Java SE 8》

## 一、lambda表达式
###详见
[http://www.cnblogs.com/figure9/archive/2014/10/24/4048421.html](http://www.cnblogs.com/figure9/archive/2014/10/24/4048421.html)
###语法
* 匿名类写法:参数、箭头(->)、表达式
* 方法引用写法:
    + 对象::实例方法
    + 类::静态方法
    + 类::实例方法

```java
//定义列表
List<String> l = Arrays.asList("a", "b", "d");
/*遍历列表, forEach() 接收一个 @FunctionalInterface 标记的接口 Consumer，并把 String 作为参数传递，下面三种写法效果都是一样的*/
//创建内部类写法
l.forEach(new Consumer<String>() {
  @Override
  public void accept(String s) {
      System.out.println(s);
  }
});

//lambda 写法，如果表达式只有一句，可以省略"{}"
l.forEach((String s) -> {
  System.out.println(s);
});

//lambda 的 方法引用写法
l.forEach(System.out::println);
```
###重点
* `@FunctionalInterface`标记的接口中，必须只有一个抽象方法
* `this`区分

```java
    //第一种情况 this 指向调用者的实例，可以理解 lambda 是一个表达式，而不是真正生成内部类
    Runnable r = () -> System.out.println(this.getClass());
    new Thread(r).start();

    //第二种情况  this 是内部类Runnable实例
    new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(this.getClass());
            }
    }).start();
```
* `lambda`引用变量的作用域增强，相当于内部类需要引用外部变量`a`，那`a`则需要`final`关键字声明

```java
    //name自动升级为final变量
    Callable<String> helloCallable(String name) {
        String hello = "Hello";
        //name = "1";导致程序编译失败
        return () -> hello + ", " + name;
    }
```

##二、`Stream`用法
###定义
* `Stream`是元素的集合，这点让`Stream`看起来用些类似`Iterator`；
* 可以支持顺序和并行的对原`Stream`进行汇聚的操作

![Alt text](http://img04.taobaocdn.com/imgextra/i4/90219132/T2ycFgXQ8XXXXXXXXX_!!90219132.jpg)

###语法

```java
//1.distinct（有状态转换，去重的过程中必须记录之前的元素）: 对于Stream中包含的元素进行去重操作（去重逻辑依赖元素的equals方法），新生成的Stream中没有重复的元素
//2.filter: 对于Stream中包含的元素使用给定的过滤函数进行过滤操作，新生成的Stream只包含符合条件的元素；
//3.map: 对于Stream中包含的元素使用给定的转换函数进行转换操作，新生成的Stream只包含转换生成的元素。
//4.flatMap：和map类似，不同的是其每个元素转换得到的是Stream对象，会把子Stream中的元素压缩到父集合中
//5.peek: 生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数；
//6.limit: 对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素；
//7.skip: 返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream
//8.聚合操作:count(), min(), max(), collect(), 一旦执行聚合操作，将会关闭Stream
//静态工厂方法
Stream<Integer> integerStream = Stream.of(1, 2, 3, 5, 5);
//去重
System.out.println(integerStream.distinct().collect(Collectors.toList()));
//Collection 转换 Stream
Stream<String> stringStream = Arrays.asList("1", "2", "3", "4").stream();
//排除 null 
List<Integer> nums = new ArrayList<>(Arrays.asList(1,1,null,2,3,4,null,5,6,7,8,9,10));
int size = nums.size();
List<Integer> numsWithoutNull = nums.stream().filter(num -> num != null).
collect(Collectors.toCollection(() -> new ArrayList<Integer>((int)(size * 1.5))));
System.out.println(numsWithoutNull);
```
###重点

* `Stream`与`Collection`区别
    + `Stream`自己不会存储元素。元素可能而被存储在底层的集合中，或者根据需要产生出来。
    + `Stream`操作符不会改变源对象。相反，它们会返回一个持有结果的新`Stream`
    + `Stream`操作符可能是延迟执行的。这意味着它们会等到需要结果的时候才执行
    
* 当我们使用`Stream`时，你会通过三个阶段来建立一个流水线：
    + 创建一个`Stream`
    + 在一个或多个步骤中，指定将初始`Stream`转换为另一个`Stream`的中间操作
    + 使用一个终止操作来产生一个结果。该操作会强制它之前的延迟操作立即执行。在这之后，该`Stream`就不会再被使用了

##三、`Optional<T>`类型

###定义
这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

###语法及API
[http://www.importnew.com/6675.html](http://www.importnew.com/6675.html)

##四、新的日期和时间API

###语法及API
* 所有`java.time.*`对象都是不可变的，线程安全的（`LocalDate`, `LocalTime`, `LocalDateTime`）
* `instant`是时间线上的一个点（与Date类似）
* 在Java事件中，每天都是86400秒（即没有闺秒）
* `Duration`表示两个`Instant`之间的时间
* `TempoalAdjuster`的方法处理常用的日历计算
* 使用`DateTimeFormattter`来格式化和解析日期和时间

##五、杂项改进

###1.`String.join()`多个字符串以指定字符串分割底层依赖`StringJoiner`

> `String.join(‘,’, String[])`只能接受字符`CharSequence`数组，比较郁闷。推荐使用`commoms:StringUtils.join`或者`guava:Joiner.on(',').join(intArray)`

```java
String spliter = ",";
String a = "a";
String b = "b";
System.out.println(String.join(spliter, a, b));//a,b
System.out.println(String.join(",", new String[]{"a", "b"}));//a,b
System.out.println(String.join(",").length());//0
```
###2.`Comparator.comparing()`改进
```java
//使用Stream方式找到数组最大值
int[] array2 = new int[]{1, 3, 5, 6, 7, 8, -2, -3, 100, 99, 97};
int max1 = IntStream.of(array2).max().getAsInt();//效率最高

List<Integer> list = new ArrayList<>(Arrays.asList(new Integer[]{1, 3, 5, 6, 7, 8, -2, -3, 100, 99, 97}));
Optional<Integer> max2 = list.stream().reduce(Integer::max);//分解数组，与 collect 对应
Optional<Integer> max3 = list.stream().collect(Collectors.maxBy(naturalOrder()));
int max4 = list.stream().collect(Collectors.summarizingInt(Integer::intValue)).getMax();

//4.根据字符串排序
String[] stringArray = new String[]{"aa", "bbb", "cccc", "dd"};
//自然排序
System.out.println(Arrays.toString(Stream.of(stringArray).sorted(Comparator.naturalOrder()).toArray()));//[aa, bbb, cccc, dd]
//长度排序
System.out.println(Arrays.toString(Stream.of(stringArray).sorted(Comparator.comparing(String::length, reverseOrder())).toArray()));//[cccc, bbb, aa, dd]
```
###3.集合新增方法
* 3.1.`Iterable.forEach()`

```java
List<String> list2 = new ArrayList<>(Arrays.asList("a", "c", "b"));
list2.forEach(System.out::println);
System.out.println(list2);
```
> 注意：通过源码可知，不同的数据结构，会有不同的遍历方式

```java
/*Iterable.forEach() 默认实现:
default void forEach(Consumer<? super T> action) {
  Objects.requireNonNull(action);
  for (T t : this) {
      action.accept(t);
  }
}
*/
/*ArrayList 实现
default void forEach(Consumer<? super T> action) {
  Objects.requireNonNull(action);
  final int expectedModCount = modCount;
  @SuppressWarnings("unchecked")
  final E[] elementData = (E[]) this.elementData;
  final int size = this.size;
  for (int i=0; modCount == expectedModCount && i < size; i++) {
      action.accept(elementData[i]);
  }
  if (modCount != expectedModCount) {
      throw new ConcurrentModificationException();
  }
}
*/
```
###4.`Collection.removeIf()`

```java
List<Integer> list3 = new ArrayList<>(Arrays.asList(new Integer[]{1, 3, 5, 6, 7, 8, -2, -3, 100, 99, 97}));
list3.removeIf((i) -> i.intValue() == 1);
//remove 1 from list [3, 5, 6, 7, 8, -2, -3, 100, 99, 97]
System.out.println("remove 1 from list " + list3);
```

###5.`List.replaceAll()/sort()`

```java
List<Integer> list4 = new ArrayList<>(Arrays.asList(new Integer[]{1, 6, 3, 7, 5}));
list4.replaceAll((x) -> x + 1);
//element increment [2, 7, 4, 8, 6]
System.out.println("element increment " + list4);
list4.sort(Comparator.reverseOrder());
//sorted [8, 7, 6, 4, 2]
System.out.println("sorted " + list4);
```
###6.`forEach/replace/replaceAll/remove/putIfAbsent/compute/computeIf(Absent|Present)/merge`

```java
Map<String, String> m3 = new HashMap<>();
m3.put("k1", "v1");
m3.put("k2", "v2");
m3.put("k3", "v3");
//Map:forEach/replace/replaceAll/remove/putIfAbsent/compute/computeIf(Absent|Present)/merge

//forEach
m3.forEach((k, v) -> System.out.println(k + "->" + v));

//replace
String k1 = "k1";
m3.replace(k1, m3.get(k1), "v4");//只有键值对同时符合才会执行
/* 等价于
* if (map.containsKey(key) && Objects.equals(map.get(key), value)) {
*     map.put(key, newValue);
*     return true;
* } else
*     return false;
* }
* */

//putIfAbsent
m3.putIfAbsent("k4", "v4");//非线程安全

//compute/computeIf(Absent|Present) 同理
m3.compute("k4", (k, v) -> k.concat(v) + v);//两个参数分别是key和value，返回值替换旧的value
System.out.println("k4'value = " + m3.get("k4"));

//merge
m3.merge("k4", "null", (v1, v2) -> v2 + v1);//v1为k4键的值，v2为声明的参数并传入
System.out.println("k4'value = " + m3.get("k4"));

//merge 合并两个Map,并把相同key的value相加
Map<String, Integer> m1 = new HashMap<String, Integer>();
m1.put("a", new Integer(1));
m1.put("b", new Integer(2));
m1.put("c", new Integer(3));
Map<String, Integer> m2 = new HashMap<>();
m2.put("a", new Integer(101));
m2.put("b", new Integer(102));

m1.forEach((key1, value1) -> {
    m2.merge(key1, value1, Integer::sum);
});
System.out.println(m2);
```
###7.`Collections.checkedList()`防止类型转换

```java
List<Integer> list1 = Arrays.asList(1, 3, 1);
Collections.checkedList(list1, Integer.class);
//禁止泛型转换
list1.add(1);//将会抛异常
```
###8.`Comparator`新增的静态方法
```java
//字符串根据长度排序(小到大)，如果长度相同，根据自然顺序的倒序排列
String[] stringArray1 = new String[]{"a", "b", "cc", null, "dd", "aa", "eee", "ee", "e"};
//thenComparing 可以添加多个比较条件
Arrays.sort(stringArray1, nullsLast(Comparator.comparing(String::length).thenComparing(Comparator.reverseOrder())));
//stringArray1 = e,b,a,ee,dd,cc,aa,eee,null
System.out.println("stringArray1 = " + String.join(",", stringArray1));
```
###9.`Base64`
```java
Base64.Encoder encoder = Base64.getMimeEncoder();
String s = "lizebin";
String encoded = encoder.encodeToString(s.getBytes(StandardCharsets.UTF_8));
System.out.println("encoded = " + encoded);

Base64.Decoder decoder = Base64.getMimeDecoder();
System.out.println("encoded = " + new String(decoder.decode(encoded), StandardCharsets.UTF_8));
```
###10.`Files`读取文件返回`Stream<String>`
```java
//try-with-resource 语句的 catch 和 finally 分支，而且都会在【关闭资源】之后执行
//如果在finally代码块close()，可能会抛出异常，但这样就保证了 close() 抛出的异常能够被捕获到
//等价于:try (Stream<String> lines = Files.lines(Paths.get("/Users/lizebin/work/yao_dian/购药记录.sql"), StandardCharsets.UTF_8).onClose(() -> System.out.println("stream closed"))) {
try (Stream<String> lines = Files.lines(Paths.get("/", "Users","lizebin","work", "yao_dian", "购药记录.sql"), StandardCharsets.UTF_8).onClose(() -> System.out.println("stream closed"))) {
    lines.forEach((line) -> System.out.println(line));
    System.out.println("wait");
} catch (IOException e) {
    //获取二次异常
    //Throwable[] secondaryExceptions = e.getSuppressed();
    e.printStackTrace();
} finally {
    System.out.println("finally execute");
}
```
###11.`Path`|`Paths`|`Files`用法

* `Path`与`File`对象类似，`Path`对象并不一定非要对应一个世纪存在的文件。它只是一个文件名的抽象序列。要创建一个文件，首先要构造一个路径，然后调用其他方法来创建对应的文件。
* `java.io.File`能实现的功能，都可以用`Files`|`Path`|`Paths`组合完成，JDK更推荐后者

```java
Path path1 = null;
//下面两句等价
//Path path1 = Paths.get("/", "Users", "lizebin", "Desktop", "test.txt");
//path1 = Paths.get("/Users/lizebin/Desktop/test.txt");
//互相转换:File <--> Path
path1 = new File("/Users/lizebin/Desktop/test.txt").toPath();
File file1 = path1.toFile();

//读取文件
byte[] bytes = Files.readAllBytes(path);
//往文件写内容
Files.write(path, Stream.of("1", "2", "3").collect(Collectors.toList()), StandardCharsets.UTF_8, StandardOpenOption.APPEND);
//转换成原生IO流操作
/*try (
   InputStream is = Files.newInputStream(path);
   BufferedReader br = Files.newBufferedReader(path);) {
}*/

//Files.deleteIfExists(path1);

Path path2 = Paths.get("/Users/lizebin/Desktop/test");
//Files.createDirectory(path2);//如果目录存在将会报错
//Files.createDirectories(path2);//创建多级目录 mkdir -p [dir]
//创建临时文件(随机数)--4799633418917146900.tmp
//Files.createTempFile(Paths.get("/Users/lizebin/Desktop/"), "", ".tmp");
```
###12.`Objects`API
* `Objects.equals()`|`Objects.deepEquals()`

```java
//过去写法:a.equals(b)，推荐使用:Objects.equals(a, b);
//对象是数组类型，应该采用 deepEquals() 实际调用的是 Arrays.deepEquals()
String[] firstArray  = {"a", "b", "c"};
String[] secondArray = {"a", "b", "c"};
String[] thirdArray = {"a", "c", "b"};//顺序不一样
String s1 = new String("a");
String s2 = new String("a");
System.out.println(Objects.equals(s1, s2));//true
System.out.println(firstArray.equals(secondArray) );//false
System.out.println(Objects.equals(firstArray, secondArray) );//false
System.out.println(Objects.equals(firstArray, thirdArray));//false
System.out.println(Arrays.deepEquals(firstArray, secondArray));//true
System.out.println(Objects.deepEquals(firstArray, secondArray));//true
System.out.println(Objects.deepEquals(firstArray, thirdArray));//false
System.out.println(Objects.equals(null, null));//true
```
* `Objects.toString()`

```java
//toString()方法，并且可以设置默认值
System.out.println(Objects.toString(1, "2"));//1
System.out.println(Objects.toString(null, "2"));//2
//对于四类八种类型转字符串是有特定方法的
System.out.println(Long.toString(1));//看源码可知，等价于 Objects.toString();
```
###13.多个字段的`hashCode`

```java
//原写法
return 31 * Objects.hashCode(firstName) + Objects.hashCode(lastName);
//多字段hashCode
return Objects.hash(firstName, lastName);
```
###14.包装类型提供`compare()`

```java
 //Person.age 排序
Person[] persons = new Person[]{new Person(10), new Person(14), new Person(15), new Person(13), new Person(28), new Person(26)};
Comparator<Person> cp = new Comparator<Person>() {
  @Override
  public int compare(Person o1, Person o2) {
      //return o1.getAge() - o2.getAge();//这种方式会导致溢出风险
      //更优做法-Integer, Byte, Long, Short, Boolean 都各自提供了 compare() 静态方法
      return Integer.compare(o1.getAge(), o2.getAge());
  }
};
Arrays.sort(persons, cp);
//简化写法
Arrays.sort(persons, Comparator.comparing(Person::getAge, Integer::compare));
System.out.println(Arrays.toString(persons));
```








