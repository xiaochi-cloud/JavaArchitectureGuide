# 一、影响性能的因素

## 1.1 商业需求影响性能

### 1.1.1 不合理需求

> 需求：一个论坛帖子总量的统计
> 附加要求：实时更新


1. 初级阶段：SELECT COUNT(*)
2. 新建一个表，在这个表中更新这个汇总数据（频率问题）
3. 真正的问题在于，实时？
   1. 创建一个统计表，隔一段时间统计一次并存入；

## 1.1.2 无用功能堆积

1. 无用的列堆积；
2. 错误的表设计；
3. 无用的表关联；

## 1.2  系统架构及实现对性能影响

### 1.2.1 哪些数据不适合放在数据库中

1.  二进制数据；
2.  流水队列数据；
3.  超大文本；

### 1.2.2 合理的 Cache

哪些数据适合放到cache中?
1，系统配置信息；
2，活跃的用户的基本信息；
3，活跃用户的定制化信息；
4，基于时间段的统计数据；
5，读>>>写的数据；

### 1.2.3 减少数据库交互次数

N+1问题的解决：

1.  使用链接查询；
2.  使用1+1查询；

### 1.2.4 过度依赖数据库SQL 语句的功能

- 交叉表；
- 不必要的表连接；

### 1.2.5 重复执行相同的SQL

- 在一个页面中，有相同内容，但是使用2条SQL去查询；

## 1.3 其他常见系统架构和实现问题

1. Cache 系统的不合理利用导致Cache 命中率低下造成数据库访问量的增加，同时也浪费了Cache系统的硬件资源投入；
2. 过度依赖面向对象思想，对系统可扩展性的过渡追求，促使系统设计的时候将对象拆得过于离散，造成系统中大量的复杂Join语句，而MySQL Server 在各数据库系统中的主要优势在于处理简单逻辑的查询，这与其锁定的机制也有较大关系；
3. 对数据库的过渡依赖，将大量更适合存放于文件系统中的数据存入了数据库中，造成数据库资源的浪费，影响到系统的整体性能，如各种日志信息；
4. 过度理想化系统的用户体验，使大量非核心业务消耗过多的资源，如大量不需要实时更新的数据做了实时统计计算。

## 1.4  其他因素

### 1.4.1 SQL引起性能问题的原因

SQL的执行过程；

1. 客户端发送一条查询给服务器
2. 服务器先会检查查询缓存，如果命中了缓存，则立即返回存储在缓存中的结果。否则进入下一阶段；
3. 服务器端进行SQL解析、预处理，再由优化器根据该SQL所涉及到的数据表的统计信息进行计算，生成对应的执行计划；
4. MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询；
5. 将结果返回给客户端。

磁盘IO

- SQL执行的最大瓶颈在于磁盘的IO，即数据的读取；不同SQL的写法，会造成不同的执行计划的执行，而不同的执行计划在IO的上面临完全不一样的数量级，从而造成性能的差距；

### 1.4.2 Schema 设计对系统的性能影响

1.  冗余数据的处理；
2.  大表拆小表，有大数据的列单独拆成小表；
3.  根据需求的展示设置更合理的表结构；
4.  把常用属性分离成小表；

### 1.4.3  硬件环境对性能影响

1.  提高IO指标
    1. IOPS：每秒可提供的IO 访问次数；
    2. IO 吞吐量：每秒的IO 总流量；
2.  提高CPU计算能力；
3.  如果是单独的数据库服务器，提高网络能力；

### 1.4.4  数据库系统场景 

两种典型数据库应用场景：
1，联机事务处理OLTP（on-line transaction processing）：OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。
2，联机分析处理OLAP（On-Line Analytical Processing）：OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果，数据仓库就是一个典型应用场景；

#### OLTP

OLTP的数据库应用特点：
1，系统总体数据量较大，但活动数据量较小；
2，IO访问频繁，单涉及数据量较小，分布离散；
3，并发很高；
4，网络交互数据量较小，但交互频繁；

