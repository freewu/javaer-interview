## mysql 隔离级别

```
事务隔离级别					脏读	不可重复读	幻读
读未提交（read-uncommitted）	 是		是		是
不可重复读（read-committed）	否		是		是
可重复读（repeatable-read）	 否		否		是
串行化（serializable）	     否		否		否

不可重复读（read-committed）大多数据库系统默认的隔离级别(不是mysql的默认隔离级别)
可重复读（repeatable-read）mysql的默认隔离级别
```

## mysql复制原理

```
```

## mysql锁的类型

```
属性: 共享锁 排他锁
粒度: 行级锁(innodb) 表级锁(innodb myisam) 页级锁(innodb) 记录锁 间隙锁 临时锁

默认情况下，表锁和行锁都是自动获得的， 不需要额外的命令。

# 全局锁 对整个数据库实例加锁
	Flush tables with read lock

# 表级锁
	lock tables t1 read,t2 wirte;

# 行级锁
    显式锁定 ：
    select ... lock in share mode //共享锁 
    select ... for update //排他锁 

```

## 聚簇索引和非聚簇索引

```
聚簇索引:
	找到了索引就找到了需要的数据，那么这个索引就是聚簇索引，所以主键就是聚簇索引，修改聚簇索引其实就是修改主键。
	
非聚簇索引:
	索引的存储和数据的存储是分离的，也就是说找到了索引但没找到数据，需要根据索引上的值(主键)再次回表查询,非聚簇索引也叫做辅助索引。
	
主键一定是聚簇索引，MySQL的InnoDB中一定有主键，即便研发人员不手动设置，则会使用unique索引，没有unique索引，则会使用数据库内部的一个行的id来当作主键索引,其它普通索引需要区分SQL场景，当SQL查询的列就是索引本身时，我们称这种场景下该普通索引也可以叫做聚簇索引，MyisAM引擎没有聚簇索引。
```

## 索引的基本原理

```
```

## 索引数据结构有哪些

```
数据库索引，是数据库管理系统中一个排序的数据结构，主要有 B树索引、Hash索引两种

B树索引

Hash索引
	哈希索引就是采用一定的哈希算法，把键值换算成新的哈希值，检索时不需要类似B+树那样从根节点到叶子节点逐级查找，只需一次哈希算法即可立刻定位到相应的位置，速度非常快。
	
Hash索引只支持等值比较，例如使用=，IN( )和<=>。对于WHERE price>100并不能加速查询
Hash 索引无法被用来避免数据的排序操作
Hash 索引不支持多列联合索引的最左匹配规则
Hash 索引在任何时候都不能避免表扫描
```

## mysql 执行计划

```
explain + sql 语句

select_type	说明
    SIMPLE	简单查询
    PRIMARY	最外层查询
    SUBQUERY	映射为子查询
    DERIVED	子查询
    UNION	联合
    UNION RESULT	使用联合的结果
    
table : 正在访问的表名
possible_keys : 可能使用的索引
key : 真实使用的
key_len : MySQL中使用索引字节长度
rows : mysql 预估为了找到所需的行而要读取的行数

type:
	ALL	全数据表扫描
    index	全索引表扫描
    RANGE	对索引列进行范围查找
    INDEX_MERGE	合并索引，使用多个单列索引搜索
    REF	根据索引查找一个或多个值
    EQ_REF	搜索时使用primary key 或 unique类型
    CONST	常量，表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次。
    SYSTEM	系统，表仅有一行(=系统表)。这是const联接类型的一个特例。
    
性能：all < index < range < index_merge < ref_or_null < ref < eq_ref < system/const 性能在 range 之下基本都可以进行调优

extra:
	Using index	此值表示mysql将使用覆盖索引，以避免访问表。
	Using where	mysql 将在存储引擎检索行后再进行过滤，许多where条件里涉及索引中的列，当(并且如果)它读取索引时，就能被存储引擎检验，因此不是所有带where子句的查询都会显示“Using where”。有时“Using where”的出现就是一个暗示：查询可受益于不同的索引。
	Using temporary	mysql 对查询结果排序时会使用临时表。
	Using filesort	mysql会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。mysql有两种文件排序算法，这两种排序方式都可以在内存或者磁盘上完成，explain不会告诉你mysql将使用哪一种文件排序，也不会告诉你排序会在内存里还是磁盘上完成。
	Range checked for each record(index map: N)	没有好用的索引，新的索引将在联接的每一行上重新估算，N是显示在possible_keys列中索引的位图，并且是冗余的
```

## Myisam和Innodb区别

