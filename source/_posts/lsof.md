---
title: linux删除文件后没有释放空间
date: 2018-01-17 17:33:35
tags:
---
linux删除文件后没有释放空间

 

今天发现一台服务器的home空间满了，于是要清空无用的文件，当我删除文件后，发现可用空间没有变化

 

os：centos4.7


现象：

 

发现当前磁盘空间使用情况：

 

[root@ticketb ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             981M  203M  729M  22% /
none                   16G     0   16G   0% /dev/shm
/dev/sda9             2.9G   37M  2.7G   2% /tmp
/dev/sda7             4.9G  1.9G  2.7G  42% /usr
/dev/sda8             2.9G  145M  2.6G   6% /var
/dev/mapper/vghome-lvhome
                       20G   19G   11M 100% /home
/dev/mapper/vgoradata-lvoradata
                      144G   48G   90G  35% /u01/oradata
/dev/mapper/vgbackup-lvbackup
                      193G  7.8G  175G   5% /u01/backup


通过下面的命令找到无用的文件，然后删除


[root@ticketb ~]# find /home/oracle/admin/dbticb/udump/ -name "dbticb_*.trc" -mtime +50 | xargs rm -rf

 

然后在查看磁盘空间使用情况，发现没有/home空间没有变化

 

[root@ticketb ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             981M  203M  729M  22% /
none                   16G     0   16G   0% /dev/shm
/dev/sda9             2.9G   37M  2.7G   2% /tmp
/dev/sda7             4.9G  1.9G  2.7G  42% /usr
/dev/sda8             2.9G  145M  2.6G   6% /var
/dev/mapper/vghome-lvhome
                       20G   19G   11M 100% /home
/dev/mapper/vgoradata-lvoradata
                      144G   48G   90G  35% /u01/oradata
/dev/mapper/vgbackup-lvbackup
                      193G  7.8G  175G   5% /u01/backup

 

这个郁闷啊，明明删除文件了，怎么空间没有被释放啊，rm命令应该是直接删除啊，在查看下/home下还有什么占用空间

 

[root@ticketb ~]# du -h --max-depth=1  /home
16K     /home/lost+found
2.6G    /home/oracle
2.6G    /home

 

可这里显示空间已经释放了啊，于是google下，

 

未释放磁盘空间原因：

 

在Linux或者Unix系统中，通过rm或者文件管理器删除文件将会从文件系统的目录结构上解除链接(unlink).然而如果文件是被
打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。而我删除的是oracle的告警log文件
删除的时候文件应该正在被使用

 

解决方法

 

首先获得一个已经被删除但是仍然被应用程序占用的文件列表，如下所示：


[root@ticketb ~]# lsof |grep deleted
oracle    12639  oracle    5w      REG              253,0         648     215907 /home/oracle/admin/dbticb/udump/dbticb_ora_12637.trc (deleted)
oracle    12639  oracle    6w      REG              253,0 16749822091     215748 /home/oracle/admin/dbticb/bdump/alert_dbticb.log (deleted)
oracle    12639  oracle    7u      REG              253,0           0      36282 /home/oracle/oracle/product/10.2.0/db_1/dbs/lkinstdbticb (deleted)
oracle    12639  oracle    8w      REG              253,0 16749822091     215748 /home/oracle/admin/dbticb/bdump/alert_dbticb.log (deleted)
oracle    12641  oracle    5w      REG              253,0         648     215907 /home/oracle/admin/dbticb/udump/dbticb_ora_12637.trc (deleted)
oracle    12641  oracle    6w      REG              253,0 16749822091     215748 /home/oracle/admin/dbticb/bdump/alert_dbticb.log (deleted)
。
。
。
oracle    23492  oracle    6w      REG              253,0 16749822091     215748 /home/oracle/admin/dbticb/bdump/alert_dbticb.log (deleted)
oracle    23492  oracle    7u      REG              253,0           0      36282 /home/oracle/oracle/product/10.2.0/db_1/dbs/lkinstdbticb (deleted)
oracle    23492  oracle    8w      REG              253,0 16749822091     215748 /home/oracle/admin/dbticb/bdump/alert_dbticb.log (deleted)
oracle    23494  oracle   10u      REG              253,0           0      36307 /home/oracle/oracle/product/10.2.0/db_1/dbs/lkinstrmandb (deleted)

 


从输出结果可以看到/home/oracle/admin/dbticb/bdump/alert_dbticb.log还被使用，未被释放空间

 

如何让进程释放呢？

 

一种方法是kill掉相应的进程，或者停掉使用这个文件的应用，让os自动回收磁盘空间

我这个环境有很多进程在使用的这个文件，停掉进程有点麻烦，再有就是风险很大

 

当linux打开一个文件的时候,Linux内核会为每一个进程在/proc/ 『/proc/nnnn/fd/目录（nnnn为pid）』建立一个以其pid
为名的目录用来保存进程的相关信息，而其子目录fd保存的是该进程打开的所有文件的fd（fd：file descriptor）。

kill进程是通过截断proc文件系统中的文件可以强制要求系统回收分配给正在使用的的文件。
这是一项高级技术，仅到管理员确定不会对运行中的进程造成影响时使用。应用程序对这种方
式支持的并不好，当一个正在使用的文件被截断可能会引发不可预知的问题

 

所以我还是采用停应用来解决

 

restart oracle数据库，发现/home/oracle/admin/dbticb/bdump/alert_dbticb.log对应的空间被释放

 

在查看磁盘空间的使用情况，发现空间已经回收了


[root@ticketb ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             981M  203M  729M  22% /
none                   16G     0   16G   0% /dev/shm
/dev/sda9             2.9G   37M  2.7G   2% /tmp
/dev/sda7             4.9G  1.9G  2.7G  42% /usr
/dev/sda8             2.9G  145M  2.6G   6% /var
/dev/mapper/vghome-lvhome
                       20G  2.6G   16G  15% /home
/dev/mapper/vgoradata-lvoradata
                      144G   48G   90G  35% /u01/oradata
/dev/mapper/vgbackup-lvbackup
                      193G  7.8G  175G   5% /u01/backup


ok，问题解决，然后做下收尾工作即可

 

-------------------------------------------------------------------------------------------------

 

学习下lsof命令

 

lsof全名list opened files，也就是列举系统中已经被打开的文件。我们都知道，linux环境中，任何事物都是文件，
设备是文件，目录是文件，甚至sockets也是文件。所以，用好lsof命令，对日常的linux管理非常有帮助。

 

lsof是linux最常用的命令之一，通常的输出格式为：

 

引用
COMMAND     PID   USER   FD      TYPE     DEVICE     SIZE       NODE NAME

 

常见包括如下几个字段：更多的可见manual。

1、COMMAND
默认以9个字符长度显示的命令名称。可使用+c参数指定显示的宽度，若+c后跟的参数为零，则显示命令的全名
2、PID：进程的ID号
3、PPID
父进程的IP号，默认不显示，当使用-R参数可打开。
4、PGID
进程组的ID编号，默认也不会显示，当使用-g参数时可打开。
5、USER
命令的执行UID或系统中登陆的用户名称。默认显示为用户名，当使用-l参数时，可显示UID。
6、FD
是文件的File Descriptor number，或者如下的内容：
（这里很难翻译对应的意思，保留英文）

 

引用
cwd  current working directory;
Lnn  library references (AIX);
jld  jail directory (FreeBSD);
ltx  shared library text (code and data);
Mxx  hex memory-mapped type number xx.
m86  DOS Merge mapped file;
mem  memory-mapped file;
mmap memory-mapped device;
pd   parent directory;
rtd  root directory;
tr   kernel trace file (OpenBSD);
txt  program text (code and data);
v86  VP/ix mapped file;

 

 

文件的File Descriptor number显示模式有：

 

引用
r for read access;
w for write access;
u for read and write access;
N for a Solaris NFS lock of unknown type;
r for read lock on part of the file;
R for a read lock on the entire file;
w for a write lock on part of the file;
W for a write lock on the entire file;
u for a read and write lock of any length;
U for a lock of unknown type;
x for an SCO OpenServer Xenix lock on part  of the file;
X  for an SCO OpenServer Xenix lock on the entire file;
space if there is no lock.

 

 

7、TYPE

引用
IPv4 IPv4的包；
IPv6 使用IPv6格式的包，即使地址是IPv4的，也会显示为IPv6，而映射到IPv6的地址；
DIR 目录
LINK 链接文件

详情请看manual中更多的注释。

 

8、DEVICE
使用character special、block special表示的设备号
9、SIZE
文件的大小，如果不能用大小表示的，会留空。使用-s参数控制。
10、NODE
本地文件的node码，或者协议，如TCP等
11、NAME
挂载点和文件的全路径（链接会被解析为实际路径），或者连接双方的地址和端口、状态等

 

常用示例：

 

1.显示开启文件/home/oracle/10.2.0/db_1/bin/tnslsnr的进程

 

[root@svr-db-test ~]# lsof /home/oracle/10.2.0/db_1/bin/tnslsnr
COMMAND  PID   USER  FD   TYPE DEVICE   SIZE     NODE NAME
tnslsnr 3520 oracle txt    REG  253,5 431062 11408866 /home/oracle/10.2.0/db_1/bin/tnslsnr

 

2.知道22端口现在运行什么程序

 

[root@svr-db-test ~]# lsof -i :22
COMMAND  PID USER   FD   TYPE  DEVICE SIZE NODE NAME
sshd    3101 root    3u  IPv6    8670       TCP *:ssh (LISTEN)
sshd    4545 root    3u  IPv6 4237972       TCP 203.aibo.com:ssh->win-avbmq9e8ka7.gdgg.local:nsjtp-ctrl (ESTABLISHED)

 

3.显示init进程现在打开的文件

 

[root@svr-db-test ~]# lsof -c init
COMMAND PID USER   FD   TYPE DEVICE    SIZE   NODE NAME
init      1 root  cwd    DIR  253,0    4096      2 /
init      1 root  rtd    DIR  253,0    4096      2 /
init      1 root  txt    REG  253,0   43496 524446 /sbin/init
init      1 root  mem    REG  253,0  130448 917826 /lib64/ld-2.5.so
init      1 root  mem    REG  253,0 1678480 917827 /lib64/libc-2.5.so
init      1 root  mem    REG  253,0   23520 917686 /lib64/libdl-2.5.so
init      1 root  mem    REG  253,0  247528 917844 /lib64/libsepol.so.1
init      1 root  mem    REG  253,0   95480 917845 /lib64/libselinux.so.1
init      1 root   10u  FIFO   0,16           2311 /dev/initctl

 

4. 看进程号为1的进程打开了哪些文件

 

[root@svr-db-test ~]# lsof -p 1
COMMAND PID USER   FD   TYPE DEVICE    SIZE   NODE NAME
init      1 root  cwd    DIR  253,0    4096      2 /
init      1 root  rtd    DIR  253,0    4096      2 /
init      1 root  txt    REG  253,0   43496 524446 /sbin/init
init      1 root  mem    REG  253,0  130448 917826 /lib64/ld-2.5.so
init      1 root  mem    REG  253,0 1678480 917827 /lib64/libc-2.5.so
init      1 root  mem    REG  253,0   23520 917686 /lib64/libdl-2.5.so
init      1 root  mem    REG  253,0  247528 917844 /lib64/libsepol.so.1
init      1 root  mem    REG  253,0   95480 917845 /lib64/libselinux.so.1
init      1 root   10u  FIFO   0,16           2311 /dev/initctl

 

5. 显示归属3520的进程情况

 

[root@svr-db-test ~]# lsof -g 3520
COMMAND  PID PGID   USER   FD   TYPE             DEVICE      SIZE     NODE NAME
tnslsnr 3520 3520 oracle  cwd    DIR              253,5      4096 11059201 /home/oracle
tnslsnr 3520 3520 oracle  rtd    DIR              253,0      4096        2 /
tnslsnr 3520 3520 oracle  txt    REG              253,5    431062 11408866 /home/oracle/10.2.0/db_1/bin/tnslsnr
tnslsnr 3520 3520 oracle  mem    REG              253,0    130448   917826 /lib64/ld-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,0   1678480   917827 /lib64/libc-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,0     23520   917686 /lib64/libdl-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,0    615136   917834 /lib64/libm-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,0    141208   917829 /lib64/libpthread-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,0    109824   917839 /lib64/libnsl-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,5  20706622 11405436 /home/oracle/10.2.0/db_1/lib/libclntsh.so.10.1
tnslsnr 3520 3520 oracle  mem    REG              253,5   3803097 11410641 /home/oracle/10.2.0/db_1/lib/libnnz10.so
tnslsnr 3520 3520 oracle  mem    REG              253,5     83493 11407251 /home/oracle/10.2.0/db_1/lib/libons.so
tnslsnr 3520 3520 oracle  mem    REG              253,0     53880   917532 /lib64/libnss_files-2.5.so
tnslsnr 3520 3520 oracle  mem    REG              253,5      8545 11407615 /home/oracle/10.2.0/db_1/lib/libskgxn2.so
tnslsnr 3520 3520 oracle  mem    REG              253,5    513705 11410332 /home/oracle/10.2.0/db_1/lib/libocrutl10.so
tnslsnr 3520 3520 oracle  mem    REG              253,5    636161 11410330 /home/oracle/10.2.0/db_1/lib/libocr10.so
tnslsnr 3520 3520 oracle  mem    REG              253,5    657825 11410331 /home/oracle/10.2.0/db_1/lib/libocrb10.so
tnslsnr 3520 3520 oracle  mem    REG              253,5   1745769 11410365 /home/oracle/10.2.0/db_1/lib/libhasgen10.so
tnslsnr 3520 3520 oracle  mem    REG              253,5     61985 11410366 /home/oracle/10.2.0/db_1/lib/libclsra10.so
tnslsnr 3520 3520 oracle    0u   CHR                1,3               2553 /dev/null
tnslsnr 3520 3520 oracle    1u   CHR                1,3               2553 /dev/null
tnslsnr 3520 3520 oracle    2u   CHR                1,3               2553 /dev/null
tnslsnr 3520 3520 oracle    3w   REG              253,5 318853012 11633459 /home/oracle/10.2.0/db_1/network/log/listener.log
tnslsnr 3520 3520 oracle    4r  FIFO                0,6              15661 pipe
tnslsnr 3520 3520 oracle    5r   REG              253,5     11776 11410579 /home/oracle/10.2.0/db_1/network/mesg/nlus.msb
tnslsnr 3520 3520 oracle    6r   REG              253,5     46592 11407160 /home/oracle/10.2.0/db_1/network/mesg/tnsus.msb
tnslsnr 3520 3520 oracle    7w  FIFO                0,6              15662 pipe
tnslsnr 3520 3520 oracle    8u  IPv4              15665                TCP 203.aibo.com:ncube-lm (LISTEN)
tnslsnr 3520 3520 oracle    9u  unix 0xffff81021b7d6980              15666 /var/tmp/.oracle/s#3520.1
tnslsnr 3520 3520 oracle   10u  unix 0xffff81021b7d66c0              15668 /var/tmp/.oracle/s#3520.2


6.依照文件夹/home/oracle来搜寻，但不会打开子目录，用来显示目录下被进程开启的文件

 

[root@svr-db-test ~]# lsof +d /home/oracle
COMMAND  PID   USER   FD   TYPE DEVICE SIZE     NODE NAME
tnslsnr 3520 oracle  cwd    DIR  253,5 4096 11059201 /home/oracle

 

7. 打开/home/oracle文件夹以及其子目录搜寻，用来显示目录下被进程开启的文件

 

[root@svr-db-test ~]# lsof +D /home/oracle


显示内容太多了，不显示了


8. lsof -i 用以显示符合条件的进程情况

 

语法: lsof -i[46] [protocol][@hostname|hostaddr][:service|port]

 

46 --> IPv4 or IPv6

protocol --> TCP or UDP

hostname --> Internet host name

hostaddr --> IPv4位置

service --> /etc/service中的 service name (可以不只一个)

port --> 端口号 (可以不只一个)

 

例：


[root@svr-db-test ~]# lsof -i tcp@192.168.2.245:1521 -n
COMMAND   PID   USER   FD   TYPE  DEVICE SIZE NODE NAME
oracle  15633 oracle   16u  IPv4 4069605       TCP 192.168.2.203:31580->192.168.2.245:ncube-lm (ESTABLISHED)

 

或

 

[root@svr-db-test ~]# lsof -i tcp@192.168.2.245:1521 
COMMAND   PID   USER   FD   TYPE  DEVICE SIZE NODE NAME
oracle  15633 oracle   16u  IPv4 4069605       TCP 203.aibo.com:31580->192.168.2.245:ncube-lm (ESTABLISHED)

 

lsof -n 不将IP转换为hostname，缺省是不加上-n参数

 

9. 显示某用户的已经打开的文件（或该用户执行程序已经打开的文件）

 

[root@svr-db-test ~]# lsof -u oracle
或
[root@svr-db-test ~]# lsof -u 0


10. 仅打印进程，方便shell脚本调用

[root@svr-db-test ~]# lsof -tc sshd
3101
4545

 

 

关注：

 

进程调试命令:truss、strace和ltrace

 

进程无法启动，软件运行速度突然变慢，程序的"SegmentFault"等等都是让每个Unix系统用户头痛的问题，而这些问题都可以通过使用truss、strace和ltrace这三个常用的调试工具来快速诊断软件的"疑难杂症"
