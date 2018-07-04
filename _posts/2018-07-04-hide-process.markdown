---
layout: post
title:  "浅谈进程隐藏之术"
date:   2018-07-04 19:30:38 +0800
categories: cert
category: cert
---

<p>
	<span style="color:#00B050;"><strong>前段时间遇到了一些【进程隐藏】相关的应急事件，故此心生一念，对网上一些资料和部分个人经验做了总结，以飨众人。</strong></span>
</p>

### windows进程隐藏

####  基于系统服务的进程隐藏技术

在WIN 9X系列操作系统中, 系统进程列表中不能看到任何系统服务进程, 因此只需要将指定进程注册为系统服务就能够使该进程从系统进程列表中隐形。

在WIN 9X下用RegisterServiceProcess函数隐藏进程，NT架构下用不了 即win2000和xp等什么的用不了此方法。


替换tasklist、ps、top
https://blog.csdn.net/qq_27446553/article/details/54591099
windows高版本内核难以进行真正的进程隐藏，除非编写底层驱动。
在进程LoadLibrary某个DLL文件后，这个DLL是不可以被删除的，但是可以改名和移动位置（当然，就算移动了位置也不可以删除它），所以代码中可以用MoveFile移动到某个角落去。
这样，DLL就从原来的位置消失了，而新位置在资源管理器中又无法访问到，达到了简单隐藏DLL的目的。


```
CreateDirectory('d:\test\....\', nil);
MoveFile('D:\test\Hack.dll', 'd:\test\....\Hack.dll');
```



#### 基于API HOOK的进程隐藏技术

API HOOK指的是通过特殊的编程手段截获WINDOWS系统调用的API函数,并将其丢弃或者进行替换。 通过API
HOOK编程方法,截获系统遍历进程函数并对其进行替换,可以实现对任意进程的隐藏。


#### 基于 DLL 的进程隐藏技术:远程注入Dll技术

先编写一个API的DLL，将它远程注入进程，写入远程进程的内存地址空间，并建立远程线程执行。

不触发PG（patchguard），又能隐藏驱动：
当驱动加载时 会将驱动信息加入那个链表，可以直接阻止这个加入的过程。
"MiProcessLoaderEntry"，这个函数将驱动信息加入链表和移除链表：

```
调用MiProcessLoaderEntry(pDriverObject->DriverSection, 0);
PCHunter显示为红色~

能不能完全隐藏?
pTargetDriverObject->DriverSection = NULL;

// 破坏驱动对象特征
pTargetDriverObject->DriverStart = NULL;
pTargetDriverObject->DriverSize = NULL;
pTargetDriverObject->DriverUnload = NULL;
pTargetDriverObject->DriverInit = NULL;
pTargetDriverObject->DeviceObject = NULL;
```

#### 基于远程线程注入代码的进程隐藏技术

这种方法与远程线程注入DLL的原理一样,都是通过在某进程中创建远程线程来共享该进程的内存空间。

所不同的是,远程线程注入代码通过直接拷贝程序代码到某进程的内存空间来达到注入的目的。

因为程序代码存在于内存中,不仅进程列表中无法检测,即使遍历进程加载的内存模块也无法找到被隐藏程序的踪迹。

#### Rootkit方式

Intel CPU 有4 个特权级别： Ring 0， Ring 1， Ring 2， Ring 3。Windows 只使用了其中的 Ring  0 和  Ring  3 两个级别。

操作系统分为内核和外壳两部分：内核运行在Ring0级，通常称为核心态（或内核态），用于实现最底层的管理功能，在内核态可以访问系统数据和硬件，包括处理机调度、内存管理、设备管理、文件管理等；外壳运行在 Ring 3级，通常称为用户态，是基于内核提供的交互功能而存在的界面，它负责指令传递和解释。通常情况下，用户态的应用程序没有权限访问核心态的地址空间。

