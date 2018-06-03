CentOS7 yum groupinstall chinese-support提示没有可用的安装包？
```
yum groupinstall "fonts"
```
临时设定系统语言
```
LANG='zh_CN.UTF-8'
```
修改系统默认语言
```
vim /etc/locale.conf 
LANG=zh_CN.UTF-8
```

# Windows文件在Linux乱码
在windows上使用ftp上传文件到linux上，中文名称在Linux系统中显示为乱码。虽然将Linux的env设置了LANG=en_US.UTF-8，并且本地的Shell客户端编码也设置成UTF-8，但在Shell中（或通过http访问），仍是乱码……

原因在于，Windows 的文件名中文编码默认为GBK，压缩或者上传后，文件名还会是GBK编码，而Linux中默认文件名编码为UTF8，由于编码不一致所以导致了文件名乱码的问题，解决这个问题需要对文件名进行转码。

 - 转换文件格式
```
yum install convmv
convmv -f gbk -t utf-8 -r --notest /File_Path
```
 - 常见参数
```
常用参数：
-r 递归处理子文件夹
–notest 真正进行操作，默认情况下是不对文件进行真实操作
–list 显示所有支持的编码
–unescap 可以做一下转义，比如把%20变成空格
-i 交互模式（询问每一个转换，防止误操作）

linux下有许多方便的小工具来转换编码：
文本内容转换 iconv
文件名转换 convmv
mp3标签转换 python-mutagen
```