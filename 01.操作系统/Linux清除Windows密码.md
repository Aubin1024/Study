# 下载安装ntfs-3g
 - 下载驱动让linux挂载windows磁盘
```
https://tuxera.com/opensource/ntfs-3g_ntfsprogs-2017.3.23.tgz
```

 - 安装
```
tar -xvf ntfs-3g_ntfsprogs-2017.3.23.tgz
cd ntfs-3g_ntfsprogs-2017.3.23
./configure
make
make install 
```

# 下载安装chntpw
 - 下载chntpw
```
https://pkgs.org/download/chntpw
wget http://li.nux.ro/download/nux/dextop/el7/x86_64//chntpw-0.99.6-22.110511.el7.nux.x86_64.rpm
yum -y install ./hntpw-0.99.6-22.110511.el7.nux.x86_64.rpm
```

# 挂载windows磁盘

 - 挂载windows的系统盘  
默认第二个分区才是C盘
```
mkdir /win
mount -t  ntfs-3g /dev/xvdb2 /win
```

 - 备份SAM文件
```
cd /win/Windows/System32/config/
cp SAM{,.bak}
```

 - 清除Windows密码
```
chntpw SAM
- - - - User Edit Menu:
 1 - Clear (blank) user password
 2 - Edit (set new) user password (careful with this on XP or Vista)
 3 - Promote user (make user an administrator)
(4 - Unlock and enable user account) [seems unlocked already]
 q - Quit editing user, back to user select
Select: [q] > 1                         # 1 清除密码
Password cleared!

Hives that have changed:
 #  Name
 0  <SAM>
Write hive files? (y/n) [n] : y         # y保存
 0  <SAM> - OK

```

 - 将windows系统盘重新挂载回windows中即可