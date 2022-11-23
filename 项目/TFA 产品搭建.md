## 初始化tfa
![image](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220624162330661-1799320418.png)

账号：adm  密码：adm@123


## 功能配置
功能配置-网络接口-接口-编辑-流量接受
功能配置-网络接口-网络-新建-名字任意，接口都勾上，总带宽与物理带宽一致，1000，方向混合，复选框都勾上

## 在宿主机上向网卡打数据包
```
brctl
```
`tcpreplay -i vnet1 dns+icmp.pcapng`

![image](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220624182446877-1318172467.png)

命令具体意思稍后细看
## brctl 
brctl 用来管理以太网桥，在内核中建立、维护、检查网桥配置。一个网桥一般用来连接多个不同的网络，这样这些不同的网络就可以像一个网络那样进行通讯。

## 重新编译tfa报错收集
### cannot find -lrdkafka
```bash
/bin/ld: cannot find -lrdkafka
collect2: error: ld returned 1 exit status
make: *** [fpc] Error 1

```
解决办法：
```
cd 3rd_src/
cd librdkafka-1.0.0/
cat readme.txt
然后按照里面的命令安装。
```

### can't create obj
```bash
Fatal error: can't create obj/src_mp_server/mp_server.obj: No such file or directory
make: *** [obj/src_mp_server/mp_server.obj] Error 1
make: *** Waiting for unfinished jobs....
Assembler messages:
Fatal error: can't create obj/src_flowlog/flowlog_clickhouse.obj: No such file or directory
make: *** [obj/src_flowlog/flowlog_clickhouse.obj] Error 1

```

解决办法：
```bash
[root@localhost fpc-npb]# mkdir -p obj/src_mp_server/
[root@localhost fpc-npb]# mkdir -p obj/src_flowlog/
```

### 运行时报错
```bash
[root@localhost fpc-npb]# ./fpc
-------------------------------------------------------------------------
BACKTRACE: NUMBER OF ADDRESSES IS:4

./fpc(debug_backtrace_init+0x39)[0x6aa349]
./fpc(main+0x4e)[0x52d18e]
/lib64/libc.so.6(__libc_start_main+0xf5)[0x7f9479e59555]
./fpc[0x530674]
-------------------------------------------------------------------------
read default configure init file Error, ret:-1
conf init errro
```

说是代码是最新的，安装包没有更新
解决办法：
```bash
[root@localhost fpc-npb]# cp /opt/machloop/config/fpc-engine/conf.ini /opt/machloop/config/fpc-engine/conf_default.ini
[root@localhost fpc-npb]# ./fpc
```

## 安装tfa带suricata版本
### 安装
```bash
tar -zxvf tfa-npmd-cms-install-182.tar.gz
sh install.sh

###################################################################
Machloop FPC-CMS software will be installed on centos7 or kylinV10.
###################################################################
First at all, please tell us something about how to install Machloop FPC.
1) Do you want initialize database?[Y/N] (Input Y if there's never installed before or you want recreate database, otherwise please input N.) n
2) Please tell us the management device(nic) ip address of Machloop FPC.
10.0.2.145
3) Please tell us the management device(nic) name of Machloop FPC.
ens3
4) would you like to keep the original settings of business ports?[Y/N]y
5) Please tell us the peak flow.1,5,10,20 is supported(the unit is Gbps).
1
6) Would you like to enable metadata?[Y/N]y
7) Would you like to enable suricata?[Y/N]y
Now, everything is ready, please check your input:
-------------------------------------------------------
initialize database:              false
management nic ip   address:      10.0.2.145
management nic name:              ens3
business nic name:
peak_flow:                        1Gbps
meta_data:                        true
suricata:                         true
-------------------------------------------------------
After confirm, please input anything to start installer, or type CTRL+C to stop installer.
###############################################
Thanks for your work, the installer will be on.
###############################################
```

```
服务文件的路径：/usr/lib/systemd/system/

suricata路径：/opt/components/suricata/bin/suricata

查看suricata服务路径： systemctl status scd

/opt/components/suricata/bin/suricata --runmode workers --dpdk -S /opt/components/suricata/suricata.rules
```
### suricata命令含义
```bash
-S <path>  
	path to signature file loaded exclusively (optional)
	以独占方式加载的签名文件的路径（可选）
-s <path>
	path to signature file loaded in addition to suricata.yaml settings (optional)
	除suricata外加载的签名文件的路径。yaml设置（可选）
	yaml位置：/opt/components/suricata/etc/suricata/suricata.yaml
-r <path> 
	run in pcap file/offline mode
	在pcap文件/脱机模式下运行
-l <dir>
	default log directory
	默认日志路径
```

