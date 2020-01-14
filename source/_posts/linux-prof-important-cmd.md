---
title: Linux常用命令
tags: [prof]
toc: true
date: 2020-01-13 22:10:15
category: Linux
---

本文将介绍Linux常用几个命令，主要是用来定位线上环境出现问题用，如果现场环境出了问题该怎么去找出问题，如何定位出有问题的代码段。

<!-- more -->

## top

`top`命令可以看到系统的整体运行情况。主要看3个值，cpu，mem和load average，cpu和memory是系统当前的使用百分比，load average是系统负载，有三个值，代表的是系统1分钟，5分钟，15分钟的平均负载值，如果这三个值取平均值大于0.6，说明系统当前负载比较高。

`top`还有一个精简命令`uptime`。

```shell
top - 22:35:45 up 16 min,  2 users,  load average: 0.04, 0.08, 0.06
Tasks:  95 total,   1 running,  94 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882152 total,  1117464 free,   385520 used,   379168 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1345460 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND               
 1238 root      20   0  318988  27388  11504 S  0.7  1.5   0:01.20 docker-containe       
 1697 jenkins   20   0 2589952 192284  18548 S  0.3 10.2   0:14.67 java                  
 2088 root      20   0       0      0      0 S  0.3  0.0   0:00.08 kworker/0:0           
    1 root      20   0  127944   6604   4172 S  0.0  0.4   0:01.15 systemd               
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd              
    4 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H          
    5 root      20   0       0      0      0 S  0.0  0.0   0:00.03 kworker/u2:0          
    6 root      20   0       0      0      0 S  0.0  0.0   0:00.11 ksoftirqd/0           
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0           
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh                
    9 root      20   0       0      0      0 S  0.0  0.0   0:00.52 rcu_sched             
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain         
   11 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 watchdog/0            
   13 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kdevtmpfs             
   14 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 netns                 
   15 root      20   0       0      0      0 S  0.0  0.0   0:00.00 khungtaskd            
   16 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 writeback             
   17 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kintegrityd           
   18 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset                
   19 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset                
   20 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset                
```

这是top的交互命令
```shell
d： 更改刷新频率
f： 增加或减少要显示的列(选中的会变成大写并加*号)
F： 选择排序的列
h： 显示帮助画面
H： 显示线程
i： 忽略闲置和僵死进程
k： 通过给予一个PID和一个signal来终止一个进程。（默认signal为15。在安全模式中此命令被屏蔽）
l:  显示平均负载以及启动时间（即显示影藏第一行）
m： 显示内存信息
M： 根据内存资源使用大小进行排序
N： 按PID由高到低排列
o： 改变列显示的顺序
O： 选择排序的列，与F完全相同
P： 根据CPU资源使用大小进行排序
q： 退出top命令
r： 修改进程的nice值(优先级)。优先级默认为10，正值使优先级降低，反之则提高的优先级
s： 设置刷新频率（默认单位为秒，如有小数则换算成ms）。默认值是5s，输入0值则系统将不断刷新
S： 累计模式（把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+ ）
T： 根据进程使用CPU的累积时间排序
t： 显示进程和CPU状态信息（即显示影藏CPU行）
u： 指定用户进程
W： 将当前设置写入~/.toprc文件，下次启动自动调用toprc文件的设置
<： 向前翻页
>： 向后翻页
?： 显示帮助画面
1(数字1)： 显示每个CPU的详细情况
```

## vmstat

`vmstat` 主要用来查看cpu，但是不仅限于cpu。

`vmstat -n 2 3` 代表每隔2秒采集一次，共采集3次。主要观察procs和cpu的指标。

```shell
root@dev ~]# vmstat -n 2 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1117248   2108 377076    0    0   236    12  121  240  2  1 97  0  0
 1  0      0 1117248   2108 377092    0    0     0     1   85  182  0  0 100  0  0
 0  0      0 1117248   2108 377092    0    0     0     0  209  392  6  8 86  0  0
```

主要观察两个参数的值

 - `proc` 
    - `r` : 运行和等待CPU时间片的进程数，原则上1核的CPU的运行队列不要超过2，否则代表系统压力大。
    - `b` : 等待资源的进程数，

- `cpu`
    - `us` : 用户进程消耗的CPU时间百分比，长期大于50%则需要优化程序。
    - `sy` : 内核进程消耗的CPU时间百分比。

## free

`free`用于查看内存使用情况
```shell
[root@dev ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1838         358        1099           8         380        1329
Swap:          2047           0        2047

```
## df

`df`用于查看磁盘剩余空间

```shel
[root@dev ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 908M     0  908M   0% /dev
tmpfs                    920M     0  920M   0% /dev/shm
tmpfs                    920M  8.6M  911M   1% /run
tmpfs                    920M     0  920M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  3.3G   14G  20% /
/dev/sda1               1014M  191M  824M  19% /boot
tmpfs                    184M     0  184M   0% /run/user/0
```

## iostat

`iostat`用于磁盘I/O性能评估，如果没有这个命令需要先安装`yum install sysstat`

```shell
[root@dev ~]# iostat -xdk 2 3
Linux 3.10.0-1062.4.1.el7.x86_64 (dev)  01/14/2020      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.01    0.28    0.15    13.48     2.25    72.09     0.00    1.36    1.22    1.64   0.82   0.04
dm-0              0.00     0.00    0.22    0.12    12.64     2.20    85.20     0.00    1.67    1.31    2.33   0.92   0.03
dm-1              0.00     0.00    0.00    0.00     0.06     0.00    50.09     0.00    0.42    0.42    0.00   0.33   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-1              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-1              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

```

主要看两个参数

