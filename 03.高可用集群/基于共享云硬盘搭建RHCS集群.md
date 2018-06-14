# 环境准备
 - 准备云主机，3台云主机  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/49428119.jpg)

 - 准备共享云硬盘  
创建VBD模式的共享磁盘，centos系统目前只能挂载VBD模式的共享磁盘。
磁盘挂载至node1、node2节点
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/18164264.jpg)

 - 挂载共享云硬盘  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/49871973.jpg)

# node1、node2节点配置
node1、node2节点配置一模一样

 - 配置主机名
```
hostname node1
exec bash
vim /etc/sysconfig/netowrk
NETWORKING=yes
HOSTNAME=node1
```

 - 配置hosts
```
vim /etc/hosts
192.168.1.142 node1
192.168.1.178 node2
192.168.1.33 server
```

 - 安装软件包
```
yum -y install ricci openais cman rgmanager lvm2-cluster gfs2-utils
```

 - 关闭防火墙
```
service iptables stop
```

 - 关闭ACPI服务
```
service acpid stop
chkconfig acpid stop
```

 - 设置ricci密码
```
passwd ricci
```

 - 启动ricci
```
service ricci restart 
chkconfig ricci on
```

 - 查看挂载到的磁盘
```
[root@node1 ~]# fdisk -l

Disk /dev/xvda: 42.9 GB, 42949672960 bytes
255 heads, 63 sectors/track, 5221 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000b6d96

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1               1         523     4194304   82  Linux swap / Solaris
Partition 1 does not end on cylinder boundary.
/dev/xvda2   *         523        5222    37747712   83  Linux

Disk /dev/xvdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

```
 
 - 建立挂载点
```
mkdir /data
```

 - 为磁盘创建gfs2文件系统
```
mkfs.gfs2 -p lock_dlm -t mycluster:gfs_vbd -j 4 /dev/xvdb
    # -t 锁表名称，锁表名称为clustername:fsname，其中的clustername必须跟集群配置文件中的集群名称保持一致
    # -p LockProtoName：所使用的锁协议名称，通常为lock_dlm或lock_nolock之一；
    # -j Number：指定创建gfs2文件系统时所创建的日志区域个数，一般需要为每个挂载的客户端指定一个日志区域；
    
    
[root@node1 ~]# mkfs.gfs2 -p lock_dlm -t cluster:gfs_vbd -j 4 /dev/xvdb
This will destroy any data on /dev/xvdb.
It appears to contain: Linux GFS2 Filesystem (blocksize 4096, lockproto lock_dlm)

Are you sure you want to proceed? [y/n] y

Device:                    /dev/xvdb
Blocksize:                 4096
Device Size                10.00 GB (2621440 blocks)
Filesystem Size:           10.00 GB (2621438 blocks)
Journals:                  4
Resource Groups:           40
Locking Protocol:          "lock_dlm"
Lock Table:                "cluster:gfs_vbd"
UUID:                      78082cbe-d5b3-07c2-52af-b8d9eb65ee6a

```


# 管理节点配置
 - 修改主机名
```
hostname server
exec bash
vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=server
```

 - 配置hosts文件
```
vim /etc/hosts
192.168.1.142 node1
192.168.1.178 node2
192.168.1.33 server
```

 - 关闭防火墙
```
service iptables stop
```

 - 安装集群控制软件
```
yum -y install luci
```

 - 启动luci  
启动后使用https://serverIP:prot访问  
```
[root@server ~]# service luci start
Adding following auto-detected host IDs (IP addresses/domain names), corresponding to `server' address, to the configuration of self-managed certificate `/var/lib/luci/etc/cacert.config' (you can change them by editing `/var/lib/luci/etc/cacert.config', removing the generated certificate `/var/lib/luci/certs/host.pem' and restarting luci):
	(none suitable found, you can still do it manually as mentioned above)

Generating a 2048 bit RSA private key
writing new private key to '/var/lib/luci/certs/host.pem'
Starting saslauthd:                                        [  OK  ]
Start luci...                                              [  OK  ]
Point your web browser to https://server:8084 (or equivalent) to access luci

```

 - 浏览器访问控制节点
地址为https,
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/20060034.jpg)

 - 使用root账号密码登录
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/73970230.jpg)

 - 创建集群  
Manage Clusters ---> Create
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/80839084.jpg)

 - 填写集群信息  
填写相关的信息，密码为ricci的密码，填写完毕后点击Create Cluster等待集群创建
注意：名称要与gfs2中指定的集群名称一致
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/97214563.jpg)

 - 集群创建成功
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/25598831.jpg)

 - 创建Resource
创建前先将共享磁盘挂载到主机中
```
mount /dev/xvdb /data
```
点击Resource--->add  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/3632580.jpg)

 - 创建故障转移域
Failover Domains ---> Add --->create  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/24816748.jpg)

 - 创建服务组group_1  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/41066956.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/15574833.jpg)

 - 启动服务组group_1  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/7737475.jpg)

 - 创建服务器group_2  
用同样的方法创建group_2并启动  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/32013357.jpg)

# 查看集群状态
 - 查看集群状态  
```
[root@node1 ~]# clustat 
Cluster Status for cluster @ Thu Jun 14 16:05:52 2018
Member Status: Quorate

 Member Name                                                    ID   Status
 ------ ----                                                    ---- ------
 node1                                                              1 Online, Local, rgmanager
 node2                                                              2 Online, rgmanager

 Service Name                                          Owner (Last)                                          State         
 ------- ----                                          ----- ------                                          -----         
 service:group_1                                       node1                                                 started       
 service:group_2                                       node2                                                 started       
You have new mail in /var/spool/mail/root

```

 - 查看挂载信息
```
[root@node1 ~]# df -H
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda2       38G  3.2G   33G   9% /
tmpfs           511M   33M  478M   7% /dev/shm
/dev/xvdb        11G  543M   11G   6% /data

```

# 验证共享磁盘
在node2中创建文件，验证node1中同步成功。  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/46416128.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/11745259.jpg)
  
  
  
使用node2编辑ctyun文本文件，在node1中无法编辑，说明GFS2有同步和锁机制  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/35598089.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/87706881.jpg)

验证文本同步功能  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/33316862.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/40296145.jpg)