### 检测
```
suricata -r pcap文件名 -l 自定义输出位置

检测pcap数据包命令：
/opt/components/suricata/bin/suricata -r /root/Work/packet/PythonSynFlood.pcapng  -l /root/Work/suricata_output/ -S /root/Work/suricata_rules/syn.rules

问题：没有输出fast.log
解决办法：
修改配置文件，将fast.log上面的no改为yes。
vi /opt/components/suricata/etc/suricata/suricata.yaml
问题：规则太多导致suricata卡住
解决办法：
[root@localhost ~]# ps -ef|grep suricata
root      4207 13265  0 15:24 pts/0    00:00:00 /opt/components/suricata/bin/suricata -r /root/Work/packet/dns.pcap -l /root/Work/suricata_output/ -S /root/Work/suricata_rules/syn.rules
root      6746  4968  0 15:25 pts/1    00:00:00 grep --color=auto suricata
machloop 13607     1 92 12:31 ?        02:40:35 /opt/components/suricata/bin/suricata --runmode workers --dpdk-pcap -S /opt/components/suricata/suricata.rules
machloop 13703     1 95 12:31 ?        02:46:32 /opt/components/suricata/bin/suricata --runmode workers --dpdk -S /opt/components/suricata/suricata.rules
[root@localhost ~]# kill -9 4207
[root@localhost ~]#

```

### 规则含义

```
alert tcp any any -> any any \
(\
    msg:"检测到synflood攻击";\
    flags:S;\
    threshold:type both,track by_dst,count 15,seconds 30;\
)

alert tcp any any -> any any \
(\
    msg:"LOCAL DOS SYN packet flood inbound, Potential DOS";\
    flow:to_server; \
    flags: S,12; \
    threshold: type both, track by_dst, count 15, seconds 5; \
    classtype:misc-activity; \
    sid:5;\
)
```
- threshold:type both
阈值参数设置为 both，其中包括:
threshold to set a minimum threshold for a rule before it generates alerts.
阈值设置最小阈值的规则
limit to make sure system does not get flooded with alerts.
限制，以确保系统不会被警报淹没。
- flow: to _ server
使用flow: to _ server  或 flow: to _ client 强制检查请求或响应。
- Classstype 
Classstype 关键字提供有关规则和警报分类的信息。它由一个短名称、一个长名称和一个优先级组成。它可以告诉例如，一个规则是否只是信息或是关于黑客等等。对于每种类型，classfication.config 都有一个优先级，该优先级将在规则中使用。
- sid
关键字 sid 为每个签名提供自己的 id。这个 id 用一个数字表示。Sid 的格式如下:
- Ack
是对 TCP 连接的另一端发送的所有以前(数据)字节的接收的确认。在大多数情况下，TCP 连接的每个数据包在第一个 SYN 之后都有一个 ACK 标志和一个 ACK-number，后者随着每个新数据字节的接收而增加。可以在签名中使用 ack 关键字来检查特定的 TCP 确认号。
`ack:1;`
metadata
元数据关键字允许将其他非功能性信息添加到签名中。虽然格式是自由格式，建议坚持键，值对，因为 Suricata 可以包括这些在夏娃警报。格式如下:

## pgsql 设置可以外部访问
1. 编辑配置文件，改为可以监听
2. 重启pgsql

```bash
vi /opt/machloop/data/postgres-fpc/data/postgresql.conf
listen_addresses = '*'
systemctl restart postgresql-11
```
#### PostgreSQL执行SQL文件
1. 连接到PSQL以后
```
psql -d database -U username
 
\i /path/xxx.sql
```

2.未连接到PSQL直接执行
```
psql -d database-U userName -f /path/xxx.sql
```

## shell 向psql中插入数据
```bash
# 初始化参数
export PGPASSWORD='Machloop@123'
db="fpc"
user="machloop"
pwd="Machloop@123"
host="127.0.0.1"
port="5432"
table="fpc_analysis_suricata_rule_classtype"
import_file="IndicatorLabels.txt"
# 数据导入
cat $import_file |tr -d '\r' | while read line
	do
		label_id=$(echo $line|cut -d " " -f1)
		label_name=$(echo $line|cut -d " " -f2)
        cur_dateTime=$(date +%Y-%m-%d' '%H:%M:%S.%N | cut -b 1-23)
		#echo "insert into fpc_analysis_suricata_rule_classtype(id,name,deleted,create_time,update_time) values($label_id,$label_name,0,$cur_dateTime,$cur_dateTime);">>1.sql
        post_data=(`psql -X -A -d ${db} -U ${user} -h ${host} -p ${port} -t -c"SELECT name FROM $table WHERE name='$label_name'"`)
        # echo $post_data
        # echo $label_name
        if [ "${post_data}" == "$label_name" ]; then
            if [ -n "$post_data" ];then
                echo "已存在'$label_name'"
            fi
        else
            psql "host=${host} port=$port user=$user  password=$pwd dbname=$db"<<EOF
            insert into $table values('$label_id','$label_name',0,'$cur_dateTime','$cur_dateTime');
            
EOF
fi
    done

```

