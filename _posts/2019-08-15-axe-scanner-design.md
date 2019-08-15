---
layout: post
title:  "浅谈被动漏扫思路"
date:   2019-08-15 21:13 +0800
categories: scanner
category: scanner
---

<p>
	<span style="color:#DAA520;"><strong>本文中的规则分析，大部分来自于以前被动漏扫研发的实践，以后会再接触更多业务后，会继续进行更新。其中如有纰漏或者差错，希望诸位给出修正意见。
</strong></span>
</p>


### SQL注入
对于SQLI注入，可以使用sqlmapapi进行检测。

优点在于，对于一些带回显和延时注入，企业内部检测没有waf拦截，会较为准确的定位。

缺点在于，对于这类检测，由于属于大规模扫描，一般会采用level和risk为初级的检测。当注入所需条件比较苛刻时，可能会覆盖不全，造成遗漏。

### XSS
对于XSS的检测的话，已经解决的有两个点，是针对反射性XSS和DOM型XSS的检测。

以前对awvs一类的扫描器做过调研，通常是用变形的常用的payload去fuzz，检测效率是比较低的，也容易被拦截。

针对反射性XSS（包含POST型存储XSS），采用的是检测部分特殊字符，两两成对，如果未曾过滤再去借助专门的XSS扫描器进行扫描，可尝试的字符如下：

```
'
"
< 
> 
/
```

其实还可以做二次复检，加入敏感html标签再次尝试。这样检测成本提高不多，但会更准确一些。

对于DOM型XSS检测，提供有两种方案：

- 采用守护进程的方式去利用headless浏览器来hook渲染，从而检测出漏洞。
不过这种耗时比较长，有时候也因为浏览器渲染失败，以及同时发包量过大，导致内存崩溃漏掉部分漏洞。
- 采用类phamtomjs的引擎，直接增改数据包参数，像检测反射性XSS那样加入特殊字符，通过Ajax方式发送数据包，挨个去检测对比返回值。

针对存储型XSS，主要存在一个痛点，返回的页面不一定是输出XSS的点，那我们应该怎么办呢？

个人有个想法，我们可以通过数据库临时表记录下每个打了暗桩的post数据包，通过md5生成的是随机的值，我们可以采用这两种写法：


```
A = payload+定位符+md5
B = 特殊字符+定位符+md5
```


然后在抓到的后续访问数据包中，如果response含有完整的A或者B，我们可以通过标记去寻找回溯存储型XSS的源头。

但有人可能会说了，你这随意下暗桩，搞得满站都是垃圾数据怎么办？

以前笔者就遇到过几次众测，个别人坏了规矩，登录后拿着扫描器一顿扫，搞得每个页面都在弹窗。

当然这个问题sqlmap等工具也是存在的，但这跟工具本身没关系，是设计和用途的问题。

这就需要业务扫描平台去做去重规则和黑名单限制了。

### 越权漏洞

越权漏洞本身是不太好全自动化检测的，但我们可以做一些半自动化的工作，比如：
- 配置多组关键参数对，交替去替换原request中的参数对，看是否会有关键的差异response返回。
- 采用混淆过的或者置空的cookie，看看返回数据是否与原response相同。
  
### 敏感文件泄露

在web目录可能会存在敏感配置文件或者临时文件，我们需要去通过黑盒探测，做好防治工作。

我们可以采用关键词命中、状态码命中、header命中等方式，多维度进行判断和探测，正确率会相对较高。

目前已经有部分开源扫描器采用了这种方式，可以直接调用或者模拟使用它们的规则。

### 命令执行漏洞

命令执行的判定主要是通过以下几种方式：
- 通过回显，直接判断有没有读取到文件。
- 通过dns服务器，判断有无读取到漏洞主机的请求。
- 通过server反馈时差，判断是否执行了sleep。

但是一般命令执行会有一定的过滤和其他限制，我们需要通过拼接和替换参数值的方式，去构造执行命令的语句，这时候我们就需要用payload去fuzz了。

### 敏感信息泄露

对于敏感信息检测，可以通过关键词进行定位，方式主要有以下几种：
- 配置信息
- 日志信息
- 敏感api和路径
- cookie/token/明文密码/手机号等

### CRLF 注入

检测是否成功注入header，这里有两个点需要注意：
- 不建议使用敏感的头部参数，可以生造一个set-header键值对，也方便检测。
- 返回码需要为30x。


```
payload：
%0aset-header：ceshi;%0a

关键词：
ceshi
```


### SSTI 注入

我们可以通过对response中的回显进行关键词匹配：

```
$\{\{11 * 11\}\}

121
```

### SSI 注入

检测方式类似于命令执行漏洞，采用替换参数值的方式，换取回显关键词和dns请求匹配。


```
<!--#exec cmd="cat /etc/passwd"-->
root

<!--#exec cmd="type c:\windows\win.ini"-->
[extensions]
```


### CORS漏洞

cors漏洞检测主要通过response中的header关键词，进行相应的定位匹配：


```
Origin: http://www.baidu.com

Access-Control-Allow-Origin: http://www.baidu.com
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```


### JSONP漏洞检测

jsonp漏洞需要依靠callback，利用场景简单提一下：

在响应中回调函数被硬编码：
- 基础函数调用
- 对象方法调用

动态调用回调函数：
-  URL完全可控（GET变量）
-  URL部分可控（GET变量），但是附加有一个数字
-  URL可控，但最初不会显示在请求之中

那我们的检测方法可以这么做，先判断header是否存在：

```
Content-Type: application/json
```

