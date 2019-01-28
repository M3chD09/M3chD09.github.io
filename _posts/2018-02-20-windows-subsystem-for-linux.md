---
layout: post
title: '适用于Linux的Windows子系统'
date: 2018-02-20
author: M3chD09
tags: wsl

---
>在Windows下运行多个Linux子系统

## 安装
Windows的子系统目前有两种方法安装：  
1.应用商店  
2.lxrun  
无论哪种方法都要先开启Windows子系统功能，以管理员身份打开PowerShell并运行：  
`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`  
根据提示重启系统。

### 应用商店  
直接安装即可。  
Ubuntu文件位置：  
`%localappdata%\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs`

### lxrun  
打开命令提示符并运行：  
`lxrun /install`  
根据提示安装即可。  
文件位置：  
`%localappdata%\lxss\`

## 管理多子系统  

### wslconfig  
打开命令提示符并运行*（示例）*：  
`wslconfig /list` 列出已安装的子系统  
`wslconfig /setdefault <DistributionName>` 设置默认子系统  
`lxrun /setdefaultuser root` 设置默认子系统的默认用户为root  
`bash` 进入默认子系统

### WSL-Distribution-Switcher  
***该管理工具只适用于lxrun***  
首先安装python，并将其加入PATH。  
打开命令提示符并运行：`python`即可检验是否安装成功。  
下载[WSL-Distribution-Switcher](https://github.com/RoliSoft/WSL-Distribution-Switcher)，在其所在位置用命令提示符打开并运行*（示例）*：  
`python get-source.py centos` 下载Centos  
`python get-prebuilt.py kalilinux/kali-linux-docker` 下载Kali  
`python install.py centos` 安装Centos  
`python install.py rootfs_kalilinux_kali-linux-docker_latest.tar.gz` 安装Kali  
`python switch.py` 列出已有的子系统  
`python switch.py centos` 切换到Centos  
`python switch.py kalilinux:kali-linux-docker_latest` 切换到Kali

## 效果图
![Ubuntu](/assets/img/wsl/ubuntu.png)
![Centos](/assets/img/wsl/centos.png)
![Kali](/assets/img/wsl/kali.png)
