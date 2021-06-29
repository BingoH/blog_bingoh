---
title: "Jupyter and Nginx"
date: 2021-06-23T06:35:07Z
draft: true
---

Nginx用于搭建反向代理服务器，将客户端的请求转发到目标服务器。特别是后台有多个http服务器提供服务时，使用nginx可以统一对外入口。另一方面，对于我遇到的客户端无法直接访问目标服务器地址，也可以通过在中转的机器上搭建nginx代理解决。目标服务器上的jupyter notebook通过该方式暴露给了客户端所在网络。
<!--more-->

