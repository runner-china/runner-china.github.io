---
author: runner
date: 2018-02-08 20:50+08:00
layout: post
title: "云服务器上安装部署OpenVPN"
description: ""
comments : true
categories:
- 技术
tags:
- vpn
---


![](/blog/images/180212_logo.png)
# 0x00 前言

VPN（Virtual Private Network）中文称为虚拟专用网，它透过公网来传送内网的信息，这种技术可以利用不安全的网络（例如：互联网）来发送可靠、安全的消息。它的基本原理是利用已加密的隧道协议（Tunneling Protocol）来达到保密、发送端认证、消息准确性等安全效果。本文利用开源软件[OpenVPN](https://en.wikipedia.org/wiki/Virtual_private_network)搭建VPN服务器，客户端拨号连接到VPN服务器这种的方法就能实现信息远程的安全通信，保障网络传输的保密性、完整性、身份认证、访问控制等。另外，VPN服务器的自身安全性也是考虑因素之一。

# 0x01 VPN与OpenVPN
常见的VPN的分类如下  

- 按协议层次：可以分为二层VPN，三层VPN、四层VPN和应用层VPN。
- 按应用范围：远程访问VPN、内联网VPN和外联网VPN。
- 按体系结构：网关到网关VPN、主机到网关VPN和主机到主机VPN。

<!--more-->

OpenVPN是一款著名VPN服务器软件，除此以外还有SoftEther，strongswan，Openswan，Libreswan等开源软件可以实现VPN功能，[参见这里](https://www.geckoandfly.com/5710/free-vpn-for-windows-mac-os-x-linux-iphone-ubuntu/)。  

OpenVPN具有如下特点：  

- 开源、免费
- 综合利用虚拟网卡技术、SSL协议等多种技术
- 具有跨平台特性
- 支持主机到网关的远程访问功能


# 0x02 VPN组建方案与身份验证
1. 云上用一台ECS安装OpenVPN服务端、搭建PKI。  
2. 本地计算机上安装OpenVPN客户端。  
3. 本地计算机通过VPN拨号连接到VPN服务端后，再通过VPN网络访问云上的其他服务器。
4. 通过在客户端增加路由，服务端启用NAT的模式：VPN拨号成功以后，客户端访问互联网，不走VPN网络。客户端访问云（192.168.0.0/16）,走VPN网络。  
5. 安全的身份验证：不同用户用各自的客户端证书，进行基于PKI的双向身份验证（维护阶段可以撤销客户端证书等进行控制）；不同用户用各自的用户名/密码方式进行身份验证，即PAM方式（维护阶段可以停用账号，修改密码等进行控制）;  

# 0x03 VPN服务端的安装部署

安装EPEL源:  yum install epel-release  
yum install  openvpn  -y  
yum install easy-rsa  （此软件包用于建立PKI）
其他需要安装的依赖包：openssl、lzo、pam等

## 生成CA的证书与私钥
cd /etc/openvpn/easy-rsa/2.0  
编辑vars文件，根据需要设置参数 KEY_COUNTRY, KEY_PROVINCE, KEY_CITY, KEY_ORG, and KEY_EMAIL，也可以默认不改变。  

	source ./vars  
	./clean-all  
	./build-ca  
keys目录下面生成 ca.crt、ca.key 两个文件，分别是ca的证书与私钥；  

## 生成服务器的证书与私钥
	./build-key-server server  
在交互式命令中，最后两步选择y（yes），表明用CA私钥对服务器的证书进行签名。  
keys目录下面生成server.crt、server.key 两个文件，分别是server的证书与私钥。  

## 生成客户端的证书与私钥
	./build-key client  （备注：每个客户端应该有一个独立的名字，这里用了client）  
在交互式命令中，最后两步选择y（yes），表明用CA私钥对客户端的证书进行签名。  
keys目录下面生成client.crt、client.key 两个文件，分别是client的证书与私钥。  

## 生成Diffie Hellman参数
	./build-dh  
keys目录下面生成dh1024.pem (vars文件里面有KEY_SIZE项， 如果KEY_SIZE = 2048，这里产生的文件是dh2048.pem)  

## 生成ta文件
生成ta.key文件，用于避免DoS 攻击 and UDP端口的flooding攻击。  

	openvpn --genkey --secret keys/ta.key    


## 文件清单
![](/blog/images/180212_files.png)  

注意，这里根CA服务器与VPN服务端简化为了同一台服务器。

## 复制服务端的文件

	cd /etc/openvpn/easy-rsa/2.0/keys  
	cp ca.crt dh2048.pem server.crt server.key ta.key  /etc/openvpn

## 修改系统参数，允许IP转发
	vim /etc/sysctl.conf  
	net.ipv4.ip_forward = 1  
	sysctl -p /etc/sysctl.conf  

## 开放ECS的安全组
源地址：any  
协议：udp  
端口号：11941  

## 利用iptables做SNAT与转发的访问控制
	systemctl status firewalld.service  #centos7默认安装的firewalld
	systemctl stop firewalld   #关闭Centos7默认的 firewall防火墙  
	systemctl disable firewalld  
	yum install -y iptables-services  #安装iptables服务
	systemctl enable iptables  
	systemctl start iptables   #启动iptables  
	iptables -t filter -F     #清空默认的iptables规则  
	iptables -t nat  -F  
	iptables -t filter -P FORWARD DROP  #默认为DROP  
	iptables -t nat -A POSTROUTING -s 10.8.0.0/24  -o  eth0 -j MASQUERADE   #设置iptables NAT转发规则  
	iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT  
	iptables -A FORWARD -s 10.8.0.0/24 -d 192.168.0.0/24  -j ACCEPT  #设置iptables 转发的访问控制  
	service iptables save   #保存防火墙规则，写入配置  
	iptables -t nat -L  #查看nat表  
	iptables -t filter -L  #查看filter表  

## 配置文件的改写
在Linux系统上，配置文件命名为server.conf 和 client.conf；在Windows系统上，配置文件命名为server.ovpn 和 client.ovpn。  

	cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf   /etc/openvpn  

修改文件的配置项

	port 11941 #修改端口号，默认是1194
	proto udp
	dev tun
	ca ca.crt
	cert server.crt
	key server.key
	dh dh2048.pem
	server 10.8.0.0 255.255.255.0 #虚拟网卡所用的网段
	ifconfig-pool-persist ipp.txt
	#push "redirect-gateway def1 bypass-dhcp" #这段要注释，避免所有流量往vpn走
	push "route 192.168.0.0 255.255.240.0" #客户端路由
	push "dhcp-option DNS 208.67.222.222"
	push "dhcp-option DNS 8.8.8.8"
	keepalive 10 120
	tls-auth ta.key 0 # This file is secret
	cipher AES-256-CBC
	comp-lzo #压缩
	user nobody
	group nobody
	persist-key
	persist-tun
	status openvpn-status.log #日志
	log-append  openvpn.log #日志
	verb 3
	explicit-exit-notify 1
	#Use PAM authentication，客户端用用户名/密码进行验证
	plugin  /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so  login
	reneg-sec 5400 #该参数是指n秒钟之后重新验证key

## 启动VPN服务
	systemctl start openvpn@server  
	systemctl enable openvpn@server  
	systemctl status  openvpn@server  

## 账号创建
注意：user、group根据实际情况填写  

	groupadd group  
	useradd  user -g user -M  -s /sbin/nologin  
	passwd user  

# 0x04 VPN客户端的安装部署
安装： openvpn-install-2.4.4-I601.exe  
取回之前生成的位于/etc/openvpn/easy-rsa/2.0/keys中的4个文件：  
ca.crt、client.crt、client.key、ta.key  
将这几个文件放入到 C:\Program Files\OpenVPN\config 或者 C:\Users\Bruce\OpenVPN\config中。  
将  C:\Program Files\OpenVPN\sample-config\client.ovpn  复制到C:\Program Files\OpenVPN\config下  
修改该文件的配置项  

	client
	dev tun
	proto udp
	remote xxx.xxx.xxx.xxx 11941 #配服务端的IP与端口
	ca ca.crt
	cert client1.crt
	key client1.key
	comp-lzo
	tls-auth ta.key 1  
	auth-nocache
	auth-user-pass #客户端用用户名/密码验证

## 0x05 VPN服务端的安全性考虑
1. 修改OpenVPN的默认端口。
2. 服务器自身的安全加固。
3. 安全组、iptables等进行访问控制。
4. VPN日志的审计。
5. 多重身份验证，包括基于PKI的验证与基于PAM的验证。
6. VPN网关采用双机热备架构，故障发生时能够自动切换。
 

**注：如需转载这篇文章请注明出处**  




