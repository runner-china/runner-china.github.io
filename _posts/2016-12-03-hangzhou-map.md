---
author: runner
date: 2016-12-03 11:50+08:00
layout: post
title: "杭州景点分布与高德地图API"
description: ""
comments : true
categories:
- 技术
tags:
- 杭州
- 高德地图
- JavaScript
---


# 0x00 前言

杭州为中国八大古都之一，因为风景秀丽，自古有“人间天堂”的美誉。杭州现有西湖、千岛湖（淳安县）、西溪湿地共三个国家5A级旅游景区，除此以外的各类景点更是繁多。这篇文章要做的是对那些值得游玩且评价较高的景点进行统计、标记其在高德地图上分布，作为备忘，方便以后有需要时候查看浏览甚至在规划短途旅游时候作为参考资料。


# 0x01 景点信息筛选

景点信息取材于大众点评网，维基百科和蚂蜂窝等，然后按照自然景观、人文景观、特色大学校区等进行筛选。比如西湖十景属于自然景观，博物馆属于人文景观，中国美院象山校区属于特色大学校区。结合游客对景点的评价与景点本身的知名度，在进行一轮筛选后，最终只保留了87个景点（仅杭州市区），利用[高德坐标拾取器](http://lbs.amap.com/console/show/picker)获得这87个景点的地理坐标。最后的数据形式如下：

    苏堤春晓（西湖十景之一）	120.13796,30.24388	
    曲院风荷（西湖十景之一）	120.135502,30.251182
    平湖秋月（西湖十景之一）	120.14612,30.252218
    断桥残雪（西湖十景之一）	120.151195,30.258105
    柳浪闻莺（西湖十景之一）	120.158,30.239366
    花港观鱼（西湖十景之一）	120.142121,30.232194
    三潭印月（西湖十景之一）	120.145409,30.238695

# 0x02 高德地图API

因为要做成Web的形式，所以这里用的是[JavaScript API](http://lbs.amap.com/api/javascript-api/summary)。按照官网介绍的流程，首先注册高德开发者、控制台创建应用、获取Key。然后开始学习示例程序的代码，掌握基本点标注、点聚合等基本方法。最后实现的效果如下:  

![](/blog/images/17040301.png)  

[点击这里](https://runner-china.github.io/demo/map_demo.html)还可以查看在线演示版本。

# 0x03 效果分析
- 快速了解景点地理信息：比如快速了解还存在哪些没去过的景点，它们的地理位置在哪里。
- 分析景点数量分布：以西湖周边为最多，其次是灵隐、钱塘江大桥周边。
- 短途旅游规划参考：使一次出门游玩可以尽量多去几个位置比较靠近的景点，提高效率。比如打算去参观六和塔，那么顺便去一下周围2公里范围以内的浙大之江校区、白塔公园、江洋畈生态公园等景点也是不错的选择。

**注：如需转载这篇文章请注明出处**  



