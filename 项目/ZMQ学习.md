原文链接：https://blog.csdn.net/lfsf802/article/details/38238007
## ZMQ是什么

阅读了ZMQ的Guide文档后，我的理解是，这是个类似于Socket的一系列接口，他跟Socket的区别是：普通的socket是端到端的（1:1的关系），而ZMQ却是可以N：M 的关系，人们对BSD套接字的了解较多的是点对点的连接，点对点连接需要显式地建立连接、销毁连接、选择协议（TCP/UDP）和处理错误等，而ZMQ屏蔽了这些细节，让你的网络编程更为简单。ZMQ用于node与node间的通信，node可以是主机或者是进程。

## ZMQ 的三个基本模型

ZMQ 提供了三个基本的通信模型，分别是“Request-Reply “，”Publisher-Subscriber“，”Parallel Pipeline”，我们从这三种模式一窥 ZMQ 的究竟。

### ZMQ 的 Request-Reply,  应答模式REQ/REP模式
参考：[](https://blog.csdn.net/bjbz_cxy/article/details/114384483)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a944accf3cc14afbbe5777a5a00de227.png)
客户端
```c
#include <zmq.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
 
int main(void)
{
    printf("Connecting to server...\n");
 
    void * context = zmq_ctx_new();
    void * socket = zmq_socket(context, ZMQ_REQ);
    zmq_connect(socket, "tcp://localhost:6666");
 
    while(1)
    {
        char buffer[10];
        const char * requestMsg = "Hello";
        int bytes = zmq_send(socket, requestMsg, strlen(requestMsg), 0);
        printf("[Client][%d] Sended Request Message: %d bytes, content == \"%s\"\n", i, bytes, requestMsg);
 
        bytes = zmq_recv(socket, buffer, 10, 0);
        buffer[bytes] = '\0';
        printf("[Client][%d] Received Reply Message: %d bytes, content == \"%s\"\n", i, bytes, buffer);
 
    }
 
    zmq_close(socket);
    zmq_ctx_destroy(context);
 
    return 0;
}
```

服务端
```c
#include <zmq.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
 
int main(void)
{
    printf("Connecting to server...\n");
 
    void * context = zmq_ctx_new();
    void * socket = zmq_socket(context, ZMQ_REQ);
    zmq_connect(socket, "tcp://localhost:6666");
 
    while(1)
    {
        char buffer[10];
        const char * requestMsg = "Hello";
        int bytes = zmq_send(socket, requestMsg, strlen(requestMsg), 0);
        printf("[Client][%d] Sended Request Message: %d bytes, content == \"%s\"\n", i, bytes, requestMsg);
 
        bytes = zmq_recv(socket, buffer, 10, 0);
        buffer[bytes] = '\0';
        printf("[Client][%d] Received Reply Message: %d bytes, content == \"%s\"\n", i, bytes, buffer);
 
    }
 
    zmq_close(socket);
    zmq_ctx_destroy(context);
 
    return 0;
}
```

### ZMQ 的 Publish-subscribe 模式

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7456023773a47ff88447a6126bee919.png)

#### 服务端

```c
#include <stdio.h>
#include "zmq.h"

int main()
{
    // 1 创建zmq上下文
    void* context = zmq_ctx_new();
    if (context == NULL)
    {
        printf("zmq_ctx_new error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    /* 2 创建zmq套接字
     * 不同的flag有不同的组合方式。比如ZMQ_REQ ZMQ_REP,在这种模式下,
     * 我们的程序必须要遵守recv()和send()配对使用的编程模式,也就是说，
     * 在服务程序中，必须要有完整的recv()和send()成对出现
     * 本样例使用ZMQ_PUB方式，就不需要成对出现
     */
    void* socket = zmq_socket(context, ZMQ_PUB);
    if (socket == NULL)
    {
        printf("zmq_socket error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    // 3 绑定节点，下面例子中"127.0.0.1"为IP，"10086" 为 端口号
    int ret = zmq_bind(socket, "tcp://127.0.0.1:10086");
    if(ret == -1)
    {
        printf("zmq_bind error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    // 定义一块缓冲区用于发送
    char send_buf[1024] = { 0 };

    while (true)
    {
        // 将缓冲区内容置为0
        memset(send_buf, 0, sizeof(send_buf));

        printf("请输入发送内容(输入0结束):");
        int ret = scanf("%s",send_buf);
        if (send_buf[0] == '0')
        {
            break;
        }

        // 发送数据
        ret = zmq_send(socket, send_buf, strlen(send_buf), 0);
        if (ret != -1)
        {
            printf("发送了%d个****字符\n", ret);
        }
        else
        {
            printf("zmq_send error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        }
    }

    // 关闭socket
    zmq_close(socket);

    // 销毁上下文
    zmq_ctx_destroy(context);
}


```

