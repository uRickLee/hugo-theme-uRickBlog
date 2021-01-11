---
title: MySQ高级特性
date: 2019-12-22 23:08:24
image: https://gitee.com/uRick/oss/raw/master/blog/MYSQL体系结构.png
description: 索引(Index)是帮助MySQL高校获取数据的数据结构，当谈到MySQL性能优化时，一定是绕不开的话题，因为它决定SQL性能的好坏，好的SQL离不开一个好的索引，二者是相辅相成的。
categories:
 - Database
tags:
 - MySQL
---

## 1.1. MySQL体系结构
![MySQL体系结构](https://gitee.com/uRick/oss/raw/master/blog/MYSQL体系结构.png)
1.  **Client Connectors：** 接入方支持的协议。
2. **Management Serveices & Utilities：** 系统管理和控制工具，mysqldump、 mysql复制集群、分区管理等。
3. **Connection Pool：** 连接池，管理缓冲用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求。
4. **SQL Interface：** SQL接口，接受用户的SQL命令，并且返回用户需要查询的结果。
5. **Parser：** 解析器，SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的。
6. **Optimizer：** 查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化。
7. **Cache和Buffer（高速缓存区）：** 查询缓存，如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。
8. **Pluggable Storage Engines：** 插件式存储引擎。存储引擎是MySql中具体的与文件打交道的子系统。
9. **File System：** 文件系统，数据、日志（redo，undo）、索引、错误日志、查询记录、慢查询等。

MySQL官方文档：[https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)

## 1.2. 索引
> 索引(Index)是帮助MySQL高校获取数据的数据结构，当谈到MySQL性能优化时，一定是绕不开的话题，因为它决定SQL性能的好坏，好的SQL离不开一个好的索引，二者是相辅相成的。

> 索引能极大的减少存储引擎需要扫描的数据量，提高数据查询的效率，减少IO操作次数，提高性能。

> MySQL主要使用B+Tree作为索引算法，InnoDB引擎就是使用这种算法，它的特点有如下几点：

- 节点关键字搜索采用闭合区间；
- 非叶节点不保存数据相关信息，只保存关键字和子节点的引用；
- 关键字对应的数据保存在叶子节点中；
- 叶子节点是顺序排列的，并且相邻节点具有顺序引用的关系；

`后续补充详细………… 😁 `

### 1.2.1. 索引优化技巧
**【优化总结口诀】**  
①全值匹配我最爱，最左前缀要遵守；  
②带头大哥不能死，中间兄弟不能断；  
③索引列上少计算；范围之后全失效；  
④like百分写最右；覆盖索引不写量；  
⑤不等空值还有OR；索引失效要少用；  
⑥VAR引号不可丢；SQL高级也不难；  

> **下面通过案例理解上面的技巧：**  
> 假设表Y建立索引`index(a,b,c)`，Y表还有很多其他字段未列出；  
> a是字符类型，b、c 是整型；  
> 若测试表Y只有a、b、c三个字段，不能达到测试效。

**①②** 的含义是：若是多列索引，要遵守最左前缀法则；指的是查询从索引的最左前列开始并且不跳过索引中的列，同时如果能够所有索引列都能匹配是最完美的。  

```sql
-- 使用到索引列a，其他b、c并未使用到
select *from Y where a='1';
-- 使用到索引a、b，未使用到c
select *from Y where a='1' and b=2;
-- 所有的索引列都使用到，最好的最完美的方式
select *from Y where a='1' and b=2 and c=3;
-- 跳过了索引a，所有的索引都未使用到（所以带头大哥不能死哟）
select *from Y where b=2 and c=3;
select *from Y where b=2;
select *from Y where c=3;
-- 使用到索引a，并未使用到索引c，因为跳过了中间索引b
select *from Y where a='1' and c=3;
```

**③** 的含义是：不在索引列上做任何操作`（计算、函数、（自动or手动）类型转换）`，否则会导致索引失效而转向全表扫描；若使用索引列中，使用到范围查找，则范围查找右边使用到的索引列会失效。

```sql
-- 通过索引列计算，则索引列a未使用到
select *from Y where right(a,2)='1';
-- b列通过范围查找，则范围右边的列c未使用到
select *from Y where a='1' and b>2 and c=3;
```

**④** 的含义是：like模糊查找以通配符开头（'%abc'、'%abc%'）mysql索引失效会变成全表扫描操作；查询中尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少使用`select*`。

```sql
-- abc索引列都使用到了
select *from Y where a like 'xx%' and b=2  and c=3;
-- 同上
select *from Y where a like 'k%xx%' and b=2  and c=3;
-- 左匹配模糊查询，导致索引失效，所有索引都未使用到
select *from Y where a like '%xx' and b=2  and c=3;
-- 同上
select *from Y where a like '%xx%' and b=2  and c=3;
-- 使用覆盖索引，不使用select*，覆盖索引前提是保证查询条件和查询内容列一样
select a,b,c from Y where a='1' and b=2 and c=3;
```

**⑤** 的含义是：MySQL在使用（`!=或<>或is null或is not null 或or`）的时候无法使用索引会导致全表扫描。

```sql
-- a、b、c索引失效
select *from Y where a='1' or b='2' and c='3';
-- 索引a未使用
select *from Y where a is null;
-- 索引a未使用
select *from Y where a != '1';
```

**⑥** 的含义是：字符类型的字段作为条件查询时，不加单引号会导致索引失效，因为它存在隐式类型转换。

```sql
-- 存在隐式类型转换
select *from Y where a = 1;
-- 不带引号，索引列a是用不到
select *from Y where a = '1';
```

### 1.2.2. 其他技巧
#### 1.2.2.1. in与exists
> 原则：小表驱动大表，即小的数据集驱动大的数据集。  

假设存在小表B，大表A，则查询语句写法：

```sql
-- in(subquery)子查询中适合查询小表，这种方式效率更高
select * from A where id in (select id from B)
-- exists(subquery) 子查询适合大表，主查询数据需要放到子查询中匹配
-- 通过返回true或false判断是否保留主查询结果
select * from B where exists (select * from A)
```

#### 1.2.2.2. Order By 与Group By
> 对于`order by`，尽量使用Index方式排序，避免使用FileSort方式排序。  
> `group by` 实质是先排序后进行分组，优化技巧同order by一样。

>  MySQL支持Index和FileSort排序，Index比FileSort排序效率高，Index是基于索引实现排序，而FileSort是基于外部文件排序的。

若想要实现Index排序，需要遵照索引建的最佳左前缀原则，下面通过案例（伪代码）说明：  

假设一个表建立索引：` index_abc(a,b,c)`   

```sql
-- order by 使用到索引排序，遵守最佳左前缀排序
order by a
order by a,b
order by a,b,c
order by a DESC,b DESC,c DESC
```

```sql
-- 如果WHERE使用索引的左前缀为常量，则order by 能使用索引
where a = const order by b,c
where a = const and b = const order by c
where a = const and b>const order by b,c
```

```sql
-- 不能使用索引排序
-- 排序不一致
order by a ASC,b DESC,c DESC 
-- 丢掉a索引
where g=const order by b,c
-- 丢掉b索引
where a=const order by c
-- d不是索引的一部分
where a=const order by a,d
-- 对于排序来说,多个相等的条件也是范围查询
where a in(…………) order by b,c
```



## 1.3. 存储引擎
1. **CSV：** 以逗号隔开的数据存储格式文件,它不支持索引，一般常用于数据交互格式，数据导出、导入。
2. **ARCHIVE：** 采用压缩协议进行数据的存储，数据存储以ARZ格式存储。
    - 只支持insert和select两种操作；
    - 只允许自增ID列建立索引；
    - 支持行级锁，不支持事务；
    - 因为占用磁盘空间小，常用日志采集、大量数据采集场景。

3. **MERMORY：** 所有数据存储在内存中，访问数据极快，数据容易丢失。
    - 数据都是存储在内存中，IO效率要比其他引擎高很多；
    - 服务重启数据丢失，内存数据表默认只有16M；
    - 支持hash索引，Btree索引，默认hash（查找复杂度0(1)）字段长度都是固定长度varchar(32)=char(32)；
    - 不支持大数据存储类型字段如 blog，text表级锁；
    - 常用于等值查找热度较高数据、内存数据查询计算，查询中的临时表就是使用当前引擎。

4. **MyISAM：** MySQL 5.5版本之前的默认存储引擎，有很多系统表也使用MyISAM引擎。
    - MyISAM的索引与行记录是分开存储的，分别以`.MYI`和`.MYD`，使用的索引叫做非聚集索引；
    - 支持表级锁，不支持行锁，不支持事务；
    - `select count(*) from tableName `无需进行数据的扫描MyISAM会保存数据行数;
    - 多用数据查询场景，不建议使用更新和删除操作。
5. **InnoDB：** Mysql5.5及以后版本的默认存储引擎，它是默认的事务引擎，也是最重要、使用最广泛的引擎。
    - InnoDB的主键索引与行记录是存储在一起的，使用的索引叫做聚集索引；
    - 所有的数据查找都是基于聚集索引。
    > 存在主键，则以主键作为聚集索引，否则以一个非空的unique作为聚集索引，否则创建一个隐藏的row-id作为聚集索引；  
    > 当通过辅助索引查找数据时，通过索引查找树，查找到叶子节点中存储聚集索引，最后才通过聚集索引查找到对应的数据。

**[InnoDB与MyISAM区别详解](https://mp.weixin.qq.com/s/FUXPXKfKyjxAvMUFHZm9UQ)**

## 1.4. 查询优化
> 数据查询时日常开发中，操作频次最高的，所以练好这部分内功是必不可少的；所有的查询SQL要想性能最好，必须结合MySQL索引的特点实现。

![SQL执行流程](https://gitee.com/uRick/oss/raw/master/blog/SQL执行流程.png)

### 1.4.1. 客户端/服务器通信协议
1. 通信方式
    - 全双工通信：双向通信的，发送方和接收方可以同时接收或发送消息。
    - 半双工通信：双向通信的，发送方和接收方不能同时接收或者发送消息。同一时刻，发送方发送消息，接收方只能接收消息，不能发送消息，发送方也不能够接收消息。
    - 单工通信：单一方向发送消息，只能从A到B发送，不能逆向。发送方和接收方职责都是单一的，角色是固定的。

2. 查看MySQL客户端与服务端通信连接信息
可以通过`show full processlist/show processlist`命令查询，如：

|  Id  | User |      Host       |  db   | Command | Time |  State   |         Info          |
| :--: | :--: | :-------------: | :---: | :-----: | :--: | :------: | :-------------------: |
| 4122 | root | 127.0.0.1:20114 | mysql |  Sleep  | 4075 |          |         NULL          |
| 4127 | root | 127.0.0.1:53839 | NULL  |  Sleep  |  29  |          |         NULL          |
| 4128 | root | 127.0.0.1:53841 | blog  |  Query  |  0   | starting | show full processlist |

*参数说明：*  
- Id：线程id，可以通过kill命令杀掉；
- User：连接的用户名；
- Host：连接的主机地址；
- db：连接的数据库名；
- Command：当前命令状态信息；
    - Sleep：线程正在等待客户端发送数据；
    - Query：连接线程正在执行查询；
    - Locked：线程正在等待表锁的释放；
    - Sorting result：线程正在对结果进行排序；
    - Sending data：向请求端返回数据。
- Time：连接时间；
- State：状态信息；
- Info：命令信息。

### 1.4.2. 查询缓存
> 查询缓存主要是缓存SQL语句查询的结果和SQL语句，可以理解为key-value存储，sql作为key，查询结果作为value。  

默认情况下MySQL的缓存是没有开启的，为了减少资源浪费，可以通过`set global query_cache_type=1`命令开启或者配置文件配置（需要重启服务）；  

1. 查询流程
执行查询SQL，先查找缓存中是否存在结果，若存在则直接返回结果，不存在则执行查询，并缓存查询结果（`缓存命中SQL需要完全一样，SQL是区分大小写的`）。

2. 查询缓存信息
> 通过命令`show variables like 'query%'`命令查看

| Variable_name                | Value   |
| :--------------------------- | :------ |
| query_alloc_block_size       | 8192    |
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 1048576 |
| query_cache_type             | OFF     |
| query_cache_wlock_invalidate | OFF     |
| query_prealloc_size          | 8192    |

*参数说明：*  

- **query_cache_type：**
    - 0：不启用查询缓存 ，默认值 ；
    - 1：启用查询缓存，只要符合查询缓存的要求，客户端的查询语句和记录集都可以缓存起来，供其他客户端使用 ，加上 SQL_NO_CACHE 将不缓存；
    - 2：启用查询缓存，只要查询语句中添加了参数： SQL_CACHE ，且符合查询缓存的要求，客户端的查询语句和记录集，则可以缓存起来，供其他客户端使用；
- **query_cache_size：** 允许设置 query_cache_size 的值最小为 40K ，默认1M ， 推荐设置 为： 64M/128M ；
- **query_cache_limit：** 限制查询缓存区最大能缓存的查询记录集，默认设置为1M；
- **query_cache_min_res_unit：**  分配内存块时的最小单位大小，设置查询缓存Query Cache每次分配内存的最小空间大小，即每个查询的缓存最小占用的内存空间大小；
- **query_cache_wlock_invalidate：**  如果某个数据表被锁住,是否仍然从缓存中返回数据,默认是OFF,表示仍然可以返回；

3. 查询缓存状态
> 通过命令`show status like 'qcache%'`命令查看

| Variable_name           | Value   |
| :---------------------- | :------ |
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1031832 |
| Qcache_hits             | 0       |
| Qcache_inserts          | 0       |
| Qcache_lowmem_prunes    | 0       |
| Qcache_not_cached       | 4623    |
| Qcache_queries_in_cache | 0       |
| Qcache_total_blocks     | 1       |

*参数说明：*  

- **Qcache_free_blocks：** 缓存池中空闲块的个数，数目大说明可能有碎片。FLUSH QUERY CACHE会对缓存中的碎片进行整理，从而得到一个空暇块；
- **Qcache_free_memory：** 缓存中的空暇内存；
- **Qcache_hits：** 缓存命中次数；
- **Qcache_inserts：** 缓存写入次数，每次插入一个查询时就增大，命中次数除以插入次数就是不中比率；
- **Qcache_lowmem_prunes：** 缓存出现内存不足而且必需要进行清理以便为很多其它查询提供空间的次数；
- **Qcache_not_cached：** 查询未被缓存次数,例如查询结果超出缓存块大小,查询中包含可变函数（now()、datetime()等）等；
- **Qcache_queries_in_cache：** 当前缓存中缓存的SQL数量；
- **Qcache_total_blocks：** 缓存中块的数量。

4. 缓存失效情况
    1. 当查询语句中有一些不确定的数据时，则不会被缓存。如包含函数 NOW() ，CURRENT_DATE() 等类似的函数，或者用户自定义的函数，存储函数，用户变量等都不会被缓存；
    2. 当查询的结果大于`query_cache_limit`设置的值时，结果不会被缓存；
    3. 对于 InnoDB 引擎来说，当一个语句在事务中修改了某个表，那么在这个事务提交之前，所有与这个表相关的查询都无法被缓存。因此长时间执行事务，会大大降低缓存命中率。
    4. 查询的表是系统表；
    5. 查询语句不涉及到表。

*MySQL查询缓存可用于以读为主的业务，数据生成之后就不常改变的业务比如门户类 、新闻类、报表 类 、论坛类*

### 1.4.3. 查询优化处理
> 查询优化处理主要由解析器、预处理器、查询优化器。

- **解析器：** 通过lex词法分析,yacc语法分析将sql语句解析成解析树；
- **预处理器：** 根据mysql的语法的规则进一步检查解析树的合法性，如：检查数据的表和列是否存在，解析名字和别名的设置。还会进行权限的验证；
- **查询优化器：** 找出最优的查询计划，然后通过查询引擎查询数据。

#### 1.4.3.1. 执行计划explain
> 使用 **explain** 可以模拟优化器执行SQL语句，从而知道MySQL是何处理你的SQL语句的，分析你的查询语句或是结构的性能瓶颈。

> 通过explain，能够很清晰的知道SQL查询读取表的顺序、哪些索引被使用到、表直接的引用关系、每张表有多少条数据被扫描等等。

通过`explain + sql` 执行分析，数据参数如下：  
![](https://gitee.com/uRick/oss/raw/master/blog/1578320943_20190222171036687_17009.png)

*参数说明：*  

1. **id：** 表示select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。
    - id相同，执行顺序由上至下；
    - id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行；
    - id相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行。

2. **select_type：** 主要是用于区分普通查询、联合查询、子查询等。
    - **SIMPLE：** 简单的select查询，查询中不包含子查询或者UNION；
    - **PRIMARY：** 查询中若包含任何复杂的子部分，最外层查询则被标记为primary；
    - **SUBQUERY：** 在SELECT或者WHERE列表中包含了子查询；
    - **DERIVED：** 在FROM列表中包含的子查询被标记为DERIVED（衍生） MySQL会递归执行这些子查询，把结果放在临时表里；
    - **UNION：** 若第二个SELECT出现在UNION之后，则被标记为UNION; 若UNION包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED；
    - **UNION RESULT：** 从UNION表获取结果的SELECT。

3. **table：** 查询涉及到的表，直接显示表名或者表的别名。
    - `<unionM,N>` 由ID为M,N 查询union产生的结果；
    - `<subqueryN>` 由ID为N查询生产的结果。

4. **type：** 访问类型，sql 查询优化中一个很重要的指标，结果值从好到坏依次是：`system > const > eq_ref > ref > range > index > ALL`。
    - **system：** 表只有一行记录（等于系统表），const类型的特例，基本不会出现，可以忽略不计；
    - **const：** 表示通过索引一次就找到了，const用于比较primary key 或者 unique索引；
    - **eq_ref：** 唯一索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描；
    - **ref：** 非唯一性索引扫描，返回匹配某个单独值的所有行，本质是也是一种索引访问；
    - **range：** 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引一般就是在你的where语句中出现了between、<、>、in等的查询 这种范围扫描索引扫描比全表扫描要好，因为他只需要开始索引的某一点，而结束语另一点，不用扫描全部索引。
    - **index：** Full Index Scan，索引全表扫描，把索引从头到尾扫一遍；
    - **ALL：** Full Table Scan，遍历全表以找到匹配的行。

5. **possible_keys：** 查询涉及的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用,也就是可能使用到的索引。
6. **key：** 实际使用的索引。如果为null则没有使用索引，查询中若使用了覆盖索引，则索引和查询的select字段重叠。
7. **key_len：** 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好key_len显示的值为索引最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。
8. **ref：** 显示索引那一列被使用了，如果可能的话，是一个常数。那些列或常量被用于查找索引列上的值。
9. **rows：** 据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。
10. **Extra：** 包含不适合在其他列中显示但十分重要的额外信息。
    - **Using filesort：** 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。 MySQL中无法利用索引完成排序操作成为“文件排序”；
    - **Using temporary：** 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by 和分组查询 group by；
    - **Using index：** 表示相应的select操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！ 如果同时出现using where，表明索引被用来执行索引键值的查找； 如果没有同时出现using where，表面索引用来读取数据而非执行查找动作,覆盖索引（Covering Index） ；
    - **Using where：** 表明用到where条件过滤；
    - **Using join buffer：**  使用了连接缓存；
    - **Impossible where：** 子句的值总是false，不能用来获取任何元组；
    - **Select tables optimized away：** 在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算， 查询执行计划生成的阶段即完成优化；
    - **distinct：** 优化distinct，在找到第一匹配的元组后即停止找同样值的工作。
    - 更多参考：[https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-extra-information](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-extra-information)
11. **partitions：** 查询记录来自哪个分区。若没有匹配分区，该值为NULL。
12. **filtered：** 查询过滤行所占百分比，若为100则数据未过滤，过滤掉的行数为：总行数×filtered百分比值（单位%）。

*关于explain详细说明可参考官方：[https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_filtered](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)*
### 1.4.4. 查询执行引擎
> 根据生成的查询计划，调用存储引擎接口执行查询，直到完成所有的数据查询。

### 1.4.5. 结果返回
> 将SQL查询的数据返回给客户端，若需要做缓存，则将结果插入缓存；  

> MySQL返回结果给客户端是一个增量、逐步返回的过程，目的是为了减轻服务端的压力，服务端直接将结果返回，不需要储存，浪费过多的内存资源。

## 1.5. 慢查询日志
MySQL提供的SQL监控的一种日志，记录在MySQL中SQL执行响应的时间的语句，SQL响应时间超过long_query_time的时间就回被记录到慢查询日志中；当SQL语句执行响应时间超过给定的long_query_time时，可以针对性的优化相关SQL语句。  

*MySQL默认未开启慢查询日志功能，需要手动开启，因为开启或多或少会带来性能上的一点开销。*  

1. 常用命令

```sql
--查看是否开启
SHOW VARIABLES LIKE '%slow_query_log%'
--开启慢查询
set global slow_query_log = 1
--关闭慢查询
set global slow_query_log = 0
--查看当前多少秒算慢
SHOW VARIABLES LIKE 'long_query_time%';
--慢的阙值时间（设置后需要另外开启一个回话生效）或者使用第二个命令查看
set global long_query_time=3;
show global variables like 'long_query_time';
--查询当前系统中有多少条慢查询记录
show global status like '%slow_queries%'
```

2. 配置文件配置
> 慢日志查询开启，除了使用上面的命令方式开启，还可以在配置文件my.ini中配置

3. mysql 开启慢查询在配置文件my.ini中配置

```ini
[mysqld]
# 开启慢查询
slow_query_log=1
# 指定慢查询输出的日志文件名（包含路径）
slow_query_log_file=xxx.log
# 指定慢查询阀值时间，单位s（秒）
long_query_time=3
# 指定慢查询输入的方式
log_output=file
```

4. 日志分析工具
> 针对大量日志，须要手动分析相当浪费时间，可以使用mysql 的工具 *mysqldumpshow*

```cmd
 perl mysqldumpslow.pl --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]
Parse and summarize the MySQL slow query log. Options are
  --verbose    verbose
  --debug      debug
  --help       write this text to standard output
  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

*常用参数说明：*  

```cmd
s:是表示按何种方式排序
c:访问次数
l:锁定时间
r:返回记录
t:查询时间
al:平均锁定时间
ar:平均返回记录数
at:平均查询时间
t:即为返回前面多少条的数据
g:后边搭配一个正则匹配模式，大小写不敏感的

案例：
    返回记录集最多的10个SQL。
    mysqldumpslow -s r -t 10 slow_log1.log
    访问次数最多的10个SQL
    mysqldumpslow -s c -t 10 slow_log1.log
    按照时间排序的前10条里面含有左连接的查询语句。
    mysqldumpslow -s t -t 10 -g “left join” slow_log1.log
    另外建议在使用这些命令时结合|和more 使用，否则有可能出现刷屏的情况。
    mysqldumpslow -s r -t 20 slow_log1.log | more
```

*详见官网：[https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html](https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html)*

## 1.6. show profiles
>show profiles是MySQL提供可以用来分析当前会话中语句执行的资源消耗情况，可以用于SQL的调优测量  
>默认情况下，参数处于关闭状态，可以通过命令`set profiling=on`开启，并保存最近15次的运行结果

1. 分析步骤
    1. 通过`set profiling=on`命令开启show profiles，也可以通过 `show variables like 'profiling'`命令查询mysql 是否支持profiling；
    2. 通过`show profiles`命令查看profiles 记录执行的sql，如下：

![](https://gitee.com/uRick/oss/raw/master/blog/Mysql_show_profiles.png)

   3. 诊断sql，分析sql的执行步骤，根据Query_ID 通过 `show profile`命令查看sql执行状态 。

```sql
-- type为需要查看的类型，n对应Query_ID
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]

type: {
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS
}
```

*参数说明：*  

```
ALL：显示所有的开销信息
BLOCK IO：显示块IO开销
CONTEXT SWITCHES：上下文切换开销
CPU：显示CPU开销信息
IPC：显示发送和接收开销信息
MEMORY：显示内存开销信息
PAGE FAULTS：显示页面错误开销信息
SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息
SWAPS：显示交换次数开销信息

# =======注意（sql生命周期中出现以下周期，说明sql很糟糕）=====
# 查询结果太大，内存都不够用了往磁盘上搬了
converting HEAP to MyISAM
# 创建临时表（拷贝数据到临时表、用完再删除）
Creating tmp table
# 把内存中临时表复制到磁盘，危险！！！
Copying to tmp table on disk
# 被锁住
locked
```

*详见官网：[https://dev.mysql.com/doc/refman/5.7/en/show-profile.html](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)*