Rootkit 是攻击者用来隐藏自己的踪迹和保留 root 访问权限的工具，它能使攻击者一直保持对目标机器的访问，以实施对目标计算机的控制[1]。从 Rootkit 运行的环境来看，可将
其分为用户级 Rootkit 和内核级 Rootkit。
用户态下，应用程序会调用 Win32 子系统动态库（包括Kernel32.dll， User32.dll， Gdi32.dll等）提供的Win32 API函数，它们是Windows提供给应用程序与操作系统的接口，运行在Ring3级。用户级 Rootkit 通常就是通过拦截 Win32 API，建立系统钩子，插入自己的代码，从而控制检测工具对进程或服务的遍历调用，实现隐藏功能。

内核级Rootkit 是指利用驱动程序技术或其它相关技术进入Windows 操作系统内核，通过对 Windows 操作系统内核相关的数据结构或对象进行篡改，以实现隐藏功能。

由于Rootkit运行在Ring0级别，甚至进入内核空间，因而可以对内核指令进行修改，而用户级检测却无法发现内核操作被拦
截。
下面介绍两种使用Rootkit技术来实现进程隐藏的方法。注册表来实现启动,因而易于被检测出来。显然,要增强进程的隐蔽性,关键在于增强加载程序文件的隐藏性。


- SSDT Hook

参考资料：

```
进程隐藏与进程保护（SSDT Hook 实现）（一）
http://www.cnblogs.com/zmlctt/p/3979105.html

进程隐藏与进程保护（SSDT Hook 实现）（二）
https://www.cnblogs.com/zmlctt/p/3979108.html

进程隐藏与进程保护（SSDT Hook 实现）（三）
http://www.cnblogs.com/BoyXiao/archive/2011/09/05/2168115.html
```

 
- DKOM（Direct Kernel Object Manipulation，直接内核对象操作）

```
使用DKOM方法进行进程隐藏。在Windows操作系统中，系统会为每一个活动进程创建一个进程对象EPROCESS，为进程中的每一个线程创建一个线程对象 ETHREAD。

在 EPROCESS 进程结构中有个双向链表 LIST_ENTRY，LIST_ENTRY结构中有FLINK 和BLINK 两个成员指针，分别指向当前进程的前驱进程和后继进程。

如果要隐藏当前进程，只需把当前进程的前驱进程的BLINK 修改为当前进程的BLINK，再把当前进程的后继进程的FLINK修改为当前进程的FLINK。
```


- 关于断链隐藏进程

Windows系统中的所有进程通过其ActiveProcessLinks结构中的指针来引用。它们构成了诸如taskmgr.exe（任务管理器）或某些SysInternals（例如procexp.exe）等工具使用的双链表。

DKOM技术【直接内核对象操纵（Direct Kernel Object Manipulation）】隐藏了一个取消链接它自己的ActiveProcessLinks的进程，并将“前一个”和“下一个”进程直接相互链接。
事实上，许多监控/系统工具（例如SysInternals Microsoft套件）都是基于双链表的进程枚举。

不过断链隐藏进程，容易蓝屏，貌似也过不了pg。


### Linux进程隐藏

- 一种简单的方法：系统启动时会依据 /etc/fstab 文件内容来挂载分区，在 proc 分区挂载参数中加入 hidepid=2 参数后，登陆系统的用户只能查看到当前用户启动的进程的信息。也就是说， tomcat 用户只能看到属于 tomcat 用户进程的信息。

- 在内核中新增两个信号，当进程向内核发出 hide 信号时，内核将不会为该进程在 /proc 目录下生成对应的目录，从而也就从底层铲除了进程的信息，即使黑客获得了 root 权限也无法通过常规手段察觉到蛛丝马迹。除此之外，新增的unhide信号作用恰好与 hide 信号相反。

- 对其他用户隐藏
 
如果你使用的linux kernel(内核)是3.2以上的版本(或者使用的RHEL/CentOS是6.5以上的版本)，你就可以对其他用户隐藏进程。
只有root用户可以看到所有的进程，而非root用户，只能看到属于自己的进程信息。你所需要做的仅仅是开启linux kernel加固选项 "hidepid "来重新挂载 /proc文件系统。

```
https://www.jb51.net/LINUXjishu/347787.html
```

- linux下根据进程名称隐藏进程的PID

