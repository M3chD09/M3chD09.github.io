---
layout: post
title: 'SSH隧道动态端口转发'
date: 2018-02-11
author: M3chD09
tags: gfw ssh

---

>最原始的就是最厉害的

## 配置公钥登录

本地执行`ssh-keygen -t rsa -f test -C "test key"`  
-f 文件名，-C 备注。可以不要密码。  
`cat ~/.ssh/id_rsa.pub` 查看公钥并复制到远程机器的`~/.ssh/authorized_keys`内。  
最后远程机器执行`chmod -R 700 ~/.ssh`

## 动态端口转发

建议先安装autossh，能让 SSH 隧道一直保持执行，它会启动一个 SSH 进程，并监控该进程的健康状况；当 SSH 进程崩溃或停止通信时，AutoSSH 将重启动 SSH 进程。  
本地执行`autossh -D 1080 root@ip`  
其中ip换成远程机器地址，可以加上参数N，表示只连接远程主机，不打开远程shell。  
穿墙完成！  
检查`ps axu | grep ssh`  
关闭`killall ssh`
