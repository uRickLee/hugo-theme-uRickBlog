---
title: MySQL基础
date: 2019-12-22 23:08:24
image: https://gitee.com/uRick/oss/raw/master/blog/事务隔离级别.png
description: DCL是Data Control Language的缩写，主要包括数据权限有关的操作指令，常见操作指令如下
categories:
 - Database
tags:
 - MySQL
 - 数据库
---

## 1.1. 简介
> MySQL是关系型数据存储容器，它将数据以特定的格式存储到内存或者文件中，MySQL是目前主流的数据库之一。

`如下命令中，若上文已经表述的字段，下文不再阐述，若有疑问，ctrl+f搜索即可`

## 1.2. 数据类型
> [http://www.runoob.com/mysql/mysql-data-types.html](http://www.runoob.com/mysql/mysql-data-types.html)

## 1.3. 常用操作指令
### 1.3.1. DCL
> DCL是Data Control Language的缩写，主要包括数据权限有关的操作指令，常见操作指令如下：

**用户管理操作**

```sql
-- 创建用户
-- username:用户名，ip:该用户访问ip，password：密码，newPassword:新密码
create user 'username'@'ip' identified by 'password';
-- 删除用户
drop user 'username'@'ip';
-- 修改用户
rename user 'username'@'ip'; to 'newUserName'@'ip';;
-- 修改密码
set password for 'username'@'ip' = Password('newPassword')
```

**授权操作**

```sql
-- privilege:授予的权限，dbName:数据库名，tableName：表名
-- 查看权限
-- @ip表示用户只能在当前ip下才能访问数据库，它支持通配%(表示任意) 可以表示为@%（任意ip）
show grants for 'username'@'ip'
-- 授权
-- password表示用户密码，flush privileges：表示刷新权限，
grant privilege  on dbName.tableName to   'username'@'ip' identified by "password" flush privileges；
-- 取消权限
revoke privilege on dbName.tableName from 'username'@'ip'
```

**privilege** 权限可选项包括如下：

```sql
all privileges  -- 除grant外的所有权限
select          -- 仅查权限
select,insert   -- 查和插入权限
usage                   -- 无访问权限
alter                   -- 使用alter table
alter routine           -- 使用alter procedure和drop procedure
create                  -- 使用create table
create routine          -- 使用create procedure
create temporary tables -- 使用create temporary tables
create user             -- 使用create user、drop user、rename user和revoke  all privileges
create view             -- 使用create view
delete                  -- 使用delete
drop                    -- 使用drop table
execute                 -- 使用call和存储过程
file                    -- 使用select into outfile 和 load data infile
grant option            -- 使用grant 和 revoke
index                   -- 使用index
insert                  -- 使用insert
lock tables             -- 使用lock table
process                 -- 使用show full processlist
select                  -- 使用select
show databases          -- 使用show databases
show view               -- 使用show view
update                  -- 使用update
reload                  -- 使用flush
shutdown                -- 使用mysqladmin shutdown(关闭MySQL)
…………
```


### 1.3.2. DDL
> DDL是Data Definition Language的缩写，主要包括create、alter、drop等常用指令，用于操作表或者数据库结构的指令，常见指令如下：

```sql
-- 创建数据库dbName
create database dbName default charset utf8 collate utf8_general_ci;
-- 使用数据库
use dbName;
drop database dbName;
-- 创建表
-- column_name列名称，column_type列类型
create table if not exists tableName (column_name column_type);
-- 删除表
drop table tableNamea;
```

```sql
-- 添加列
alter table tableName add column_name column_type;
-- 添加实例
ALTER TABLE `user_info` ADD COLUMN `id` VARCHAR (11) DEFAULT NULL COMMENT '平台ID' AFTER `id_number`;
-- 删除列
alter table tableName drop column column_name;
-- 修改列
alter table tableName modify column column_name column_type;
alter table tableName change old_column_name new_column_name column_type;
-- 添加主键
alter table tableName add primary key(column_name);
-- 删除主键
alter table tableName drop primary key;
alter table tableName  modify  column_name int, drop primary key;
-- 添加外键，stable从表名称,fk_name外键名称，fk外键，mtable主表名称，pk主键字段
alter table stable add constraint fk_name（如：FK_从表_主表） foreign key stable(fk) references mtable(pk);
-- 删除外键
alter table tableName drop foreign key fk_name
-- 修改默认值,default_value默认值
alter table tableName alter column_name set default default_value;
-- 删除默认值
alter table tableName alter column_name drop default;
```

```sql
-- 创建索引,indexName索引名称
create index indexName ON tableName(column_name(length));
-- 删除索引
drop index [indexName] ON tableName;
-- 创建唯一索引
create unique index indexName ON tableName(column_name(length));
```

```sql
-- 添加主键,column_list主键列表,因为主键可以是多个
alter table tableName add primary key (column_list)
-- 创建唯一索引
alter table tableName add unique (column_list)
-- 修改表结构，添加索引（普通索引）
alter table tableName add index indexName(column_list);
-- 添加全文索引
alter table tableName add fulltext indexName(column_list);
-- 删除索引
alter table tableName drop index indexName;
-- 删除主键
alter table tableName drop primary key;

```

```sql
-- 重置数据库表自增序列,id为主键
alter table tableName drop id;
alter table tableName add id int unsigned not null auto_increment first,add primary key (id);
-- 指定自增序列开始值
alter table tableName auto_increment = 100;
```

### 1.3.3. DML
> DML是Data Manipulation Language的缩写，也就是SQL语句，指令如下：

1. **insert**

```sql
-- 插入一条
insert into tableName (field1,field2,...fieldN) values (value1,value2,...valueN);
-- 插入多条
insert into tableName (field1,field2,...fieldN) values (value1,value2,...valueN),(value1,value2,...valueN),(……);
-- 全值插入
insert into tableName (value1,value2,...valueN);
-- 基于已经存在的表插入数据
insert into tableName1 (field1,field2...) select (field2,field2...) from tableName2
```

2. delete

```sql
-- 删除表数据， []表示可选项，condition表示条件
delete from tableName [[where condition1 [and [or]] condition2.....];
-- 清空表所有数据，保留表结构
truncate table tableName;
-- 删除表，表结构、数据全部删除，同时释放磁盘空间，也就是表彻底从当前数据库消失了
drop table tableName;
```

3. **query**

```sql
-- n数字是检索的行数，m数字是指从第m行开始，也就是查询的数据从m行开始计算
select field1, field2,...fieldN from tableName [[where condition1 [and [or]] condition2.....] [limit n][offset m];
-- 模糊查询语句语法
select field1, field2,...fieldN from tableName where field1 like condition1 [and [or]] condition2.....
-- 查询合并[all | distinct] 条件可选，默认union具有distinct去重功能
select field1, field2,...fieldN from tableName1 [where conditions]
union [ALL | distinct]
select field1, field2,...fieldN from tableName2 [where conditions];
-- 排序查询语句语法,order by 默认asc升序
select field1, field2,...fieldN from tableName1, tableName2... order by field1, [field2...] [asc [desc]]
-- 分组查询,这里需要注意，where与having的区别,where 分组前过滤，having分组后过滤，分组查询中，条件中有聚合函数必须使用having
-- where/having/function 在from后面的执行顺序：where>function>having
select column_name, function(column_name) from tableName
[where conditions]
group by column_name
[having [function_conditions|simple_conditions]];
-- 连接查询
select [distinct] field1, field2,...fieldN from left_tableName
[inner|left|right] join right_tableName on conditions [and|or conditions]
[where conditions]
[group by column_name]
[having [function_conditions|simple_conditions]]
[order by field1, [field2...] [asc [desc]]]
[limit n][offset m]
```

4. **update**

```sql
-- 更新数据,new-value新值
update tableName set field1=new-value1, field2=new-value2
[where conditions]
-- 关联更新
update left_tableName
[inner|left|right] join right_tableName on conditions
set field1=new-value1, field2=new-value2,……
[where conditions]
-- 替换某个字段中的字符
update tableName set field=replace(field, 'old-string', 'new-string')
[where conditions]
```

5. **JSON数据操作常用方法**

| 方法                                                       | 功能                                                   |
| :--------------------------------------------------------- | :----------------------------------------------------- |
| `JSON_ARRAY([val[, val] …])`                               | 根据一系列元素创建一个JSON                             |
| `JSON_OBJECT(key, val[, key, val] …)`                      | 根据一系列 K/V 对创建一个JSON                          |
| `JSON_QUOTE(string)`                                        | 返回一个字符串，该字符串为带引号的JSON数据             |
| `JSON_CONTAINS(target, candidate[, path])`                  | 通过返回1或0来表示目标JSON是否包含给定的candidate JSON |
| `JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] …)` | 通过返回0或1来表示一个JSON在给定路径是否包含数据       |
| `JSON_INSERT(json_doc, path, val[, path, val] …)`          | 在JSON中在某一路径下插入子元素                         |
| `JSON_REMOVE(json_doc, path[, path] …)`                    | 移除JSON文档中某一路径下的子元素                       |
| `JSON_REPLACE(json_doc, path, val[, path, val] …)`         | 替换JSON文档中的某一路径下的子元素                     |
| `JSON_SET(json_doc, path, val[, path, val] …)`             | 在JSON文档中为某一路径设置子元素                       |
| `JSON_UNQUOTE(json_val)`                                    | 移除JSON值外面的引号，返回结果为字符串                 |
| `JSON_TYPE(json_val)`                                      | 检查某JSON内部内容的类型                               |
| `->`                                                        | 返回执行路径后面的JSON列的值；是`JSON_EXTRACT`的语法糖  |
| `->>`                                                       | 返回不带引号的JSON数据                                 |

可参考文档：

- JSON数据类型基础使用方法： [https://dev.mysql.com/doc/refman/8.0/en/json.html](https://dev.mysql.com/doc/refman/8.0/en/json.html)
- JSON数据操作函数： [https://dev.mysql.com/doc/refman/5.7/en/json-functions.html](https://dev.mysql.com/doc/refman/5.7/en/json-functions.html)

### 1.3.4. TCL
> TCL是Transaction Control Language的缩写，事务控制语言，指令如下：

1. **事务**
> 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行，用来管理 insert,update,delete 语句。

2. **事务的特性**
- **原子性（Atomicity）** 原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
- **一致性（Consistency）** 事务必须使数据库从一个一致性状态变换到另外一个一致性状态。
- **隔离性（Isolation）** 事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
- **持久性（Durability）** 持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

3. **指令**

```sql
-- 显示开启事务，两种开启方式是等价的
begin/start transaction;
-- 显示提交事务，两种提交方式是等价的
commit/commit work;
-- 事务回滚，回滚结束用户的事务，并撤销正在进行未提交的修改
rollback/rollback work;
-- 创建事务保存点，一个事务可以有多个保存点，保存点主要用于事务内部回滚
savepoint idenfier;
-- 删除事务保存点
release savepoint idenfier;
-- 将事务回滚到某一个保存点
rollback to identifier;
-- 查看设置事务隔离级别,隔离级别有read uncommit、read commit、repeatable read、serializable
select @@global.tx_isolation, @@session.tx_isolation;
-- 设置当前会话事务隔离级别
set session transaction isolation level REPEATABLE READ;
-- 设置全局事务隔离级别
set global transaction isolation level REPEATABLE READ;
-- 禁止自动提交
set autocommit=0
-- 开启自动提交
set autocommit=1
```

### 1.3.5. 常用show指令

```sql
-- 显示当前数据库中所有表的名称。
show tables或show tables from database_name;
-- 显示mysql中所有数据库的名称。
show databases;
-- 显示表中列名称。
show columns from table_name from database_name; 或show columns from database_name.table_name;
-- 显示一个用户的权限，显示结果类似于grant 命令。
show grants for user_name;
-- 显示表的索引。
show index from table_name;
-- 显示一些系统特定资源的信息，例如，正在运行的线程数量。
show status;
-- 显示系统变量的名称和值。
show variables;
-- 显示系统中正在运行的所有进程，也就是当前正在执行的查询。大多数用户可以查看他们自己的进程，但是如果他们拥有process权限，就可以查看所有人的进程，包括密码。
show [full] processlist;
-- 显示当前使用或者指定的database中的每个表的信息。信息包括表类型和表的最新更新时间。
show table status;
-- 显示服务器所支持的不同权限。
show privileges;
-- 显示create database 语句是否能够创建指定的数据库。
show create database database_name;
-- 显示create database 语句是否能够创建指定的数据库。
show create table table_name;
-- 显示innoDB存储引擎的状态。
show innodb status;
-- 显示BDB存储引擎的日志。
show logs;
-- 显示最后一个执行的语句所产生的错误、警告和通知。
show warnings;
-- 只显示最后一个执行语句所产生的错误。
show errors;
--显示安装后的可用存储引擎和默认引擎。
show [storage] engines;
-- 查看表锁，Table_locks_waited数值高，锁表锁频次高
show status like 'table%';
-- 数值高说明，表锁比较严重
show status like 'innodb_row_lock%';
```

### 1.3.6. MYSQL常用函数
[http://www.runoob.com/mysql/mysql-functions.html](http://www.runoob.com/mysql/mysql-functions.html)

### 1.3.7. 事务的隔离级别
> MySQL中的事务隔离级别的存在时为了防止多个事务并发执行时由于交叉执行而导致数据的不一致。  
> 在事务并发情况下，会出现如下几种情况：

![](https://gitee.com/uRick/oss/raw/master/blog/事务隔离级别.png)

- 脏读
> 指事务A读取到事务B未提交的数据，若要解决脏读可以在事务B执行操作时加上排他锁。
- 不可重复读
> 指事务A两次读取到的数据不一致，因为事务B更新了数据，可以通过在事务A读取数据时添加共享锁，当事务B执行更新时就需要等待事务A释放共享锁。
- 幻读
> 事务A读取数据时，是读取某个范围的数据，当事务B向数据表中插入数据后，事务A就读取到新增的数据导致幻读；这种情况需要在读取数据时锁住范围内的数据行，对于MySQL InnoDB引擎能解决幻读，或者串行话的隔离级别。

## 1.4. 主从复制
> 通过服务器配置多个库来实现数据同步，实现主从复制来达到负载均衡、高可用、高扩展行，实现数据分布式读取等。

*原理：*

- 主库变更的数据以二进制的形式输出保存到磁盘上；
- 从库读取主库中的二进制文件，将二进制文件复制到中继日志中；
- 从库从中继日志中读取，并执行备份，复制方式异步串行化的。

![](https://gitee.com/uRick/oss/raw/master/blog/MYSQL主从复制.png)


*配置步骤：*

- 配置主库my.ini文件

```ini
[mysqld]
# 主从复制配置
#设置server-id,保证主从唯一
server-id=1 
#开启二进制日志
log-bin=logPath
# 主库可以读写
read-only=0
# 设置不要复制的数据库
binlog-ignore-db=mysql
# 可设置定复制的数据库
# binlog-do-db=mysql
```

- 配置从库my.ini文件

```ini
[mysqld]
# 主从复制配置
# 设置server-id,保证主从唯一
server-id=2
# 开启二进制日志
log-bin=logPath
```

- 分别重启主从库服务器
- 创建主库复制用户权限并从库复制权限

```ini
# 创建用户rickslave为用户名，可自行替换  localhost为从库ip IDENTIFIED BY '密码'
create user 'rickslave'@'localhost' IDENTIFIED BY 'rickslave';
# 授权
grant replication slave on *.* to 'rickslave' @'localhost';
# 查看主库的状态，记住file和position的值，mysqlbin.000006，67需要在从库进行热备份配置。
show master status
# 处理的线程
show processlist
# 刷新权限
flush privileges;
```

- 配置从库复制功能

```ini
# 从库复制功能主库配置, master_host为主库地址，master_user为从库用户，master_password为密码，master_log_file和master_log_pos为show master status 查询出对应的值
change master to master_host = 'localhost',
master_user = 'rickslave', master_password = 'rickslave',
master_log_file = 'mysqlbin.000006', master_log_pos =2031

# 启动从服务器复制功能
start slave
# 停用从库服务器复制功能
stop slave
显示从库状态,当Slave_IO_Running和Slave_SQL_Running为Yes时说明功能正常
show slave status
```

- 最后执行数据更新，操作验证主从数据变化

*详见官网：[https://dev.mysql.com/doc/refman/5.7/en/replication-configuration.html](https://dev.mysql.com/doc/refman/5.7/en/replication-configuration.html)*