---
layout: post
title: Selenium+phantomjs刷量
date: 2016-11-12 21:52
author: DEMON
comments: true
categories: spider
---
<p>
	<span style="color:#DAA520;"><strong>最近有这方面的需求，要帮人在某网站上刷点击量。由于该网站是一家比较知名的大型门户，反作弊机制肯定是有的，所以突发奇想用Selenium试试。</strong></span>
</p>

<p>
	与Selenium类似的东西有lxml，它们采用正则xpath路径匹配标签会多一些。当然，也有人会使用beautifulsoup4去解析网页结构，最后再得到需要的标签。达成目的的路不止一条，这点不再赘述。
</p>

<p>
	本来想用Selenium随便找个浏览器模拟人的浏览网页行为，结果每次需要重新打开一次浏览器，几乎让人抓狂。最后还是采用了phantomjs，这是一种后台浏览器，同样满足模拟行为。
</p>

<p>
	这里提一下，Selenium需要跟浏览器版本的更新度一致。本来我采用的firefox28.0+Selenium3.01,结果踩了半天坑没找到原因。最后，将火狐更新到最新版47.01才解决。
</p>

<p>
	像这类调度，我发现的大概需要注意的有两点：
</p>

<p>
	第一：他们都需要有个浏览器调用的中间件【windows下举例】：
</p>

<p>
	<span style="color:#FF0000;">比如phantomjs.exe（同类的有chromedriver.exe【谷歌】、geckodriver.exe【火狐】），这些需要放在python环境的script目录下。</span>
</p>

<p>
	第二：浏览器需要装在默认目录：
</p>

<p>
	<span style="color:#FF0000;">当然在浏览器目录添加系统环境变量（PATH）也是可以的，这个不是默认添加的，否则会报错。</span>
</p>

<p>
	<span style="color:#000000;">完成了这些工作以后，我们就可以后台对</span>Selenium进行调度了，另外phantomjs具有JS特性，自然也是可以对后台加载的网页进行一些诸如定位、拖动滚动条之类的操作。
</p>

<p>
	比如下面这段代码片段【案例来自于网络】：
</p>

<pre class="brush:python;">
&nbsp; &nbsp; driver.get(pageURL)&nbsp;
&nbsp; &nbsp; js1 = &#39;return document.body.scrollHeight&#39;
&nbsp; &nbsp; js2 = &#39;window.scrollTo(0, document.body.scrollHeight)&#39;
&nbsp; &nbsp; old_scroll_height = 0
&nbsp; &nbsp; while(driver.execute_script(js1) &gt; old_scroll_height):
&nbsp; &nbsp; &nbsp; &nbsp; old_scroll_height = driver.execute_script(js1)
&nbsp; &nbsp; &nbsp; &nbsp; driver.execute_script(js2)
&nbsp; &nbsp; &nbsp; &nbsp; time.sleep(1) &nbsp;</pre>

<p>
	我们刷流量嘛，自然需要代理。这个可以在网上买，大概5块几千个没问题，不保证稳定性。当然，你去其他免费代理网站爬下来也是可以的。
</p>

<p>
	设置代理的法子大概写一下【案例来自于网络】：
</p>

<pre class="brush:python;">
service_args = [
 &#39;--proxy=127.0.0.1:9999&#39;,
 &#39;--proxy-type=socks5&#39;,
 ]
driver = webdriver.PhantomJS(&#39;../path_to/phantomjs&#39;,service_args=service_args)</pre>

<p>
	UA在放在list池子里，需要的时候自行启用：
</p>

<pre class="brush:python;">
from random import choice</pre>

<p>
	header里设置UA【案例来自于网络】：
</p>

<pre class="brush:python;">
from selenium import webdriver
from selenium.webdriver import DesiredCapabilities
driver=webdriver.PhantomJS(executable_path=&#39;存放路径\phantomjs.exe&#39;)
desired_capabilities= DesiredCapabilities.PHANTOMJS.copy()
headers = {&#39;Accept&#39;: &#39;*/*&#39;,
&#39;Accept-Language&#39;: &#39;en-US,en;q=0.8&#39;,
&#39;Cache-Control&#39;: &#39;max-age=0&#39;,
&#39;User-Agent&#39;: &#39;Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36&#39;,#这种修改 UA 也有效
&#39;Connection&#39;: &#39;keep-alive&#39;
&#39;Referer&#39;:&#39;http://www.baidu.com/&#39;
}
for key, value in headers.iteritems():
    desired_capabilities['phantomjs.page.customHeaders.{}'.format(key)] = value
desired_capabilities['phantomjs.page.customHeaders.User-Agent'] = &#39;Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36&#39;
driver= webdriver.PhantomJS(desired_capabilities=desired_capabilities)
driver.get(&quot;http://www.myip.cn/judge.php&quot;)
print driver.page_source</pre>

<p>
	其他涉及具体细节的，这里就不多谈了，网上很多。
</p>

<p>
	我这里也是临时用下，没有太高深的技术。希望对某些朋友有帮助，Good luck!
</p>

<p>
	<strong><span style="color:#FFD700;">[转载请注明来源</span><a href="http://blog.hellsec.net/" target="_blank"><span style="color:#FFD700;">本站</span></a><span style="color:#FFD700;">,谢谢。]</span></strong>
</p>