- `await` : I/O请求的平均等待时间，单位毫秒，值越小性能越好。
- `util` : 一秒中有百分之几的时间用于I/O操作。接近100%时，表示磁盘带宽跑满，需要优化程序或者增加磁盘。

## ifstat

`ifstat`用于查看网络I/O，最好先卸载在安装

卸载
```shell
[root@dev ~]# find / -name ifstat
/usr/sbin/ifstat
[root@dev ~]# rm -rf /usr/sbin/ifstat 
```

安装，需要有gcc
```shell
wget http://distfiles.macports.org/ifstat/ifstat-1.1.tar.gz
tar xzvf ifstat-1.1.tar.gz
cd ifstat-1.1
./configure
make && make install
cp istate /usr/sbin/
```

查看各个网卡进出的网络负载情况
```shell
      enp0s3              enp0s8             docker0         br-adc7174eea54  
 KB/s in  KB/s out   KB/s in  KB/s out   KB/s in  KB/s out   KB/s in  KB/s out
    0.00      0.00      0.06      0.28      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.48      0.43      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.12      0.26      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
    0.00      0.00      0.06      0.20      0.00      0.00      0.00      0.00
```

## 如何定位生产CPU占用过高？

1. 先用`top`命令找出占比最高的进程,根据cpu使用百分比排序，`P`或者`shift+p`。

```shell
top - 09:30:12 up 11:11,  5 users,  load average: 0.31, 0.17, 0.10
Tasks: 101 total,   1 running, 100 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.8 us,  3.8 sy,  0.0 ni, 92.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882152 total,   761932 free,   435836 used,   684384 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1265840 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND               
 5262 root      20   0 2472080  40824  12132 S 10.0  2.2   0:11.31 java                  
 3225 root      20   0  160160   6740   4384 S  0.7  0.4   0:00.67 sshd                  
 5311 root      20   0  161888   2176   1540 R  0.3  0.1   0:00.37 top                   
    1 root      20   0  128076   6728   4192 S  0.0  0.4   0:02.36 systemd               
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.01 kthreadd              
    4 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H          
    5 root      20   0       0      0      0 S  0.0  0.0   0:00.49 kworker/u2:0          
    6 root      20   0       0      0      0 S  0.0  0.0   0:01.93 ksoftirqd/0           
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0           
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh                
    9 root      20   0       0      0      0 S  0.0  0.0   0:00.98 rcu_sched             
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain         
   11 root      rt   0       0      0      0 S  0.0  0.0   0:01.43 watch
```

目前可以知道pid为`5262`这个进程占用的cpu百分比最高。

2. 用`ps -ef ` 或者 `jps ` 定位是哪一个后台程序惹的事。

```shell
[root@dev ~]# jps -l | grep 5262
5262 Demo
[root@dev ~]# ps -ef | grep 5262 | grep -v grep
root      5262  3241  6 09:26 pts/2    00:00:37 java Demo
```

3. 定位到具体的线程或者代码

使用命令`ps -mp [pid] -o THREAD,tid,time`

```shell
[root@dev ~]# ps -mp 5262  -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID     TIME
root      6.5   -    - -         -      -     - 00:00:44
root      0.0  19    - futex_    -      -  5262 00:00:00
root      6.3  19    - n_tty_    -      -  5263 00:00:43
root      0.0  19    - futex_    -      -  5264 00:00:00
root      0.0  19    - futex_    -      -  5265 00:00:00
root      0.0  19    - futex_    -      -  5266 00:00:00
root      0.0  19    - futex_    -      -  5267 00:00:00
root      0.0  19    - futex_    -      -  5268 00:00:00
root      0.0  19    - futex_    -      -  5269 00:00:00
root      0.0  19    - futex_    -      -  5270 00:00:00
root      0.1  19    - futex_    -      -  5271 00:00:01
```
可以看到pid为`5262`的进程下tid为`5263`线程占用CPU最多。

4. 将线程id转换为16进制格式，英文小写

因为程序堆栈中是使用16进制来记录的，`5263`的16进制为`0x148f`

5. 打印程序运行堆栈信息

使用命令`jstack [pid] | grep [tid] -A60`,tid为16进制线程id英文小写，打印出前60行信息。

```shell
[root@dev ~]# jstack 5262 | grep 0x148f  -A60
"main" #1 prio=5 os_prio=0 tid=0x00007fed4804b800 nid=0x148f runnable [0x00007fed51b18000]
   java.lang.Thread.State: RUNNABLE
        at java.io.FileOutputStream.writeBytes(Native Method)
        at java.io.FileOutputStream.write(FileOutputStream.java:326)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        - locked <0x00000000ecd68b00> (a java.io.BufferedOutputStream)
        at java.io.PrintStream.write(PrintStream.java:482)
        - locked <0x00000000ecd60c38> (a java.io.PrintStream)
        at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
        at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
        at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
        - locked <0x00000000ecd68b40> (a java.io.OutputStreamWriter)
        at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
        at java.io.PrintStream.write(PrintStream.java:527)
        - eliminated <0x00000000ecd60c38> (a java.io.PrintStream)
        at java.io.PrintStream.print(PrintStream.java:597)
        at java.io.PrintStream.println(PrintStream.java:736)
        - locked <0x00000000ecd60c38> (a java.io.PrintStream)
        at Demo.main(Demo.java:4)

"VM Thread" os_prio=0 tid=0x00007fed480cb800 nid=0x1490 runnable 

"VM Periodic Task Thread" os_prio=0 tid=0x00007fed4811a000 nid=0x1497 waiting on condition 

JNI global references: 5
```

可以看到线程信息，代码在Demo类的main方法，Demo.java这个文件的第4行，然后去看这个方法以及这一行代码相关的逻辑，加以分析应该就可以找出原因了。


