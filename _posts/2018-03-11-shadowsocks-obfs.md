---
layout: post
title: '用obfs伪装shadowsocks'
date: 2018-03-11
author: M3chD09
tags: gfw

---
>让shadowsocks监听443端口，并与https共存  

***部署环境：ubuntu 16.04 x64***  

## 安装shadowsocks-libev和simple-obfs

[shadowsocks-libev项目地址](https://github.com/shadowsocks/shadowsocks-libev)  
[simple-obfs项目地址](https://github.com/shadowsocks/simple-obfs)  
```bash
apt-get install software-properties-common -y
add-apt-repository ppa:max-c-lv/shadowsocks-libev -y
apt-get update
apt install shadowsocks-libev -y
apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake rng-tools -y
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
make install
```

## 安装nginx

```bash
apt update
apt install nginx -y
```

## 配置shadowsocks

### 修改配置
``vi /etc/shadowsocks-libev/config.json``  
写入以下内容并保存  
```javascript
{
    "server":["[::0]","0.0.0.0"],
    "server_port":443,
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin": "obfs-server",
    "plugin_opts": "obfs=http;obfs-host=www.icloud.com;failover=[::0]:8443",
    "fast_open":true
}
```
其中password是密码
### 加入启动项  
`vi /etc/rc.local`  
在`exit 0`前加上`ss-server -c /etc/shadowsocks-libev/config.json`来将其加入启动项

## 配置nginx

### 通过[Let’s Encrypt](https://letsencrypt.org/)开启https  
```bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto --nginx
```
### 修改配置  
``vi  /etc/nginx/sites-enabled/default``  
找到  
``listen [::]:443 ssl ipv6only=on; # managed by Certbot``  
``listen 443 ssl ; # managed by Certbot``  
改成  
``listen [::]:8443 ssl ipv6only=on; # managed by Certbot``  
``listen 8443 ssl http2; # managed by Certbot``  
可以在``server_name``添加多个域名，用空格分开  
添加多个域名后，需要再次运行``./certbot-auto --nginx``来获取证书  
最后重启服务器即可
