---
layout: post
title: '配合nginx使用v2ray插件'
date: 2019-02-20
author: M3chD09
tags: gfw
cover:

---
之前一直让Shadowsocks监听443端口，这样就和HTTPS冲突了。而现在，两者可以兼得。

## 服务端部署
***部署环境：CentOS 7.6 x64***
### nginx配置
在默认配置文件`/etc/nginx/nginx.conf`中新建一个`server`，其中：
* `server_name`写访问v2ray-plugin所用的的域名，需要A记录解析到服务器IP上。
* `ssl_certificate`写SSL证书路径。
* `ssl_certificate_key`写SSL私钥路径。
* `proxy_pass`中的端口号要和下文中`server_port`一致。

示例如下：
```
server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  example.com;
        root         /usr/share/nginx/html/;
        ssl_certificate "/path/to/cert";
        ssl_certificate_key "/path/to/key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        location / {
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_pass http://localhost:8008;
            proxy_set_header Host $http_host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}
```
### Shadowsocks配置
修改Shadowsocks-libev默认配置文件`/etc/shadowsocks-libev/config.json`，其中：
* `server_port`和上文的`proxy_pass`端口号一致。

示例如下：
```javascript
{
    "server":"0.0.0.0",
    "server_port":8008,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server;loglevel=none"
}
```
## 客户端设置
插件程序填`v2ray-plugin`，插件选项填`tls;host=example.com`即可。
