## man命令
man命令的可用帮助文档分类有

|代码|代表内容|
|--|--|
| 1 | 普通的命令 |
| 2 | 内核调用命令的函数与工具 |
| 3 | 常见的函数与函数库 |
| 4 | 设备文件的说明 |
| 5 | 配置文件 |
| 6 | 游戏 |
| 7 | 惯例与协议 |
| 8 | 管理员可用的命令 |
|9|内核相关文件|

帮助文档的目录结构
|结构名称|代表意义|
|--|--|
|NAME|命令名称及功能简要说明|
|SYNOPSYS| 用法说明，包括可用的选项|
|DESCRIPTION|命令功能的详细说明，可能包括每一个选项的意义|
|OPTIONS|说明每一项的意义|
|EXAMPLES|使用示例|
|OVERVIEW|概述|
|SEE ALSO|另外参照|

man命令的操作按键
/关键词  从上至下搜索
?关键词  从下至上搜索
n 定位到下一个搜索到的关键词
N 上一个

## 常用系统工作命令
### echo
用于在终端显示字符串或变量
echo $SHELL
/bin/bash
echo $HOSTNAME
localhost.localdomain
### date  
date "+%Y-%m-%d %H:%M:%S"

### DNS 配置相关
- 1. DNS服务器地址配制 /etc/resolv.conf
- 2. host 主机名配制，/etc/hosts
- 3. 配置文件添加DNS服务器地址。/etc/sysconfig/network-scripts/ifcfg-eth0 
生效顺序是：
hosts文件指定 >  网卡设置配置 > DNS服务器地址

### journalctl
-k  查看内核日志
-b  查看系统本次启动的日志
-u  查看指定服务的日志
-n  指定日志条数
-f  追踪日志
--disk-usage  查看当前日志占用磁盘的空间的总大小
### iptables
==先放放==

### docker
==先放放==

### ifconfig 
```bash
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.145  netmask 255.255.0.0  broadcast 10.0.255.255
        inet6 fe80::5054:ff:fe72:5e24  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:72:5e:24  txqueuelen 1000  (Ethernet)
        RX packets 269780  bytes 13331234 (12.7 MiB)
        RX errors 0  dropped 311  overruns 0  frame 0
        TX packets 7434  bytes 5455285 (5.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
- ether和HWaddr是一个意思，表示MAC地址。
- inet用来表示网卡的IP地址
- inet6 addr：是IPv6的版本的IP，我们没有使用，所以略过。

- RX：那一行代表的是网络由启动到目前为止的数据包接收情况，packets代表数据包数、errors代表数据包发生错误的数量、dropped代表数据包由于有问题而遭丢弃的数量等。

- TX：与RX相反，为网络由启动到目前为止的传送情况。

- collisions：代表数据包碰撞的情况，如果发生太多次，表示你的网络状况不太好。

- txqueuelen：代表用来传输数据的缓冲区的储存长度。

- RX Bytes、TX Bytes：总传送、接收的字节总量。

- Interrupt、Memory：网卡硬件的数据，IRQ岔断与内存地址。

## 系统状态检测命令
### uname
查看系统内核版本信息:
uname -a
查看系统详细版本信息：
cat /etc/redhat-release

### uptime
查看系统负载情况
输出内容分别为：系统当前时间、系统已运行时间、当前在线用户以及平均负载值。而平均负载分为最近1分钟、5分钟、15分钟的系统负载情况，负载值越低越好(小于1是正常)。
常用`watch -n 1 uptime`每秒刷新一次当前系统负载情况。
### watch
watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行。在watch命令可以实时全屏监控当前命令执行的动态变化结果。watch命令的常用参数有`“-n”、“-d”、“-t”`分别表示“`时隔多少秒刷新”`、“`高亮显示动态变化`”、“`关闭命令顶部的时间间隔,命令显示”`
退出watch：Ctrl + C

### free 
显示当前系统中内存的使用情况
free -h 人性化展示
### who
查看当前登入主机的用户名

### last
该命令用来列出目前与过去登录系统的用户相关信息。

### history 
历史命令会被保存到用户家目录中的”.bash_history“文件中。Linux系统中以点()开头的文件均代表隐藏文件，一般会是系统文件。

### top
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。top命令可以动态显示进程的资源使用情况，即可以通过用户按键来不断刷新当前状态。如果在前台执行该命令，它将独占前台，直到用户终止该程序为止。比较准确的说，top命令提供了实时的对系统处理器的状态监视，它将显示系统中CPU最“敏感”的任务列表，该命令可以按CPU使用、内存使用和执行时间来对任务进行排序，而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定。 
选项	功能
```
-d	指定每两次屏幕信息刷新之间的时间间隔，如希望每秒刷新一次，则使用：top -d 1
-p	通过指定PID来仅仅监控某个进程的状态
-S	指定累计模式
-s	使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险
-i	使top不显示任何闲置或者僵死的进程
-c	显示整个命令行而不只是显示命令名
-H  会列出所有Linux线程。在top运行时，你也可以通过按“H”键将线程查看模式切换为开或关。
```
top 运行中可以通过 top 的内部命令对进程的显示方式进行控制。内部命令如下：
```
s – 改变画面更新频率
l – 关闭或开启第一部分第一行 top 信息的表示
t – 关闭或开启第一部分第二行 Tasks 和第三行 Cpus 信息的表示
m – 关闭或开启第一部分第四行 Mem 和 第五行 Swap 信息的表示
N – 以 PID 的大小的顺序排列表示进程列表
P – 以 CPU 占用率大小的顺序排列进程列表
M – 以内存占用率大小的顺序排列进程列表
h – 显示帮助
n – 设置在进程列表所显示进程的数量
q – 退出 top
s – 改变画面更新周期
```
top中一些字段的含义：
```
VIRT：virtual memory usage 虚拟内存
1、进程“需要的”虚拟内存大小，包括进程使用的库、代码、数据等
2、假如进程申请100m的内存，但实际只使用了10m，那么它会增长100m，而不是实际的使用量

