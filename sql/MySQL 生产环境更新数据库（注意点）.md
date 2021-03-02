# MySQL 生产环境更新数据库（注意点）

一般来说，很多程序员操作数据库，不管是测试环境还是生产环境。执行`insert`/`update`/`delete`等DCL语句，很少会注意到事务开启与提交的问题，最多就是看多两眼语句，单凭感觉不会有错。可能最直接的原因，就是觉得自己不是DBA或者没有在这些情景上犯过错，就不会引起注意。

##1.情景
今天刚好需要手动更新数据，而且是相同的语句，只要把值修改就行。下面代码重复执行几次，每执行一次，就确认一次，直到操作完毕，再`commit`，总觉得这样没问题。语句大概如下:

```sql
    START TRANSACTION;-- 开启事务
    set autocommit = 0;-- 关闭事务自动提交
    set @a := 123;
    set @b := 456;
    set @c := 678;
    UPDATE table_a SET field_a = @a where field_c = @c;
    UPDATE table_b SET field_b = @b where field_c = @c;
    -- 先确认数据
    select field_a from table_a where field_c = @c;
    select field_b from table_b where field_c = @c;
```
##2.问题
在执行的过程中，我再打开另一个数据库连接，确认数据并未修改，但结果是数据已经发生改变。我在想，整个执行过程，我并没有执行`commit`，而且我执行了`set autocommit = 0`，为什么事务还会提交？难道跟事务隔离级别有关系？MySQL默认的事务隔离级别是__read repeated__，而不是__read uncommied__

##3.结论
经过本机测试，有了几点概括:
> - `START TRANSACTION`执行的时候，如果上一个事务未提交，会自动执行`commit`（感觉就是一个`connection`不会有两个未提交的事务）
> - 声明在`START TRANSACTION`与`commit`之间，无需`set autocommied = 0`，虽然`select @@autocommit = 1`，但实际上只要开启事务，只有`commit`才会生效，跟`autocommit`的值无关
> - 如果需要多次对数据库进行`delete`/`update`等操作，__只需开启一次事务（`START TRANSACTION`），执行语句，通过`select`确认，无误则执行`commit`__。
> - 以后操作生产数据库，除了确保参数正确，还是养成一个开启事务和提交事务的好习惯，还有一个很重要的问题，而且要说三遍的：
>   - `update`/`delete`语句一定要带过滤条件！
>   - `update`/`delete`语句一定要带过滤条件！
>   - `update`/`delete`语句一定要带过滤条件！
> - __PS:MySQL有一个变量`sql_safe_updates`默认是关闭的，如果此变量打开__
> - __1.update在有limit时是可以执行更新的，而delete严格一些，只要where条件为常量或者为空是会被拒绝的__
> - __2.update和delete在修改数据时，如果不带limit，需要where条件可以走索引，否则会报错__


