---
layout: post
title: 'CentOS下手动编译安装shadowsocks-libev'
date: 2018-03-18
author: M3chD09
tags: gfw

---

>告别一键脚本，从此手动安装

***部署环境：CentOS 7.4 x64***

[shadowsocks-libev项目地址](https://github.com/shadowsocks/shadowsocks-libev)  
[simple-obfs项目地址](https://github.com/shadowsocks/simple-obfs)  

## 编译安装
```bash
yum update -y
yum install -y epel-release rng-tools
yum install -y git wget gettext gcc autoconf libtool automake make asciidoc xmlto c-ares-devel libev-devel zlib-devel openssl-devel
rngd -r /dev/urandom

# 安装Libsodium
export LIBSODIUM_VER=1.0.16
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
make install
popd
ldconfig

# 安装MbedTLS
export MBEDTLS_VER=2.7.0
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
make DESTDIR=/usr install
popd
ldconfig

# 安装shadowsocks
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
./autogen.sh && ./configure && make
make install
cd ..

# 安装simple-obfs
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
make install
```
## 修改配置
`mkdir /etc/shadowsocks-libev`  
`vi /etc/shadowsocks-libev/config.json`  
写入以下内容并保存  
```javascript
{
    "server":["[::0]","0.0.0.0"],
    "server_port":443,
    "local_port":1080,
    "password":"password",
    "timeout":60,
    "method":"aes-256-gcm",
    "plugin": "obfs-server", 
    "plugin_opts": "obfs=http"
}
```
其中443是端口号，password是密码  
## 加入启动项
`vi /etc/rc.d/rc.local`  
在最后加上`ss-server -c /etc/shadowsocks-libev/config.json`来将其加入启动项，并加可执行权限  
`chmod +x /etc/rc.d/rc.local`  