RES：resident memory usage 常驻内存
1、进程当前使用的内存大小，但不包括swap out
2、包含其他进程的共享
3、如果申请100m的内存，实际使用10m，它只增长10m，与VIRT相反
4、关于库占用内存的情况，它只统计加载的库文件所占内存大小

SHR：shared memory 共享内存
1、除了自身进程的共享内存，也包括其他进程的共享内存
2、虽然进程只使用了几个共享库的函数，但它包含了整个共享库的大小
3、计算某个进程所占的物理内存大小公式：RES – SHR
4、swap out后，它将会降下来

DATA
1、数据占用的内存。如果top没有显示，按f键可以显示出来。
2、真正的该程序要求的数据空间，是真正在运行中要使用的。

S 进程状态。（D=不可中断的睡眠状态，R=运行，S=睡眠，T=跟踪/停止，Z=僵尸进程）
PR 优先级
NI nice值。负值表示高优先级，正值表示低优先级
TIME+ 进程使用的CPU时间总计，单位1/100秒
COMMAND 命令名/命令行
```

要让top输出某个特定进程`<pid>`并检查该进程内运行的线程状况：
`top -H -p <pid>`

### systemctl
systemctl命令是系统服务管理器指令，主要负责控制systemd系统和服务管理器，它实际上将 service 和 chkconfig 这两个命令组合到一起。
```
start	启动服务
stop	停止服务
restart	重启服务
enable	使某服务开机自启
disable	关闭某服务开机自启
status	查看服务状态
list-units -–type=service 列举所有已启动服务
```

### ps
最常用的监控进程的命令，通过此命令可以查看系统中所有运行进程的详细信息。
```
a：显示一个终端的所有进程，除会话引线外；
u：显示进程的归属用户及内存的使用情况；
x：显示没有控制终端的进程；
-l：长格式显示更加详细的信息；
-e：显示所有进程

