---
layout: post
title: "Docker on Shadowsocks"
date: 2019-03-25 18:07:57 +0800
---

## [Get Docker](https://get.docker.com)

~~~ shell
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
~~~

### For Mainland China:

~~~ shell
sh get-docker.sh --mirror Aliyun
~~~

[Configure Docker Hub registry mirrors]([configure-dockerhub-registry-mirrors]: https://mirrors.ustc.edu.cn/help/dockerhub.html)

### For CentOS:

~~~ shell
systemctl start docker && systemctl enable docker
~~~

## Run Shadowsocks server

~~~ bash
export SS_PASSWORD="it's a secrect"
export SS_METHOD=rc4-md5 SS_PORT=8388 KCP_PORT=4000
docker run -d --restart=unless-stopped \
  -p ${SS_PORT}:8388 -p ${SS_PORT}:8388/udp -p ${KCP_PORT}:4000/udp \
  -e "SS_CONFIG=-s 0.0.0.0 -p 8388 -m ${SS_METHOD} -k ${SS_PASSWORD}" \
  -e 'KCP_CONFIG=-t 127.0.0.1:8388 -l :4000 -mode fast2' \
  -e 'KCP_FLAG=true' \
  --name ss-server mritd/shadowsocks
~~~

## Run Shadowsocks client

### Configure:

~~~ bash
export SERVER_IP=1.2.3.4
export SS_PASSWORD="it's a secrect"
export SS_METHOD=rc4-md5 SS_PORT=8388 KCP_PORT=4000
export SOCKS_PORT=1080
~~~

### KCP vs SS

#### KCP Client:

~~~ bash
docker run -d --restart=unless-stopped \
  -p ${SOCKS_PORT}:1080 \
  -e 'SS_MODULE=ss-local' -e "SS_CONFIG=-s 127.0.0.1 -p 8388 -b 0.0.0.0 -l 1080 -m ${SS_METHOD} -k ${SS_PASSWORD}" \
  -e 'KCP_MODULE=kcpclient' -e "KCP_CONFIG=-r ${SERVER_IP}:${KCP_PORT} -l :8388 -mode fast2" \
  -e 'KCP_FLAG=true' \
  --name ss-client mritd/shadowsocks
~~~

#### SS Client:

~~~ bash
docker run -d --restart=unless-stopped \
  -p ${SOCKS_PORT}:1080 \
  -e 'SS_MODULE=ss-local' -e "SS_CONFIG=-s ${SERVER_IP} -p ${SS_PORT} -b 0.0.0.0 -l 1080 -m ${SS_METHOD} -k ${SS_PASSWORD}" \
  --name ss-client mritd/shadowsocks
~~~
