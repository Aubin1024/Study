# Docker daemon安全
Docker向外界提供服务有四种通信形式，默认为Unix域套接字
- 域套接字
- TCP/IP
- 未知
- 未知
#### 1.TCP/IP
Docker提供了TLS传输层安全协议。在Docker中可以设定以下参数
[官方文档](http://note.youdao.com/)
```
--tlsverify         安全传输校验
--tlscacert         信任的证书(CA证书)
--tlskey            服务器或客户端密钥(私钥)
--tlscert           证书位置
```

证书申请流程： 

 - 生成私钥(对应tlskey)
 - 根据私钥生成申请证书文件csr，等待CA签署
 - CA签署后生成文件crt(对应tlscert)
 - CA自己的证书(对应tlscacert)
 - 
# 镜像安全
registry v2时有镜像签名功能，防止官方镜像被篡改或损坏

# 内核安全
#### 1.cgroups资源限制
#### 2.namespace资源隔离

# 网络安全
Docker daemon指定 --icc时，可以禁止容器与容器之间的通信。