## 批量删除数据shell
```bash
# 初始化参数
db="fpc"
user="machloop"
pwd="Machloop@123"
host="127.0.0.1"
port="5432"
table="fpc_analysis_suricata_rule_classtype"
num=100
for((i=100;i<=111;i++));
do
psql "host=${host} port=$port user=$user  password=$pwd dbname=$db"<<EOF

    delete from $table where id='$i';

EOF
done
psql "host=${host} port=$port user=$user  password=$pwd dbname=$db"<<EOF
delete from fpc_analysis_suricata_rule_classtype where create_time='2022-07-18 15:38:59.791+08';
EOF

```
## 读取数据库中的数据shell
```bash
#!bin/bash
# time=$(date "+%Y-%m-%d %H:%M:%S").$((`date "+%N"`/1000000))
export PGPASSWORD='Machloop@123'
db="fpc"
user="machloop"
pwd="Machloop@123"
host="127.0.0.1"
port="5432"
table="fpc_analysis_suricata_rule_classtype"
import_file="IndicatorLabel.txt"
time=$(date +%Y-%m-%d' '%H:%M:%S.%N | cut -b 1-23)
#echo $time
# psql -d ${db} -U ${user} -c  "SELECT 1 FROM $table WHERE name='挖矿行为'"
post_data=(`psql -X -A -d ${db} -U ${user} -h ${host} -p ${port} -t -c"SELECT 1 FROM $table WHERE name='挖矿1行为'"`)

if [ -n "$post_data" ];then
echo $post_data
echo "[ -n ]"
fi
```

## Python 高效字符串拼接
https://blog.csdn.net/zzy979481894/article/details/95039935

### 解决终端乱码
https://zhuanlan.zhihu.com/p/354940478

日志文件：
/opt/machloop/log/fpc-manager

调用顺序
app-layer:

Pop3ProbingParserTs/Pop3ProbingParserTc 识别应用层协议
Pop3StateAlloc 创建应用层协议解析上下文
Pop3ParseRequest/Pop3ParseResponse 解析应用层数据
output:

Pop3GetTxCnt 获取当前应用层交互个数
Pop3GetTx 获取一个应用层交互
Pop3GetAlstateProgressCompletionStatus & Pop3GetStateProgress 获取应用层交互当前状态和结束状态
Pop3GetTxLogged & Pop3SetTxLogged 获取应用层交互是否已输出，设置应用层交互已输出
对于一般协议，仅需根据协议格式，修改Pop3ParseRequest/Pop3ParseResponse 解析实现即可。主要是将packet中信息记录到应用层交互上下文中，以供检测模块或输出模块使用。需要注意，交互上下文的状态会影响其内存释放，异常检测和日志输出。

3. 输出模块实现
输出模块可以视为Suricata的日志输出插件。Suricata会在每包解析结束后检查各类输出模块是否达到输出条件，对于符合输出条件的输出模块，调用其回调接口。

Suricata还对输出模块进行了分类，用于输出不同类型的日志。个人理解其是按照输出时机进行分类的：

OutputRegisterPacketModule 每包解析完成后调用的包输出模块
OutputRegisterTxModule 每个应用层协议交互完成后调用的交互输出模块
OutputRegisterFlowModule 每个Flow结束（超时）后或重用时调用的流输出模块
OutputRegisterFileModule 每个文件还原开始后到输出模块返回大于1之前调用的文件信息（
————————————————
版权声明：本文为CSDN博主「人潮没冲散当初那伙」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_50638175/article/details/108622785



## perf 分析函数CPU使用情况

## alloca 向栈申请内存
alloca是向栈申请内存,因此无需释放。

## 文件路径
配置文件
```
psql 配置文件路径
/opt/machloop/data/postgres-fpc/data
北向配置文件路径
vi /opt/machloop/config/northbound-service/northbound_service.ini

```
查看日志
```
查看北向日志
write-log函数写的，不会进journalctl
tail -f /opt/machloop/log/fpc-engine/northbound_service.log

查看北向发日志
/opt/components/fpc-engine/northbound_service/zmq_dbg --get-tem|grep suricata -A7

查看suricata日志
journalctl -n 20 -u scd
journalctl -fu scd

psql
systemctl restart postgresql-11

fpc日志
journalctl -u fpc-engine -n 20



```

## 查看oom信息
```
dmesg -T
```

```
cp s_zmq /opt/components/fpc-engine/northbound_service/
```

## 更新时间