OLTP系统硬件架构选型：
1，大量的合理的cache设计，能够大大减少数据库的交互；应尽量的扩大内存容量；
2，IOPS指标要求较高；
3，CPU的计算能力，并行计算能力要求较高；
4，对外网络要求较高；

#### OLAP

特点：
1，数据量非常大，数据访问集中，数据活跃度分布平均；
2，并发访问较低，
3，每次检索的数量非常多；

架构选型：
1，硬盘存储容量需要非常大；
2，对存储设备的IO吞吐量要求很高；
3，CPU要求较低；
4，对外网络要求不高；

## 1.5 综合考虑

- 需求和架构及业务实现优化：55%
- Query 语句的优化：30%
- 数据库自身的优化：15%

# 二、 SQL优化

## 2.1 SQL优化原则

### 2.1.1 选择需要优化的SQL

- 优先优化高并发低消耗的SQL；
  - 1小时请求1W次，1次10个IO；
  - 1小时请求10次，1次1W个IO；

从IO消耗，优化难度，CPU消耗进行比较；

### 2.2.2  定位性能瓶颈

- SQL运行较慢有两个影响原因，IO和CPU，明确性能瓶颈所在；

### 2.2.3  明确优化目标

#### Explain和Profile

1.  任何SQL的优化，都从Explain语句开始；Explain语句能够得到数据库执行该SQL选择的执行计划；
2.  首先明确需要的执行计划，再使用Explain检查；
3.  使用profile明确SQL的问题和优化的结果；


### 2.2.4 永远用小结果集驱动大的结果集

##### JOIN原则

1. 不是小表驱动大表，是小结果集驱动大结果集；

### 2.2.5 在索引中完成排序

### 2.2.6 使用最小Columns

1.  特别是需要使用column排序的时候；
2.  减少网络传输数据量；
3.  MYSQL排序原理，是把所有的column数据全部取出，在排序缓存区排序，再返回结果；如果column数据量大，排序区容量不够的时候，就会使用先column排序，再取数据，再返回的多次请求方式；

### 2.2.7 使用最有效的过滤条件

1.  过多的WHERE条件不一定能够提高访问性能；
2.  一定要让where条件使用自己预期的执行计划；

### 2.2.8  避免复杂的JOIN和子查询

1.  复杂的JOIN和子查询，需要锁定过多的资源，MYSQL在大量并发情况下处理锁定性能下降较快；
2.  不要过多依赖SQL的功能，把复杂的SQL拆分为简单的SQL；
3.  MySQL子查询性能较低，应尽量避免使用；

## 2.3 使用Explain和Profiling

