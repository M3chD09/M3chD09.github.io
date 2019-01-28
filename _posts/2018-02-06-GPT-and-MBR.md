---
layout: post
title: 'GPT与MBR的相互转换命令'
date: 2018-02-06
author: M3chD09
tags: cmd

---

>命令行是最简单高效的工具

## 安装Windows时，在“您想将Windows安装在何处”的界面按下“Shift+F10”，调出命令行窗口

`diskpart` （启动Diskpart程序）  
`list disk` （查看电脑中有哪些磁盘）   
`select disk 0`（选中编号为0的磁盘）  
`clean`（清除磁盘所有分区）  

### MBR转GPT

`convert gpt`（将磁盘转换成GPT格式）  
`list partition`（查看当前磁盘分区情况）  
`create partition efi size=100`（建立EFI分区，默认大小为M）  
`create partition msr size=128`（建立微软默认128M的MSR分区）  
`create partition primary size=102400` （此处为你想设置C盘的大小）  

### GPT转MBR

`create partition primary size=102400` （创建主分区，即你要当做系统盘的分区）  
`active` （激活主分区，以便启动电脑时BIOS会检测到主分区的操作系统）  
`format quick` （快速格式化主分区）  
`create partition extended` （创建扩扩展分区，后面会拿来当做逻辑分区进行划分，将剩余磁盘全部作为扩展分区）  
`create partition logical size=102400` （创建100G逻辑分区）  
`create partition logic` （最后一次分区，将剩余扩展分区全部作为逻辑分区）  

## 两次输入exit，重新启动电脑进行系统安装