```
virsh list
virsh start czs-2

ntpdate cn.pool.ntp.org
```

tcpreply
```
tcpreplay -i vnet1 -M 8 -l 0  dns.cap
```

/opt/cpmmponents/fpc-engine/npb-dbg --mpcli | grep 

![](Pasted%20image%2020220930155637.png)


linux 访问windows 共享文件
```bash
mount -t cifs -o username=Admin,password=User@czs123 //10.0.0.79/share /mnt/share
```

### fpc 掉电后分配不到内存
/opt/machloop/config/fpc-engine/dpdk-run.sh
这个文件的38,39行，就是分别给node0和node1分配大页内存
如果日志里面出现了socket0或者1上大页内存不够，就手动echo一个更大的值；或者改dpk-run.sh里的这两行echo的值，然后重启设备

## 元数据添加url
### metadata_to_json.c :729
```

yyjson_mut_val *url_arr = yyjson_mut_arr((doc));

    MimeDecUrl *curr = pstMime->url_list;

    while (curr != NULL)

    {

        yyjson_mut_val *val = yyjson_mut_strcpy((doc), curr->url);        

        yyjson_mut_arr_append(url_arr, val);

        curr = curr->next;

    }

    yyjson_mut_obj_add_val((doc), (metadata_val), MAIL_URL_LIST, url_arr);
```

### util_mime.c
```
static void PrintUrl(MimeMsg *pstMimeMsg)

{

    MimeDecUrl *curr = pstMimeMsg->url_list;

    while (curr != NULL)

    {

        printf("%s-%s\n",curr->url_flags,curr->url);

        curr = curr->next;

    }

}

static void PrintChars(int log_level, const char *label, const uint8_t *src, uint32_t len)

{

    printf("[%s]: ", label);

    for(int i=0;i<len;i++){

        printf("%c",src[i]);

    }

    printf("\n");

}

```

### util_mail.h



### metadata_log.c: 987
```
mime_free(pstMime);
```

```bash

cp fpc-npb/src/npb/metadata/util_mime.c new/fpc-npb/src/npb/metadata/
cp fpc-npb/include/src/npb/util_mail.h new/fpc-npb/include/src/npb/

cp fpc-npb/src/npb/outer/metadata_to_json.c new/fpc-npb/src/npb/outer/

cp fpc-npb/src/npb/common/metadata_log.c new/fpc-npb/src/npb/common/
cp fpc-npb/include/src/npb/metadata_log.h new/fpc-npb/include/src/npb/
```
/opt/machloop/config/northbound-service/template/

### 根据地址定位文件
```c
addr2line -e /opt/components/suricata/bin/suricata 0x6e456d
```

## 虚拟机重建

```bash
virsh destroy czs-1
virsh undefine czs-1

./common-create.py --name=czs  --cdrom=/home/tfa_auto_tfa-225.iso --virnet=virbrczs --ssdimg=/home/vm2_hdd/tfa-ssd-czs.img --ssdsize=120 --vncport=5902 --ip=10.0.2.145

```
## 扩大虚拟机内存
```

扩大内存 报错：
Call to virDomainSetMaxMemory failed: Requested operation is not valid: initial memory size of a domain with NUMA nodes cannot be modified with this API

解决办法：
virsh edit czs-1 需要改numa节点

```
## 重建后无法上网
```
原因：没有配置域名服务器
方法1：
echo nameserver 8.8.8.8 > /etc/resolv.conf

方法2：
cd /etc/sysconfig/network-scripts/
vi ifcfg-ens3
添加
DNS1=8.8.8.8
DNS2=114.114.114.114

service network restart
```

## 基础包构建
```bash
/tmp/mnt/infrastructure
sh all_rpm.sh  #默认可以不装
sh infrastructure_install.sh
```

## 宿主机查看虚拟机网卡
```
brctl show
```

## 重组后编译tfa报错

```
报错 dpdk.h

scp -r root@10.0.0.101:/home/dpdk-stable-20.11.1 /home
export RTE_SDK=/home/dpdk-stable-20.11.1
export RTE_TARGET=build_1

vi /etc/profile
新添
export RTE_SDK=/home/dpdk-stable-20.11.1
export RTE_TARGET=build_1

yum install zlib-devel.x86_64

报错：linpq.h:
yum install postgresql-devel
yum install  bzip2-devel

 cannot find -lsm4
 将 svn上的 lib 目录拷贝下来

suricata:
configure: error: pcre.h not found ...

yum -y install pcre-devel

yum install -y epel-release

make: *** [aclocal.m4] Error 127
yum install automake
make 报错 zmq.h 没有那个文件
将zmq.h 和3rd_lib中的动态库加入到/home/zmq下
```
### 查看动态库
nm libzmq.so |grep zmq_msg_send