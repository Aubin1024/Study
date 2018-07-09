# 简介
Flannel使集群中所有node节点创建的Docker容器都具有全集群唯一的新IP地址。并在这些IP地址之间建立一个覆盖网络(Overlay Network),通过覆盖网络，将数据包传送到目标容器。


#### 同一pod内通信
同一个pod共享网络名称空间、共享TCP/IP协议栈。所以可以直接通过localhost+port进行通信。

#### 同一node中不同pod通信
pod1与pod2在同一个node中，可以直接通过docker0网桥直接转发数据包，数据不需要经过Flannel。

#### 不同node中的不同pod通信
发送  
 - 数据从容器发送到ethXXX
 - vethXXX将数据包发送给docker0,报文通过bridge docker0发出去
 - 查找路由表，不同node容器的ip报文都会转发到flannel0虚拟网卡
 - flannel0将报文发给flanneld
 - flanneld通过etcd维护了一张路由表，将原来的报文UDP封装一层，通过ethXXX发出去  

接收
 - 报文通过node之间的网络找到目标node的ethXXX
 - ethXXX将报文交给坚挺在8285的flanneld
 - 数据解包，发送给flannel0
 - flannel0将报文交给docker0
 - docker找到自己连接的容器，将报文发送出去