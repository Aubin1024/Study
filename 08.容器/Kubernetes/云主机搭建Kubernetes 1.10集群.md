# 一、基础环境

#### 云主机
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-7-4/16169198.jpg)

#### 下载软件包
将所有软件下载至`/data`目录
```
# 链接：https://pan.baidu.com/s/13DlR1akNBCjib5VFaIjGTQ 密码：1l69
```

#### master到node做免密认证
```
ssh-keygen
ssh-copy-id root@192.168.1.237
ssh-copy-id root@192.168.1.100
ssh-copy-id root@192.168.1.188
```

#### 设定主机名与host文件
```
# 分别设定node与master的主机名
hostnamectl set-hostname master
exec bash

# 同步所有主机的hosts文件
vim /etc/hosts
192.168.1.78 master localhost
192.168.1.237 node1
192.168.1.100 node2
192.168.1.188 node3
```

#### 解决DNS解析localhost
此云主机的DNS解析localhost会解析到一个鬼地址，这是个大坑。kubeadm初始化是会用到localhost。如果你的主机能解析到自己的IP，那么这步可以跳过。如果不能则需要自己搭建一个DNS，将localhost解析到自己。
```
# 1.检测
[root@node2 ~]# nslookup localhost
Server:		118.118.118.9
Address:	118.118.118.9#53

Non-authoritative answer:
Name:	localhost.openstacklocal
Address: 183.136.168.91

# 2.搭建DNS
yum -y install dnsmasq
cp /etc/resolv.conf{,.bak}
rm -rf /etc/resolv.conf
echo -e "nameserver 127.0.0.1\nnameserver $(hostname -i)" >> /etc/resolv.conf
chmod 444 /etc/resolv.conf
chattr +i /etc/resolv.conf
echo -e "server=8.8.8.8\nserver=8.8.4.4" > /etc/dnsmasq.conf
echo -e "$(hostname -i)\tlocalhost.$(hostname -d)" >> /etc/hosts
service dnsmasq restart

# 3.再次检测
[root@master ~]# nslookup localhost
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	localhost
Address: 192.168.1.78
```

#### 同步系统时间
```
ntpdate 0.centos.pool.ntp.org
```

#### 关闭防火墙
```
iptables -F
systemctl stop firewalld
systemctl disable firewalld
```

#### 关闭SELinux & 关闭swap

```
swapoff -a 
sed -i 's/.*swap.*/#&/' /etc/fstab
setenforce 0
```

#### 确认时区
```
timedatectl set-timezone Asia/Shanghai 
systemctl restart chronyd.service 

```

#### 修改系统参数

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

#### 安装docker
```
tar -xvf docker-packages.tar
cd docker-packages
yum -y install local *.rpm
systemctl start docker && systemctl enable docker
```

#### 配置镜像加速器
```
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://lw9sjwma.mirror.aliyuncs.com"]
}

systemctl daemon-reload 
systemctl restart docker
```

#### 配置k8s的yum源
```
vim /etc/yum.repos.d/k8s.repo
[k8s]
name=k8s
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
```

#### 获取kube软件包
```
cd kube-packages-1.10.1                 # 软件包在网盘中下载    
tar -xvf kube-packages-1.10.1.tar
cd kube-packages-1.10.1
yum -y install local *.rpm 
systemctl start kubelet && systemctl enable kubelet
```

#### 统一k8s与docker的驱动
```
# 1.查看docker驱动
 docker info | Cgroup Driver
Cgroup Driver: cgroupfs

# 修改k8s配置文件与docker保持一致
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```


#### 导入基础镜像
```
cd /data
docker load -i k8s-images-1.10.tar.gz 
```

# 二、初始化master节点
```
# 初始化master 指定的版本要与kubeadm版本一致
# kubeadm只给定了最少选项，集群名称等等都没有指定，kubeadm init
[root@master ~]# kubeadm init --kubernetes-version=v1.10.1 --pod-network-cidr=10.244.0.0/16

# 初始化完成后得到如下信息

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.1.78:6443 --token qabol0.c2gq0uyfxvpqr8bu --discovery-token-ca-cert-hash sha256:2237ec7b8efd5a8f68adcb04900a0b17b9df2a78675a7d62b4aef644a7f62c05
# kubeadm join 是node节点加入集群的命令，注意token的有效期
```

#### 若要通过普通用户使用群集，需要执行以下步骤
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 基本命令
```
# 查看pods
kubectl get pods


# 查看系统pods 
[root@master ~]# kubectl get pods -n kube-system
NAME                             READY     STATUS     RESTARTS   AGE
etcd-master                      0/1       Pending    0          1s
kube-apiserver-master            0/1       Pending    0          1s
kube-controller-manager-master   0/1       Pending    0          1s
kube-dns-86f4d74b45-d42zm        0/3       Pending    0          8h
kube-proxy-884h6                 1/1       NodeLost   0          8h
kube-scheduler-master            0/1       Pending    0          1s

# 查看集群各组件状态信息
[root@master ~]# kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
You have new mail in /var/spool/mail/root

```

# 三、node加入集群

```
# 确保node节点cgroup驱动保持一致
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# 命令来自集群初始化之后额显示中
kubeadm join 192.168.1.78:6443 --token v0866r.u7kvg5js1ah2u1bi --discovery-token-ca-cert-hash sha256:7b36794f4fa5121f6a5e309d0e312ded72997a88236a93ec7da3520e5aaccf0e

# master节点查看nodes信息
[root@master data]# kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
master    NotReady      master    57m       v1.10.1
node1     NotReady      <none>    27m       v1.10.1
node2     NotReady      <none>    11s       v1.10.1
node3     NotReady   <none>    4s        v1.10.1
You have new mail in /var/spool/mail/root

```

