# 一、参数优化

## 1.1 内存参数优化

### 1.1.1 Buffer Pool参数优化

#### 缓冲池内存大小配置

一个大的日志缓冲区允许大量的事务在提交之前不写日志到磁盘。因此，如果你有很多事务的更新，插入或删除操作，通过设置这个参数会大量的减少磁盘I/O的次数数。
建议: 在专用数据库服务器上，可以将缓冲池大小设置为服务器物理内存的60% - 80%.

-  查看缓冲池大小 

```sql
mysql> show variables like '%innodb_buffer_pool_size%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+

mysql> select 134217728 / 1024 / 1024;
+-------------------------+
| 134217728 / 1024 / 1024 |
+-------------------------+
|            128.00000000 |
+-------------------------+
```


-  在线调整InnoDB缓冲池大小
   innodb_buffer_pool_size可以动态设置，允许在不重新启动服务器的情况下调整缓冲池的大小. 

```sql
mysql> SET GLOBAL innodb_buffer_pool_size = 268435456; -- 512
Query OK, 0 rows affected (0.10 sec)

mysql> show variables like '%innodb_buffer_pool_size%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 268435456 |
+-------------------------+-----------+
```

监控在线调整缓冲池的进度 

```sql
mysql> SHOW STATUS WHERE Variable_name='InnoDB_buffer_pool_resize_status';
+----------------------------------+----------------------------------------------------------------------+
| Variable_name                    | Value                                                        |
+----------------------------------+----------------------------------------------------------------------+
| Innodb_buffer_pool_resize_status | Size did not change (old size = new size = 268435456. Nothing to do. |
+----------------------------------+----------------------------------------------------------------------+
```

####  配置多个buffer pool实例

- 当buffer pool的大小是GB级别时，将一个buffer pool分割成几个独立的实例能降低**多个线程同时读写缓存页的竞争性而提高并发性**。
- 通过innodb_buffer_pool_instances参数可以调整实例个数。如果有多个实例，则缓存的数据页会随机放置到任意的实例中，且每个实例都有独立的buffer pool所有的特性。
- buffer pool 可以存放多个 instance，每个instance由多个chunk组成。instance的数量范围和chunk的总数量范围分别为1-64，1-1000。
- `Innodb_buffer_pool_instances` 的默认值是1，最大可以调整成64 

```sql
mysql> show variables like 'innodb_buffer_pool_instances';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_buffer_pool_instances | 1     |
+------------------------------+-------+
```

#### chunk(块)大小配置

增大或减小缓冲池大小时，将以chunk的形式执行操作.chunk 大小由 `innodb_buffer_pool_chunk_size` 决定引入chunk 是为了方便在线修改缓冲池大小，修改时以 chunk 为单位拷贝 buffer pool。

```sql
mysql> show variables like 'innodb_buffer_pool_chunk_size';
+-------------------------------+-----------+
| Variable_name                 | Value     |
+-------------------------------+-----------+
| innodb_buffer_pool_chunk_size | 134217728 | 
+-------------------------------+-----------+
```

缓冲池大小`innodb_buffer_pool_size`必须始终等于或者是`chunk_size * instances`的倍数(不等于则MySQL会自动调整)。

```
假设
 innodb_buffer_pool_chunk_size=128MB 
 innodb_buffer_pool_instances=16
那么
 innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances=2GB 

如果我们设置innodb_buffer_pool_size=9GB，则会被自动调整为10GB
```

### 1.1.2 InnoDB 缓存性能评估

当前配置的innodb_buffer_pool_size是否合适，可以通过分析InnoDB缓冲池的缓存命中率来验证。

-  以下公式计算InnoDB buffer pool 命中率: 

```
命中率 = innodb_buffer_pool_read_requests / (innodb_buffer_pool_read_requests+innodb_buffer_pool_reads)* 100

参数1: innodb_buffer_pool_reads：表示InnoDB缓冲池无法满足的请求数。需要从磁盘中读取。
参数2: innodb_buffer_pool_read_requests：表示从内存中读取页的请求数。
```

