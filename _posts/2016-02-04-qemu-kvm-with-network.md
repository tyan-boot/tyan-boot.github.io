---
layout: post
title:  "使用QEMU虚拟机以及KVM加速 附带网络支持"
date:   2016-02-04 19:32:00 +0000
categories: tec
---

>本文主要概述如何使用QEMU虚拟机以及KVM相关模块.并且启用网络支持

注意:本文仅限于Linux用户.不给予Windows用户支持
---

首先安装QEMU.具体如何安装自行百度
这里列举几个常见的发行版  
Ubuntu:apt-get install qemu  
CentOS:yum install qemu  
Arch: pacman -S qemu  

然后是KVM.这个需要内核支持.当你确定支持之后启用KVM

网络支持
---

本文重点其实是说怎么设置网络  
QEMU内的客户机使用网络大概有两种.  
第一种是usermode.即直接使用主机的网络.这种模式简单.但是客户机无法和主机所在的局域网的机子互通  
第二种是tap模式.虚拟一个网卡出来,然后进行桥接.这种方法可以让客户机访问到内网的其他用户.  

首先QEMU配置网络需要配置两部分.  
第一部分官方叫做network backend.这里设置的就是上面提到的上网模式  
第二部分是网卡设备.这个网卡设备指的是你客户机内看到的网卡设备

usermode
-----

这个用起来很简单.启动时候加上两个参数  
-netdev user,id=net0
-device pcnet,netdev=net0  
第一个参数指定使用user模式,id参数则指定这个netdev的名字.接下来device参数创建了一个网卡设备并且绑定到net0这个netdev上  
接下来启动你的QEMU之后客户机应该就可以上网了

Tap模式
-----

这里需要你有root权限  
首先需要安装网桥相关的包,一般叫bridge-utils.这个自己安装- -  
接下来:  

brctl addbr br0  
brctl addif eth0    #这里指定你自己主机的网卡设备  
ip tuntap add mode tap tap0    #创建一个tap设备供虚拟机使用  
brctl addif tap0    #把这个tap0加入到网桥中  
ip link set br0 up    #启用网桥  

dhclient br0    #从网桥获取ip地址  

\#下面启动你的QEMU  

qemu-system-i386 -enable-kvm -netdev tap,id=net0,ifname=tap0,script=no,downscript=no  

如果正常的话.就可以食用虚拟机了  