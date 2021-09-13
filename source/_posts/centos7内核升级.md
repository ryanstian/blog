---
title: centos7内核升级
date: 2021-09-11 19:16:00
categories: centos7
tags: []
copyright: true #新增,开启
---

<!--more-->
导入elrepo源公钥  
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

导入elrepo源  
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

列出可用内核（lt长期维护，ml稳定）  
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

下载最新稳定版内核  
yum -y --enablerepo=elrepo-kernel --enablerepo=elrepo-kernel install kernel-ml

重启（建议开机手动选内核，遇到异常可重选旧内核）  
reboot

查看内核版本号  
uname -r

删除旧内核  
yum remove 内核包名