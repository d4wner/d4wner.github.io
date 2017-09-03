---
layout: post
title: PYTHON学习札记（五）
date: 2014-04-20 05:09
author: DEMON
comments: true
categories: tips
---
<span style="color: #ff0000;">博主考虑了许久，因为前面写的札记都较为基础，所以以后记录的都以代码和简要说明为主，精简于型.</span>

1.实现类似于字典--{1:'xx',2,['zz','yy']}的形式：
<pre lang="python">from collections import defaultdict
d = defaultdict(lambda: [])
with open('data.txt', 'rb') as f:
    for line in f:
        key, value = line[:-1].split(",")
        d[key].append(value)
for k,v in d.iteritems():
    print k, v</pre>
附上参考链接：http://www.cnblogs.com/herbert/archive/2013/01/09/2852843.html
<pre lang="python">number_result = {}                                                              

for line in sys.stdin:                                                          
|   l_s = line.strip().split(' ')                                               
|   key, val = l_s[0], l_s[1]                                                   
|   if key in number_result:                                                    
|   |   #number_result[key].append(val)                                         
|   |   l = []                                                                  
|   |   l.extend(number_result[key])                                            
|   |   l.append(val)                                                           
|   |   number_result[key] = l                                                  
|   |   #number_result[key] = number_result[key].append(val)                    
|   |   #print val                                                              
|   else: number_result[key] = [val]</pre>
字典亦可直接append，将value转为list。
上述亦可将dict转化为字符串，用表达式解决。（博主未亲测）

if key in number_result: == if number_result.has_key(key) == True:

2.py判断一个变量属性方法：直接type后打印。
PS: 上面的代码==&gt;
else: number_result[key] = [val]
直接把值赋为了列表属性，属性统一后面处理显得更加便捷。

3.如例子1：
lambda的匿名函数功能挺好用：
举个例子
一般的函数是这样:

def f(x):
return x+1

这样使用 print f(4)

用lambda的话，写成这样:
g = lambda x : x+1
这样使用 print g(4)

&nbsp;

4.python 链接外部数据库

MySQLdb  pymssql。

&nbsp;

<span style="color: #ff0000;">------------项目中，持续更新-------------</span>

&nbsp;

&nbsp;