```sql
mysql> show status like 'innodb_buffer_pool_read%';
+---------------------------------------+-------+
| Variable_name                         | Value |
+---------------------------------------+-------+
| Innodb_buffer_pool_read_ahead_rnd     | 0     |
| Innodb_buffer_pool_read_ahead         | 0     |
| Innodb_buffer_pool_read_ahead_evicted | 0     |
| Innodb_buffer_pool_read_requests      | 12701 |
| Innodb_buffer_pool_reads              | 455   |
+---------------------------------------+-------+

-- 此值低于90%，则可以考虑增加innodb_buffer_pool_size。
mysql> select 12701 / (455 + 12701) * 100 ;
+-----------------------------+
| 12701 / (455 + 12701) * 100 |
+-----------------------------+
|                     96.5415 |
+-----------------------------+
```

### 1.1.3 Page管理相关参数

查看Page页的大小(默认16KB),`innodb_page_size`只能在初始化MySQL实例之前配置，不能在之后修改。如果没有指定值，则使用默认页面大小初始化实例。

```sql
mysql> show variables like '%innodb_page_size%'; 
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
```

-  Page页管理状态相关参数 

```sql
mysql> show global status like '%innodb_buffer_pool_pages%';
+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| Innodb_buffer_pool_pages_data    | 515   |
| Innodb_buffer_pool_pages_dirty   | 0     |
| Innodb_buffer_pool_pages_flushed | 334   |
| Innodb_buffer_pool_pages_free    | 15868 |
| Innodb_buffer_pool_pages_misc    | 0     |
| Innodb_buffer_pool_pages_total   | 16383 |
+----------------------------------+-------+
```

- **pages_data**: InnoDB缓冲池中包含数据的页数。 该数字包括脏页面和干净页面。
- **pages_dirty**: 显示在内存中修改但尚未写入数据文件的InnoDB缓冲池数据页的数量（脏页刷新）。
- **pages_flushed**: 表示从InnoDB缓冲池中刷新脏页的请求数。
- **pages_free**: 显示InnoDB缓冲池中的空闲页面
- **pages_misc**: 缓存池中当前已经被用作管理用途或hash index而不能用作为普通数据页的数目
- **pages_total**: 缓存池的页总数目。单位是page。 

#### 优化建议

innodb_page_size的官方描述: ![](../../../../../../Documents/02_%25E5%259B%25BE%25E7%2589%2587/56.jpg)

- MySQL 5.7增加了对32KB和64KB页面大小的支持。默认的16KB或更大的页面大小适用于各种工作负载，特别是涉及表扫描的查询和涉及批量更新的DML操作。对于涉及许多小写操作的OLTP工作负载，较小的页面大小可能更有效. 
- Page大小对于行存储的影响
  - 对于4KB、8KB、16KB和32KB的页大小，最大行大小(不包括存储在页外的任何可变长度的列)略小于页大小的一半。 
- Page大小对于索引的影响
  - 如果在创建MySQL实例时通过指定innodb_page_size选项将InnoDB页面大小减少到8KB或4KB，索引键的最大长度将按比例降低，这是基于16KB页面大小的3072字节限制。也就是说，当页面大小为8KB时，最大索引键长度为1536字节，而当页面大小为4KB时，最大索引键长度为768字节。 

### 1.1.4 Change Buffer相关参数优化

change buffering是MySQL5.5加入的新特性，change buffering是insert buffer的加强，insert buffer只针对insert有效，change buffering对insert、delete、update(delete+insert)、purge都有效。

#### 配置change buffer使用模式

innodb_change_buffering 配置参数说明 

```sql
mysql> show variables like '%innodb_change_buffering%';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_change_buffering | all   |
+-------------------------+-------+
```

