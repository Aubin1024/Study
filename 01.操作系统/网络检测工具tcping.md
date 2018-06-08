# 简介
ping命令走ICMP协议，有些IDC会专门针对ICMP进行优化。  
tcping是走tcp协议，如果只是针对ICMP进行优化，那么使用tcping就能看出真正的延迟

# tcping功能
 - TCP协议的延迟检测
 - 端口检测
 - 禁ping后延迟检测

# 下载
[官网](https://elifulkerson.com/projects/tcping.php)
[32位](http://www.elifulkerson.com/projects/downloads/tcping-0.26/tcping.exe)
[64位](http://www.elifulkerson.com/projects/downloads/tcping-0.26/tcping64.exe)

# 安装
下载后放到`C:\Windows\System32\`

# 使用

```
tcping [参数]   域名/IP  [端口]

-t      连续ping
-d      显示时间
-4      优先pingIPV4
-6      优先pingIPV6
-w 0.5  设定超时时间
-s      tcping通之后立即停止ping(在超时时间之内ping通)
```

# 实例

 - 基本
```
tcp www.baidu.com
tcp -d www.baidu.com 80
```

 - 禁ping后检测延迟  
```
# 禁ping后检测延迟的前提是要是指哪个端口开放，可以用namp扫描端口
tcp -d -t 36.103.245.171 22
```