再通过关键词定位json数据中，是否含有敏感数据。

### WEBSOCKET劫持检测

对于该漏洞，我们需要先检测header中是否含有Sec-WebSocket-Accept字段。

可以把Origin: https://www.baidu.com修改Origin: http://www.evil.com，若看到服务器端response status返回了 101，可以判定有漏洞。

### URL跳转漏洞

首先我们需要检测是否存在任意跳转漏洞，替换参数值为下面的内容：


```
www.evil.com
@www.evil.com
\www.evil.com
\.www.evil.com
#www.evil.com
www.evil.com?vulnweb.cn
http://www.evil.com
http://www.evil.com?vulnweb.cn
evil.com
http://evil.com
```


然后，首先我们可以检测状态码为30x的response数据包里，是否Location解析出的域名为evil.com，亦或是定位evil.com特有可控的关键词。

其次，某些返回包可以能会通过中转页面，借助js进行跳转。

这时，我们可以选择去页面检测meta-refresh标签，以及检测以下标签：

```
第1种：
<script language="javascript" type="text/javascript">
　　window.location.href="login.jsp?backurl="+window.location.href;
　　</script>
第2种：
<script language="javascript">
　　alert("返回");
　　window.history.back(-1);
　　</script>
第3种：
<script language="javascript">
　　window.navigate("top.jsp");
　　</script>
第4种：
<script language="JavaScript">
　　self.location=’top.htm’;
　　</script>
第5种：
<script language="javascript">
　　alert("非法访问！");
　　top.location=’xx.jsp’;
　　</script>
```

再者，我们可以采用更耗资源的暴力做法，直接用headless浏览器去发包，看是否落地域名是否为evil.com。

### 文件读取漏洞（LFI和RFI）

如果我们需要检测是否存在LFI，替换参数值对为下面的内容：


```
追加：
../../../../../../../../../../../etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
../../../../../../../../../../windows/win.ini

替换：
c:\windows\win.ini
/etc/passwd

匹配关键词：
root
[extensions]
```
至于RFI在放在下面一起讲。


### SSRF漏洞

本漏洞可以可以跟RFI采用类似的检测办法，采用DNS服务器来读取请求。

这里提一下，如果是内网搭建DNS服务器，我们可以规避不能外联和内网IP限制的问题，能更精确的去进行检测。

检测关键词如下：

```
http://vuln.evil.com/index.jsp
http://127.0.0.1/index.jsp
```

发包之后，如果存在漏洞，可以去DNS服务接口去读取到匹配vuln.evil.com（内网DNS服务器为evil.com）的结果；
又或是读取到的内容，是否已经匹配到index.jsp里的动态脚本标签。

### 文件下载漏洞

这里用的payload是文件读取漏洞的那套，但是定位的返回关键词不同，我们可以定位response里的header：


```
Content-Type:application/octet-stream
```

### XXE漏洞

检测xxe漏洞的时候，有两种情况可以尝试fuzz。

- post参数值里含有xml标签时，这种情况可能需要把xxe payload编码发送。
- 直接post了整个xml区块到server端，这种情况可以直接将整块xml区块，替换为xxe payload。
- 整块的xml标签就不写出来了，这里给出附加的payload对,，同样是关键词+DNS服务监听：

```
payload:
file:///etc/passwd
file:///c:/windows/win.ini
http://xxx.evil.com

匹配关键词：
root
[extensions]
以及dns监听请求evil.com
```

### 上传漏洞

上传漏洞的可操作的地方比较多，目前公布出的payload中，暂时还没有能完全囊括所有hack点，下面我们会以java为例简单讲讲。

#### 后缀
在对java应用后缀绕过的时候，我们可以尝试对下面这些后缀进行fuzz：

```
jjspsp
jspx
jspa
jtml
jsw
jsv
jspf
jsp
```

#### 冗余绕过
有些地方，我们进行了hack，但是却不影响数据包发送，比如：


```
Content-Disposition: form-data; name="myfile";;; filename="t3.jsp"

Content-Disposition2: form-data; name="Upload"; filename="1.jsp"

合成一行：
Content‐Disposition: form-data; name="img_crop_file"; filename="1.jsp"Content-Type: image/jpeg

Content‐Disposition: form‐data; name="up_picture"; filename="xss.js
p"

boundary后面加空格：
Content-Type: multipart/form-data; boundary= —————————47146314211411730218525550

Content-Disposition: form-data; filename="xx.jsp"; name="up_picture"

Content-Disposition: form-data; name="file_x"; filename="test.jpg"; filename="test.jsp"

Content-Disposition: form-data; name="Fhq"; test="5W个字符"; filename="test.jsp"

多个Content-Disposition可以用来绕过waf，一般server默认取的是第一个，但研究范围没有覆盖全java web server，有待验证。

绕过文件类型的验证：
Content-Type: image/jpeg

或者直接删除文件类型的验证：
Content-Type: xxxxx
```


检测上传文件有一个痛点，那就是不好自动化定位上传点。

对于**有回显的情况**，暂时有以下的解决办法：

- 利用标签正则，手工定位返回的上传点，这种方法对于大规模扫描时不适用。

- 寻找同一批response包里带有shell后缀的路径，做好web url拼接，暂存后利用shell标识去挨个判断，是否上传成功shell。

对于**无回显的情况**下：

如果黑盒测试，只能根据常用上传目录去爆破shell文件名，以获取到shell标识为成功。

否则的话，需要结合白盒定位，手动配置好路径，估算生成的文件名，去进行黑盒探测。
 
 