| 选项    | 说明                         |
| ------- | ---------------------------- |
| inserts | 插入缓冲                     |
| deletes | 删除标记缓冲                 |
| changes | 更新缓冲,由两个缓冲区组成    |
| purges  | 缓冲在后台发生的物理删除操作 |
| all     | 表示启用上面所有配置(默认)   |
| none    | 表示不启用任何配置           |

####  配置change buffer 大小

ChangeBuffer占用BufferPool空间，默认占25%，最大允许占50%，可以根据读写业务量来进行调整。参数`innodb_change_buffer_max_size`; 

```sql
mysql> show variables like 'innodb_change_buffer_max_size';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| innodb_change_buffer_max_size | 25    |
+-------------------------------+-------+
1 row in set (0.00 sec)
```

####   查看change buffer的工作状态 

```sql
-- 查看change buffer的工作状态
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
```

-  **size:** 表示已经合并到辅助索引页的数量； 
-  **free list len:** 表示空闲列表长度； 
-  **seg size：**表示当前Change Buffer的大小，2*16KB； 
-  **merges**：表示合并的次数； 
-  **merged operations：**表示每个具体操作合并的次数； 
   - insert：表示插入操作；
   - delete mark：表示删除标记操作；
   - delete：表示物理删除操作；

## 1.2 日志参数优化

### 日志缓冲区相关参数配置

日志缓冲区的大小。一般默认值16MB是够用的，但如果事务之中含有blog/text等大字段，这个缓冲区会被很快填满会引起额外的IO负载。配置更大的日志缓冲区,可以有效的提高MySQL的效率.

-  **innodb_log_buffer_size 缓冲区大小** 

```sql
mysql> show variables like 'innodb_log_buffer_size';
+------------------------+----------+
| Variable_name          | Value    |
+------------------------+----------+
| innodb_log_buffer_size | 16777216 |
+------------------------+----------+
```

-  **innodb_log_files_in_group 日志组文件个数**
   日志组根据需要来创建。而日志组的成员则需要至少2个，实现循环写入并作为冗余策略。 

```sql
mysql> show variables like 'innodb_log_files_in_group';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_files_in_group | 2     |
+---------------------------+-------+
```

-  **innodb_log_file_size 日志文件大小**
   参数innodb_log_file_size用于设定MySQL日志组中每个日志文件的大小(默认48M)。此参数是一个全局的静态参数，不能动态修改。
   参数innodb_log_file_size的最大值，二进制日志文件大小（innodb_log_file_size * innodb_log_files_in_group）不能超过512GB.所以单个日志文件的大小不能超过256G. 

```sql
mysql> show variables like 'innodb_log_file_size';
+----------------------+----------+
| Variable_name        | Value    |
+----------------------+----------+
| innodb_log_file_size | 50331648 |
+----------------------+----------+
```

### 日志文件参数优化

首先我们先来看一下日志文件大小设置对性能的影响