#### 客户端
```c
#include <stdio.h>
#include "zmq.h"

int main()
{
    // 1 创建zmq上下文
    void* context = zmq_ctx_new();
    if (context == NULL)
    {
        printf("zmq_ctx_new error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    /* 2 创建zmq套接字
     * 不同的flag有不同的组合方式。比如ZMQ_REQ ZMQ_REP,在这种模式下,
     * 我们的程序必须要遵守recv()和send()配对使用的编程模式,也就是说，
     * 在服务程序中，必须要有完整的recv()和send()成对出现
     * 本样例使用ZMQ_PUB方式，就不需要成对出现
     */
    void* socket = zmq_socket(context, ZMQ_SUB);
    if (socket == NULL)
    {
        printf("zmq_socket error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }
    
    // 3 连接节点 下面例子中"127.0.0.1"为IP，"10086" 为 端口号
    int ret = zmq_connect(socket, "tcp://127.0.0.1:10086");
    if (ret == -1)
    {
        printf("zmq_connect error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    /* 设置标记位,如果这里设置了如这种代码
     * zmq_setsockopt(socket, ZMQ_SUBSCRIBE, "mark", 4)
     * 表示接收时会检查字符串前面是否有“mark”，如果有才会接收，起到筛选作用
     */
    ret = zmq_setsockopt(socket, ZMQ_SUBSCRIBE, "", 0);
    if (ret == -1)
    {
        printf("zmq_setsockopt error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        return -1;
    }

    // 定义接收缓冲区
    char recv_buf[1024] = { 0 };    

    while (true)
    {
        // 将缓冲区内容置为0
        memset(recv_buf, 0, sizeof(recv_buf));

        printf("等待数据\n");

        // 接收数据，如果没有接收到数据，程序会一直阻塞在这里
        ret = zmq_recv(socket, recv_buf, sizeof(recv_buf), 0);
        if (ret != -1)
        {
            printf("接收%d个字符\n", ret);
            printf("%s\n", recv_buf);
        }
        else
        {
            printf("zmq_recv error,errorno = %d,error info:%s\n", zmq_errno(), zmq_strerror(zmq_errno()));
        }
    }

    // 关闭socket
    zmq_close(socket);

    // 销毁上下文
    zmq_ctx_destroy(context);
}

```

### ZMQ的分布式推送PUSH/PULL模式
分发者 ventilator
执行者 worker
收集结果的接收者 sink
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a2058b5ef5b4a1e8dfd0683fbcf4c25.png)

#### ventilator：
```c
#include <zmq.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
 
int main(void)
{
    void * context = zmq_ctx_new();
    void * sender = zmq_socket(context, ZMQ_PUSH);
    zmq_bind(sender, "tcp://*:6666");
	printf ("Press Enter when the workers are ready: ");
    getchar ();
	printf ("Sending tasks to workers...\n");
    while(1)
    { 
        const char * replyMsg = "World";
        zmq_send(sender, replyMsg, strlen(replyMsg), 0);
        printf("[Server] Sended Reply Message content == \"%s\"\n", replyMsg);
    }
 
    zmq_close(sender);
    zmq_ctx_destroy(context);
 
    return 0;
}
```

#### worker：
```c
#include <zmq.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
 
int main(void)
{
void * context = zmq_ctx_new();
void * recviver = zmq_socket(context, ZMQ_PULL);
zmq_connect(recviver, "tcp://localhost:6666");
 
void * sender = zmq_socket(context, ZMQ_PUSH);
zmq_connect(sender, "tcp://localhost:5555");
 
while(1)
{
char buffer [256];
int size = zmq_recv (recviver, buffer, 255, 0);
if(size < 0)
{
return -1;
}
printf("buffer:%s\n",buffer);
const char * replyMsg = "World";
zmq_send(sender, replyMsg, strlen(replyMsg), 0);
printf("[Server] Sended Reply Message content == \"%s\"\n", replyMsg);
}
 
zmq_close(recviver);
zmq_close(sender);
zmq_ctx_destroy(context);
 
return 0;
}
```