# 四、部署网络
#### 部署
[flannel官网](https://github.com/coreos/flannel)  
flannel下载时不用科学上网,flannel的yml文件会自动去quay.io网站中下载镜像。
```
# 1.1使用软件包中的flannel,并指pod映射到哪个主机的网卡上面。
vim kube-flannel.yml
command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr","-iface=eth0" ]
# 以下要按顺序创建，先创建rbac，之前没有穿件rbac导致pod正常创建，但是pin不同
kubectl apply -f kube-flannel-rbac.yml
kubectl apply -f kube-flannel.yml
# 后，节点的状态会变为ready
[root@master1 kubernetes1.10]# kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready      master    57m       v1.10.1
node1     Ready      <none>    27m       v1.10.1
node2     Ready      <none>    11s       v1.10.1
node3     Ready   <none>    4s        v1.10.1

# 2.从官网下载最新的flannel，k8s1.7+ 直接执行以下命令即可
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### flannel配置文件修改
```
kube-flannel.yml中指定使用的网段
"Network": "10.244.0.0/16"

默认使用16位掩码，则在各node中都分配一个10.244.0.0/8的网络
```

# 五、部署dashboard
```
kubectl apply -f kubernetes-dashboard-http.yam
kubectl apply -f admin-role.yaml
kubectl apply -f kubernetes-dashboard-admin.rbac.yaml
```

#### 命令行常用命令
```
# 查看pod信息，默认显示default名称空间下的pod
[root@master ~]# kubectl get pods
No resources found.

# 指定名称空间写pod
[root@master ~]# kubectl get pods -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-master                             1/1       Running   0          3h
kube-apiserver-master                   1/1       Running   0          3h
kube-controller-manager-master          1/1       Running   0          3h
kube-dns-86f4d74b45-bzbvc               3/3       Running   0          3h
kube-flannel-ds-5ghhj                   1/1       Running   0          2h
kube-flannel-ds-ht4xd                   1/1       Running   0          3h
kube-flannel-ds-kbm5g                   1/1       Running   0          3h
kube-flannel-ds-mlj4r                   1/1       Running   0          2h
kube-proxy-9xxnd                        1/1       Running   0          3h
kube-proxy-n9w5x                        1/1       Running   0          3h
kube-proxy-nkn8c                        1/1       Running   0          2h
kube-proxy-shd6l                        1/1       Running   0          2h
kube-scheduler-master                   1/1       Running   0          3h
kubernetes-dashboard-5c469b58b8-rjfx6   1/1       Running   0          1h


# 显示更详细的pod信息，此时各pod中都运行了一个kube-proxy和flannel容器
-o wide 显示更详细的信息，报错node节点iP、主机名
[root@master ~]# kubectl get pods -n kube-system -o wide
NAME                                    READY     STATUS    RESTARTS   AGE       IP              NODE
etcd-master                             1/1       Running   0          3h        192.168.1.78    master
kube-apiserver-master                   1/1       Running   0          3h        192.168.1.78    master
kube-controller-manager-master          1/1       Running   0          3h        192.168.1.78    master
kube-dns-86f4d74b45-bzbvc               3/3       Running   0          3h        10.244.0.2      master
kube-flannel-ds-5ghhj                   1/1       Running   0          2h        192.168.1.188   node3
kube-flannel-ds-ht4xd                   1/1       Running   0          3h        192.168.1.78    master
kube-flannel-ds-kbm5g                   1/1       Running   0          3h        192.168.1.237   node1
kube-flannel-ds-mlj4r                   1/1       Running   0          2h        192.168.1.100   node2
kube-proxy-9xxnd                        1/1       Running   0          3h        192.168.1.237   node1
kube-proxy-n9w5x                        1/1       Running   0          3h        192.168.1.78    master
kube-proxy-nkn8c                        1/1       Running   0          2h        192.168.1.100   node2
kube-proxy-shd6l                        1/1       Running   0          2h        192.168.1.188   node3
kube-scheduler-master                   1/1       Running   0          3h        192.168.1.78    master
kubernetes-dashboard-5c469b58b8-rjfx6   1/1       Running   0          1h        10.244.0.3      master
```

# 六、kubeadm清空配置
```
# 清空kubectl
kubeadm reset

# 清空网络信息
ip link del cni0
ip link del flannel.1
```

# 七、踩过的那些坑
 - 确保master与node的DNS解析localhost能解析到自己的IP
 - node加入master确保token不过期
 - node确保kubelet正常启动并运行
 - flannel网络要先创建kube-flannel-rbac.ymal再创建 kube-flannel.yml


# 八、token过期的解决办法
```
# 1.查看已经存在的token
kubeadm token list

# 2.创建token
kubeadm token create

# 3.查看ca证书的sha256编码
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

# 4.node使用新的token加入集群
kubeadm join --token acb123 --discovery-token-ca-cert-hash sha256:efg456  172.16.6.79:6443 --skip-preflight-checks
    # abc123    新创建的Token
    # efg456    证书的sha256编码
    # IP+Port   Master的IP+Port

```

# 感谢
 - [无痴迷，不成功](https://www.cnblogs.com/justmine/p/8886675.html)
 - [discsthnew](https://blog.csdn.net/mailjoin/article/details/79686934)







