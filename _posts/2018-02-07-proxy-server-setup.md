---
layout: post
title: '科学上网服务器端部署'
date: 2018-02-07
author: M3chD09
tags: gfw
cover: '/assets/img/bg/bypass-gfw.jpg'

---

>我们不生产代码，我们只做代码的搬运工

***部署环境：ubuntu 16.04 x64***

## brook

[项目地址](https://github.com/txthinking/brook)  
```bash
wget https://github.com/txthinking/brook/releases/download/v20180112/brook
chmod +x brook
nohup ./brook server -l :9999 -p password &
```
其中9999是端口号，password是密码，可以换成其它的  
注意下载链接要更新

## shadowsocks-libev + simple-obfs

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
vi /etc/shadowsocks-libev/config.json
```
写入以下内容并保存  
```javascript
{
    "server":["[::0]","0.0.0.0"],
    "server_port":443,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin": "obfs-server", 
    "plugin_opts": "obfs=http"
}
```
其中443是端口号，password是密码，最后：  
`nohup ss-server -c /etc/shadowsocks-libev/config.json &`  
#### 加入启动项  
`vi /etc/rc.local`  
在`exit 0`前加上`ss-server -c /etc/shadowsocks-libev/config.json`来将其加入启动项

## ShadowsocksR

原作者已经删除了项目，我从其它地方fork了一份  
```bash
apt-get update
apt-get install python-pip -y
git clone -b manyuser https://github.com/M3chD09/shadowsocksr.git
cd shadowsocksr
bash initcfg.sh
vi user-config.json
```  
根据需要修改，最后：
```bash
cd shadowsocks
python server.py -d start
```