1. Explain可以让我们查看MYSQL执行一条SQL所选择的执行计划；
2. Profiling可以用来准确定位一条SQL的性能瓶颈；
   1. [https://dev.mysql.com/doc/refman/8.0/en/show-profile.html](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)

```plsql
BEGIN
	DECLARE i int default 0;
  WHILE i<num DO
		insert into test(name,age) values (UUID(),i/10);
    set i=i+1;
  END WHILE;
END
```

### Explain命令

1. 使用方式：
   explain SQL;
2. 返回结果： 
   1. ID：执行查询的序列号；
   2. select_type：使用的查询类型 
      1. DEPENDENT SUBQUERY：子查询中内层的第一个SELECT. 依赖于外部查询的结果集；
      2. DEPENDENT UNION：子查询中的UNION. 且为UNION 中从第二个SELECT 开始的后面所有SELECT. 同样依赖于外部查询的结果集；
      3. PRIMARY：子查询中的最外层查询. 注意并不是主键查询；
      4. SIMPLE：除子查询或者UNION 之外的其他查询；
      5. SUBQUERY：子查询内层查询的第一个SELECT. 结果不依赖于外部查询结果集；
      6. UNCACHEABLE SUBQUERY：结果集无法缓存的子查询；
      7. UNION：UNION 语句中第二个SELECT 开始的后面所有SELECT. 第一个SELECT 为PRIMARY
      8. UNION RESULT：UNION 中的合并结果；
   3. table：这次查询访问的数据表；
   4. type：对表所使用的访问方式： 
      1. all：全表扫描
      2. const：读常量. 且最多只会有一条记录匹配. 由于是常量. 所以实际上只需要读一次；
      3. eq_ref：最多只会有一条匹配结果. 一般是通过主键或者唯一键索引来访问；
      4. fulltext：全文检索. 针对full text索引列；
      5. index：全索引扫描；
      6. index_merge：查询中同时使用两个（或更多）索引. 然后对索引结果进行merge 之后再读取表数据；
      7. index_subquery：子查询中的返回结果字段组合是一个索引（或索引组合）. 但不是一个主键或者唯一索引；
      8. rang：索引范围扫描；
      9. ref：Join 语句中被驱动表索引引用查询；
      10. ref_or_null：与ref 的唯一区别就是在使用索引引用查询之外再增加一个空值的查询；
      11. system：系统表. 表中只有一行数据；
      12. unique_subquery：子查询中的返回结果字段组合是主键或者唯一约束；
   5. possible_keys：可选的索引；如果没有使用索引. 为null；
   6. key：最终选择的索引；
   7. key_len：被选择的索引长度；
   8. ref：过滤的方式. 比如const（常量）. column（join）. func（某个函数）；
   9. rows：查询优化器通过收集到的统计信息估算出的查询条数；
   10. Extra：查询中每一步实现的额外细节信息 
      11. Distinct：查找distinct 值. 所以当mysql 找到了第一条匹配的结果后. 将停止该值的查询而转为后面其他值的查询；
      12. Full scan on NULL key：子查询中的一种优化方式. 主要在遇到无法通过索引访问null值的使用使用；
      13. Impossible WHERE noticed after reading const tables：MySQL Query Optimizer 通过收集到的统计信息判断出不可能存在结果；
      14. No tables：Query 语句中使用FROM DUAL 或者不包含任何FROM 子句；
      15. Not exists：在某些左连接中MySQL Query Optimizer 所通过改变原有Query 的组成而使用的优化方法. 可以部分减少数据访问次数；
      16. Select tables optimized away：当我们使用某些聚合函数来访问存在索引的某个字段的时候. MySQL Query Optimizer 会通过索引而直接一次定位到所需的数据行完成整个查询。当然. 前提是在Query 中不能有GROUP BY 操作。如使用MIN()或者MAX（）的时候；
      17. Using filesort：当我们的Query 中包含ORDER BY 操作. 而且无法利用索引完成排序操作的时候. MySQL Query Optimizer 不得不选择相应的排序算法来实现。
      18. Using index：所需要的数据只需要在Index 即可全部获得而不需要再到表中取数据；
      19. Using index for group-by：数据访问和Using index 一样. 所需数据只需要读取索引即可. 而当Query 中使用了GROUP BY 或者DISTINCT 子句的时候. 如果分组字段也在索引中. Extra 中的信息就会是Using index for group-by；
      20. Using temporary：当MySQL 在某些操作中必须使用临时表的时候. 在Extra 信息中就会出现Using temporary 。主要常见于GROUP BY 和ORDER BY 等操作中。
      21. Using where：如果我们不是读取表的所有数据. 或者不是仅仅通过索引就可以获取所有需要的数据. 则会出现Using where 信息；
      22. Using where with pushed condition：这是一个仅仅在NDBCluster 存储引擎中才会出现的信息. 而且还需要通过打开Condition Pushdown 优化功能才可能会被使用。控制参数为engine_condition_pushdown 。

### profiling的使用

Query Profiler是MYSQL5.1之后提供的一个很方便的用于诊断Query执行的工具，能够准确的获取一条查询执行过程中的CPU，IO等情况；

1. 开启profiling：set profiling=1;
2. 执行QUERY，在profiling过程中所有的query都可以记录下来；
3. 查看记录的query：show profiles；
4. 选择要查看的profile：show profile cpu, block io for query 6；=

- status是执行SQL的详细过程；
- Duration：执行的具体时间；
- CPU_user：用户CPU时间；
- CPU_system：系统CPU时间；
- Block_ops_in：IO输入次数；
- Block_ops_out：IO输出次数；
- profiling只对本次会话有效；

## 2.4 合理使用索引 

### 2.4.1 理解MYSQL的索引

MySQL中索引类型： 

1. B-TREE：使用平衡树实现索引. 是mysql中使用最多的索引类型；
   1. 在innodb中. 存在两种索引类型. 
      1. 第一种是主键索引（primary key）. 在索引内容中直接保存数据的地址；
      2. 第二种是其他索引. 在索引内容中保存的是指向主键索引的引用；所以在使用innodb的时候. 要尽量的使用主键索引. 速度非常快；
2. Hash：
   1. 把索引的值做hash运算. 并存放到hash表中. 使用较少. 一般是memory引擎使用；因为使用hash表存储. 按照常理. hash的性能比B-TREE效率高很多。
      hash索引的缺点：
3. hash索引只能适用于精确的值比较. =. in. 或者<>；
4. 无法使用索引排序；
5. 组合hash索引无法使用部分索引；
6. 如果大量索引hash值相同. 性能较低；
7. full-text索引：全文检索索引. 效率低. 限制多. 不做介绍；
8. R-TREE：针对空间数据索引. 使用很少；不做介绍；

### 2.4.2  索引的利弊

1.  索引的好处： 
    1. 提高表数据的检索效率；
    2. 如果排序的列是索引列. 大大降低排序成本；
    3. 在分组操作中如果分组条件是索引列. 也会提高效率；
2.  索引的问题：索引需要额外的维护成本； 

### 2.4.3 如何创建索引

1.  较频繁的作为查询条件的字段应该创建索引；
2.  唯一性太差的字段不适合单独创建索引. 即使频繁作为查询条件；
3.  更新非常频繁的字段不适合创建索引；
4.  不会出现在WHERE 子句中的字段不该创建索引；

### 2.4.4 单值索引和组合索引

1. 单值索引即一列作为索引；
2. 组合索引即多列创建为一个索引；

### 2.4.5 MySQL中索引使用限制

- BLOB 和TEXT 类型的列只能创建前缀索引
- MySQL 目前不支持函数索引
- 使用不等于（!= 或者<>）的时候MySQL 无法使用索引
- 过滤字段使用了函数运算后（如abs(column)）. MySQL 无法使用索引
- Join 语句中Join 条件字段类型不一致的时候MySQL 无法使用索引
- 使用LIKE 操作的时候如果条件以通配符开始（ '%abc...'）MySQL 无法使用索引
- 使用非等值查询的时候MySQL 无法使用Hash 索引

## 2.5 优化JOIN

### 2.5.1 理解JOIN原理

- mysql中使用Nested Loop Join来实现join；
- A JOIN B：通过A表的结果集作为循环基础. 一条一条的通过结果集中的数据作为过滤条件到下一个表中查询数据. 然后合并结果；

### 2.5.2 join优化原则

1. 尽可能减少Join 语句中的Nested Loop 的循环总次数. 用小结果集驱动大结果集；
2. 优先优化Nested Loop 的内层循环；
3. 保证Join 语句中被驱动表上Join 条件字段已经被索引；
4. 扩大join buffer的大小；

### 2.5.3 其他优化

#### 优化ORDER BY

ORDER BY 实现原理：

1. 通过有序索引而直接取得有序的数据；
2. 通过MySQL 的排序算法将存储引擎中返回的数据进行排序然后再将排序后的数据返回；

优化方案：

1. 加大max_length_for_sort_data 参数；
2. 去掉不必要的返回字段；
3. 增大sort_buffer_size 参数；

#### 优化GROUP BY

#### 优化distinct

### 2.5.4 其他优化

#### Query Cache

1.  QueryCache的实现原理； 
2.  QueryCache的负面影响： 
    1. Query的hash性能问题和命中率问题；
    2. 查询缓存及其容易失效；当表内容发生变化或者表结构发生变化. 对应的查询缓存内容都会失效；
    3. 查询缓存中的结果容易产生重复；因为查询缓存中缓存的是查询结果. 所以不同的查询的结果很容易重复；
3.  Query Cache的使用： 
    1.  设置query_cache_limit为查询缓存大小. 如果为0. 则不使用查询缓存； 
    2.  使用SQL_CACHE或者SQL_NO_CACHE来强制是否使用查询缓存； 
    3.  查询查询缓存设置：show variables like '%query_cache%'; 
        1. “have_query_cache”：该MySQL 是否支持Query Cache；
        2. “query_cache_limit”：Query Cache 存放的单条Query 最大Result Set . 默认1M；
        3. “query_cache_min_res_unit”：Query Cache 每个Result Set 存放的最小内存大小. 默认4k；
        4. “query_cache_size”：系统中用于Query Cache 内存的大小；
        5. “query_cache_type”：系统是否打开了Query Cache 功能；
    4.  查询查询缓存使用情况：show status like 'Qcache%'; 
        1. “Qcache_free_blocks”：Query Cache 中目前还有多少剩余的blocks。如果该值显示较大. 则说明Query Cache 中的内存碎片较多了
        2. “Qcache_free_memory”：Query Cache 中目前剩余的内存大小；
        3. “Qcache_hits”：多少次命中；
        4. “Qcache_inserts”：多少次未命中然后插入；Query Cache 命中率= Qcache_hits / ( Qcache_hits + Qcache_inserts )；
        5. “Qcache_lowmem_prunes”：多少条Query 因为内存不足而被清除出Query Cache；
        6. “Qcache_not_cached”：因为query_cache_type 的设置或者不能被cache 的Query 的数量；
        7. “Qcache_queries_in_cache”：当前Query Cache 中cache 的Query 数量；
        8. “Qcache_total_blocks”：当前Query Cache 中的block 数量；
    5.  query cache的使用限制： 
        1. mysql query cache内容为 select 的结果集, cache 使用完整的 sql 字符串做 key, 并区分大小写. 空格等。即两个sql必须完全一致才会导致cache命中。
        2. prepared statement永远不会cache到结果. 即使参数完全一样,
        3. where条件中如包含了某些函数永远不会被cache, 比如current_date, now等。
        4. 太大的result set不会被cache (< query_cache_limit)
    6.  query cache的使用方式： 
        1. 如果没有绝对的使用把握. 可以关闭查询缓存；
        2. 如果要使用查询缓存. 最好能够精确的控制那些表内容放到查询缓存. 哪些表不用查询缓存；

#### Innodb_buffer_pool_size

1.  Innodb_buffer_pool_size：innodb的缓存. 可以用于缓存索引. 同时还会缓存实际的数据；
    innodb_buffer_pool_size 参数用来设置Innodb 最主要的Buffer(Innodb_Buffer_Pool)的大小. 对Innodb 整体性能影响也最大. 可以按需要设置大一些； 
2.  可以通过show status like 'Innodb_buffer_pool_%';查看innodb buffer的状态 
    1. Innodb_buffer_pool_pages_data：使用到了的缓存页数；
    2. Innodb_buffer_pool_pages_flushed：刷新过的缓存页数；
    3. Innodb_buffer_pool_pages_free：剩余的缓存页数；
    4. Innodb_buffer_pool_pages_total：总缓存页数；
    5. Innodb_buffer_pool_read_requests：从缓存中读取的数据量；
    6. Innodb_buffer_pool_reads：直接从磁盘读取的数据量；
3.  可以通过这些参数计算出缓存命中率和缓存利用率等； 

## 2.6 事务优化

### 2.6.1 隔离级别优化

1. innodb实现了READ UNCOMMITTED/READ COMMITTED/REPEATABLE READ/SERIALIZABLE四种隔离级别；
2. 默认使用REPEATABLE READ隔离级别；
3. 可以通过SELECT @@tx_isolation;查看当前的事务隔离级别；
4. 可以通过set tx_isolation='read-committed';来修改默认的事务隔离级别；

### 2.6.2 innodb_flush_log_at_trx_commit

1. 理解Innodb事务机制： 
   1. 事务在buffer中对数据进行修改；
   2. 事务的变化记录在事务日志中；
   3. 在合适的时机同步事务日志中的数据到数据库中；
2. 所以什么时候提交事务日志文件. 对系统性能影响较大. 可以通过设置innodb_flush_log_at_trx_commit来修改事务日志同步时机： 
   1. innodb_flush_log_at_trx_commit = 0. 每1秒钟同步一次事务日志文件；
   2. innodb_flush_log_at_trx_commit = 1. 默认设置. 每一个事务完成之后. 同步一次事务日志文件；
   3. innodb_flush_log_at_trx_commit = 2. 事务完成之后. 写到事务日志文件中. 等到日志覆盖再同步数据；
      注意. 1性能最差. 2不能完全保证数据是写到数据文件中. 如果宕机. 可能会有数据丢失现象. 但性能最高；1. 性能和安全性居中；

# 三、 MySQL复制

## 3.1 MySQL复制机制原理

1. MySQL的复制是异步执行的. 因为MySQL的特殊机制. 让复制的延迟控制较小；
2. MySQL的复制是从一个MySQL进程复制到另一个MySQL进程. 被复制方我们称为Master；复制方我们称为Slave；
3. MySQL的复制依赖一种叫做bin-log的日志文件. bin-log记录了所有在Master端执行的DDL/DML/事务操作序列. 并同步到Slave端. Slave根据日志复现操作序列. 即完成同步；
4. 复制流程： 
   1. Slave 上面的IO 线程连接上Master. 并请求从指定日志文件的指定位置之后的日志内容；
   2. Master 接收到来自Slave 的IO 线程的请求后. 通过负责复制的IO线程根据请求信息读取指定日志指定位置之后的日志信息. 返回给Slave 端的IO线程；
   3. Slave 的IO 线程接收到信息后. 将接收到的日志内容依次写入到Slave 端的Relay Log 文件(mysql-relay-bin.xxxxxx)的最末端；
   4. Slave 的SQL 线程检测到Relay Log 中新增加了内容后. 会马上解析该Log 文件中的内容成为在Master 端真实执行时候的那些可执行的Query 语句. 并在自身执行这些Query；
5. 复制一定会存在延迟和数据丢失的风险；

## 3.2  复制级别

## 3.3  安装新的MySQL实例

要完成复制. 需要准备两个MYSQL实例；

1. 停止第一个MYSQL服务；
2. 将原MYSQL目录拷贝到新的地址；
3. 修改my.ini. 主要修改以下内容： 
   1. port；新的mysql实例端口；
   2. basedir：新的mysql目录；
   3. datadir：新的mysql实例文件地址；
   4. 拷贝原数据库文件中的mysql目录. 或者执行mysql_install_db命令；
   5. 为新的MYSQL实例创建服务：mysqld install MySQL2  --defaults-file="E:\MySQL\mysql_base\ini\my.ini"
   6. 修改注册表：HKEY_LOCAL_MACHINE-->SYSTEM-->CurrentControlSet-->Services. 将imagePath修改为新的mysql目录和配置文件；
   7. 重启两个MYSQL实例；

## 3.4  配置主/从

1. 在主服务器my.ini中添加：
   server-id=1   //给数据库服务的唯一标识. 一般为大家设置服务器Ip的末尾号
   log-bin=master-bin
   log-bin-index=master-bin.index
2. 启动主服务器；
3. 执行show master status；查看主服务器状态；
4. 复制当前主服务器中的数据库内容；
5. 在从服务器中创建主服务器数据库；
6. 在从服务器my.ini中添加：
   server-id=2
   relay-log-index=slave-relay-bin.index
   relay-log=slave-relay-bin
7. 启动从服务器；
8. 在从服务中执行：
   change master to master_host='127.0.0.1', //Master 服务器Ip
   master_port=3306,
   master_user='root',
   master_password='admin',
   master_log_file='master-bin.000001',//Master服务器产生的日志
   master_log_pos=0;
9. 启动从服务：start slave；
10. 在主服务器中添加一条数据. 查看在从服务器中是否同步成功；

## 3.5  读写分离

