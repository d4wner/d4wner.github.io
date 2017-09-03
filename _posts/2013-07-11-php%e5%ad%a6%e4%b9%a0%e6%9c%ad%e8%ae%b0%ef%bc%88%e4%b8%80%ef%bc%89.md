---
layout: post
title: PHP学习札记（一）
date: 2013-07-11 22:36
author: DEMON
comments: true
categories: coding
---
以下只是摆弄PHP时遇到的一些小问题，朋友们看到有新的见解，忘不吝赐教。
PHP的几种常见报错：
1.Fatal error: Can't use function return value in write context in ……这种情况一般是返回值问题，在即在返回值里面不能使用函数（function）返回值，而应该用获取返回值过后的变量中转替代。
2.Undefined offset: 1 in.....这种情况属于数组越界，有时候下标取值错误，或者正则匹配失误时，往往会造成这种情况。通常这种情况我会有两种做法，一是检查下标数，看看匹配正则有没有误判导致取错。二是选择了一种稍笨的方法，由于正则博大精深，不熟的话取的时候偶尔会出错，如果可以采用代替函数分割步奏执行，虽然效率稍低，仍不失为一种好方法。
3.preg_match() expects parameter 2 to be string, array given in......这种情况是使用取得的pre_match时,传入的变量并非string类型，造成这种可能有不少原因。如果只是取得值而类似不准可以用（string）强制转换传入的变量。而万一传入的变量并没有准确的取到值，则要依靠调试，echo，看看错误的缘由。                                                                                                                                                 
4.php中的Notice:undefine dindex以及Notice:Undefinedvariable:一般由于未声明变量而导致的，有时候系统环境变量也需要声明，可考虑使用 isset() 或者empty()进行预置值。

这里顺便一提，大公司的作风，除了实力强劲写的公司迅拥有猛的修补速度，还有部分公司喜欢这里有洞这里掩饰一下，那里有洞那里填补一下，导致了二次漏洞的产生。在这里笔者以为关闭报错也是种不错的方法，虽然有时候会影响web程序的运行。
补救方法下面稍稍列几项，也是从互联网采集而来，笔者做了些修改。
1.服务器配置修改
修改 php.ini 中的 error配置下错误显示方式：将error_reporting = E_ALL 修改为error_reporting = E_ALL &amp; ~E_NOTICE，改后重启下APACHE服务器，方可生效。
2.函数容错
在变量前面 加上一个 @ ，如 if (@$_GET['action']=='save')
3.变量初始化
程序猿养成一个好习惯，php定义变量很容易，但在定义时在记得考虑是否对变量进行容错初始化。
4.函数替代
时间比较紧的话，可以尝试用功能相似，而程序猿本身又相对熟悉的函数去替代不太了解函数。即使要花稍多时间写更长一段代码，也比事后多次修补和安全的不稳定性的代价交换更好。
5.禁止报错
在文件头部加上“error_reporting(E_ERROR | E_WARNING | E_PARSE);这是一剂狠药，如果交互性比较强的代码建议慎用。


