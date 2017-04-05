---
author: runner
date: 2017-04-05 19:50+08:00
layout: post
title: "用Wireshark抓包分析获得优酷视频地址"
description: ""
comments : true
categories:
- 技术
tags:
- Wireshark
- HTTP抓包
---

# 0x00 前言

优酷网是目前国内最流行的在线视频服务网站，通常如果需要下载优酷网上的视频就必须安装优酷的官方客户端。本文利用著名的抓包软件Wireshark对优酷网络视频进行抓包，通过数据包分析获得优酷网在线视频的真实URL地址，实现绕过客户端直接通过下载工具对其视频进行下载。

# 0x01 Wireshark简介

[Wireshark](https://www.wireshark.org/)是一个免费开源的网络数据包分析软件，它可以帮助网络管理员检测网络问题，帮助网络安全工程师检查信息安全相关问题。
WireShark的常用功能有如下：

- `捕获数据包功能`：可以选择需要捕获数据的网络接口、设置混杂模式、设置捕获过滤器、设置捕获到永久文件、设置自动停止捕获的条件。
- `数据包过滤功能`：数据包过滤又可以分为捕获过滤器和显示过滤器，过滤器将在下一节进行详述。
- `数据包浏览与分析功能`：显示捕获后的数据包，对每个数据包按网络协议进行分层解释，并提供十六进制数据格式的窗口。
- `文件操作功能`：能够对完成捕获的数据进行保存为多种文件格式，对捕获文件进行导入和合并等操作。
- `搜索功能`：具有按照字符串/十六进制/正则表达式对分组进行搜索的功能。
- `统计功能`：具有协议统计、IP统计、端口统计等功能。

<!--more-->

# 0x02 显示过滤器的语法

数据包过滤又可以分为捕获过滤器和显示过滤器，两者的语法有所不同。如果数据包流量不是很大，可以不对捕获过滤器进行设置即捕获所有的数据包，所以这里仅介绍在完成捕获以后用到的显示过滤器语法，用于只显示那些感兴趣的数据包。

## 比较操作符：  

- 等于： == 	
- 不等于： !=	 
- 大于： >	
- 小于： <	
- 大于等于： >=	
- 小于等于： <=
- 包含： contains
- 正则表达式： matches

## 逻辑操作符：   

- 与： and
- 或： or
- 异或： xor
- 非： not

## 常见的协议字段：

- ip： IP协议
- ip.addr： IP的源或目的地址
- ip.dst： IP目的地址
- ip.src： IP源地址
- tcp： TCP协议
- tcp.port： TCP的源或目的端口
- tcp.srcport： TCP源端口
- tcp.dstport： TCP目的端口
- http： HTTP协议 
- http.request.method： HTTP请求的方法
- http.request.uri： HTTP请求的URI
- http.file_data： HTTP的文件数据字段
- http.host： HTTP请求的HOST字段
- http.user_agent： HTTP请求的USER AGENT字段

# 0x03 HTTP抓包与分析

以youku网的电视剧**《琅琊榜》第1集**为例，展现如何获得下载地址。  

首先打开Wireshark软件，点击开始捕获分组。然后打开优酷网的[《琅琊榜》第1集](http://v.youku.com/v_show/id_XMTMzOTkzNjU0OA==.html?spm=a2h1n.8251845.0.0)，可以看到网页开始播放几个广告视频，这个时候Wireshark的窗口已经有数据开始显示。等广告过去后就进入电视部分，作者大约等了15分以后点击停止捕获。 

youku的视频格式一般为flv格式或者mp4格式的后缀。所以根据上一节的知识，在显示过滤器里面设置如下： 

    http.request.method == "GET" && (http.request.uri contains ".flv" || http.request.uri contains ".mp4")

这表示过滤出HTTP请求为GET的方法，且请求的URL包含“.flv”或者“.mp4”字符串。通过应用该过滤器，过滤出了几十个数据包。

![](/blog/images/17040501.jpg)


通过分析这些数据包的HTTP协议，去掉重复的地址和广告视频的地址，就得到了电视剧的前面十几分钟内容的三段下载地址（由于优酷对完整的视频采用了分片的方法，所以不存在整个电视剧视频的完整地址）。

    http://112.17.4.17/youku/65722A1C9CC49769046856DB0/030002070057BFFB378EAF157A2CB8CA1EE067-CF08-E943-75B9-67271B6708F5.flv
    http://218.205.72.112/youku/6572B4A3E184D83ED1EBFE6ED3/030002070157BFFB378EAF157A2CB8CA1EE067-CF08-E943-75B9-67271B6708F5.flv
    http://112.17.4.24/youku/67745438E48307448DA054FEA/030002070257BFFB378EAF157A2CB8CA1EE067-CF08-E943-75B9-67271B6708F5.flv


最后进行验证，利用下载工具下载这些地址，并能够正常播放这些视频。

![](/blog/images/17040502.jpg)

**注：如需转载这篇文章请注明出处**  




