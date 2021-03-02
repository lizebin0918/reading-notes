#读书笔记《Java8 实战》

###1.`Collectors`的汇总操作
> `minBy()`|`maxBy()`
> `summarizingXxxx()`:汇总个数，综合，最小值，最大值，平均值
> `join`:对集合进行字符串拼接
> `reducing`最终武器

###2.`collect`与`reduce`的区别
> `collect`方法的设计就是要改变容器，从而积累要输出的结果
> `reduce`旨在把两个值结合起来生成一个新值，它是一个不可变的规约，而且不能支持并发执行

