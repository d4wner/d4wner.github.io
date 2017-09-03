---
layout: post
title: PYTHON学习札记（三）
date: 2013-10-03 22:05
author: DEMON
comments: true
categories: coding
---
<span style="color: #ff0000;">1.telnetlib模块的几个小技巧：</span>
<span style="color: #00ff00;">(1)神奇的read_very_eager()</span>
telnetlib模块里的read方法，其说是兼容性较好的，在取得数据完整性和结束符定位做的比较完美，不过，记得设置延时，
待数据抓取完再行第二次查询。
<span style="color: #00ff00;">(2)telnet直接模拟登陆获取的数据，可以抓取到一些敏感信息记录。</span>
参考代码：
<pre lang="python">#encoding=utf-8
def do_telnet(Host, username, password, finish, commands):
    import telnetlib
    '''Telnet远程登录：Windows客户端连接Linux服务器'''
    # 连接Telnet服务器
    tn = telnetlib.Telnet(Host, port=23, timeout=10)
	tn.set_debuglevel(2)
    # 输入登录用户名
    tn.read_until('login: ')
    tn.write(username + '\n')
    # 输入登录密码
    tn.read_until('password: ')
    tn.write(password + '\n')     
    # 登录完毕后执行命令
    tn.read_until(finish)
    for command in commands:
        tn.write('%s\n' % command)
    #执行完毕后，终止Telnet连接（或输入exit退出）
    tn.read_until(finish)
    tn.close() # tn.write('exit\n')
if __name__=='__main__':
	 # 配置选项
	Host = '10.255.254.205' # Telnet服务器IP
	username = 'administrator'   # 登录用户名
	password = 'dell1950'  # 登录密码
	finish = ':~$ '      # 命令提示符
	commands = ['echo "test"']
	do_telnet(Host, username, password, finish, commands)</pre>
参考：http://blog.csdn.net/five3/article/details/8099997
<span style="color: #ff0000;">2.多线程小见：</span>
<span style="color: #00ff00;">(1)当多线程共享一个东西时，</span>可以采用Queue.Queue，通过队列是否为空实施控制，进行锁信号和同步。
<span style="color: #00ff00;">(2)抛弃thread，投向threading的怀抱。</span>过去小弟试过几回thread写多线程，采用了start_new_thread()，写法更加复杂,线程难以很好的控制，而且功能不如threading强大。新版py差不多抛弃了这种写法。
附上参考代码：
<pre lang="python">import threading
import time
class timer(threading.Thread): #The timer class is derived from the class threading.Thread
    def __init__(self, num, interval):
        threading.Thread.__init__(self)
        self.thread_num = num
        self.interval = interval
        self.thread_stop = False 
    def run(self): #Overwrite run() method, put what you want the thread do here
        while not self.thread_stop:
            print 'Thread Object(%d), Time:%s/n' %(self.thread_num, time.ctime())
            time.sleep(self.interval)
    def stop(self):
        self.thread_stop = True 
def test():
    thread1 = timer(1, 1)
    thread2 = timer(2, 2)
    thread1.start()
    thread2.start()
    time.sleep(10)
    thread1.stop()
    thread2.stop()
    return 
if __name__ == '__main__':
    test()</pre>
<span style="color: #00ff00;">(3)主进程结束后，子进程一般要过一段时间才结束，</span>从而完成收尾工作。如果我们想在主进程结束的时候，子进程也结束的话，我们就应该使用setDaemon（）函数。
<span style="color: #00ff00;">（4）没啥好说的，同步所需要锁</span>==&gt;threading.RLock()，所有“临界区”都封闭在同一锁对象的acquire()和release()方法调用之间。
参考：http://blog.csdn.net/lazy_tiger/article/details/3861844 ===================&gt;Python多线程学习(连载)
http://www.cnblogs.com/rollenholt/archive/2011/08/09/2131719.html===========&gt;python多线程学习(细节)
<span style="color: #00ff00;">(5)设置timeout不如sleep，</span>前者只是在初始化socket连接时起作用，而一旦连接成功后如果出现等待那就不会起作用。适当沉睡可以等到线程结束，便于同步,最后可以用join设置超时控制。
<span style="color: #ff0000;">3.os模块小总结</span>
<span style="color: #00ff00;">(1)简单说是取得a文件属性然后覆盖b文件。</span>
<pre lang="python">import os
import stat, time
infile = "samples/sample.jpg"
outfile = "out.jpg"
# copy contents
fi = open(infile, "rb")
fo = open(outfile, "wb")
while 1:
    s = fi.read(10000)
    if not s:
        break
    fo.write(s)
fi.close()
fo.close()
# copy mode and timestamp
st = os.stat(infile)
os.chmod(outfile, stat.S_IMODE(st[stat.ST_MODE]))
os.utime(outfile, (st[stat.ST_ATIME], st[stat.ST_MTIME]))
print "original", "=&gt;"
print "mode", oct(stat.S_IMODE(st[stat.ST_MODE]))
print "atime", time.ctime(st[stat.ST_ATIME])
print "mtime", time.ctime(st[stat.ST_MTIME])
print "copy", "=&gt;"
st = os.stat(outfile)
print "mode", oct(stat.S_IMODE(st[stat.ST_MODE]))
print "atime", time.ctime(st[stat.ST_ATIME])
print "mtime", time.ctime(st[stat.ST_MTIME])</pre>
<span style="color: #00ff00;">(2)命令执行和程序调用</span>
execfile() #直接编译执行
os.execl(path, arg*)
os.execvp(program,arg*) #需要指定程序类型
eval()
os.system(cmd)
<span style="color: #00ff00;">(3)使用 _ _import_ _ 函数获得特定函数</span>
<pre lang="python">def getfunctionbyname(module_name, function_name):
    module = _ _import_ _(module_name)
    return getattr(module, function_name)
print repr(getfunctionbyname("dumbdbm", "open"))</pre>
<pre><span style="color: #00ff00;">(4)使用 os 模块调用其他程序 </span>======================WIN========================</pre>
<pre lang="python">import os
import sys
def run(program, *args):
    pid = os.fork()
    if not pid:
        os.execvp(program, (program,) +  args)
    return os.wait()[0] 
run("python", "hello.py")
print "goodbye"</pre>
=====================UNIX========================
<pre lang="python">import os
import string 
def run(program, *args):
    # find executable
    for path in string.split(os.environ["PATH"], os.pathsep):
        file = os.path.join(path, program) + ".exe"
        try:
            return os.spawnv(os.P_WAIT, file, (file,) + args)
        except os.error:
            pass
    raise os.error, "cannot find executable" 
run("python", "hello.py")
print "goodbye"</pre>
<span style="color: #00ff00;">(5)</span>os.environ["PATH"],os.pathsep===&gt;系统路径分隔符,compile 函数检查语法
