---
layout: post
title: SQL历史语句的追踪宝典
date: 2014-04-20 06:33
author: DEMON
comments: true
categories: data
---
鄙人最近在进行一个小项目，前面的文章提到过，在这几天做到了数据库行为分析这块儿。


我们如果拿到历史语句，可以进行一个反黑阔的追踪，可以进行数据异常分析。查数据库历史语句时，我们还能顺带进行各种日志审计和监控记录，对于运维的朋友来说很有帮助。


那么，我们如何获得数据库的执行过的历史语句呢，能否自动将其导出？定时处理数据？或者能否做中间件将其过滤？

鄙人在这里将在这里做一个详细的讲解。



<strong><span style="line-height: 1.6; color: #ff0000;">1.Mssql：</span></strong>

mssql的历史语句的获取和处理着实让鄙人痛苦了一把，在网上曾找到以下几种方法。

（1）sql server profiler

（2）sqldiag

（3）log explorer

我们先来看第一个，这个是sql server自带的追踪分析工具（<span style="color: #33cccc;">单独下载的版本很挫，功能少，鄙人找了很久没找到好的</span>），图形化界面操作，可以选择性过滤，可以手动保存产生的trace内容。缺点：不能自动保存（<span style="color: #33cccc;">网上有人说可以计划任务启动再保存啥的，没找到方法</span>），耗资源（mssql套件通病），资源多了很不好看（也可以通过一定的方法过滤）。

至于第二个sqldiag，默认位置：C:\Program Files\Microsoft SQL Server\90\Tools\Binn\。网上说其在log目录能产生一个叫sqldiag.txt的文件，里面可以看到历史语句，估计是鄙人人品太挫了，表示没看见这个文件。找遍了所有的trc文件也没看见我要的结果。曾经找一个基友appleu0帮忙试验了下，他也仅仅是在关闭程序后才看到一个文件生成（<span style="color: #33cccc;">文件名是啥我忘了Orz..</span>），且这个程序好像最多能记录最后一百条的数据，很挫，虽然还是够轻量级。

最后看看第三个，log explorer。其中提供两个强大的工具：日志分析浏览，对象恢复。这里讲老实话，我自己试的时候没有用成功，因为我采用的是windows认证，在安装连接数据库时老出错，后来一怒就不用了，但听某基友讲这个还是蛮好用，不过也达不到我的要求。

朋友们可能要问，那我最后采用的方法是什么呢，难道比上面几个现成的工具都要好么？

没错，虽然不一定比上面几个都好，但绝对是最便捷的，最轻量级的，因为它没有借助任何第三方工具（<span style="color: #33cccc;">呕吐。。</span>），采用的是mssql的自带的追踪器trace（profiler的本质）。

如何创建一个trace？

1）执行存储过程sp_trace_create创建一个追踪器

2）执行存储过程sp_trace_setevent添加自己想订阅的事件以及最终结果集的列名

3）执行存储过程sp_trace_setfilter设置过滤器来对过滤产生数据

附上代码：
<pre lang="sql">DECLARE @return_code INT;
 DECLARE @TraceID INT;
 DECLARE @maxfilesize BIGINT;
 DECLARE @Onset BIT;  --------弄了个bit型变量
 SET @Onset = 1; 	
 SET @maxfilesize = 200; ------（单位为M）弄了个大点的，一般不会增长太多，增长太多肯定出问题了，让他自动关闭！
 --step 1: create a new empty trace definition
 EXEC sp_trace_create
                 @traceid OUTPUT
                , @options = 2           -------文件大小达到顶峰时会自增到新文件
                , @tracefile = N'C:\demon\LongRunningQueries'  ---输出文件，后缀为trc
                , @maxfilesize = @maxfilesize
     , @stoptime =NULL
     , @filecount = 2;
 -- step 2: add the events and columns

 EXEC sp_trace_setevent
                 @traceid = @TraceID
                , @eventid = 12 -- SQL:BatchCompleted ：用于sql语句追踪。
                , @columnid = 1 -- TextData ：历史语句
                , @on = @Onset;--include this column in trace

 EXEC sp_trace_setevent
                 @traceid = @TraceID
                , @eventid = 12 -- SQL:BatchCompleted
                , @columnid = 13 --Duration
                , @on = @Onset;--include this column in trace
 EXEC sp_trace_setevent
                 @traceid = @TraceID
                , @eventid = 12 -- SQL:BatchCompleted
                , @columnid = 15 --EndTime
                , @on = @Onset;--include this column in trace        
 -- step 3: add duration filter
 DECLARE @DurationFilter BIGINT;
 SET @DurationFilter = 10000000; --duration in microseconds
 EXEC sp_trace_setfilter
                 @traceid = @TraceID
                , @columnid = 13
                , @logical_operator = 0 --AND
                , @comparison_operator = 5 -- less than or equal to|---妈蛋，tmd大于100000*还抓个蛋的语句
                , @value = @DurationFilter; --filter value
 SELECT @TraceID AS TraceID;</pre>
