[TOC]: # "MySQL5.1主主复制配置"

# MySQL5.1主主复制配置
- [准备](#准备)
- [设置两个数据库相互访问权限：](#设置两个数据库相互访问权限)
- [重启数据库（systemctl restart mysqld）](#重启数据库systemctl-restart-mysqld)
- [互告bin-log信息](#互告bin-log信息)


## 准备

主数据库1：192.168.100.210 主数据库2：192.168.100.212

修改MySQL配置文件： vim /etc/my.cnf

**主数据库1：**

```editorconfig
# Example MySQL config file for very large systems.
#
# This is for a large system with memory of 1G-2G where the system runs mainly
# MySQL.
#
# You can copy this file to
# /etc/my.cnf to set global options,
# mysql-data-dir/my.cnf to set server-specific options (in this
# installation this directory is /usr/local/mysql/var) or
# ~/.my.cnf to set user-specific options.
#
# In this file, you can use all long options that a program supports.
# If you want to know which options a program supports, run the program
# with the "--help" option.

# The following options will be passed to all MySQL clients
[client]
#password       = your_password
port            = 3306
socket          = /tmp/mysql.sock

# Here follows entries for some specific programs

# The MySQL server
[mysqld]
datadir=/var/data/mysql
user=mysql

port            = 3306
socket          = /tmp/mysql.sock
skip-locking
key_buffer_size = 384M
max_allowed_packet = 128M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8
max_connections = 256
skip-name-resolve
log-bin=mysql-bin
wait_timeout=2880000
interactive_timeout=2880000
server-id       = 1

# 需要同步的数据名称,多个数据库则要重复设置: bin-do-db,bin-ignore-db为互斥关系, 只需设置其中一项即可

replicate-do-db=police_center_db


replicate-ignore-table=police_center_db.co_ap_list
replicate-ignore-table=police_center_db.co_ele_fence

# 自增长字段初始值为1

auto-increment-offset=1

# 自增长字段增量值

auto-increment-increment=2

# 跳过所有复制的错误

slave-skip-errors=all

master-connect-retry=60

[mysqld_safe]
log-error=/var/data/mysql/mysqld.log
pid-file=/var/data/mysql/mysqld.pid

[mysqldump]
quick
max_allowed_packet = 500M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout

```

**主数据库2：**

```editorconfig
# Example MySQL config file for very large systems.
#
# This is for a large system with memory of 1G-2G where the system runs mainly
# MySQL.
#
# You can copy this file to
# /etc/my.cnf to set global options,
# mysql-data-dir/my.cnf to set server-specific options (in this
# installation this directory is /usr/local/mysql/var) or
# ~/.my.cnf to set user-specific options.
#
# In this file, you can use all long options that a program supports.
# If you want to know which options a program supports, run the program
# with the "--help" option.

# The following options will be passed to all MySQL clients
[client]
#password       = your_password
port            = 3306
socket          = /tmp/mysql.sock

# Here follows entries for some specific programs

# The MySQL server
[mysqld]
datadir=/var/data/mysql
user=mysql

port            = 3306
socket          = /tmp/mysql.sock
skip-locking
key_buffer_size = 384M
max_allowed_packet = 128M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8
max_connections = 256
skip-name-resolve
log-bin=mysql-bin
wait_timeout=2880000
interactive_timeout=2880000
server-id       = 2
# 需要同步的数据名称,多个数据库则要重复设置: bin-do-db,bin-ignore-db为互斥关系, 只需设置其中一项即可

replicate-do-db=police_center_db

# 自增长字段初始值为2

auto-increment-offset=2

#自增长字段增量值

auto-increment-increment=2

# 跳过所有复制的错误

slave-skip-errors=all

master-connect-retry=60
replicate-ignore-table=police_center_db.co_ap_list
replicate-ignore-table=police_center_db.co_ele_fence

[mysqld_safe]
log-error=/var/data/mysql/mysqld.log
pid-file=/var/data/mysql/mysqld.pid

[mysqldump]
quick
max_allowed_packet = 500M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

## 设置两个数据库相互访问权限：

**主数据库1：**

```mysql
grant replication slave on *.* to '用户名2'@'%' identified by '密码2';

flush privileges;
```

**主数据库2：**
```mysql
grant replication slave on *.* to '用户名1'@'%' identified by '密码1';

flush privileges;
```

## 重启数据库（systemctl restart mysqld）

1. ps -ef |grep mysql
2. 杀掉所有与mysql相关的进程(先杀死check_mysql守护进程)
3. /usr/local/saudit-police/monitor-process/check_mysql &

## 互告bin-log信息

进入mysql命令行：
mysql -u用户名 -p密码

**主数据库1：**

查看日志：

```mysql
show master status;
```

![](https://img-blog.csdnimg.cn/20190706131343369.png)

设置同步：

```mysql
change master to master_host='数据库1IP',master_user='数据库1用户名',master_password ='数据库1密码',master_log_file='mysql-bin.000001',master_log_pos=154;
```
**备注:** master_log_file与File值一致，master_log_pos与Position值一致

开始同步：

``` mysql
start slave ;
```

查看同步情况：

```mysql
show slave status \G
```

当看到了两个Yes，即：
Slave_IO_Running: Yes
Slave_SQL_Running: Yes

说明已经配置成功了

![](https://img-blog.csdnimg.cn/20190706131343372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoaW5hX2hkeQ==,size_16,color_FFFFFF,t_70)

**主数据库2：**

查看日志：

```mysql
show master status;
```

![](https://img-blog.csdnimg.cn/20190706131343369.png)

设置同步：

```mysql
change master to master_host='数据库2IP',master_user='数据库2用户名',master_password ='数据库2密码',master_log_file='mysql-bin.000001',master_log_pos=154;
```
**备注:** master_log_file与File值一致，master_log_pos与Position值一致

开始同步：

``` mysql
start slave ;
```

查看同步情况：

```mysql
show slave status \G
```

当看到了两个Yes，即：
Slave_IO_Running: Yes
Slave_SQL_Running: Yes

说明已经配置成功了

