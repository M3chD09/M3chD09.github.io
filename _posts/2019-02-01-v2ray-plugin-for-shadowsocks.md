---
layout: post
title: '安装shadowsocks的v2ray插件'
date: 2019-02-01
author: M3chD09
tags: gfw
cover:

---
很长时间没有更新博客了，今天偶然发现[simple-obfs](https://github.com/shadowsocks/simple-obfs)已经废弃了，取而代之的是[v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin)，于是我决定尝鲜一下。  

## 服务端部署
***部署环境：CentOS 7.6 x64***
### 安装Shadowsocks-libev
这个之前博文说过，就不重复了。  
### 安装v2ray-plugin
我懒得编译，直接下载[已经编译好的](https://github.com/shadowsocks/v2ray-plugin/releases)了，解压到`/usr/local/bin/`下。  
### 获取SSL证书
我使用的是*Let's Encrypt*的免费证书，执行命令之前记得先将域名的A记录解析到服务器IP上，并且开放80和443端口。
```bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto certonly
```
成功之后会有`/etc/letsencrypt/live/域名/fullchain.pem`和`/etc/letsencrypt/live/域名/privkey.pem`两个文件。
### 修改配置文件
默认配置文件在`/etc/shadowsocks-libev/config.json`，如下文所示，在最后加上`plugin`和`plugin_opts`，把域名换成自己的就行。这里采用的是Shadowsocks over websocket (HTTPS)，其它的可以自己参考GitHub上的介绍。  
```javascript
{
    "server":"0.0.0.0",
    "server_port":443,
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server;tls;cert=/etc/letsencrypt/live/域名/fullchain.pem;key=/etc/letsencrypt/live/域名/privkey.pem;host=域名;loglevel=none"
}
```
最后执行`ss-server -c /etc/shadowsocks-libev/config.json -f /run/shadowsocks.pid`即可。
## 客户端设置
先安装对应平台的Shadowsocks客户端。  
### Windows
直接下载已经编译好的v2ray-plugin的windows版，解压后命名为`v2ray-plugin.exe`和`Shadowsocks.exe`放在同一文件夹下。  
在Shadowsocks的服务器设置中，插件程序填`v2ray-plugin`，插件选项填`tls;host=域名`。域名要和服务端设置的一致。  
### Android
安装[v2ray-plugin-android](https://github.com/shadowsocks/v2ray-plugin-android/releases)，打开Shadowsocks编辑服务器，在最下面的插件中选择v2ray，配置Transport mode为`websocket-tls`，Hostname为服务端设置的域名即可。  
### Ubuntu
直接下载已经编译好的v2ray-plugin，解压到`/usr/local/bin/`下。  
编辑配置文件`/etc/shadowsocks-libev/config.json`，如下文所示，在最后加上`plugin`和`plugin_opts`，域名要和服务端设置的一致。  
```javascript
{
    "server":"域名",
    "server_port":443,
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin":"v2ray-plugin",
    "plugin_opts":"tls;host=域名;loglevel=none"
}
```
最后执行`ss-local -c /etc/shadowsocks-libev/config.json`即可。
## 优势
你甚至可以把SS服务器放在CDN后面，免费的CDN有[Cloudflare](https://www.cloudflare.com)。  
~~GFW就拿你没办法了哈哈。~~