-  设置过小 
   - 参数`innodb_log_file_size`设置太小，就会导致MySQL的日志文件( redo log）频繁切换，频繁的触发数据库的检查点（Checkpoint），导致刷新脏页到磁盘的次数增加。从而影响IO性能。 
   - 处理大事务时，将所有的日志文件写满了，事务内容还没有写完，这样就会导致日志不能切换. 
-  设置过大
   - 参数`innodb_log_file_size`如果设置太大，虽然可以提升IO性能，但是当MySQL由于意外宕机时，二进制日志很大，那么恢复的时间必然很长。而且这个恢复时间往往不可控，受多方面因素影响。 

### 优化建议

-  如何设置合适的日志文件大小 ?
   根据实际生产场景的优化经验,一般是计算一段时间内生成的事务日志（redo log）的大小， **而MySQL的日志文件的大小最少应该承载一个小时的业务日志量**(官网文档中有说明)。 
   1.  想要估计一下InnoDB redo log的大小，需要抓取一段时间内Log [Sequence](https://so.csdn.net/so/search?q=Sequence&spm=1001.2101.3001.7020) Number的数据,来计算日志一小时内的日志大小. 

> Log sequence number 
> 自系统修改开始，就不断的修改页面，也就不断的生成redo日志。为了记录一共生成了多少日志，于是mysql设计了全局变量log sequence number，简称lsn，但不是从0开始，是从8704字节开始。 

```sql
-- pager分页工具, 只获取 sequence的信息
mysql> pager grep sequence;
PAGER set to 'grep sequence'

-- 查询状态,并倒计时一分钟
mysql> show engine innodb status\G select sleep(60);
Log sequence number 5399154
1 row in set (0.00 sec)

1 row in set (1 min 0.00 sec)

-- 一分时间内所生成的数据量 5406150
mysql> show engine innodb status\G select sleep(60);
Log sequence number 5406150

-- 关闭pager
mysql> nopager;
PAGER set to stdout
```

      2. 有了一分钟的日志量,据此推算一小时内的日志量

```sql
mysql> select (5406150 - 5399154) / 1024 as kb_per_min;
+------------+
| kb_per_min |
+------------+
|     6.8320 |
+------------+

mysql> select (5406150 - 5399154) / 1024 * 60 as kb_per_min;
+------------+
| kb_per_min |
+------------+
|   409.9219 |
+------------+
```

## 1.3 IO 线程参数优化

数据库属于 IO 密集型的应用程序，其主要职责就是数据的管理及存储工作。从内存中读取一个数据库数据的时间是微秒级别，而从一块普通硬盘上读取一个IO是在毫秒级别。要优化数据库，IO操作是必须要优化的，尽可能将磁盘IO转化为内存IO。

### 参数: query_cache_size&have_query_cache

MySQL查询缓存保存查询返回的完整结果。当查询命中该缓存，会立刻返回结果，跳过了解析，优化和执行阶段。
查询缓存会跟踪查询中涉及的每个表，如果这写表发生变化，那么和这个表相关的所有缓存都将失效。 

1. 查看查询缓存是否开启

```sql
-- 查询是否支持查询缓存
mysql> show variables like 'have_query_cache';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | YES   |
+------------------+-------+

-- 查询是否开启查询缓存 默认关闭
mysql> show variables like '%query_cache_type%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| query_cache_type | OFF   |
+------------------+-------+
```

2. 开启缓存,在my.ini中添加下面一行参数

```shell
query_cache_size=128M
query_cache_type=1

query_cache_type:
	设置为0，OFF,缓存禁用
	设置为1，ON,缓存所有的结果
	设置为2，DENAND,只缓存在select语句中通过SQL_CACHE指定需要缓存的查询
```

3. 测试能否缓存查询

```plsql
   mysql> show status like '%Qcache%';
  +-------------------------+---------+
  | Variable_name           | Value   |
  +-------------------------+---------+
  | Qcache_free_blocks      | 1       |
  | Qcache_free_memory      | 1031832 |
  | Qcache_hits             | 0       |
  | Qcache_inserts          | 0       |
  | Qcache_lowmem_prunes    | 0       |
  | Qcache_not_cached       | 1       |
  | Qcache_queries_in_cache | 0       |
  | Qcache_total_blocks     | 1       |
  +-------------------------+---------+
```

-  **Qcache_free_blocks**:缓存中目前剩余的blocks数量（如果值较大，则查询缓存中的内存碎片过多） 
-  **Qcache_free_memory**:空闲缓存的内存大小 
-  **Qcache_hits**:命中缓存次数 
-  **Qcache_inserts**: 未命中然后进行正常查询 
-  **Qcache_lowmem_prunes**:查询因为内存不足而被移除出查询缓存记录 
-  **Qcache_not_cached**: 没有被缓存的查询数量 
-  **Qcache_queries_in_cache**:当前缓存中缓存的查询数量 
-  **Qcache_total_blocks**:当前缓存的block数量 

**优化建议**: Query Cache的使用需要多个参数配合，其中最为关键的是 query_cache_size 和 query_cache_type ，前者设置用于缓存 ResultSet 的内存大小，后者设置在何场景下使用 Query Cache。

- MySQL数据库数据变化相对不多，query_cache_size 一般设置为256MB比较合适 ,也可以通过计算Query Cache的命中率来进行调整

```sql
( Qcache_hits / ( Qcache_hits + Qcache_inserts ) * 100) )
```

### 参数: innodb_max_dirty_pages_pct

该参数是InnoDB 存储引擎用来控制buffer pool中脏页的百分比，当脏页数量占比超过这个参数设置的值时，InnoDB会启动刷脏页的操作。 

```sql
-- innodb_max_dirty_pages_pct 参数可以动态调整，最小值为0， 最大值为99.99，默认值为 75。
mysql> show variables like 'innodb_max_dirty_pages_pct';
+----------------------------+-----------+
| Variable_name              | Value     |
+----------------------------+-----------+
| innodb_max_dirty_pages_pct | 75.000000 |
+----------------------------+-----------+
```

**优化建议**: 该参数比例值越大，从内存到磁盘的写入操作就会相对减少，所以能够一定程度下减少写入操作的磁盘IO。但是，如果这个比例值过大，当数据库 Crash 之后重启的时间可能就会很长，因为会有大量的事务数据需要从日志文件恢复出来写入数据文件中.最大不建议超过90,一般重启恢复的数据在超过1GB的话,启动速度就会变慢. 

### 参数: innodb_old_blocks_pct&innodb_old_blocks_time

`innodb_old_blocks_pct` 用来确定LRU链表中old sublist所占比例,默认占用37% 

```sql
mysql> show variables like '%innodb_old_blocks_pct%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_old_blocks_pct | 37    |
+-----------------------+-------+
```

`innodb_old_blocks_time`  用来控制old sublist中page的转移策略，新的page页在进入LRU链表中时，会先插入到old sublist的头部，然后page需要在old sublist中停留innodb_old_blocks_time这么久后，下一次对该page的访问才会使其移动到new sublist的头部，默认值1秒. 

```sql
mysql> show variables like '%innodb_old_blocks_time%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_old_blocks_time | 1000  |
+------------------------+-------+
```

**优化建议**: 在没有大表扫描的情况下，并且数据多为频繁使用的数据时，我们可以增加innodb_old_blocks_pct的值，并且减小innodb_old_blocks_time的值。让数据页能够更快和更多的进入的热点数据区。 

###  参数: innodb_io_capacity&innodb_io_capacity_max

`innodb_io_capacity`: InnoDB1.0.x版本开始提供该参数 ,它的作用在两个方面: 

      1. 合并插入缓冲时,每秒合并插入缓冲的数量为 innodb_io_capacity值的5%，默认就是 200*5%=10
      2. 在从缓冲区刷新脏页时（checkpoint）,每秒刷新脏页的数量就等于innodb_io_capacity的值，默认200

`innodb_io_capacity_max` : 若用户使用了SSD类的磁盘,或者将几块磁盘做了RAID,当存储设备拥有更高的 IO速度时,可以将 innodbio_capacity_max的值调高,直到符合磁盘IO的吞吐量 为止。 

```sql
mysql> show variables like '%innodb_io_capacity%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_io_capacity     | 200   |
| innodb_io_capacity_max | 2000  |
+------------------------+-------+
```

**优化建议**: 在有频繁写入的操作时,对该参数进行调整.并且该参数设置的大小取决于硬盘的IOPS，即每秒的输入输出量（或读写次数）。
什么样的磁盘配置应该设置innodb_io_capacity参数的值是多少,下面是一些参考: 

> 仅供参考,建议通过sysbench或者其他基准工具对磁盘吞吐量进行测试

| innodb_io_capacity | 硬盘配置      |
| ------------------ | ------------- |
| 200                | 单盘 SAS/SATA |
| 2000               | SAS*12 RAID10 |
| 8000               | SSD           |
| 50000              | FUSON-IO()    |

# 



|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