1. 把要隐藏的进程PID设置为0，因为系统默认是不显示PID为0的进程，不过缺陷比较大。
其核心思想就是把task->pid变成0，就成了0号进程。而在ps，top命令中，是不显示0号进程的相关信息。这么一来，在/proc/文件夹下就不会有该进程的相关信息了。
 
2. 修改系统调用sys_getdents（）。

```
http://blog.chinaunix.net/uid-26585427-id-5077452.html
```

3. 另外，还有一种比较简便的方法，就是把int main(int argc, char*argv[])中的参数变成0，那么就在单纯的ps命令中就不会显示进程相关信息，但是/proc/文件夹下，还会存在该进程的相关信息。

```
https://blog.csdn.net/xqhrs232/article/details/51906206
```

- 遍历PspCidTable表检测隐藏进程


```
https://www.cnblogs.com/kuangke/p/5761615.html
```


- 部分补充说明

在变更文件里可以看到一些挖矿程序，同时 /etc/ld.so.preload 文件的变更需要引起注意，这里涉及到 Linux 动态链接库预加载机制，是一种常用的进程隐藏方法，而 top 等命令都是受这个机制影响的。

可以看看其中有没有包含可疑的so文件，然后记录后去掉。
```
注意
/etc/rc.d/init.d/network
/etc/resolv.conf

cat /etc/ld.so.preload
top 查看pid
ls  -lh /proc/pid号
得到相关文件位置后进行清理
```

隐藏进程，会出现proc下面大小异常：

```
cat /pro/$$/mountinfo 
cat /proc/mounts 
mount
以上三个等价，可靠性不同。

$$ 表示当前shell进程的进程ID

#处理后可以瞒过直接mount
cp /etc/mtab .
mount –bind /bin /proc/[pid]
mv . /etc/mtab

#这样可以进行隐藏
mount –bind /tmp/empty /proc/2694
```


### Windows工具检查

* Winpmem内存转储【配合Volatility进行内存取证】
* 冰刃
* process explorer
* Filemon：查看进程和文件对应
* Regmon：查看进程和注册表对应
* PC Hunter(xuetr) 可查看硬盘上隐藏的文件

#### 手动杀进程

非常古老的pskill
```
pskill $PID
```

可以用它结束一些常见的杀毒软件进程，使用方法如下：
```
c:\> ProcessHacker.exe -c -ctype process -cobject $PID-Number -caction terminate
```

也是暂停进程的运行，如下：
```
c:\> ProcessHacker.exe -c -ctype process -cobject $PID-Number –caction suspend
```

wmic：
```
wmic process where caption="qq.exe" delete
wmic process where handle=10000 delete
```

TASKKILL：
```
TASKKILL /S system /F /IM notepad.exe /T
TASKKILL /PID 1230 /PID 1241 /PID 1253 /T
TASKKILL /F /IM QQ.exe
```

ntsd：
```
ntsd -c q -p pid
```



### Linux下工具检查：

- 暴力枚举进程 
```
通过PsLookupProcessByProcessId获得EPROCESS
通过ZwQuerySystemInformation
通过进程活动连来枚举
```
- hkrookit

- rkhunter

    具体用法，请查看：[《信安应急响应手册
》](/pentest/2018/06/25/security-emergency/)。

- 偶然发现的小工具

可检测通过Hook vfs 函数来隐藏的进程。
```
https://security.tencent.com/index.php/opensource/detail/16
```

- Volatility：
```
pslist – 通过检查双链表来检测进程
pstree – 使用了相同技术，只是显示有小小的差别
psscan – 在内存中扫描_POOL_HEADER结构（内存页池）以识别相关进程
psxview – 几种技术的组合：
pslist：如上所述
psscan：如上所述
thrdproc：线程扫描，检索调度程序使用的_KTHREAD列表（不能在不中断进程执行的情况下修改它），然后搜索相关的_EPROCESS对象。
pspcid
csrss：csrss.exe进程保留着可以在其内存中检索到的进程的独立列表。
session
deskthrd
```

### 尾声

需要强调的是，攻防都是相对的，技术是在进步的，工具需要配合手工才能变成神器。

以后在实践中遇到新东西，或者在其他资料站看到实用的内容，后面会继续给大家更新。