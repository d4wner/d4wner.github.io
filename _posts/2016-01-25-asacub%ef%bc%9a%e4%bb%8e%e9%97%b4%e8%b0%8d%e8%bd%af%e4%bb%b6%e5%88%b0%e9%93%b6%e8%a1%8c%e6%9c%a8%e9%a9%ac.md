---
layout: post
title: Asacub：从间谍软件到银行木马
date: 2016-01-25 00:17
author: DEMON
comments: true
categories: translate
---
<span style="color: #00B050;"><strong>我们最近分析了一个系列银行木马Trojan-Banker.AndroidOS.Asacub，发现了其中在用的一台CC服务器chugumshimusona.com，也在为一款名为CoreBot的Windows木马所使用,这让我们起了对这款移动端银行木马进行分析的心思。</strong></span>

<!--more-->
据我们所知，最初版本的Asacub木马出现在2015年6月。与其说当时的Asacub是银行恶意软件，不如说它是一款木马。早期的Asacub会窃取用户收到的所有短信，并上传到黑客的服务器。这款木马能够从CC服务器接收和处理下面的命令：
<span style="color: #ffcc99;">get_history: 将浏览器历史上传到服务器上。</span>
<span style="color: #ffcc99;"> get_contacts: 将联系人列表上传到服务器上。</span>
<span style="color: #ffcc99;"> get_listapp: 将安装的应用列表上传到服务器上。</span>
<span style="color: #ffcc99;"> block_phone: 锁屏。</span>
<span style="color: #ffcc99;"> send_sms: 发送带有指定内容的短信到指定号码。</span>
而新版本的Asacub出现在2015年7月，这款恶意软件在接口使用了欧洲银行的logo，取代了早期版本的美国银行的logo。
与此同时，它可以执行的命令数量也有了很大的增长：
<span style="color: #ffcc99;">get_sms: 将所有短信上传到服务器上。</span>
<span style="color: #ffcc99;"> del_sms: 删掉指定的短信。</span>
<span style="color: #ffcc99;"> set_time: 为CC服务器的联系设定新的时间间隔。</span>
<span style="color: #ffcc99;"> get_time: 指定CC目标与CC服务器联系的时间间隔。</span>
<span style="color: #ffcc99;"> mute_vol: 静默电话模式。</span>
<span style="color: #ffcc99;"> start_alarm: 启用设备白屏处理器继续运行模式。</span>
<span style="color: #ffcc99;"> stop_alarm: 禁用设备白屏处理器继续运行模式。</span>
<span style="color: #ffcc99;"> block_phone: 锁屏。</span>
<span style="color: #ffcc99;"> rev_shell: 反弹shell执行命令。</span>
<span style="color: #ffcc99;"> intercept_start: 启用短信拦截。</span>
<span style="color: #ffcc99;"> intercept_stop: 关闭短信拦截。</span>
上述的远程命令执行（反弹shell）功能，其实对这类恶意软件的来讲是不太正常的。在接收到命令后，木马会主动将远程服务器接入肉鸡设备的控制台，以便黑客在设备上执行命令和获取输出的结果。这个功能是典型的后门功能，我们其实很少发现银行类恶意软件会使用它。因为大多数银行类恶意软件，旨在从受害者银行账户里窃取资金，而不是控制设备本身。
最新版本的Asacub出现在2015年9月之后，这里的功能比起早期版本来更加关注窃取银行的敏感信息。早期版本只是使用了银行的logo图标，新版本中我们则发现了一些带着银行logo的钓鱼页面：

<img class="alignnone" src="https://cdn.securelist.com/files/2016/01/blog_corebot_1nn-768x698.jpg" alt="" width="768" height="698" />

该木马名叫“ActivityVTB24”。这听起来似乎是一家俄罗斯的大型银行，但是其却自称为乌克兰银行。

<img class="alignnone" src="https://cdn.securelist.com/files/2015/12/blog_corebot_2-768x683.jpg" alt="" width="768" height="683" />
自去年9月Asacub改版以来，钓鱼窗口出现在所有变种之中，但是其中只有银行卡的输入框可用。这意味着黑客可能只攻击他们所使用银行的客户，当然也可能这只是其中一个版本。
在启动后，“秋日版本（autumnal version）”的木马就开始窃取短信，并且还能执行下面的命令：
<span style="color: #ffcc99;">get_history: 上传浏览器历史记录到服务器上。</span>
<span style="color: #ffcc99;"> get_contacts: 上传联系人列表到服务器上。</span>
<span style="color: #ffcc99;"> get_cc: 弹出钓鱼窗口来窃取银行卡数据。</span>
<span style="color: #ffcc99;"> get_listapp: 上传已安装程序列表到服务器。</span>
<span style="color: #ffcc99;"> change_redir: 转发所有来电到指定手机号码。</span>
<span style="color: #ffcc99;"> block_phone: 锁屏。</span>
<span style="color: #ffcc99;"> send_ussd: 运行指定的USSD请求。</span>
<span style="color: #ffcc99;"> update:下载指定的文件并安装。</span>
<span style="color: #ffcc99;"> send_sms: 短信发送指定内容到指定号码。</span>
虽然目前我们并没有注意到有美国用户受到它的攻击，但是黑客对美国银行logo的使用应该是个危险的信号。该木马正在迅速发展，随时有新的特性可能会激活，然后添加进木马里。
<strong><span style="color: #ff0000;">今天的Asacub</span></strong>
2015年末，我们发现了一个新的Asacub，它增添了下面的命令：
<span style="color: #ffcc99;">GPS_track_current – 获取设备的坐标定位，发送给攻击者。</span>
<span style="color: #ffcc99;"> camera_shot – 使用设备的相机拍照。</span>
<span style="color: #ffcc99;"> network_protocol – 目前我们不知道它有任何用处，但应该在未来会和CC服务器产生交互。</span>
这些变种里没有钓鱼功能，但代码中涉及到了银行关键词。有意思的是，该木马一直试图关闭乌克兰银行的官方应用：
<img class="alignnone" src="https://cdn.securelist.com/files/2016/01/blog_corebot_3.png" alt="" width="838" height="207" />
此外，我们还分析了该木马和CC服务器的通信，它似乎对俄罗斯手机银行服务特别感兴趣，
在新年假期，新的改动在俄罗斯通过短信疯狂传播。短短一个星期内，从2015.12.28到2016.01.04，我们已发现6500名感染的用户，该木马由此跻身最活跃的恶意程序TOP5.。Asacub改版后发展的速度才有所减缓，我们将继续跟踪这类恶意软件。

<strong>[参考来源<a href="https://securelist.com/blog/research/73211/the-asacub-trojan-from-spyware-to-banking-malware/" target="_blank">securelist</a>，转载请注明本站翻译]</strong>