"ps aux" 可以查看系统中所有的进程；
"ps -le" 可以查看系统中所有的进程，而且还能看到进程的父进程的 PID 和进程优先级；
"ps -l" 只能看到当前 Shell 产生的进程；
```

### docker
```
1、创建容器
# docker run -tid centos /bin/bash 

2、查看容器
# docker ps -a

3、删除容器
[root@node01.linux.com ~]# docker rm e9ae

[root@node01.linux.com ~]# docker stop 64c1
[root@node01.linux.com ~]# docker rm 64c1

[root@node01.linux.com ~]# docker rm -f b326


4、查看容器的详细信息
[root@node01.linux.com ~]# docker inspect c18d
```

## 工作目录切换
### cd 
cd命令用于切换工作路径,
cd - 切换到上一次的工作
### ls
-d 仅看目录本身
-h 易读的文件容量
-l 文件详细信息

## 文本编辑命令
### vi编辑器
vi abc   打开文件abc
yy         复制当前行
p           粘贴复制的内容
End        光标移到行尾
:set nu    显示行号
:set nonu 不显示行号

一般模式
下翻1页\[Page Down] 
上翻1页\[Page Up]
行首\[Home]
行尾\[End]
向左箭头键(←)  光标向左移动一个字符 
向下箭头键(↓)  光标向下移动一个字符 
向上箭头键(↑)  光标向上移动一个字符 
向右箭头键(→)  光标向右移动一个字符
第1行gg
最后1行G
第n行nG
n\[移动键]，相当于按n次\[移动键]
dd    删除当前行
ndd  执行n次dd，即删除当前行在内向下n行
yy    复制当前行
nyy  执行n次yy，即复制当前行向下n行
p      粘贴在当前位置之后
u       撤销
\[Ctrl]+r  重做

命令行模式
/word 
从当前光标向下寻找一个名称为 word 的字符串。
?word  
从当前光标向上寻找一个字符串名称为 word 的字符串。 
n  重复前一个查找动作，继续向下/上查找。
N 反向进行前一个查找动作，继续向上/下查找。

:n1,n2s/word1/word2/g
:n1,n2s/word1/word2/gc

n1是开始查找替换的行号。
n2是结束行的行号。
word1是被查找替换的字符串。
word2是替换后的新字符串。
加上c代表每一处替换都要用户确认。

将n1行和n2行之间的所有word1都替换为word2。






### cat
用于查看纯文本文件（较短的）
-n 显示行号
-b 显示行号（不包含空行）
-A 显示出“不可见”的符号，空格、Tab

### more 
查看纯文本文件（较长的）
-数字 预先显示的行数
-d 显示提示语句和报错信息
### head 
查看纯文本的前N行
head -n 20 a.txt
正常输出，但不显示最后3行
head -n -3 t3.c

### tail
查看纯文本的最后N行
tail -n 10 显示后面的10行
-f 持续刷新显示的内容
### od
od命令用于查看特殊格式的文件
-t a 默认字符
-t c ASCII字符
-t o 八进制
-t d 十进制
-t x 十六进制
-t f 浮点数

### tr
用于转换文本文件
将tr.txt文件的内容转换成大写
cat tr.txt | tr\[a-z]\[A-Z]
### wc 
用于统计指定文本的行数、字数、字节数。

### cut 
cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。
如果不指定 File 参数，cut 命令将读取标准输入。必须指定 -b、-c 或 -f 标志之一。
```
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了-n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的范围之内，该字符将被写出；否则，该字符将被排除
```
获取当前系统中所有用户名称;
参数作用:-d以”:”来做分隔符，-f参数代表只看第一列的内容。
 `cut -d: -f1 /etc/passwd`

### diff
diff命令用于比较多个文本文件的差异，格式为:`diff [参数]文件`。


## 文件目录管理命令
### touch 
touch命令用于创建空白文件与修改文件时间，格式为:`touch [选项][文件]`。
对于在Linux中的文件有三种时间:
- 更改时间(mtime):内容修改时间(不包括权限的)
- 更改权限(ctime):更改权限与属性的时间
- 读取时间(atime):读取文件内容的时间
|参数|作用|
|--|--|
|-a|近修改“访问时间”(atime)|
|-m|近修改“更改时间”(mtime)|
|-d|同时修改atime与mtime|
|-t|要修改成的时间[YYMMDDhhmm]|

### mkdir
创建空白文件夹

### cp
复制文件或目录
`cp [options] source dest`
或
`cp [options] source... directory`
参数说明：

```
-a：此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容。其作用等于dpR参数组合。
-d：复制时保留链接。这里所说的链接相当于 Windows 系统中的快捷方式。
-f：覆盖已经存在的目标文件而不给出提示。
-i：与 -f 选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答 y 时目标文件将被覆盖。
-p：除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。
-r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。
-l：不复制文件，只是生成链接文件。
```

### mv
mv命令用于移动文件或改名，格式为:`“mv [选项] 文件名 [目标路径|目标文件名]`。

### rm
rm命令用于删除文件或目录，格式为: `rm [选项] 文件`。
删除普通文件并提示确认信息:`rm 文件名`
删除普通文件或目录文件，不提示: `rm -rf 文件或目录名`
|参数|作用|
|--|--|
|-f|忽略警告信息|
|-i|删除前先询问|
|-r|删除文件夹|

### dd
dd命令用于指定大小的拷贝的文件或指定转换文件，格式为:`dd [参数]`。
|参数|作用|
|--|--|
|if|输入文件名称|
|of|输出文件名称|
|bs|设置要拷贝“块”的大小。|
|count|设置要拷贝“块”的个数。|
|conv=ucase|将字母从小写转换为大写。|
|conv=lase|将字母从小写转换为小写。|



## 打包压缩文件命令
tar命令用于对文件打包压缩或解压，格式为: `tar [选项][文件]`。

打包并压缩文件:`tar -czvf 压缩包名.tar.gz文件名`
解压并展开压缩包: `tar -xzvf 压缩包名.tar.gz`
|参数|作用|
|--|--|
|-c|创建压缩文件|
|-x|解开压缩文件|
|-t|查看压缩包内有那些文件|
|-z|用Gzip压缩或解压|
|-j|用bzip2压缩或解压|
|-v|显示压缩或解压的过程|
|-f|目标文件名|
|-p|保留原始的权限与属性|
|-P|使用绝对路径来压缩|

## 文件查询搜索命令
grep命令用于对文本进行搜索，格式为: `grep[选项][文件]`
|参数|作用|
|--|--|
|-b|将可执行文件(binary)当作文本文件(text)来搜索|
|-c|仅显示找到的次数|
|-i|忽略大小写|
|-n|显示行号|
|-v|反向选择——仅列出没有“关键词”的行。|

which       查看可执行文件的位置   
whereis    查看文件的位置   
locate       配 合数据库查看文件位置   
find          实际搜寻硬盘查询文件名称





## 命令行通配符
\* 匹配零个或多个字符
？匹配任意单个字符
\[0-9] 匹配范围内的数字
\[abc]匹配已给出的任意字符

## 目录结构

|目录名称|应该放置的文件内容|
|--|--|
|/boot|开机所需文件——内核,开机菜单及所需配置文件等|
|/dev|任何设备与接口都以文件形式存放在此目录|
|/etc|配置文件|
|/home|用户主目录|
|/bin|单用户维护模式下还能够被操作的命令|
|/lib|开机时用到的函数库及/bin与/sbin下面命令要调用的函数|
|/sbin|开机过程中需要的|
|/media|一般挂载或删除的设备|
|/opt|放置第三方的软件|
|/root|系统管理员的主文件夹|
|/srv|一些网络服务的数据目录|
|/tmp|任何人均可使用的“共享”临时目录|
|/proc|虚拟文件系统，例如系统内核，进程，外部设备及网络状态等|
|/usr/local|用户自行安装的软件|
|/usr/sbin|非系统开机时需要的软件/命令/脚本|
|/usr/share|帮助与说明文件，也可放置共享文件。|
|/var|主要存放经常变化的文件，如日志。|
|/lost+found|当文件系统发生错误时，将一些丢失的文件片段存放在这里|



## cp

## 未知命令
编译好fpc、npb后手动执行后service启动不成功：
```bash
sudo -u machloop /opt/components/fpc-engine/fpc


