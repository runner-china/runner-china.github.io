---
author: runner
date: 2016-11-05 11:51+08:00
layout: post
title: "用Iptables实现网关屏蔽恶意IP和URL"
description: ""
comments : true
categories:
- 技术
tags:
- Linux
- iptables
---


# 0x00 前言
注：本文谢绝转载，如有需要请电邮联系我  

经常存在这样一种应用场景，需要网关实现对一些恶意IP和URL屏蔽（如钓鱼网址），中断客户端访问。本文以此作为出发点，以Linux网关为例，利用Linux系统的netfilter/iptables实现屏蔽恶意的IP和URL。

# 0x01 实验环境的网络拓扑
实验环境只需要一台笔记本电脑；由这台由物理计算机利用VMware Workstation虚拟机软件虚拟出2台虚拟机：VM1、VM2。其中，VM2是用户上网系统，VM1为VM2的Linux网关，实现数据包正常转发、恶意URL阻断、告警页面跳转等功能。宿主机（Host Machine）的无线网卡可以访问互联网，它与VM1之间通过VMware NAT模式进行网络通讯，即保证了VM1也能访问互联网。  
![](16110501.jpg)

<!--more-->
# 0x02 网络的基础配置

1. VMware网络配置。按照图里面的说明，分别配置子网VMnet 8（192.168.111.0）和VMnet9（192.168.112.0）；
1. 禁用SELinux功能
1. 禁用firewalld服务
1. 虚拟机网卡配置。按照图里面的说明，分别配置VM1的两张网卡和VM2的一张网卡的网络设置。
1. VM1开启IP forward
![](16110502.jpg)
![](16110503.jpg)

# 0x03 Netfilter/iptables配置
## 启用Netfilter/iptables
CentOS 7.0默认使用的是firewall作为防火墙，需改为iptables防火墙

    systemctl stop firewalld.service //停止firewall
    systemctl disable firewalld.service //禁止firewall开机启动
    firewall-cmd --state //查看默认防火墙状态
    yum install iptables-services //安装iptables
    systemctl start iptables.service //启动iptables使配置生效
    systemctl enable iptables.service //设置iptables开机启动

## 启用SNAT功能（nat table, POSTROUTING  chain）
使VMnet 9 能够访问公网

    iptables -t nat -A POSTROUTING -s 192.168.112.0/24  -o  eno16777736 –j MASQUERADE
 
## 主机防火墙功能（filter table, INPUT chain）
    iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -p icmp -j ACCEPT
    iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
    iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
    iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited


## 恶意URL阻断（filter table, FORWARD chain）
	iptables -A FORWARD -p tcp -m string --string "www.baidu.com"  --algo bm    -j REJECT --reject-with tcp-reset
	iptables -A FORWARD -p tcp -m string --string "t.cn/h5mwx"  --algo bm    -j REJECT --reject-with tcp-reset


## 恶意IP跳转到告警页面（nat table, PREROUTING chain）
	iptables -t nat -A PREROUTING  -p  tcp  -d  6.6.6.6    -j DNAT  --to-destination  192.168.112.2


# 0x04 Apache配置
在VM1的系统上安装Apache服务器，配置Web目录存放告警页面。

# 0x05 测试案例
这里假设www.baidu.com为恶意网站, 对恶意用例与正常用例分别进行测试。
## 恶意用例
- http://www.baidu.com（百度的http）
- https://www.baidu.com（百度的https） 
- http://t.cn/h5mwx （百度的短网址）
- 6.6.6.6  （假定恶意IP）


## 正常用例
- www.taobao.com
- https://www.jd.com/
- http://t.cn/hczXq  （腾讯的短网址）





