---
layout: post
title: '在CentOS下部署Nextcloud'
date: 2018-03-24
author: M3chD09
tags: web

---
>生命在于折腾

## 更新源
`yum update -y`  
`yum install epel-release -y`  

## 安装MariaDB
```bash
yum install mariadb mariadb-server -y
systemctl start mariadb
systemctl enable mariadb
# 配置数据库管理系统
mysql_secure_installation
# 新建数据库
mysql -u root -p
>create database nextcloud;
>exit
```

## 安装apache
```bash
yum install httpd -y
systemctl start httpd
systemctl enable httpd
```
## 安装php
```bash
rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum install yum-utils -y
yum update -y
yum-config-manager --enable remi-php72
yum install php php-cli php-posix php-gd php-fpm php-json php-mysql php-curl php-mbstring php-intl php-mcrypt php-imagick php-xml php-zip -y
systemctl reload httpd
```

## 下载Nextcloud
```bash
yum install wget unzip -y
wget https://download.nextcloud.com/server/releases/nextcloud-13.0.1.zip
unzip nextcloud-13.0.1.zip
mv nextcloud /var/www/
mkdir /var/www/nextcloud/data
chown apache:apache -R /var/www/nextcloud
```

## 配置apache
`vi /etc/httpd/conf.d/cloud.conf`  
写入以下内容
```html
<VirtualHost *:80>
    ServerName cloud.example.com
    ServerAlias cloudv6.example.com
    DocumentRoot /var/www/nextcloud/
    <Directory /var/www/nextcloud/>
        Options +FollowSymlinks
        AllowOverride All
        <IfModule mod_dav.c>
            Dav off
        </IfModule>
        SetEnv HOME /var/www/nextcloud
        SetEnv HTTP_HOME /var/www/nextcloud
    </Directory>
</VirtualHost>
```
其中`example.com`换成自己的域名  
然后重启apache2:`systemctl reload httpd`

## 通过[Let’s Encrypt](https://letsencrypt.org/)开启https  
```bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto 
```

## SELinux配置
```bash
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/data(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/config(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/apps(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.htaccess'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.user.ini'
restorecon -Rv '/var/www/nextcloud/'
setsebool -P httpd_can_network_connect_db 1
```

## 防火墙配置
```bash
# FirewallD
firewall-cmd --add-service http --permanent
firewall-cmd --add-service https --permanent
firewall-cmd --reload
# IPtables
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```
