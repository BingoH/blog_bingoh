---
title: "迁移站点"
date: 2021-08-23T13:16:50Z
draft: false
---
迁移本站点需要做的。
<!--more-->
aws可以免费白嫖一年，到期后可以换一个账号继续白嫖，不过站点迁移还是需要一定的折腾。
## nginx配置
### 重新生成证书
- aws安全组：开放端口80，443，否则acme.sh会超时
- 启动nginx前，acme.sh以standalone模式生成证书
  - cloudflare必须关闭proxy模式，即仅dns，待配置完毕可重新开启
```bash
# instal acme.sh
curl https://get.acme.sh | sh -s email=my@example.com
source ~/.bashrc

# dependencies
sudo yum install socat
# 允许socat打开80端口，否则permission denied when issuing cert
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/socat

# standalone模式证书生成: https://guide.v2fly.org/advanced/tls.html#%E4%BD%BF%E7%94%A8-acme-sh-%E7%94%9F%E6%88%90%E8%AF%81%E4%B9%A6
acme.sh --issue -d mydomain.me --standalone --keylength ec256 --force
acme.sh --installcert -d mydomain.me --ecc --fullchain-file /path/to/v2ray.crt --key-file /path/to/v2ray.key
```
### nginx配置变更
- 假如忘了续域名，nginx配置中的域名都得进行修改。
- 证书配置做对应修改
## blog配置
- 修改`config.toml`中域名即可

## cloudflare配置
- 修改域名即可