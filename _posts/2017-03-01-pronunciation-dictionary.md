---
author: runner
date: 2017-03-01 12:50+08:00
layout: post
title: "对韦氏词典的单词音频数据进行爬取"
description: ""
comments : true
categories:
- 技术
tags:
- 爬数据
- shell脚本
---

# 0x00 前言

最近，突然想练习一下单词的听写，而且最好能够自己按照自己定制的单词列表进行。采用的方法是到网上去爬取每个单词的发音音频，爬下来后再进行合并成一段音频在电脑上播放。通过比较各种网络词典的发音，感觉[韦氏词典](https://www.merriam-webster.com/)的发音库比较地道，所以尝试分析它的音频数据爬取方法。

# 0x01 URL地址分析

以单词[cat](https://www.merriam-webster.com/dictionary/cat)为例，发音页面的URL为：
`https://www.merriam-webster.com/dictionary/cat`  
打开这个单词页面以后，选择Chrome开发者工具，选择Network标签页，抓包得到单词cat的mp3地址为：  
`https://media.merriam-webster.com/audio/prons/en/us/mp3/c/cat00001.mp3`  
虽然不费吹灰之力得到单词cat的音频地址，下一步仍然要进一步分析这个URL是如何生成的。打开Chrome开发者工具，在Elements标签页，查看单词发音部分对应的DOM节点，找到如下的一段代码：  


    <a class="play-pron converted" data-lang="en_us" data-file="cat00001" data-dir="c" 
	href="https://www.merriam-webster.com/dictionary/cat?pronunciation&amp;lang=en_us&amp;dir=c&amp;file=cat00001" 
	title="How to pronounce cat (audio)"> play <span class="play-box"> </span></a>

<!--more-->
另外，在网页源代码中找到了下面几段重要的JS代码：

	window.mwdata.pronsDomain   = '//media.merriam-webster.com';
	lang=lang.replace("_","/")
	audioSrc=window.mwdata.pronsDomain+"/audio/prons/"+lang+'/'+'mp3'+'/'+dir+'/'+file}

综合分析，变量audioSrc定义的就是音频mp3的下载URL。它的构成其实为：`https://media.merriam-webster.com/audio/prons/en/us/mp3/` +
`dir` +  `/` + `file`+ `.mp3`。 其中，`dir`就是第一段DOM代码中date-dir属性的值， `file`就是data-file属性的值。  
最后能够拼凑成一个完整的URL：`https://media.merriam-webster.com/audio/prons/en/us/mp3/c/cat00001.mp3`

# 0x02 用Shell脚本爬取数据

通过上述的基本分析，就可以写一个程序实现批量抓取单词音频了。那究竟抓哪些单词好呢？据我所知，有几个著名的单词表，比如General Service List，Academic Words List，Ogden’s Basic English Word List等。 这里就选用GSL（General Service List）作为测试，将GSL的所有单词按行存为一个文本，命名为**gsl.txt**。

利用上述的URL地址分析，Shell脚本写了一段爬取的脚本如下：
	
{% highlight shell %}
#!/bin/bash
list=$(cat gsl.txt)
for word in $list
do
    curl  "https://www.merriam-webster.com/dictionary/$word" | grep -o 'data-file.*data-dir.*' | head -1 | awk 'BEGIN {FS="\""} {print $2 ,$4}' > tmp.txt
    file=$(cat tmp.txt| awk '{print $1}')
    dir=$(cat tmp.txt| awk '{print $2}')
    curl "https://media.merriam-webster.com/audio/prons/en/us/mp3/$dir/$file.mp3"  -o "$word.mp3"
done
{% endhighlight %}

其中，程序读取单词表文件gsl.txt，分析生成URL，最后用curl工具递归下载mp3数据。
最终得到了一份GSL的MP3，**点击这里打包下载**

# 0x03 下一步的展望

得到了这些mp3格式音频，可以进一步实现单词的“听写”功能。比如设计一个软件，在软件上按随机模式或者易错词模式进行单词读音播放，听到读音后我们在软件上以默写的方式输入该单词，让软件帮助我们进行拼写检查，找到自己对哪些单词拼写与发音听力还存在的薄弱点，记录并可以反复训练提高。

**注：如需转载这篇文章请注明出处**  