#### sink：
```c
#include <zmq.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
 
int main(void)
{
    void * context = zmq_ctx_new();
    void * socket = zmq_socket(context, ZMQ_PULL);
    zmq_bind(socket, "tcp://*:5555");
 
    while(1)
    { 
       	char buffer [256];
		int size = zmq_recv (socket, buffer, 255, 0);
		if(size < 0)
		{
			return -1;
		}
        printf("buffer:%s\n",buffer);
    }
 
    zmq_close(socket);
    zmq_ctx_destroy(context);
 
    return 0;
}
```

## 常用函数
### zmq_setsockopt
#### 描述
zmq_setsockopt()函数会对socket参数指定的socket进行设置，设置的属性由option_name参数指定，属性值由参数option_value指定。option_len参数指定属性值的数据存储空间的大小。

以下的socket属性可以通过zmq_setsockopt()函数进行设置：
- ZMQ_RCVBUF：设置内核传输缓冲区大小

ZMQ_RCVBUF属性会对socekt参数指定的socket设置底层内核的传输缓存大小，以B为单位进行设置。设置的属性值是0，则意味着使用OS的默认值。你可以查看你的操作系统手册来获得SO_RCVBUF属性更详细的信息。

|属性值的类型 |int |
|-- |--|
|属性值的单位 |B（字节） |
|默认值|0|
|可以应用的socket类型|所有类型|

- ZMQ_RCVHWM：对进入socket的消息设置高水位

ZMQ_RCVHWM属性将会设置socket参数指定的socket进入的消息的高水位。高水位是一个硬限制，它会限制每一个与此socket相连的在内存中排队的未处理的消息数目的最大值。0值代表着没有限制。

如果已经到达了规定的限制，socket就需要进入一种异常的状态，表现形式因socket类型而异。socket会进行适当的调节，比如阻塞或者丢弃被发送的消息。请从zmq_socket(3)函数中查看更多细节，获取每种类型的socket的精确的行为。

# 6-27
yyjson

MallocExtension_ReleaseFreeMemory 释放空内存

geoip 
GeoIP库可以根据IP地址(支持IPv4 和 IPv6), 定位该IP所在的 洲、经纬度、国家、省市、ASN 等信息

postgresql 学习

psql命令行学习
```
-d <数据库名> 指定连接到的数据库名（默认为$PGDATABASE或者当前登录用户名） 
-p <端口> 指定数据库服务器的端口（默认为$PGPORT或者编译期设置的默认值，通常为4321） 
-U <用户名> 指定数据库用户（默认为$PGUSER或者当前登录的用户名） 
```
psql内部命令快速参考
```
\? 列出所有的psql内部命令 
\a 在表格对齐和非对齐模式之间切换。 
\c[onnect] [dbname|- [user]] 连接到新的数据库；使用“-”作为数据库名指连接到默认数据库。可以user身份连接数据库 
\C <标题> 设置输出表格的标题；功能与“\pset 标题”相同 
\cd <目录> 改变工作目录 
\copy … Perform SQL COPY with data stream to the client machine. 
\copyright 显示PostgreSQL的使用和发布条款 
\d <表> 描述表（或者视图、索引、序列生成器） 
\d{t|i|s|v} 列出表/索引/序列生成器/视图 
\d{p|S|l} 列出访问许可/系统表/大对象 
\da 列出聚合体（aggregates） 
\db 列出表空间 
\dc 列出conversions 
\dC 列出casts 
\dd [对象] 列出表、类型、函数或者操作的注释 
\dD 列出domains 
\df 列出函数（自定义函数？？？）需要验证 
\dg 列出groups 
\dl 列出大对象；也可以写作“\lo_list” 
\dn 列出模式 
\do 列出operators 
\dT 列出数据类型 
\du 列出用户 
\e [file] 使用外部编辑器编辑当前的查询缓冲区或者file指定的文件 
\echo <文本> 将文本打印到标准输出 
\encoding <编码> 设置客户端编码 
\f <分隔符> 修改输出字段的分隔符 
\g [文件名] 将查询的结果发送到后端（结果输出到文件或者|管道） 
\h [命令] 显示SQL命令的帮助；*表示所有命令的详细说明 
\H 开启HTML模式 
\i <文件名> 从文件中读取并执行查询 
\l 列出所有的数据库 
\lo_export，\lo_import， 
\lo_list，\lo_unlink 执行大对象操作 
\o [文件名] 将所有的查询结果发送到文件或者|管道 
\p 显示当前查询缓冲区的内容 
\pset <选项> 设置表输出选项，可设置的选项可以是以下中的一个：format，border，expanded，fieldsep，footer，null，recordsep，tuples_only，title，tableattr，pager 
\q 退出psql 
\qecho <文本> 将文本写入到查询输出流（参考\o命令） 
\r 重置（清空）查询缓冲区 
\s [文件名] 打印历史或将历史存入文件中 
\set <变量> <值> 设置内部变量 
\t 只显示行（在该模式之间切换） 
\T <标记> 设置HTML表格的标记；功能和“\pset tableattr”相同 
\timing 显示命令执行的时间（在显示和不显示这两种模式间切换） 
\z 列出对表、视图和序列生成器的访问许可 
\! [命令] 切换到shell或者执行一个shell命令 
```


