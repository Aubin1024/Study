# 原理
**异步复制**

MySQL默认为异步复制，异步复制就是Master数据库吧binglog日志发送给slave数据库，不论slave是否接收到、存储好都与主节点无关。

**半同步复制**

半同步复制就是，当Master将数据发送给Slave节点时会等待Slave反馈，只有Slave将将binlog日志写入到自己的中继日志中，整个同步才算完成。开启半同步复制后，当出现超市，主数据库将会自动转为异步复制模式，直到至少有一台Slave服务器接受到Master的binlog并反馈给Master，这时才会切回半同步复制。

**注意**

半同步复制要在Master与Slave中同时开启，默认为异步复制模式。

# 搭建半同步复制

 - 搭建主从同步(参照前一篇)
 - 主节点安装插件

在主节点安装半同步插件,rpl_semi_sync_master为插件名字，semisync_master.so为模块名字。可以通过`rpm -ql mariadb-server`来查看所有模块
```
MariaDB [(none)]> install plugin rpl_semi_sync_master soname 'semisync_master.so'; 
Query OK, 0 rows affected (0.00 sec)
```

 - 开启半同步插件
```
MariaDB [(none)]> set @@global.rpl_semi_sync_master_enabled=on;
Query OK, 0 rows affected (0.00 sec)
```

 - 查看半同步状态
```
MariaDB [(none)]> show global variables like 'rpl_semi%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| rpl_semi_sync_master_enabled       | ON    |          # 状态为开启
| rpl_semi_sync_master_timeout       | 10000 |          # 超时时间为10s，10s后没有反馈结果给主节点，从节点自动降级为异步复制
| rpl_semi_sync_master_trace_level   | 32    |          # 调试级别
| rpl_semi_sync_master_wait_no_slave | ON    |          # ON表示Slave宕掉后宕Slave追上Master日志时自动切换为半同步方式。。
+------------------------------------+-------+          #↑OFF表示，Slave追赶上后也不会恢复为半同步复制，需要手动恢复
4 rows in set (0.00 sec)
```

 - 查看半同步复制统计信息
```
MariaDB [(none)]>  show global status like 'rpl_semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |  # 此时半同步客户端数量为0
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)
```

 - 从节点安装插件
```
MariaDB [(none)]> install plugin rpl_semi_sync_slave soname 'semisync_slave.so'; 
Query OK, 0 rows affected (0.00 sec)
```

 - 开启半同步复制插件
```
MariaDB [(none)]> set @@global.rpl_semi_sync_slave_enabled=on;
Query OK, 0 rows affected (0.00 sec)
```


```
MariaDB [(none)]> show global variables like 'rpl_semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | ON    |             # 开启状态
| rpl_semi_sync_slave_trace_level | 32    |             # 调试级别
+---------------------------------+-------+
2 rows in set (0.00 sec)
```

 - 查看半同步状态
```
MariaDB [(none)]> show global status like 'rpl_semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | OFF   |                  # 此时状态为OFF，需要手动重启io_thread
+----------------------------+-------+
1 row in set (0.00 sec)
```

 - 手动重启io_thread
```
MariaDB [(none)]> stop slave io_thread;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave io_thread;
Query OK, 0 rows affected (0.00 sec)
```

 - 再次查看半同步状态
```
MariaDB [(none)]> show global status like 'rpl_semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |                  #重启io_thread后为ON
+----------------------------+-------+
1 row in set (0.00 sec)

MariaDB [(none)]> 

```

 - 主节点查看半同步统计
```

MariaDB [(none)]>  show global status like 'rpl_semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |  # 半同步客户端数量为1
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)
```

# 测试半同步
 - 主节点创建测试库
```
MariaDB [(none)]> use shuaiguoxia;
# 创建测试表
MariaDB [(none)]> create table students (id int unsigned primary key auto_increment,name char(20) not null,age int unsigned,gender enum('F','M'),major varchar(30));
```

 - 主节点插入测试数据
```
insert into students (name,age,gender)values('zhang',31,'F');
```

 - 主节点查看同步统计信息
```
MariaDB [shuaiguoxia]>  show global status like 'rpl_semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 3032  |      # 网络平均等待时间，我这里平均时间长是由于主从同步失败导致
| Rpl_semi_sync_master_net_wait_time         | 18197 |
| Rpl_semi_sync_master_net_waits             | 6     |      # 网络等待次数
| Rpl_semi_sync_master_no_times              | 1     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 2876  |
| Rpl_semi_sync_master_tx_wait_time          | 17258 |
| Rpl_semi_sync_master_tx_waits              | 6     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 6     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)

```

 - 主节点查看表
```
MariaDB [shuaiguoxia]> select * from students;
+----+-------+------+--------+-------+
| id | name  | age  | gender | major |
+----+-------+------+--------+-------+
|  1 | zhang |   31 | F      | NULL  |
+----+-------+------+--------+-------+
1 row in set (0.00 sec)
```

 - 从节点查看表,发现信息一致，已经同步
```
MariaDB [shuaiguoxia]> select * from students;
+----+-------+------+--------+-------+
| id | name  | age  | gender | major |
+----+-------+------+--------+-------+
|  1 | zhang |   31 | F      | NULL  |
+----+-------+------+--------+-------+
1 row in set (0.00 sec)

```