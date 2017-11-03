---
layout: post
title: 对于python框架Scrapy+Gevent的研究
date: 2014-09-20 21:26
author: DEMON
comments: true
categories: other
---
<span style="color: #00ffff;"><strong>没错，标题党君又来了！文章只是做些第三方评论，不喜勿喷。</strong></span>

前几天看到<span style="color: #ff0000;"><a title="Freebuf" href="http://www.freebuf.com/tools/43194.html" target="_blank"><span style="color: #ff0000;">FB上的一篇文章</span></a></span>，是将用Scrapy爬虫框架加以关键词词尝试，从而将可用的关键词与相应的URL返回存到结果里，个人感觉还是有比较大的改进空间的。覆盖攻击向量字段如下：

Http头中的Referer字段
User-Agent字段
Cookie
表单（包括隐藏表单）
URL参数
RUL末尾，如 www.example.com/&lt;script&gt;alert(1)&lt;/script&gt;
跳转型XSS

由于英文说明书的原作者说该Scrapy的XSS延伸版不能进行Ajax判断，还是有点小遗憾，希望日后改进。

感觉其提供的测试页面爬下来效果不错，我这儿没有图床可用，就不展示了。（<strong><span style="color: #00B050;">话说有朋友可以给鄙人一免费图床地址么，2333333</span></strong>）

附上该Git的地址：<a title="xsscrapy" href="https://github.com/DanMcInerney/xsscrapy" target="_blank"><span style="color: #ff0000;">下载</span></a>。

还有就是，本人测试用的是Ubuntu13.04，作者建议pip安装，鄙人使用自带的安装包pip，表示有不少问题，后来在网上下了一个<span style="color: #ff0000;"><a title="pip1.5" href="https://pypi.python.org/packages/source/p/pip/pip-1.5.4.tar.gz" target="_blank"><span style="color: #ff0000;">1.5版本</span></a></span>的才成功。此外，个人感觉apt-get安装确实挺给力，除了没有的包以外，基本很难报错（知道肯定有人吐槽这B说的不是废话么^_^，以前一个朋友就挺爱骂我SB，不过可惜再也没能见到他了）。

PS：里面所需的BeautifulSoup最好用BeautifulSoup3.2.1，BeautifulSoup4.x版本已改名为bs4，坑惨小弟了，半天没反应过来。其他的pip安装（如pybloom），也可apt-get安装（如py-requests）,最后，对付某<span style="color: #00B050;">error: command 'x86_64-linux-gnu-gcc' failed with exit status 1</span>错误时，<span style="color: #00B050;">sudo apt-get install python-twisted-web python2.7-dev</span>，可能会用的上。

Scrapy是个不错的爬虫框架，最近笔者自己打算好好研究一下，结合注入工具进行爬虫式扫描，感觉应该不错的样子。如果有朋友有兴趣，或者有现货，欢迎提出宝贵建议，不胜感激。

另外，前面提到的Gevent是一名访客告诉小弟的，查看了下，是Python的一个高并发框架（高级术语名为协程）。没记错的话，还是以前那位爱骂我SB的朋友告诉我的（小感伤一下），因为以前有做分布式监控的想法。以后考虑将其纳入做项目的计划范围。

本文很水，不过以后有心得会更新的，除非有单独料，我会单独提出。

<span style="color: #00ffff;">Enjoy yourself.^_^</span>

&nbsp;

&nbsp;

&nbsp;
