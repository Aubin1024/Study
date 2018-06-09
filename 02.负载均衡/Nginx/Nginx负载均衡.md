# Nginx调度算法
 - 轮询
 - 加权轮询
 - ip_hash
 - fair
 - usl_hash

# 设定后端服务器的状态
```
down            # 此server不参与负载均衡
backup          # 备份server，当其他所有非backup主机down时，才会转发到backup server
max_fails       # 允许请求失败次数，默认为1.当超过设定的次数时，返回proxy_next_upstream模块定义错误
fail_timeout    # max_fails次失败后，后端暂停服务的时间

注：当调度算法为ip_hash时，后端服务器状态不能为backup和weight
```

# 负载均衡实例
```
http{
    upstream  myserver {
    server 10.10.10.1  weight=3  max_fails=3  fail_timeout=20s;
    server 10.10.10.2  weight=3  max_fails=3  fail_timeout=20s;
    #定义后端服务器，权重3，失败3次后暂停服务20s
    }
    server	{
 	    listen 80;
 	    server_name www.domain.com;
 	    index 	index.html;
 	    root  /date/web
    	location  /  {                      # 配置健康检查
    		proxy_pass http://myserver;
    		proxy_next_upstream http_500 http_502 error timeout
    		invalid_header;
    		# 当出现以上状态码时，转发给下一台服务器
        }
    }
}
```

