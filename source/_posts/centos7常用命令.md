---
title: centos7常用命令
date: 2021-09-08 23:07:00
categories: centos7
tags: []
copyright: true #新增,开启
---

#### 问题排查命令
strace：跟踪到进程产生的系统调用（参数，返回值，执行耗时）

lscpu：显示cpu信息，本质是读取/proc/cpuinfo文件

nslookup：显示dns记录，查看域名解析是否正常

netstat：查看链接情况

lsof：查看链接情况（会显示PID）

程序异常被系统kill，可能因为内存占用过大，或许可以在/var/log/messages找到痕迹

#### 压缩/解压命令
压缩：tar -zcvf 名字.tar.gz ./名字  

压缩：tar -cvf 名字.tar ./名字

解压：tar -zxvf 名字.tar.gz

解压：tar -xvf 名字.tar

如果需指定解压目录，在后面追加-C 目录名
#### xshell下载上传文件命令
远端下载本地：sz
本低上传远端：rz

#### 操作服务命令
列出所有服务：systemctl list-unit-files

开机自启动：systemctl enable 服务名

取消开机自启动：systemctl disable 服务名

启动：systemctl stop 服务名

停止：systemctl stop 服务名

开机自启动：  
```
vi /etc/rc.local
将服务启动命令追加到文件末尾

ps：
1、注意看rc.local文件注释，有的需要特别操作开启。docker设计上不建议用这种方式
2、注意看rc.local是否为软连接，需给予该文件及真实文件可执行权限
```

#### 操作系统命令

查看系统版本：cat /etc/redhat-release

查看当前内核版本：uname -a

查看本地全部内核包/选项：rpm -qa|grep kernel

删除内核：yum remove 内核包名

查看开机默认内核：grub2-editenv list

修改开机默认内核：grub2-set-default 'CentOS Linux (kernel-3.10.0-1127.el7.x86_64
) 7 (Core)'  
ps：括号里填自己想要的内核版本

查看当前时间：date

修改当前时间：date -s "2021-09-11 00:00:00"

查看硬件时间：hwclock

设置硬件时间：hwclock -set -date="09/11/2021 00:00:00"

系统时间写入到硬件时间：hwclock -w

硬件时间写入到系统时间：hwclock -s

查看系统最后一次启动时间：who -b

查看当前系统时间：who -r

查看历史系统启动时间：last reboot

系统运行时间：top