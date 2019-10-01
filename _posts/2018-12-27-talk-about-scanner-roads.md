---
layout: post
title:  "漫谈漏洞扫描"
date:   2018-12-27 16:42:38 +0800
categories: scanner
category: scanner
---

<p>
	<span style="color:#DAA520;"><strong>研究漏扫这块儿有段时间了，虽然是业余自己玩，但平素跟公司漏扫产品线打交道比较多，稍微有些心得，在这里简单分享下。</strong></span>
</p>


### 企业级漏扫

- 盒子扫描器

对于漏扫产品的话，部分甲方单位会按公安那边的标准，在内网部署一些盒子扫描器（硬件服务器+扫描软件）。

说实话，这玩意儿定位是比较尴尬的，虽然大的单位每年有一定的采购指标。但是有时候还是会听产品经理吐槽，每次实在卖不出量，可能一单安全服务生意卖出个一两台就不错了。

当然，现在漏扫一般会配合漏洞管理、网站监控等产品一起卖。为了覆（tong）盖（hang）产（jing）品（zheng）线，给售前和销售操控的空间，这款产品还是必须要的。

卖漏扫盒子的利润还算可观，只要销售和渠道给力，传统乙方还是愿意做的这门生意的。

代表厂商有:
```
sangfor
venus
nsfocus
topsec
...
```

- 在线漏扫服务

在线漏扫的话，一般难以对内网进行检测。大多数的操作是，在验证外网某站的所有权后，再签协议授权扫描。不过由于成本较盒子更加低廉，容易受到中小厂商的追捧。

当然，如果内网也需要享受这样的服务的话，自然还是需要安服人员带着盒子，或者类似封装好的扫描器，在企业单位进行驻场检测。

代表厂商有:
```
360
knownsec
...
```
- 定制漏扫

据笔者所知，部分云服务厂商，会对云服务客户提供了定制漏扫服务。

由于是自家的服务器，自然对客户的业务具有一定的了解。无论是做漏扫，还是做资产监控还是态势感知，都是相对容易的。

云服务厂商在对这部分客户做漏扫时，由于统一的架构部署，安服漏扫会比较精确和有效。貌似这样的漏扫服务，一般不会对外开放，算是定制的服务。

顺便提一句，部分漏洞平台，好像对于大客户也推出了一条龙服务，其中是包括定制漏扫的。

代表云厂商有:
```
alibaba
tencent
kingsoft
riskivy
...
```

- 免费商业漏扫

市面上也出现了部分优秀的商业级别漏扫，咱们这里先别讨论是免费版还是破解版。

正是有了这些漏扫产品，在驻场和分公司的苦逼兼职的安服人员，才有了一口饭吃【纯吐槽公司制度】。

这里解释下，因为公司内部的漏扫产品，不是分驻地都能拿到授权的，那最后怎么办呢？用破解的。效果不好咋办？换其他家的破解或者免费产品。

代表产品有：
```
AWVS
nessus
arachni
metasploit
sqlmap
burpsuite
appscan
netsparker
...
```


### 开源漏扫

- 社区级漏扫

这些产品一般是社区或者团队在维护的，一般为乙方渗透人员或者Bug Bounty人员所用。

一旦他们需要对企业机构，或者政府单位进行渗透测试时，可以根据情况，部署分布式节点扫描，加快漏扫速度。

笔者依稀记得曾经的bugscan，好像大家都可以接入公网节点。这听起来，其实有点像以前的迅雷p2p，可以加速所有运行的任务。

不过后来好像由于各种原因，部分人搞到了源码和payload包，自己玩起了单机。多台外网VPS一部署，扫起东西来也是美滋滋，亲测出结果还是比较快的。

不过这种漏扫有个坏处就是，一旦社区不用心再维护，渐渐就没有人再提交payload，毕竟单个漏洞的生命周期还是不长的。

当然，这种产品还有个去路，就是实现企业化。

一旦变成企业级产品，就会有更多的资源投入去维护它，自然能更好的发展下去。

比较可惜的是，升级后的版本以及payload，自然大多数就不会再开源了。

代表产品有：
```
bugscan
蚁逅
tangscan
Beebeeto
Pocscan
Osprey「鱼鹰」
...
```

- 综合扫描

由于各种脚本语言的兴起，大幅减少了coding的难度和时间，网络上涌现出一批由团队或个人维护的综合扫描器。

虽然得吐槽下，大多数质量良莠不齐，造轮子的比较多，而且后续长期作者维护的比较少，不过其中不乏优秀的个体。

综合扫描定义比较模糊，一般除了exp检测和CMS识别外，还有部分项目加入了路径爆破、资产统计、端口扫描等功能。

不过让人稍稍有点失望的是，这类综合扫描可能大同小异，暂时没有发现特别亮眼的点。

在笔者过往的系列文章中，也谈过部分关于综合扫描器细节，这里不再细说。

代表产品有:
```
w9scan
AngelSword
fenghuangscanner
猪猪侠PPT中提过的扫描器
...
```

- Gui扫描

这里之所以单独区别于前面的综合扫描器【其实是笔者实在想不出小标题了XD】，是想谈谈其开发时间、开发难度，以及插件化难度。

虽然说部分Gui扫描器也实现了插件化，但作者们大多喜欢自己更新，或者只要求邮件方式提交插件。

这样的话，把产品生态搞成了一个近似闭环，但是肯定又远不及apple store之类的体量，导致用户主动提交的漏扫插件是比较少的。

当然，有部分漏扫的功能和用户体验，还是做的很不错的，很受大家追捧，笔者当年也用的很顺手。

代表产品有：
```
椰树
北极熊扫描器
k8 team系列扫描器
千手千眼佛网站扫描器
...
```

- 代理扫描器

说起代理扫描器，可能内容就比较宽泛了，这里简单讲下，以后有机会单独谈谈。

何谓代理，中间人也，只要你能抓住中间流量，便可以作为基准去做漏洞扫描或者fuzz。

大家可能会想到利用抓包，利用网卡流量进行分析；有人也许会通过浏览器流量代理进行分析；还有人会通过浏览器本身提供扩展插件功能，直接对页面进行即时钩子探测。

说到这块儿，笔者所见的一般都是轻量级的，也可能是见识少吧。个人感觉很少有在采集存数据库以后，在离线端部署过多的exp探测任务的。

毕竟，这块儿也是要考虑到扫描效率，以及会话过期问题的。

另外，貌似代理扫描器对owasp的一些通用漏洞的fuzz，以及对敏感内容的检测，会显得多一些。对于能检测逻辑漏洞的被动扫描器，也算是比较高level的了。

代表产品有：
```
ysrc的GourdScan
burpsuite插件系列
wyproxy的衍生扫描器
浏览器插件系列
...
```

### 结语

笔者见过的常见漏扫的架构差不多就是这些了，点到为止吧。另外，笔者自研的也有类似产品，这里就不打广告了XD。

可能有部分内容，由于时间关系没能例举全，也可能有部分笔误，期待指正和建议。