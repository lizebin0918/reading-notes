##MySQL系统参数调优学习

* `innodb_buffer_pool_size=4g` #设置成机器内存的四分之三（如果只有一个实例,16g内存，分给它12g都不为过，此内存容量是用于innodb存放数据和索引的，myisam存放索引的内存设置是`key_buffer_size=16M`（默认））

* `innodb_flush_log_at_trx_commit=2` #可以设置成"2"延迟刷新到磁盘，让系统自动去刷新。默认是1，每次提交都刷新内存，并刷写到数据文件中。

> 
> 0（延迟写）： log_buff  --每隔1秒，或者内存满了--> log_file  —实时—disk
> 
> 1（实时写，实时刷）： log_buff  —实时—>  log_file  —实时—> disk
> 
> 2（实时写，延迟刷）： log_buff  —实时—> log_file --每隔1秒，或者日志满了--> disk
> 

* `sync_binlog=0` #采用默认即可

> 
> sync_binlog=0，当事务提交之后，MySQL不做fsync之类的磁盘同步指令刷新binlog_cache中的信息到磁盘，而让Filesystem自行决定什么时候来做同步，或者cache满了之后才同步到磁盘。
> 
> sync_binlog=n，当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
> 

* `innodb_log_file_size=256M` #事务日志大小(包括redo.log和undo.log，可根据硬盘大小设施，可设置1G，如果出现故障，文件越大，恢复的时间就越长)

* `query_cache_size=0` #（默认为0）这样实际上没有禁用缓存的，而且这个值是不能在线修改，只能修改配置重启才生效

* `query_cache_type=OFF` #(on 表示开启缓存（默认），OFF表示禁用，DEMAIN表示按需缓存（select * from [tablename] sql cache）就会进行缓存)，因为mysql的缓存是以键值对存储的，根据sql hash化作为键，结果值作为值。

* `innodb_log_buffer_size = 8M` #此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据.MySQL开发人员建议设置为1－8M之间

* `innodb_flush_method = O_DIRECT`： 设置InnoDB同步IO的方式：

> 
> 1) Default – 使用fsync（）。
> 
> 2) O_SYNC 以sync模式打开文件，通常比较慢。
> 
> 3) O_DIRECT(推荐)，在Linux上使用Direct IO。可以显著提高速度，特别是在RAID系统上。避免额外的数据复制和double buffering（mysql buffering 和OS buffering）。
> 

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

> 
> InnoDB表空间解释: 设置了`innodb_file_table=1`参数，只是将ibdata1空间属于自己的表的索引和数据移到独立的.ibd文件去而已，其他的比如说undo 日志、插入缓冲、双写缓存等等依然在ibdata1里面
> 

 * `innodb_data_home_dir = /data` #mysql数据目录

 * `innodb_data_file_path = ibdata1:1G:autoextend`#默认值为1G，自动增长

 * `innodb_file_table=1` #InnoDB使用独立的表空间

> 开启慢查询（`mysqlsla`或者`mysqldumpslow`对慢查询日志进行分析）

* `long_query_time = 1` #查询执行时间大于1秒的语句

* `log_slow_queries = on` #开启慢查询记录

* `slow_query_log = on` #同上

* `slow_query_log_file=mysql-slow.log` #默认会放在data目录

> 对于sql语句中的排序，特别是order by 和 group by等语句需要用到临时表排序，`tmp_table_size`规定了内部内存临时表的最大值，每个线程都要分配，如果内存临时表超出了限制，MySQL就会自动地把它转化为基于磁盘的MyISAM表，存储在指定的tmpdir目录下：

* `tmp_table_size = 64M`

* `max_heap_table_size=64M`

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

##SQL语句技巧

* 如果出现子查询，是否可以转换成JOIN

* 大数量的分页查询优化，采用第三方控件（Sphinx、Lucence），语句模板：

``` sql
SELECT * FROM `t1` INNER JOIN ( SELECT id FROM `t1`ORDER BY id DESC LIMIT 935500,10) t2 USING (id);
```

* EXPLAIN SQL语句，避免`using temprory table` 和 `using filesort`

* 对于`like`的查询，分别有三种方式，并用对应函数替换(此方式只适用'varchar', 'text', 'blob', 等等),经过测试，替代方案可行：

``` sql
select * from a where a.column like '%key%'; -- 等价于instr(column, 'key') > 0
select * from b where b.column like 'key%'; -- 考虑建议多列索引
select * from c where c.column like '%key'; -- 考虑建议多列索引,通过反转函数:reverse(column) like reserver('%key')
```
##表设计技巧

* int类型占4个字节，默认int的长度为11位，即int(11)，但此长度与int存储最大值没任何关系，并不是说int(3)只能存3位的整数，int(11)和int(3)的区别只体现在左补零的层面上。
* 只要表数据设计日期字段，表字段的数据类型尽量采用：`datetime`,`date`,`time`(由于timestamp的日期范围有限，暂不采用)

> 原因：
> 1.如果采用固定格式的字符串，`insert`的时候，只需要传字符串即可，后续查询都用字符串比较，这样比较方便。
> 2.上一点的优点只能体现在双方约定好时间格式的情况下，如果一旦系统之间需要的日期格式跟内部系统的不一致，则需要进行转换，Java代码常见的查询都是返回`List`，如果存储的字符串日期，则可以通过sql或者遍历`List`元素转换，感觉写法笨重。
> 3.如果存储的格式为时间类型，Java代码获取的都是`Date`的子类，当获取到`List`，一般都需要序列化，而在序列化过程中，可以统一对`Date`字段以特定转换成字符串 