Path :/opt/machloop/log/fpc-engine/ does not have write permission!
log file' init failed

chown -R machloop:machloop /opt/machloop/log/fpc-engine/
```

```bash
(gdb) bt
#0  strategy_judgment (pSess=<optimized out>) at ./npb_process.c:1053
#1  0x000000000049918a in npb_proto_proc (pSess=pSess@entry=0x17f6780, pMbufpktmsg=pMbufpktmsg@entry=0x144aedd80, tmp=tmp@entry=0x25c335480) at ./npb_process.c:1456
#2  0x000000000049934c in npb_pkt_handle (pMbufpktmsg=0x144aedd80, tmp=0x25c335480) at ./npb_process.c:1663
#3  0x0000000000499618 in flow_process_packet (ullRdtsc=<optimized out>, mi=0x25c335480) at ./npb_process.c:1688
#4  npb_pkt_parse (j=0, tmp=0x25c335480) at ./npb_process.c:1796
#5  mp_cli_loop (arg=<optimized out>) at ./npb_process.c:1857
#6  0x0000000000484f0b in eal_thread_loop.cold ()
#7  0x00007ffff4fb5ea5 in start_thread () from /lib64/libpthread.so.0
#8  0x00007ffff4cde9fd in clone () from /lib64/libc.so.6
(gdb) f 1
#1  0x000000000049918a in npb_proto_proc (pSess=pSess@entry=0x17f6780, pMbufpktmsg=pMbufpktmsg@entry=0x144aedd80, tmp=tmp@entry=0x25c335480) at ./npb_process.c:1456
1456                if (strategy_judgment(pSess) == 0)

