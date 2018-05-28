# 安装MySQL服务器
 - 安装数据库
```
yum install -y mariadb-server
```

 - 初始化数据库
```
mysql_secure_installation	#MySql初始化脚本，以下为每一项的翻译
    是否设置root密码
    输入密码
    确认密码
    是否设置匿名用户
    是否允许root远程登录
    删除test数据库
    现在是否生效
    添加PATH变量
```

 - 启动数据库
```
systemctl start mariadb
```
# 主服务器基础搭建

 - 设定主机名，在当前bash生效
```
hostnamectl set-hostname mysql-master
exec bash
```

 - 创建数据库
```
MariaDB [(none)]> create database shuaiguoxia ;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shuaiguoxia        |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```

 - 创建表
```
use shuaiguoxia;
create table blog (name varchar(20),age int,sex varchar(10));
```

 - 查看表结构
```
MariaDB [shuaiguoxia]> desc blog;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| name  | varchar(20) | YES  |     | NULL    |       |
| age   | int(11)     | YES  |     | NULL    |       |
| sex   | varchar(10) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

 - 插入测试数据
```
insert into blog (name,age,sex) values ("guo",27,"nan");
insert into blog (name,age,sex) values ("wu",33,"nan");
insert into blog (name,age,sex) values ("cai",31,"nv");
insert into blog (name,age,sex) values ("li",19,"nan");
insert into blog (name,age,sex) values ("zhao",18,"nan");
insert into blog (name,age,sex) values ("qian",25,"nv");
```

# 设定主服务器

 - 修改配置文件
```
vim /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
server-id=10
log-bin=mysql-bin
relay-log=mysql-relay-bin
replicate-wild-ignore-table=mysq.%
```

 - 给blog表上锁
```
mysql -u root -p

use shuaiguoxia;

MariaDB [shuaiguoxia]> lock tables blog read;
Query OK, 0 rows affected (0.00 sec)
```

 - 导出主库中已经有的数据
```
mysqldump -u root -p -h 127.0.0.1 shuaiguoxia blog > /bak.sql
Enter password: 
```

 - 将数据复制到从节点
```
scp /bak.sql 192.168.1.175:/
```

 - 给从节点创建授权用户
```
grant replication slave on *.* to 'slave_user'@'192.168.1.%' identified  by '1234'
```

# 从节点配置
 - 设定主机名，在当前bash生效
```
hostnamectl set-hostname slave-master
exec bash
```

 - 设定配置文件
```
[root@mysql-slave ~]# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
server-id=21
log-bin=mysql-bin
replicate-wild-ignore-table=mysql.%
```

 - 查看主节点的master状态
```
# 查看主节点的master状态，要在主节点执行
MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 |     2900 |              |                  |
+------------------+----------+--------------+------------------+
```

 - 设定自动复制
```
# 命令中的master_log_file和master_log_pos为主节点查询的结果

MariaDB [(none)]> change master to \
master_host='192.168.1.46',
master_user='slave_user',
master_password='1234',
master_log_file='mysql-bin.000003',
master_log_pos=2900;
```

 - 启动复制
```
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.01 sec)
```

 - 查看复制状态
 ```
 # 正常状态下Slave_IO_Running和 Slave_SQL_Running都为yes
 
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.46
                  Master_User: slave_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 3057
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 686
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: mysql.%
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 3057
              Relay_Log_Space: 982
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 10
1 row in set (0.00 sec)
 ```
 
# 主从失败常见原因

 - 网络不通：排查网络、端口
 - 用户密码不对：检查在主节点创建的用户名密码
 - 用户权限不对：MySQL中对一个用户的标识为IP@username，
 - pos不对：检查从节点设定的开始pos是否为主节点正在进行的pos
 - 开始二进制文件不对：检查从节点开始的二进制文件是否为主节点整个在进行的二进制位内按
 - 防火墙限制：关闭防火墙