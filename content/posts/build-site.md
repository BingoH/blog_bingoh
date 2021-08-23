---
title: "Build Site"
date: 2021-08-23T13:16:50Z
draft: true
---

move site problems:


configs: all domain **.tk ==> **.ml (new domain);
docker-compose.yml; nginx configures;
re-create acme.sh cert:
```bash
curl https://get.acme.sh | sh -s email=my@example.com
source ~/.bashrc

# https://guide.v2fly.org/advanced/tls.html#%E4%BD%BF%E7%94%A8-acme-sh-%E7%94%9F%E6%88%90%E8%AF%81%E4%B9%A6
# issue standalone
acme.sh --issue -d mydomain.me --standalone --keylength ec256 --force
acme.sh --installcert -d mydomain.me --ecc --fullchain-file /path/to/v2ray.crt --key-file /path/to/v2ray.key
```

# edit aws security rules on aws console; otherwise acme.sh issue cert failed (timeout); 
# open port 443 and 80; disable proxy of cloudflare currently
sudo yum install socat; # install socat on aws
# permission to socat to open 80 port; otherwise permission denied warning when issue cert
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/socat


re-build blog:
hugo config.toml: new domain; and generate static files.