```

```bash
$3 = <optimized out>
(gdb) bt
#0  strategy_judgment (pSess=<optimized out>) at ./npb_process.c:1053
Segmentation fault
[root@localhost mp_npb]# scl enable
Need at least 3 arguments.
Run scl --help to get help.
[root@localhost mp_npb]# scl enable devtoolset-10 bash
[root@localhost mp_npb]# gdb -p $(pidof mp_npb)
GNU gdb (GDB) Red Hat Enterprise Linux 9.2-10.el7

```

## python2 3切换
删除/usr/bin目录下的python link文件
```
sudo rm -rf /usr/bin/python
```
删除后再建立新的链接关系：
```
sudo ln -s /usr/bin/python3  /usr/bin/python
```
恢复：
```
sudo rm -rf /usr/bin/python
sudo ln -s /usr/bin/python2.7 /usr/bin/python
```

## crontab
执行下面命令即可编辑当前用户的定时任务：
crontab -e
```bash
*/1 * * * * /bin/python3 /tmp/share/Work/code/shell_code/1.py

```
crontab -l 可查看到已保存的定时任务：

查看安装了那些crontab
```
vi install.sh  --找到install-post.sh
vi install-post.sh   --找到ntp  
vi infrastructure/ntp/ntp-install.sh
```

## samba
```bash
yum install -y samba
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
vi /etc/samba/smb.conf
删除home、print
[share]
        comment =samba  share
        path = /tmp/share
        writable = yes
        browseable = yes
        guest ok = yes

useradd shareuser
smbpasswd -a shareuser


systemctl restart smb
```

samba不允许一个用户使用一个以上用户名与服务器或共享资源的多重连接
```
1
(https://blog.csdn.net/qq_41076734/article/details/115965075)

2
(https://blog.csdn.net/qq_31041847/article/details/109772889)
2.windows中的控制面板->程序和功能->启用或关闭windows功能中勾选SMB 1.0/CIFS文件共享支持 。
```