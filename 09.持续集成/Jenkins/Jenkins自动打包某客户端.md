# 安装JAVA环境
 - 安装JDK
```
yum =y install java-1.8.0-openjdk-devel
```

 - 设定环境变量
```
vim /etc/profile.d/java.sh
export JAVA_HOME=/usr/java/latest
export PATH=$JAVA_HOME/bin:$PATH

source /etc/profile.d/java.sh
```

 - 验证JDK
```
java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-b10)
OpenJDK 64-Bit Server VM (build 25.171-b10, mixed mode)
```

# 安装apktools
apktools是打包安装所用到的工具
```
cd /usr/local/bin
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.2.0.jar
chmod +x *
```

# 安装SVN
 - 安装
```
yum install subversion
```

 - 创建根路径
```
mkdir /svn
svnadmin create /svn/workspace
```

- 配置SVN
```
vim /svn/workspace/conf/svnserver.conf
[general]
anon-access = read
auth-access = write
password-db = passwd 
authz-db = authz 
```
- 创建用户
```
vim /svn/workspace/conf/passwd
[users]
libin=IAAS+.libin
android=IAAS+.android
```
 - 设定用户权限
```
vim /svn/workspace/conf/authz
[/svn/workspace/android]
android = rw
```

 - 启动SVN
```
svnserve -d -r /svn
```

 - 客户端连接
```
/svn目录为根目录,则android的连接应该为svn://IP/workspace/android
```
 - Windows客户端连接  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-25/53559562.jpg)

 - 将apk解包后放到本地SVN目录  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/13969925.jpg)

 - 右击commit提交  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/68699587.jpg)


# 安装Jenkins
 - 下载安装Jenkins [下载地址](https://jenkins.io/download/)

```
yum -y install wget 
wget https://pkg.jenkins.io/redhat-stable/jenkins-2.73.2-1.1.noarch.rpm
rpm -ivh jenkins-2.73.2-1.1.noarch.rpm
```

 - 配置端口
```
vim /etc/sysconfig/jenkins
JENKINS_PORT="8088"
```

 - 启动服务
```
systemctl restart jenkins
```
 - 访问Jenkins
```
浏览器打开IP:port
获取登录密钥，填入页面后可以打开页面
cat /var/lib/jenkins/secrets/initialAdminPassword
f6ee991ef5654a3a84a4b2180130a5b4
```
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-25/71989128.jpg)
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-25/86946836.jpg)

# 配置Jenkins
 - 创建一个新任务  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/23720440.jpg)

 - 创建为自由风格软件包  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/72232821.jpg)

 - 填写项目名称  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/42512002.jpg)

 - 添加源码地址  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/13463419.jpg)

 - 添加触发器  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/62828214.jpg)

 - 配置构建环境  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/98686626.jpg)

 - 配置构建操作  
此命令也就是package.bat中的命令拷贝出来的，脚本有华为提供
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/77785755.jpg)

 - 保存构建信息  
保存后，点击立即构建即可
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/98768898.jpg)

 - 查看构建详情  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/75822857.jpg)

 - 查看构建日志  
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/12989535.jpg)

 - 查看结果  
在日志中可以看到打包结果，可以在在/var/lib/jenkins/workspace/android/Client_for_Android_v1.6.10008/dist中看到输出的apk文件
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-6-4/27737027.jpg)
