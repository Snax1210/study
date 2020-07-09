[TOC]: # "MySQL8.0新特性集锦"

# MySQL8.0新特性集锦
- [1.默认字符集从Latin1变为utf8mb4](#1默认字符集从latin1变为utf8mb4)
- [2.MyISAM系统表全部换成InnoDB表](#2myisam系统表全部换成innodb表)
- [3.自增变量持久化](#3自增变量持久化)
- [4.DDL原子化](#4ddl原子化)
- [5.参数修改持久化](#5参数修改持久化)
- [6.新增降序索引](#6新增降序索引)
- [7.group by 不再隐式排序](#7group-by-不再隐式排序)
- [8.JSON 特性增强](#8json-特性增强)
- [9. redo & undo 日志加密](#9-redo--undo-日志加密)
- [10.innodb select for update跳过锁等待](#10innodb-select-for-update跳过锁等待)
- [11.增加SET_VAR语法](#11增加set_var语法)
- [12.支持不可见索引](#12支持不可见索引)
- [13.支持直方图](#13支持直方图)
- [14.新增innodb_dedicated_server参数](#14新增innodb_dedicated_server参数)
- [15.日志分类更详细](#15日志分类更详细)
- [17.增加资源组](#17增加资源组)
- [18.增加角色管理](#18增加角色管理)


## 1.默认字符集从Latin1变为utf8mb4

在8.0版本之前，默认字符集为latin1，utf8指向的是utf8mb3,,8.0版本默认字符集为utf8mb4，utf8默认指向的也是utf8mb4。  
注：在percona Server 8.0.15版本上测试，utf8仍然指向的是utf8mb3，与官方文档有出入。

```bash
Warning | 3719 | 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous. |
```

## 2.MyISAM系统表全部换成InnoDB表

系统表全部换成事务型的innodb表，默认的MySQL实例将不包含任何MyISAM表，除非手动创建MyISAM表。

```bash
# MySQL 5.7
mysql> select distinct(ENGINE) from information_schema.tables;
+--------------------+
| ENGINE             |
+--------------------+
| MEMORY             |
| InnoDB             |
| MyISAM             |
| CSV                |
| PERFORMANCE_SCHEMA |
| NULL               |
+--------------------+
6 rows in set (0.00 sec)
 
# MySQL 8.0
mysql> select distinct(ENGINE) from information_schema.tables;
+--------------------+
| ENGINE             |
+--------------------+
| NULL               |
| InnoDB             |
| CSV                |
| PERFORMANCE_SCHEMA |
+--------------------+
4 rows in set (0.00 sec)
```

## 3.自增变量持久化

在8.0之前的版本，自增组件AUTO_INCREMENT的值如果大于max(primary key)+1，在MySQL重启后，会重置AUTO_INCREMENT = max(primary key)+1,这种现象在某些情况下会导致业务主键冲突或者难以发现的问题。自增主键重启重置的问题很早就被发现 [https://bugs.mysql.com/bug.php?id=199](https://bugs.mysql.com/bug.php?id=199)，一直到8.0才被解决，8.0版本将会对AUTO_INCREMENT值进行持久化，MySQL重启后，该值将不会改变。

## 4.DDL原子化

InnoDB表的DDL支持事务完整性，要么成功要么回滚，将DDL操作回滚日志写入到data dictionary数据字典表mysql.innodb_ddl_log中用于回滚操作，该表是隐藏的表，通过show tables无法看到。通过设置参数，可见ddl操作日志打印输出到mysql错误日志中。

```bash
mysql> set global log_error_verbosity=3;
mysql> set global innodb_print_ddl_logs=1;
mysql> create table t1(c int) engine=innodb;
 
# MySQL错误日志:
2018-06-26T11:25:25.817245+08:00 44 [Note] [MY-012473] [InnoDB] InnoDB: DDL log insert : [DDL record: DELETE SPACE, id=41, thread_id=44, space_id=6, old_file_path=./db/t1.ibd]
2018-06-26T11:25:25.817369+08:00 44 [Note] [MY-012478] [InnoDB] InnoDB: DDL log delete : by id 41
2018-06-26T11:25:25.819753+08:00 44 [Note] [MY-012477] [InnoDB] InnoDB: DDL log insert : [DDL record: REMOVE CACHE, id=42, thread_id=44, table_id=1063, new_file_path=db/t1]
2018-06-26T11:25:25.819796+08:00 44 [Note] [MY-012478] [InnoDB] InnoDB: DDL log delete : by id 42
2018-06-26T11:25:25.820556+08:00 44 [Note] [MY-012472] [InnoDB] InnoDB: DDL log insert : [DDL record: FREE, id=43, thread_id=44, space_id=6, index_id=140, page_no=4]
2018-06-26T11:25:25.820594+08:00 44 [Note] [MY-012478] [InnoDB] InnoDB: DDL log delete : by id 43
2018-06-26T11:25:25.825743+08:00 44 [Note] [MY-012485] [InnoDB] InnoDB: DDL log post ddl : begin for thread id : 44
2018-06-26T11:25:25.825784+08:00 44 [Note] [MY-012486] [InnoDB] InnoDB: DDL log post ddl : end for thread id : 44
```

来看另外一个例子，库里只有一个t1表，drop table t1,t2;试图删除t1,t2两张表，在5.7中，执行报错，但t1表被删除，在8.0中执行报错，但是t1表没有没删除，证明了8.0DDL操作的原子性，要么全部成功，要么回滚。

```bash
# MySQL 5.7
mysql> show tables;
+---------------+
| Tables_in_db |
+---------------+
| t1            |
+---------------+
1 row in set (0.00 sec)
mysql> drop table t1, t2;
ERROR 1051 (42S02): Unknown table 'db.t2'
mysql> show tables;
Empty set (0.00 sec)
 
# MySQL 8.0
mysql> show tables;
+---------------+
| Tables_in_db |
+---------------+
| t1            |
+---------------+
1 row in set (0.00 sec)
mysql> drop table t1, t2;
ERROR 1051 (42S02): Unknown table 'db.t2'
mysql> show tables;
+---------------+
| Tables_in_db |
+---------------+
| t1            |
+---------------+
1 row in set (0.00 sec)
```

## 5.参数修改持久化

MySQL8.0版本支持在线修改全局参数并持久化，通过加沙红PERSIST关键字，可以将修改的参数持久化到新的配置文件(mysql-auto.conf)中，重启MySQL时，可以从该配置文件获取到最新的配置参数。  
例如执行：

```mysql
set PERSIST expire_logs_days = 10;
```

系统会在数据目录下生成一个包含json格式的mysqld-auto.cnf的文件，格式化后如下所示，当my.cnf和mysqld-auto.cnf同时存在时，后者具有更高优先级。

```json
{
    "Version": 1,
    "mysql_server": {
        "expire_logs_days": {
            "Value": "10",
            "Metadata": {
                "Timestamp": 1529657078851627,
                "User": "root",
                "Host": "localhost"
            }
        }
    }
}
```

## 6.新增降序索引

MySQL在语法上很早就已经支持降序索引，但是实际上创建的仍然是升序索引，如下MySQL5.7所示，c2字段降序，但是从show create table看c2仍然是升序。8.0可以看到，c2字段降序。

```bash
# MySQL 5.7
mysql> create table t1(c1 int,c2 int,index idx_c1_c2(c1,c2 desc));
Query OK, 0 rows affected (0.03 sec)
mysql> show create table t1\G
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int(11) DEFAULT NULL,
  `c2` int(11) DEFAULT NULL,
  KEY `idx_c1_c2` (`c1`,`c2`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
 
# MySQL 8.0
mysql> create table t1(c1 int,c2 int,index idx_c1_c2(c1,c2 desc));
Query OK, 0 rows affected (0.06 sec)
mysql> show create table t1\G
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int(11) DEFAULT NULL,
  `c2` int(11) DEFAULT NULL,
  KEY `idx_c1_c2` (`c1`,`c2` DESC)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=DYNAMIC
1 row in set (0.00 sec)
```

再来看看降序索引在执行计划中的表现，在t1表插入10万条随机数据，查看select * from t1 order by c1,c2 desc;的执行计划。从执行计划上可以看出，5.7的扫描数100113远远大于8.0的5行，并且使用了filesort。

```bash
DELIMITER ;;
CREATE PROCEDURE test_insert ()
BEGIN
DECLARE i INT DEFAULT 1;
WHILE i<100000
DO
insert into t1 select rand()*100000, rand()*100000;
SET i=i+1;
END WHILE ;
commit;
END;;
DELIMITER ;
CALL test_insert();
 
# MySQL 5.7
mysql> explain select * from t1 order by c1 , c2 desc limit 5;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows   | filtered | Extra                       |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
|  1 | SIMPLE      | t1    | NULL       | index | NULL          | idx_c1_c2 | 10      | NULL | 100113 |   100.00 | Using index; Using filesort |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
1 row in set, 1 warning (0.00 sec)
 
# MySQL 8.0
mysql> explain select * from t1 order by c1 , c2 desc limit 5;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | index | NULL          | idx_c1_c2 | 10      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

降序索引只是对查询中特定的排序顺序有效，如果使用不当，反而查询效率更低，比如上述查询条件改为order by c1 desc, c2 desc，这种情况下，5.7的执行计划要明显好于8.0的，如下：

```bash
# MySQL 5.7
mysql> explain select * from t1  order by c1 desc , c2 desc limit 5;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | index | NULL          | idx_c1_c2 | 10      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
 
# MySQL 8.0
mysql> explain select * from t1 order by c1 desc , c2 desc limit 5;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows   | filtered | Extra                       |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
|  1 | SIMPLE      | t1    | NULL       | index | NULL          | idx_c1_c2 | 10      | NULL | 100429 |   100.00 | Using index; Using filesort |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+--------+----------+-----------------------------+
1 row in set, 1 warning (0.01 sec)
```

## 7.group by 不再隐式排序

mysql8.0对于group by字段不再隐式排序，如需要排序，必须显式加上order by子句。

```bash
# 表结构
mysql> show create table tb1\G
*************************** 1. row ***************************
       Table: tb1
Create Table: CREATE TABLE `tb1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `group_own` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=DYNAMIC
1 row in set (0.00 sec)
 
# 表数据
mysql> select * from tb1;
+----+------+-----------+
| id | name | group_own |
+----+------+-----------+
|  1 | 1    |         0 |
|  2 | 2    |         0 |
|  3 | 3    |         0 |
|  4 | 4    |         0 |
|  5 | 5    |         5 |
|  8 | 8    |         1 |
| 10 | 10   |         5 |
+----+------+-----------+
7 rows in set (0.00 sec)
 
# MySQL 5.7
mysql> select count(id), group_own from tb1 group by group_own;
+-----------+-----------+
| count(id) | group_own |
+-----------+-----------+
|         4 |         0 |
|         1 |         1 |
|         2 |         5 |
+-----------+-----------+
3 rows in set (0.00 sec)
 
# MySQL 8.0.11
mysql> select count(id), group_own from tb1 group by group_own;
+-----------+-----------+
| count(id) | group_own |
+-----------+-----------+
|         4 |         0 |
|         2 |         5 |
|         1 |         1 |
+-----------+-----------+
3 rows in set (0.00 sec)
 
# MySQL 8.0.11显式地加上order by进行排序
mysql> select count(id), group_own from tb1 group by group_own order by group_own;
+-----------+-----------+
| count(id) | group_own |
+-----------+-----------+
|         4 |         0 |
|         1 |         1 |
|         2 |         5 |
+-----------+-----------+
3 rows in set (0.00 sec)
```


## 8.JSON 特性增强

MySQL8大幅改进了对JSON的支持，添加了基于路径查询参数从JSON字段中抽取数据的JSON_EXTRACT()函数，以及用于将数据分别组合到JSON数组和对象中的JSON_ARRAYAGG()和JSON_OBJECTAGG()聚合函数。

在主从复制中，新增参数binlog_row_value_options，控制json数据的传输方式，允许对于json类型部分修改，在binlog中只记录修改的部分，减少json大数据只有少量修改的情况下，对资源的占用。

## 9. redo & undo 日志加密

增加如下两个参数，用于控制redo、undo日志的加密。

[innodb_undo_log_encrypt](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_undo_log_encrypt)  
[innodb_undo_log_encrypt](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_undo_log_encrypt)

## 10.innodb select for update跳过锁等待

select ... for update , select ... for share(8.0新增语法)添加NOWAIT、SKIP LOCKED语法，跳过锁等待，或者跳过锁定。  
在5.7及之前的版本，select ... for update ，如果获取不到锁，会一直等待，知道innodb_lock_wait_timeout超时。

在8.0版本，通过添加nowait，skip locked语法，能够立即返回。如果查询的行已经加锁，那么nowait会立即报错返回，而skip locked也会立即返回，只是返回的结果中不包含被锁定的行。

```bash
# session1:
mysql> begin;
mysql> select * from t1 where c1 = 2 for update;
+------+-------+
| c1   | c2    |
+------+-------+
|    2 | 60530 |
|    2 | 24678 |
+------+-------+
2 rows in set (0.00 sec)
 
# session2:
mysql> select * from t1 where c1 = 2 for update nowait;
ERROR 3572 (HY000): Statement aborted because lock(s) could not be acquired  immediately and NOWAIT is set.
mysql> select * from t1 where c1 = 2 for update skip locked;
Empty set (0.00 sec)
```

## 11.增加SET_VAR语法

在sql语法中增加SET_VAR语法，动态调整部分参数，有利于提升语句性能。

- select /*+ SET_VAR(sort_buffer_size = 16M) */ id from test order id ;
- insert /*+ SET_VAR(foreign_key_checks=OFF) */ into test(name) values(1);

## 12.支持不可见索引

使用INVISIBLE关键字在创建表或者进行表变更中设置索引是否可见。索引不可见只是在查询时优化器不使用该索引，即使使用force index，优化器也不会使用该索引，同时优化器也不会报索引不存在的错误，因为索引仍然真实存在，在必要时，也可以快速的恢复成可见。

## 13.支持直方图

优化器会利用column_statistics的数据，判断字段的值的分布，得到更准确的执行计划。

可以使用ANALYZE TABLE table_name \[UPDATE HISTOGRAM on col_name with N BUCKETS | DROP HISTOGRAM ON clo_name\] 来收集或者删除直方图信息。

直方图统计了表中某些字段的数据分布情况，为优化选择高效的执行计划提供参考，直方图与索引有着本质的区别，维护一个索引有代价。每一次insert、update、delete都会需要更新索引，会对性能有一定影响。而直方图一次创建永不更新，除非明确去更新它。所以不会影响insert、update、delete的性能。

```bash
# 添加/更新直方图
mysql> analyze table t1 update histogram on c1, c2 with 32 buckets;
+--------+-----------+----------+-----------------------------------------------+
| Table  | Op        | Msg_type | Msg_text                                      |
+--------+-----------+----------+-----------------------------------------------+
| db.t1 | histogram | status   | Histogram statistics created for column 'c1'. |
| db.t1 | histogram | status   | Histogram statistics created for column 'c2'. |
+--------+-----------+----------+-----------------------------------------------+
2 rows in set (2.57 sec)
 
# 删除直方图
mysql> analyze table t1 drop histogram on c1, c2;
+--------+-----------+----------+-----------------------------------------------+
| Table  | Op        | Msg_type | Msg_text                                      |
+--------+-----------+----------+-----------------------------------------------+
| db.t1 | histogram | status   | Histogram statistics removed for column 'c1'. |
| db.t1 | histogram | status   | Histogram statistics removed for column 'c2'. |
+--------+-----------+----------+-----------------------------------------------+
2 rows in set (0.13 sec)
```

## 14.新增innodb_dedicated_server参数

能够让innodb根据服务器上检测到的内存大小自动配置innodb_buffer_pool_size,innodb_log_file_size,innodb_flush_method三个参数。

## 15.日志分类更详细

在错误信息中添加了错误信息编号\[MY-010311\]和错误所属子系统\[server\]

```bash
# MySQL 5.7
2018-06-08T09:07:20.114585+08:00 0 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2018-06-08T09:07:20.117848+08:00 0 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2018-06-08T09:07:20.117868+08:00 0 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
 
 
# MySQL 8.0
2018-06-21T17:53:13.040295+08:00 28 [Warning] [MY-010311] [Server] 'proxies_priv'  entry '@ root@localhost' ignored in --skip-name-resolve mode.
2018-06-21T17:53:13.040520+08:00 28 [Warning] [MY-010330] [Server] 'tables_priv'  entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2018-06-21T17:53:13.040542+08:00 28 [Warning] [MY-010330] [Server] 'tables_priv'  entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
```

## 17.增加资源组

MySQL8.0新增了一个资源组功能，用于调控线程优先级以及绑定CPU核。  
MySQL用户需要有RESOURCE_GROUP_ADMIN权限才能创建、修改、删除资源组。
在Linux环境下，MySQL进程需要有CAP_SYS_NICE权限才能使用资源组完整功能。

```bash
[root@localhost~]# sudo setcap cap_sys_nice+ep /usr/local/mysql8.0/bin/mysqld
[root@localhost~]# getcap /usr/local/mysql8.0/bin/mysqld
/usr/local/mysql8.0/bin/mysqld = cap_sys_nice+ep
```

默认提供两个资源组，分别是USR_default，SYS_default

创建资源组：  
create resource group test_resource_group type=USER vcpu=0,1 thread_priority=5;  
将当前线程加入资源组：  
SET RESOURCE GROUP test_resouce_group;  
将某个线程加入资源组：  
SET RESOURCE GROUP test_resouce_group FOR thread_id;  
查看资源组里有哪些线程：  
select * from Performance_Schema.threads where RESOURCE_GROUP='test_resouce_group';  
修改资源组：  
alter resource group test_resouce_group vcpu = 2,3 THREAD_PRIORITY = 8;  
删除资源组 ：  
drop resource group test_resouce_group;

```bash
# 创建资源组
mysql>create resource group test_resouce_group type=USER vcpu=0,1 thread_priority=5;
Query OK, 0 rows affected (0.03 sec)

mysql> select * from RESOURCE_GROUPS;
+---------------------+---------------------+------------------------+----------+-----------------+
| RESOURCE_GROUP_NAME | RESOURCE_GROUP_TYPE | RESOURCE_GROUP_ENABLED | VCPU_IDS |  THREAD_PRIORITY |
+---------------------+---------------------+------------------------+----------+-----------------+
| USR_default         | USER                |                      1 | 0-3      |                0 |
| SYS_default         | SYSTEM              |                      1 | 0-3      |                0 |
| test_resouce_group  | USER                |                      1 | 0-1      |                5 |
+---------------------+---------------------+------------------------+----------+-----------------+
3 rows in set (0.00 sec)

# 把线程id为60的线程加入到资源组test_resouce_group中，线程id可通过Performance_Schema.threads获取
mysql> SET RESOURCE GROUP test_resouce_group FOR 60;
Query OK, 0 rows affected (0.00 sec)

# 资源组里有线程时，删除资源组报错
mysql> drop resource group test_resouce_group;
ERROR 3656 (HY000): Resource group test_resouce_group is busy.

# 修改资源组
mysql> alter resource group test_resouce_group vcpu = 2,3 THREAD_PRIORITY = 8;
Query OK, 0 rows affected (0.10 sec)
mysql> select * from RESOURCE_GROUPS;
+---------------------+---------------------+------------------------+----------+-----------------+
| RESOURCE_GROUP_NAME | RESOURCE_GROUP_TYPE | RESOURCE_GROUP_ENABLED | VCPU_IDS | THREAD_PRIORITY |
+---------------------+---------------------+------------------------+----------+-----------------+
| USR_default         | USER                |                      1 | 0-3      |               0 |
| SYS_default         | SYSTEM              |                      1 | 0-3      |               0 |
| test_resouce_group  | USER                |                      1 | 2-3      |               8 |
+---------------------+---------------------+------------------------+----------+-----------------+
3 rows in set (0.00 sec)

# 把资源组里的线程移出到默认资源组USR_default
mysql> SET RESOURCE GROUP USR_default FOR 60;
Query OK, 0 rows affected (0.00 sec)

# 删除资源组
mysql> drop resource group test_resouce_group;
Query OK, 0 rows affected (0.04 sec)
```

## 18.增加角色管理

角色可以认为是一些权限的集合，为用户赋予统一的角色，权限的修改直接通过角色来进行，无需为每个用户单独授权。

```bash
# 创建角色
mysql> create role role_test;
Query OK, 0 rows affected (0.03 sec)
 
# 给角色授予权限
mysql> grant select on db.* to 'role_test';
Query OK, 0 rows affected (0.10 sec)
 
# 创建用户
mysql> create user 'read_user'@'%' identified by '123456';
Query OK, 0 rows affected (0.09 sec)
 
# 给用户赋予角色
mysql> grant 'role_test' to 'read_user'@'%';
Query OK, 0 rows affected (0.02 sec)
 
# 给角色role_test增加insert权限
mysql> grant insert on db.* to 'role_test';
Query OK, 0 rows affected (0.08 sec)
 
# 给角色role_test删除insert权限
mysql> revoke insert on db.* from 'role_test';
Query OK, 0 rows affected (0.10 sec)
 
# 查看默认角色信息
mysql> select * from mysql.default_roles;
+------+-----------+-------------------+-------------------+
| HOST | USER      | DEFAULT_ROLE_HOST | DEFAULT_ROLE_USER |
+------+-----------+-------------------+-------------------+
| %    | read_user | %                 | role_test         |
+------+-----------+-------------------+-------------------+
1 row in set (0.00 sec)
 
# 查看角色与用户关系
mysql> select * from mysql.role_edges;
+-----------+-----------+---------+-----------+-------------------+
| FROM_HOST | FROM_USER | TO_HOST | TO_USER   | WITH_ADMIN_OPTION |
+-----------+-----------+---------+-----------+-------------------+
| %         | role_test | %       | read_user | N                 |
+-----------+-----------+---------+-----------+-------------------+
1 row in set (0.00 sec)
 
# 删除角色
mysql> drop role role_test;
Query OK, 0 rows affected (0.06 sec)
```

