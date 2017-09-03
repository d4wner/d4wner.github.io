---
layout: post
title: python学习札记（一）
date: 2013-08-05 02:55
author: DEMON
comments: true
categories: coding
---
本文本无高深内容，仅是实验时自行整理，看官莫喷。
由于本人不是特聪明，正则之类的特不喜欢用，因此，字符串截取、分割之类的成了小弟的最爱。
不过这篇正则写的不错，留下作为参考。
http://wiki.ubuntu.org.cn/Python正则表达式操作指南

<span style="color: #ff0000;">第一：取字符串与查找</span>
<span style="color: #00ffff;">1.直接寻找字符串</span>
&gt;&gt;&gt; str = "xxxabxxx"
&gt;&gt;&gt; str.find("ab")
返回值为-1代表没有取到。
<strong>demo：string.find( substr, [start, [end]] )</strong>
记住去返回值最好单独赋值，不然容易出错。
参考文献：http://blog.csdn.net/ataraxia2010/article/details/6907907 （没有给出返回值）
<span style="color: #00ffff;">2.去掉指定字符串中指定的字符串</span>
参考文献：http://blog.csdn.net/ataraxia2010/article/details/6907907
import string
string.replace(s,"asd","",1)
<strong>or:</strong>
import re
re.sub("^asd","",s)
与上方不同，直接print打印值会好些。
<span style="color: #00ffff;">3."分割前中后"</span>
比如读一行到s，然后r,_,s=s.partition('指定字符串')现在，r是不要的部分，s就是指定字符串后的部分，如果有结果的话，_的值也是指定字符串。

<span style="color: #ff0000;">第二:python里的循环</span>
提醒一下，中断为continue,break为终断。循环和判断后的‘：’千万别忘了。
参考资料：
http://developer.51cto.com/art/201003/187652.htm
http://www.douban.com/note/242320366/
已经比较全面，略去不提。

<span style="color: #ff0000;">第三:如何输出内容到文件</span>
<span style="color: #00ffff;">1. python test.py&gt;1.txt</span>
<span style="color: #00ffff;">2. 先调用以下语句就可以把print结果保存到文件了</span>
import sys
origin = sys.stdout
f = open('file.txt', 'w')
sys.stdout = f

处理完之后,
sys.stdout = origin
f.close()
PS:网上摘录，使用时可能会出现一定问题。

<span style="color: #00ffff;">3.c="a string to print to file"</span>
f=open('out.txt','w')
print &gt;&gt;f,c
f.close()
注意&gt;&gt;f后面要加逗号，否则会出错
书上说f=open('out.txt','a')
试了不行，估计是权限问题。(网摘)

附上引起以上研究的学习代码：http://blog.sina.com.cn/s/blog_6b60096f01017c0f.html
