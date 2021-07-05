---
title: "Jupyter and Nginx"
date: 2021-06-23T06:35:07Z
draft: false
---

Nginx用于搭建反向代理服务器，将客户端的请求转发到目标服务器。特别是后台有多个http服务器提供服务时，使用nginx可以统一对外入口。另一方面，对于我遇到的客户端无法直接访问目标服务器地址，也可以通过在中转的机器上搭建nginx代理解决。目标服务器上的jupyter notebook通过该方式暴露给了客户端所在网络。
<!--more-->

## Nginx
Nginx提供了[docker image](https://hub.docker.com/_/nginx)，通过docker使用更为方便。在准备搭建nginx的服务器上拉取镜像即可。若其无法连接internet，则可以先在可联网机器上拉取镜像，并导出拷贝至目标机器。
```bash
docker pull nginx
docker save -o nginx.tar nginx:latest
# copy to nginx server
docker load -i nginx.tar

# on nginx server
docker run -d --name nginx nginx  # 基础容器用于拷贝
mkdir nginx # 创建host目录
mkdir nginx/logs
docker cp [container id]:/etc/nginx ./nginx/conf
docker cp [container id]:/usr/share/nginx/html ./nginx/html

docker stop nginx
docker rm nginx # 删除基础容器

docker run -d --name nginx -p 8080:80 -p 443:443 \ # 容器80端口映射为host8080端口
    -v /[host nginx path]/conf:/etc/nginx/ \
    -v /[host nginx path]/html:/usr/share/nginx/html \
    -v /[host nginx path]/logs:/var/log/nginx nginx # 正式容器
```

### Nginx配置
只需要修改host/nginx/conf/nginx.conf即可。基础配置，通过[host域名或者ip]:8080 访问后台服务
```bash
http {
    # …… other configs
    
    # 可以有多个server
    server {
        listen 80;  # 监听80端口
        listen [::]:80;
        server_name localhost;
        
        # 对/所有做负载均衡+反向代理
        location / {
            proxy_pass http://target-server; # 代理到目标地址
        }
    }
    
    # 负载均衡后台服务器列表
    upstream target-server {
        server 10.192.106.133;
        server 10.192.106.134;
    }
}
```

#### nginx配置语法
- 设置变量: `set $variable value;`
- `if`控制流: `if (condition) {rewrite directives}`

#### nginx内置变量
- $host: 按照如下优先级，host name from the request line, or host name from the “Host” request header field, or the server name matching a request
- \$http_host: Value of the “Host:” header in the request (same as all \$http_<headername> variables
- $request_method: request method, usually “GET” or “POST”
- $request_uri: full original request URI (with arguments)
- $scheme: request scheme, e.g., “http” or “https”
- $server_name: name of the server which accepted a request
- $server_port: port of the server which accepted a request

## Jupyter over nginx
```bash
\# in nginx.conf
…… 
\# gzip on
include /etc/nginx/conf.d/*.conf

\# in conf.d/jupyter.conf
upstream jupyter-notebook {
    server jupyter_notebook_ip:port;
}

server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://jupyter-notebook;
        proxy_set_header Host $host;
    
        # websocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection “Upgrade”;
        proxy_read_timeout 2d;
    }
    
}
```

## v2ray
参考[V2ray in docker](https://asfuyao.github.io/2019/06/13/Docker%E5%BF%AB%E9%80%9F%E9%83%A8%E7%BD%B2v2ray%E7%BF%BB%E5%A2%99/).
```bash
\# in nginx.conf
…… 
\# gzip on
include /etc/nginx/conf.d/*.conf

\# in conf.d/v2ray.conf
server {
    listen 443 ssl;  # listening port
    ssl_certificate /path/to/v2ray.crt;  # path to v2ray cert
    ssl_certificate_key /path/to/v2ray.key;  # path to v2ray key
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    server_name mydomain.com;  # domain
    location /UUID {  # UUID: v2ray uuid
        proxy_redirect off;
        proxy_pass http://v2ray:8002;  # websocket location&port
                                       # v2ray: container name when raise v2ray in docker
                                       # port: v2ray listening port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection “upgrade”;
        proxy_set_header Host $http_host;
    }
}
```