连接fpc数据库：
` psql -d fpc -U machloop -p 5432`

使用\\d命令查看postgresql中所有表

\\d+ 命令更加详细的显示表
![](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220627171412344-1047525617.png)

### s_send()
s_send(send,json,json_len)
先把json转为可发送的message
然后通过send发送数据

# 6-28
C语言:stat,fstat和lstat函数
这三个函数的功能是一致的，都用于获取文件相关信息，但应用于不同的文件对象。对于函数中给出pathname参数，stat函数返回与此命名文件有关的信息结构，fstat函数获取已在描述符fields上打开文件的有关信息，lstat函数类似于stat但是当命名的文件是一个符号链接时，lstat返回该符号链接的有关信息，而不是由该符号链接引用文件的信息。第二个参数buf是指针，它指向一个用于保存文件描述信息的结构，由函数填写结构内容。该结构的实际定义可能随实现有所不同.
### 线程
#### local_timer
```
每隔3秒
{
	向4444端口发送json数据("trigger_offline_log", real_time);
	
	清理空内存;
}
每隔10秒
{
	读配置文件;
	刷新geoip规则;
}
每隔60秒
{
	report_http_analysis_result--上报http分析结果;
	
	trigger_log_sending--向4444端口发送json数据（"trigger_log_timestamp", real_time）;
}
```


#### pgsql_obtain_loop
```
在pgsql中读取配置，发给kafka
```
#### clickhouse_router
保存元数据和ETA结果，入库组件
![image](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220628182531592-1973939225.png)
#### kafka_router
kafka外发组件
worker线程拼接字符串外发给kafka。
![image](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220628180741039-1847721779.png)
#### output_router
消息通过zmq 发出去，tls日志
 
![image](https://img2022.cnblogs.com/blog/1865959/202206/1865959-20220628182847445-1549408312.png)
#### zmq_debug_loop
客户端调试的
#### receive_analysis_result
收检测结果，和output_router有关联
#### analysis_router
进行http数据分析，http_analysis(buf, buf_len);
#### clickhouse_offline_router
进行离线分析，上面的3秒触发发送的
#### statistics_router
统计信息，用的3秒的接口。

最后curl线程和clickhouse进行http连接，收到的消息通过http写入clickhouse

### suricata 简介
Suricata是一个免费、开源、成熟、快速、健壮的网络威胁检测引擎。Suricata引擎能够进行实时入侵检测(IDS)、内联入侵预防(IPS)、网络安全监控(NSM)和离线pcap处理。Suricata使用强大而广泛的规则和签名语言来检查网络流量，并提供强大的Lua脚本支持来检测复杂的威胁。使用标准的输入和输出格式(如YAML和JSON)，使用现有的SIEMs、Splunk、Logstash/Elasticsearch、Kibana和其他数据库等工具进行集成将变得非常简单。Suricata项目和代码由开放信息安全基金会(OISF)拥有和支持，OISF是一个非盈利基金会，致力于确保Suricata作为一个开源项目的开发和持续成功。 

###  Kafka 概述
  Kafka 是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。