# 应用场景
[基础知识](http://blog.51cto.com/wzde2012/2054015)  
node1与node2搭建WSFC高可用集群，MSDTC配置共享存储单活。此时只有在主节点(例如node1)中可以看到共享云硬盘，服务器上面运作的内容转移至其它活着的服务器上继续运行(包括共享存储)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/3725882.jpg)

# 支持挂载SCSI磁盘的公共镜像

SCSI类型共享云磁盘只支持以下操作系统挂载，需要安装最新的pvscsi 驱动
 - Windows Server 2012 R2 Datacenter 64bit
 - Windows Server 2012 R2 Standard 64bit
 - Windows Server 2012 R2 Essentials 64bit
 - Windows Server 2008 R2 Datacenter 64bit
 - Windows Server 2008 R2 Enterprise 64bit
 - Windows Server 2008 R2 Standard 64bit

# 支持挂载SCSI磁盘的私有镜像

[下载驱动](https://github.com/UVP-Tools/SAP-HANA-Tools)
 - Novell SUSE Linux Enterprise Server 11 SP4 64bit (内核版本号为3.0.101-68-default or 3.0.101-80-default)
 - Novell SUSE Linux Enterprise Server 12 64bit (内核版本号为3.12.51-52.31-default)
 - Novell SUSE Linux Enterprise Server 12 SP1 64bit (内核版本号为3.12.67-60.64.24-default)
 
# 共享磁盘
VBD：存在空间分配冲突、数据不同步、数据不一致  
SCSI：只能挂载到Windows上的指定镜像

# 域控制器搭建
 - 与控制器的主机名与域名保持一致
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/76432767.jpg)

 - 主机IP设置为静态IP
静态IP为创建主机时指定或者自动获取的，关于自动获取一旦获取后无法使用其他IP
DNS第一条写自己的iP，第二条可以写其他外网IP

主机IP设置为静态IP，天翼云主机只有在创建云主机时才能手动设定IP。如果创建云主机时选择了DHCP，那么可以将IP地址设置为获取到的地址。  
注：如果创建云主机时选择了自动获取IP地址，又在网卡中设定其他地址会网络不通。
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/67561808.jpg)

 - 添加角色  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/24605955.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/69745773.jpg)

 - 基于角色或基于功能安装  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/69423797.jpg)

 - 选择角色  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/61163620.jpg)

 - 功能默认  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/6803174.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/42519296.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/65553152.jpg)

 - 安装  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/51457894.jpg)

 - 将此服务器提升为域控制器  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/31280168.jpg)
 
 - 使用新的林  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/73648310.jpg)

 - 设定密码  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/65828932.jpg)

 - DNS委派不用选  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/26404334.jpg)

 - 填写名称  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/89695281.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/13708759.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/7475041.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/32347293.jpg)

 - 安装，安装完毕后会自动重启  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/58233721.jpg)



# 集群节点配置

 - 购买共享磁盘  
在天翼云够买两块共享云硬盘(SCSI、共享云硬盘)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/73401673.jpg)

 - 挂载共享磁盘  
将共享云磁盘分别挂载到集群节点中(node1、node2)，在集群几点中添加磁盘，然后找到计算机->管理->文件和存储服务->磁盘，右键挂载的磁盘，选择联机，联机后，再右键，初始化，选择新建卷，并分配驱动器(这里用的是E盘、F盘)。如果磁盘无法使用，可以重启服务器，若共享磁盘无法显示则需要更新pvdriver驱动。

 - 清理sysprep  
在node1、node2中执行如下命令  
```
c:/windows/System32/Sysprep/Sysprep.exe  
```
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/56773222.jpg)

 - 创建虚拟私有云  
节点需要用到第二块网卡，所以创建一个与之前不同网段的虚拟私有云，并创建一个网卡在新的网段中。  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/37181764.jpg)

 - 添加网卡  
添加网卡，并选择新创建的网段  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/2360701.jpg)

 - 第一张网卡配置DNS  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/23560512.jpg)

 - 去掉第二块网卡的  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/30350877.jpg)

 - 禁用第二块网卡的  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/73785994.jpg)

 - 显示工具栏  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/69784151.jpg)

 - 高级设置  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/16567337.jpg)

 - 设定优先级  
将绑定EIP的网卡放到最上面  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-12/42120934.jpg)

 - 添加故障转移集群功能  
 ![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/80555922.jpg)

 - 选择故障转移集群功能  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/21856860.jpg)

 - 点击安装  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/96774839.jpg)

 - 集群节点加入域  
node1、node2打开计算机属性->更改设置，点击更改，输入正确的主机名与域名  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/32865091.jpg)

 - 加入成功  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/41528759.jpg)

 - 允许域用户远程登录  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/92793600.jpg)

 - 注意事项  
  - node2的ip需要根据节点二自己的ip来配置    
  - node2的计算机名需要与节点一不同  


# 故障转移集群搭建
 - 使用域管理员登录任意一节点  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/56271852.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/81118521.jpg)

 - 打开故障转移集群  
 ![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/42926783.jpg)

 - 验证配置  
 ![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/27414816.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/10891281.jpg)

 - 输入解决节点的名称    
主机名.域名，这里是node1.libin.com  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/14731275.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/98406968.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/42317128.jpg)
 

 - 验证  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/92345531.jpg)

 - 等待验证完成  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/49995767.jpg)

 - 验证完成  
如果这一步提示该配置不合适进行群集，说明前面配置有问题，需要查看报告解决完问题再继续下面的步骤。  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/80446775.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/73917625.jpg)

 - 创建集群  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/78708801.jpg)

 - 定名称  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/1057522.jpg)

 - 下一步  
 ![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/64169403.jpg)


 - 等待创建集群  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/39337286.jpg)

 - 集群创建成功  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-13/71313759.jpg)

# 集群状态验证  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/4052802.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/25505634.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/17923850.jpg)
# MSDTS集群搭建  
 - 配置角色  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/88824270.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/8783169.jpg)

 - 选择分布式事务协调器  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/1501628.jpg)

 - 集群名称  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/66554802.jpg)

 - 选择磁盘  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/74029991.jpg)

 - 下一步  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/19822160.jpg)

 - 角色配置成功  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/8834241.jpg)

# 共享卷验证  
 - 此时node2为主节点，共享磁盘挂载在node2节点，对于node1磁盘不可见  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/80761474.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/53839819.jpg)

 - 故障转移  
关闭node2节点  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/38363459.jpg)
磁盘自动转移到node1节点  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-14/75995972.jpg)

# 故障排除  
解决方法:远程计算机需要网络级别身份验证  
https://jingyan.baidu.com/article/380abd0a713f061d91192c63.html


 - 无法建立集群  
两台节点可以pingIP通，ping域名通，如果ping不同，排查网络，排查域控制器中的DNS正向解析