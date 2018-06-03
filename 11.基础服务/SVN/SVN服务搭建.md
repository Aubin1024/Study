# 安装SVN
```
yum install subversion
```

# 创建SVN根目录
```
mkdir /svn
svnadmin create /svn/workspace
```

# 配置SVN
```
vim /svn/workspace/conf/svnserver.conf
[general]
anon-access = read              # 匿名用户可读
auth-access = write             # 匿名用户可写
password-db = passwd 
authz-db = authz 
```

# 设置权限分配
```
vim /svn/workspace/conf/passwd
[/svn/workspace/android]
android = rw
```

# 启动SVN
```
svnserve -d -r /svn
```

# 客户端连接
```
/svn目录为根目录,则android的连接应该为svn://IP/workspace/android
```
 - Windows客户端连接
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-25/53559562.jpg)

 - 清除保存的登录信息
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-25/8982081.jpg)