原文分析在这儿:http://www.cnblogs.com/fzrain/p/3476434.html#commentform
文章作者还是值得肯定的，不过有几处没讲明，还有bug要报错，鄙人修修补补勉强可用了。

在原文里，还可以看到开关trace的语句，见其下文，根据需要进行开关即可。

参考资料：
http://blog.csdn.net/hb_gx/article/details/1745800 SQL Server Profiler 有关的几个存储过程和函数
http://technet.microsoft.com/zh-cn/library/ms186265%28v=sql.105%29.aspx 官方文档

什么？问我什么优势？上面引用的文章讲的很清楚了。
1）trace是语句执行，没有GUI界面，耗资源少。
2）随时开关，即时保存。
3）过滤灵活，不用三方。

&nbsp;

<strong><span style="color: #ff0000;">2.Mysql:</span></strong>

相较于mssql的历史语句获取，mysql要轻量得多，也简单的多，下面给大家附上方法。

若要获取mysql语句，不采用第三方工具的话（<span style="color: #00ffff;">事实上我不知道哪款三方可以用来获取这个的</span>），可以通过配置mysql.ini(linux下为my.cnf)，让他开启sql记录日志，

PS:开启后日志量会很大喔，跟mssql一样，可以使用脚本定时删除，至于脚本这里就不提供了，太简单了，与本文关系也不大。

网上的配置很多都不能用，估计是不同系统问题吧：
这里提供一份通用的配置（不一定管用）：
<pre lang="sql">[mysqld]
log=/var/log/mysql_query.log
#日志的路径（这里最不靠谱，个人觉得导出到表里是最容易生效的）
----------------------------------------
general_log=1
#开启的选项
# 将日志记录到mysql的table中
log_output=TABLE
#(默认导出到文件)</pre>
&nbsp;

可以将日志输出到表里，也可以输出到日志里，配置好了直接重启mysql即可，鄙人重启了计算机才管用，真心人品碉堡了。

表的位置：mysql.general_log，里面包含了sql用户名，语句执行日期，还有历史语句本身等等，还是比较全面的，不需要的时候就把日志开关关掉，自行清理。

查看开关是否开启的办法：

&nbsp;

&nbsp;
<pre lang="sql">mysql@localhost.(none)&gt;show global variables like "%genera%";
+------------------+------------------------------+
| Variable_name | Value |
+------------------+------------------------------+
| general_log | OFF |
| general_log_file | /data1/mysql9999/etch171.log |
+------------------+------------------------------+
2 rows in set (0.00 sec)

mysql@localhost.(none)&gt;set global general_log=on;
Query OK, 0 rows affected (0.02 sec)

mysql@localhost.(none)&gt;set global general_log=off;
Query OK, 0 rows affected (0.00 sec)</pre>
仔细研究下，朋友们都很容易懂得。

<strong><span style="color: #ff0000;">3.其他数据库的语句获取：</span></strong>

前面也提到了，鄙人人品也搓，人也懒，不大可能面面俱到，但总要给有兴趣的朋友一点提示不是，这不，给大家附上一份资料。

<a href="http://www.itpub.net/tree/index_1/" target="_blank">oracle</a> 里有v$sql等视图来记录执行过的sql语句,<a href="http://www.itpub.net/tree/index_51/" target="_blank">db2</a>里有事件监控器,<a href="http://www.itpub.net/thread-1430388-1-1.html" target="_blank">sybase</a> 里有mon监控表来记录，另外，附上progresql的记录方法：

&nbsp;

&nbsp;

&nbsp;
<pre lang="sql">digoal=# alter database digoal set log_min_duration_statement=0;
ALTER DATABASE
或者
digoal=# alter database digoal set log_statement='all';
ALTER DATABASE
-- 只记录下digoal数据库的所有SQL, 其他数据库则按系统默认的配置.
语法 : 
ALTER DATABASE name SET configuration_parameter { TO | = } { value | DEFAULT }
ALTER DATABASE name SET configuration_parameter FROM CURRENT
ALTER DATABASE name RESET configuration_parameter
ALTER DATABASE name RESET ALL
PostgreSQL的权限分级较多, 例如可以按角色配置权限, 也可以按数据库配置权限, 也可以按会话配置权限等等。</pre>


把握历史记录，利用好日志搭建蜜罐神马的，不用多久,你就能升职加薪、当上总经理、出任CEO、迎娶白富美、走上人生巅峰,想想还有点小激动呢。^_^
这里还得感谢下该死的B哥，虽然这货挺可恶的，但确实帮了我不少忙。

<span style="color: #00ff00;"><strong>--------------Enjoy yourself!------------------</strong></span>