```
1 MyISAM不支持事务，而InnoDB支持。InnoDB的AUTOCOMMIT默认是打开的，即每条SQL语句会默认被封装成一个事务，自动提交，这样会影响速度，所以最好是把多条SQL语句显示放在begin和commit之间，组成一个事务去提交。
2 InnoDB支持数据行锁定，MyISAM不支持行锁定，只支持锁定整个表。即MyISAM同一个表上的读锁和写锁是互斥的，MyISAM并发读写时如果等待队列中既有读请求又有写请求，默认写请求的优先级高，即使读请求先到，所以MyISAM不适合于有大量查询和修改并存的情况，那样查询进程会长时间阻塞。因为MyISAM是锁表，所以某项读操作比较耗时会使其他写进程饿死。
3 InnoDB支持外键，MyISAM不支持。
4 InnoDB的主键范围更大，最大是MyISAM的2倍。
5 InnoDB不支持全文索引，而MyISAM支持。全文索引是指对char、varchar和text中的每个词（停用词除外）建立倒排序索引。MyISAM的全文索引其实没啥用，因为它不支持中文分词，必须由使用者分词后加入空格再写到数据表里，而且少于4个汉字的词会和停用词一样被忽略掉。
6 MyISAM支持GIS数据，InnoDB不支持。即MyISAM支持以下空间数据对象：	Point,Line,Polygon,Surface等。
7 没有where的count(*)使用MyISAM要比InnoDB快得多。因为MyISAM内置了一个计数器，count(*)时它直接从计数器中读，而InnoDB必须扫描全表。所以在InnoDB上执行count(*)时一般要伴随where，且where中要包含主键以外的索引列。为什么这里特别强调“主键以外”？因为InnoDB中primary index是和raw data存放在一起的，而secondary index则是单独存放，然后有个指针指向primary key。所以只是count(*)的话使用secondary index扫描更快，而primary key则主要在扫描索引同时要返回raw data时的作用较大。
```

## 索引类型

````
1.普通索引
2.唯一索引
    与通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一
3.主键索引
	一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引
4.组合索引
	指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合
5.全文索引
	主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。fulltext索引配合match against操作使用，而不是一般的where语句加like
	
使用索引时，有以下一些技巧和注意事项：
1.索引不会包含有null值的列
	只要列中包含有null值都将不会被包含在索引中，复合索引中只要有一列含有null值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为null。
2.使用短索引
	对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个char(255)的列，如果在前10个或20个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。
3.索引列排序
	查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。
4.like语句操作
	一般情况下不推荐使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。
5.不要在列上进行运算
	这将导致索引失效而进行全表扫描，例如
	SELECT * FROM table_name WHERE YEAR(column_name)<2017;
6.不使用not in和<>操作
````

## mysql慢查询

```
运行时间超过long_query_time值的SQL语句，则会被记录到慢查询日志中。
long_query_time的默认值为10，意思是记录运行10秒以上的语句。
默认情况下，MySQL数据库并不启动慢查询日志，需要手动来设置这个参数
	set global slow_query_log = 1;

MySQL 慢查询的相关参数解释：
slow_query_log：是否开启慢查询日志，1表示开启，0表示关闭。
log-slow-queries ：旧版（5.6以下版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件host_name-slow.log
slow-query-log-file：新版（5.6及以上版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件host_name-slow.log
long_query_time：慢查询阈值，当查询时间多于设定的阈值时，记录日志。(默认10)
log_queries_not_using_indexes：未使用索引的查询也被记录到慢查询日志中（可选项）。
log_output：日志存储方式。log_output='FILE'表示将日志存入文件，默认值是'FILE' log_output='TABLE'表示将日志存入数据库。

select * from mysql.slow_log;
```

## 索引的设计原则

```
1 选择唯一性索引
	唯一性索引的值是唯一的，可以更快速的通过该索引来确定某条记录。
	
2 为经常需要排序、分组和联合操作的字段建立索引
	经常需要 ORDER BY、GROUP BY、DISTINCT 和 UNION 等操作的字段，排序操作会浪费很多时间。如果为其建立索引，可以有效地避免排序操作

3 为常作为查询条件的字段建立索引
	如果某个字段经常用来做查询条件，那么该字段的查询速度会影响整个表的查询速度。因此，为这样的字段建立索引，可以提高整个表的查询速度。

4 限制索引的数目
	索引的数目不是“越多越好”。每个索引都需要占用磁盘空间，索引越多，需要的磁盘空间就越大。在修改表的内容时，索引必须进行更新，有时还可能需要重构。因此，索引越多，更新表的时间就越长

5 尽量使用数据量少的索引
	如果索引的值很长，那么查询的速度会受到影响。例如，对一个 CHAR(100) 类型的字段进行全文检索需要的时间肯定要比对 CHAR(10) 类型的字段需要的时间要多。

6 数据量小的表最好不要使用索引
	由于数据较小，查询花费的时间可能比遍历索引的时间还要短，索引可能不会产生优化效果

7 尽量使用前缀来索引
	如果索引字段的值很长，最好使用值的前缀来索引。例如，TEXT 和 BLOG 类型的字段，进行全文检索会很浪费时间。如果只检索字段的前面的若干个字符，这样可以提高检索速度

8 删除不再使用或者很少使用的索引
	表中的数据被大量更新，或者数据的使用方式被改变后，原有的一些索引可能不再需要。应该定期找出这些索引，将它们删除，从而减少索引对更新操作的影响。
```

## 数据的冷热分离

```
1 数据双写
	把对冷热库的插入,更新,删除操作,全部放在一个统一的事务里面
	由于热库和冷库的类型不同，大概率会是分布式事务
2 MQ 分发
	通过 MQ 的发布订阅功能,在进行数据操作的时候,先不落库，而是发送到 MQ中
	单独启动消费进程,将 MQ 中的数据分别落到热库，冷库中
	使用这种方式改造业务，逻辑清晰,结构优雅
3 使用Binlog同步
	针对 Mysql，可以使用 Binlog 的方式同步
	使用 Canal 组件,可以持续的获取最新的 Binlog 数据，结合MQ，可以将数据同步到其它的数据源中
```

