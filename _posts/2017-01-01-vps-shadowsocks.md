---
author: runner
date: 2017-01-01 21:50+08:00
layout: post
title: "一种基于开源软件的翻墙方案"
description: ""
comments : true
categories:
- 技术
tags:
- 网络工具
---
![](/blog/images/17010101.jpg)
# 0x00 前言

有时候我们必须借助一些工具进行有效翻墙，获取墙外的信息。[翻墙方法](https://zh.wikipedia.org/wiki/%E7%AA%81%E7%A0%B4%E7%BD%91%E7%BB%9C%E5%AE%A1%E6%9F%A5)有很多，其中有一种较为安全、稳定、可靠的方法就是利用一台海外VPS，通过部署shadowsocks开源软件进行socks代理方式翻墙。

# 0x01 购买海外VPS
VPS可供选择较多，比如[搬瓦工](https://bwh1.net)，20美元一年，支持支付宝支付，具体购买步骤略去。  
最终目的是得到一个可以SSH登录的远程服务器。

<!--more-->

# 0x02 VPS安装Shadowsocks服务端
Shadowsocks是一种基于Socks5代理方式的开源软件。所有的流量都经过算法加密，允许自行选择算法。客户端覆盖多个主流操作系统和平台，包括Windows、OS X、Android、Linux和iOS系统和路由器（OpenWrt）等。与VPN的全局代理不同，Shadowsocks仅针对应用程序进行代理。


## 环境安装与更新
    yum install epel-release
    yum update
    yum install python-setuptools m2crypto supervisor
    easy_install pip
    pip install shadowsocks  

## shadowsocks.json文件配置
vi /etc/shadowsocks.json
将下面的内容粘贴后复制到shadowsocks.json文件里：

    {
    "server":"0.0.0.0",  
    "server_port":远程端口,  
    "local_port":本地端口,  
    "password":"密码",  
    "timeout":600,  
    "method":"aes-256-cfb"  
    }


## 服务器以后台进程方式启动

    ssserver -c /etc/shadowsocks.json -d start 

# 0x03 VPS安装Shadowsocks客户端
这里以Linux客户端为例，安装方式同服务端。  

## shadowsocks.json文件配置
    vi /etc/shadowsocks.json
    {
    "server":"VPS服务器IP",
    "server_port":远程端口,
    "local_address": "127.0.0.1",
    "local_port":本地端口,
    "password":"密码",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
    }

## 客户端以后台进程方式启动
    sslocal -c /etc/shadowsocks.json -d start

## 浏览器上设置代理
以Firefox为例，以下是代理设置选项  

    socks主机：VPS服务器IP  
    端口：本地端口  
    勾选远程DNS：给Firefox设置远程DNS解析，破解DNS劫持与污染  

**注：如需转载这篇文章请注明出处**  




