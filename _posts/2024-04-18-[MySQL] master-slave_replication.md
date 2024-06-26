---
layout:     post
title:      MySQL主从复制
subtitle:   配置笔记
date:       2024-04-18
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_640.jpg
catalog: true
tags:
    - MySQL
---

- 保证mysql服务均开启bin_log
- 保证mysql服务器之间,server_id不重复
- 保证mysql服务器之间,ip互通

当前MySQL版本: 8.0.23 (其他版本可查看相关版本的官方文档)

## 1. 主库

#### 查询ip

> ifconfig

192.168.0.2

#### 是否开启bin_log

> mysql -uroot -p

进入 mysql 客户端命令行交互界面

> show variables like "%log_bin%";

```
+---------------------------------+-----------------------------------+
| Variable_name                   | Value                             |
+---------------------------------+-----------------------------------+
| log_bin                         | ON                                |
| log_bin_basename                | /usr/local/var/mysql/binlog       |
| log_bin_index                   | /usr/local/var/mysql/binlog.index |
| log_bin_trust_function_creators | OFF                               |
| log_bin_use_v1_row_events       | OFF                               |
| sql_log_bin                     | ON                                |
+---------------------------------+-----------------------------------+
6 rows in set (0.00 sec)
```

#### 主库当前的binlog文件,以及Position

查看服务器正在写入 binlog 文件名和位置

> show master status\G;

```
*************************** 1. row ***************************
             File: binlog.000024
         Position: 156
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

#### 检查server_id

各个mysql服务器之间,server_id不能重复,否则复制时会报错;

> SHOW GLOBAL VARIABLES LIKE 'server_id';

```
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 1     |
+---------------+-------+
1 row in set (0.00 sec)
```

> SET GLOBAL server_id = 101;

#### 添加提供给从库使用的用户,用户从主库拉取数据


```
该用户只开放给 host为192.168.0.3的主机使用
> CREATE USER 'sync_user'@'192.168.0.3' IDENTIFIED BY 'hello@2024';
赋予从库权限
> grant replication slave on `*.*` to 'sync_user'@'192.168.0.3';

开放给任何host使用
> CREATE USER 'sync_user'@'%' IDENTIFIED BY 'hello@2024';
> grant replication slave on `*.*` to 'sync_user'@'%';
```

## 2. 从库

ip: 192.168.0.3  


> mysql -uroot -p

#### 检查server_id,保证不与主库相同

> SHOW GLOBAL VARIABLES LIKE 'server_id';

#### 执行同步命令，设置主服务器ip，同步账号密码，同步位置

> change master to master_host='192.168.0.2',master_port=3306,\
master_user='sync_user',master_password='hello@2024',\
master_log_file='binlog.000024',master_log_pos=156;

注意点:master_log_file 和 master_log_pos 是主库上的binlog文件名和位置点

#### 开启同步功能

> start slave;

检查同步状态

> show slave status\G;

```
*************************** 1. row ***************************
Slave_IO_State: Waiting for master to send event
Master_Host: 172.17.2.40
Master_User: photorepl
Master_Port: 4331
Connect_Retry: 60
Master_Log_File: mysql-bin.005502
Read_Master_Log_Pos: 64401238
Relay_Log_File: mysqld-relay-bin.015418
Relay_Log_Pos: 13456757
Relay_Master_Log_File: mysql-bin.005152
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Replicate_Do_DB: 
Replicate_Ignore_DB: mysql
Replicate_Do_Table: 
Replicate_Ignore_Table: 
Replicate_Wild_Do_Table: photo.%
Replicate_Wild_Ignore_Table: mysql.%
Last_Errno: 0
Last_Error: 
```

主要看Slave_IO_Running 和 Slave_SQL_Running 是否正常,为Yes则正常;若有异常可以查看Last_Error分析原因

#### 停止同步

> STOP SLAVE;

> RESET SLAVE;


## 可参考资料

- [https://dev.mysql.com/doc/refman/8.0/en/sql-replication-statements.html](https://dev.mysql.com/doc/refman/8.0/en/sql-replication-statements.html)
- [https://dev.mysql.com/doc/refman/8.0/en/change-master-to.html](https://dev.mysql.com/doc/refman/8.0/en/change-master-to.html)

## 扩展

> SHOW VARIABLES LIKE 'binlog_format';

```
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.00 sec)
```

ROW：不记录上下文信息（不记录 SQL ），只记录数据的变化，也就是说会记录数据变化的结果而不是 SQL 。

> show binlog events;

> \q 退出mysql客户端

> mysqlbinlog -H -vvvvvv /usr/local/var/mysql/binlog.000025


