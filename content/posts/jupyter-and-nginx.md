---
title: "Jupyter and Nginx"
date: 2021-06-23T06:35:07Z
draft: true
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

docker run -d --name nginx -p 80:80 -p 443:443 -v /[host nginx path]/conf:/etc/nginx/ \
    -v /[host nginx path]/html:/usr/share/nginx/html \
    -v /[host nginx path]/logs:/var/log/nginx nginx # 正式容器
```
### Nginx配置
只需要修改host/nginx/conf/nginx.conf即可。基础配置
## Jupyter over nginx