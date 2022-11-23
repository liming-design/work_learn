```
main ->mp_cli_create -> mp_cli_loop -> npb_pkt_parse -> flow_process_packet ->npb_pkt_handle ->npb_proto_proc
```

### st_session_head
```c
typedef struct SessionHead_st{
    uint32 ulSessionSN;          /* 此会话节点对应的ID，此内存分配空间时赋值，后续反复申请释放内存时是，此变量保持不变 */
    uint32 ulLastTime;        /* 最后一次访问此会话的时间 */
    
    struct SessionHead_st *pNext;

    union {
        /* 下面两个结构体完全相同，FPC、NPB归一后修改 */
        struct fivepuple_st fivepuple;
        struct {
            uint8  ucVersion:4;     /* V4或V6的版本识别 */
            uint8  ucDPINoSend:1;   /* 是否需要往DPI送检测包， 0为需要送，1不需要送 */
            uint8  ucDPIRes:1;      /* 是否已完成DPI检测，如果DPI已分析完成，这里将置为1 */
            uint8  ucAPPRuleFlag:1; /* 是否需要存储，如果不需要存储则置为0，需要存储为1。当ucDPIRes为1时，此变量才生效 */
            uint8  ucFlowCapture:1; /* 此条流是否需要捕获，如果命中流过滤，不需要捕获则置为0，需要捕获为1
                                       当不需要捕获时，不进行流分析，流日志生成，也不存储等操作 */
            uint8  ucProto;         /* 协议号 */
            uint16 usSPort;         /* 源端口 */
            uint16 usDPort;         /* 目的端口 */
            uint16 usVlanId;

            union {
                struct {
                    uint64 ullSIpAddr64[2];
                    uint64 ullDIpAddr64[2];
                } stValue64;
                struct {
                    uint32 ulSIpAddr[4];    /* V4只用第一个变量，V6使用整个数组 */
                    uint32 ulDIpAddr[4];    /* V4只用第一个变量，V6使用整个数组 */
                } stValue32;
                struct {
                    uint32 ulSIpAddr;       /* IPV4的源IP */
                    uint16 usSIpIdent;      /* 窗口的最大值 */
                    uint16 usReserve1;
                    uint64 ullSipIdentBit;
                    uint32 ulDIpAddr;       /* IPV4的目的IP */
                    uint16 usDIpIdent;      /* 窗口的最大值 */
                    uint16 usDipFirst:1;    /* 反向命中首包 */
                    uint16 usReserve2:15;
                    uint64 ullDipIdentBit;
                }; /* 去重功能使用 */
            } unValue;

        };
    };

    uint8 ucNetWorkId;        /* 网络Id，暂时先放这里，后续需要调整在结构体的相对位置 */
    uint8 ucStatThread:3;     /* 此变量为标识此会话在哪个统计线程处理，此值永远不变,在申请内存时初始化 */
    uint8 ucDpiDelCache:1;    /* 如果发现此会话对应的DPI为不需要存储，删除缓存的置1，避免重复进入删除 */
    uint8 ucEthType:4;        /* 此变量为二层头类型的字典ID */
    uint16 usReserve2;

    uint16 usAgeTimer:10;     /* 老化时间,最大1023秒 */
    uint16 ucSessRefresh:6;   /* 每次回收此值加1，这个值有变更，说明会话被第二次使用 */
    uint16 usDPIAppId;        /* 应用ID */
    uint16 usWormAppid;       /* 僵木蠕应用ID, 为零时识别不成功，否则成功 */
    uint16 usDPIProtoId;      /* 协议号,对应NPB需要的协议号 */
    uint16 usServiceId;       /* 此流对应的服务ID */
    uint16 usFileBlockId;     /* 离线分析使用，用于获取该包属于哪个文件 */
    uint16 usLinkType;        /* 此条流对应的linktype */

    union {
        uint32 ulFlagSet1;
        struct {
            uint8  ucStatisticIndex:1;  /* 上次统计的ID */
            uint8  ucFlagNoLog:1;       /* 不处理流日志标记，如果标记为1，此会话不上报流日志 */
            uint8  ucTCPHandShakeOK:1;     /* 是否完成三次握手，为1则为完成 */
            uint8  ucQuickAge:1;        /* 当会话启动快速老化机制：
                                           TCP时：当建会话的首包有SYN机制时，将置为1，
                                                  如果后续收到ACK包，则置为0
                                           UDP时：为UDP时收到首包置1，老化超时为快速老化，
                                                  当来了第二个包时，置为0，然后将老化超时置为正常值 */
            uint8  ucWormNoSend:1;      /* 是否需要往DPI送僵木蠕应用检测包， 0为需要送，1不需要送 */
            uint8  is_continue:1;       /* 是否为超长流，1为超长流，初始化为零 */
            uint8  ucLocalIp:2;         /* 两个bit用来标识是否命中内网IP，低bit表示源IP，高bit表示目的IP */

            uint8  ucUseFlag:1;         /* 使用标记，使用时标记为1，不使用时，标记为0 */
            uint8  ucTcpState:3;        /* 当前如果是TCP会话，此变量代表当前会话的状态 */
            uint8  ucNPBThread:4;       /* 此会话在NPB的哪个线程处理 支持0-15 */

            uint8  ucCheckMac:1;        /* 命中此会话时，是否需要对MAC地址进行匹配 */
            uint8  ucIPAddrSort:1;      /* IP的顺序分析，如果源IP大于目的IP，则为0，否则为1 */
            uint8  ucReserve3:1;
            uint8  knownDirection:1;    /* 此流是否可以确认服务/客户端 */
            uint8  serverReplyState:2;  /* 服务端响应状态，0 客户端未发送请求，1 客户端已发送请求，服务端尚未响应，2 服务端已响应 */
            uint8  ucReserve0:2;
            uint8  ucReserve1;
        };
    };

    uint16 usStatSAreaFlag:2;
    uint16 usStatSAreaId:14;

    uint16 usStatDAreaFlag:2;
    uint16 usStatDAreaId:14;

    uint32 ulForwardVersion;

    struct {
        uint8 ucChecked;            /* 已经匹配到 */
        uint8 ucTupleIndex;         /* 首包匹配到的第几行tuple */
        uint16 usTruncLen;
        uint16 usSFVersion;         /* 存储过滤配置版本号 */
    } stStoreFilterRes;             /* 匹配存储规则的结果 */

    st_statistic_idgroup stStatIdGroupNetwork[SESSION_MAX_NETWORK_SUPPORT];
    st_statistic_idgroup stStatIdGroupService[SESSION_MAX_SERVICE_SUPPORT];

    uint64  ulSessionID;       /* 会话ID，不重复，每个会话一个ID,从1开始编号，0无效 
                                  每次申请使用这个会话时，此ID都会变化 */
    void    *pDPIData;         /* DPI使用的数据指针会话回收时，需要回收内存 */
    void    *pTLSInfo;         /* 指向tls信息 */

    struct session_stat_st session_stat;
    st_statistic_public stSessStat[2];  /* 两个，跨统计周期时，分别使用 */

    union {
        uint64 ullTimeDelay[2];
        struct {
            uint32 ucTCPTimeDelayUse:1;             /* TCPTimeDelay是否可以使用(各项统计中使用)，可以使用过标记为1，否则为0，下同 */
            uint32 ucTCPTimeDelayUseServiceRT:1;    /* TCPTimeDelay是否可以使用(秒级服务中使用)，可以使用过标记为1，否则为0，下同 */
            uint32 ucTCPTimeDelayUseService:1;      /* TCPTimeDelay是否可以使用(服务中使用)，可以使用过标记为1，否则为0，下同 */
            uint32 ucTCPTimeDelayUseFlow:1;         /* TCPTimeDelay是否可以使用(流日志中使用)，可以使用过标记为1，否则为0，下同 */
            uint32 ulTCPFirstPacketSyn:1;           /* 首包是否是SYN报文 SESSION_FIRST_PACKET_SYN */
            uint32 ulTCPNotFailCheck:1;             /* 不记录建连失败的标记。在syn_sent和syn_rcvd状态下，到了ack包，即将该位置1 */
            uint32 ucTCPEstUseServiceRT:1;
            uint32 ulReserve0:1;
            uint32 ulTCPTimeDelay:24;               /* TCP三次握手时延（单位0.1毫秒），非零有效 */
            
            uint32 ucClientNetDelayUse:1;
            uint32 ucClientNetDelayUseServiceRT:1;
            uint32 ucClientNetDelayUseService:1;
            uint32 ucClientNetDelayUseFlow:1;
            uint32 ulReserve1:4;
            uint32 ulClientNetDelay:24;     /* 客户端网络时延（单位0.1毫秒），非零有效  */

            uint32 ucServerNetDelayUse:1;
            uint32 ucServerNetDelayUseServiceRT:1;
            uint32 ucServerNetDelayUseService:1;
            uint32 ucServerNetDelayUseFlow:1;
            uint32 ulReserve2:4;
            uint32 ulServerNetDelay:24;     /* 服务器网络时延（单位0.1毫秒），非零有效  */

            uint32 ucServerReplyDelayUse:1;
            uint32 ucServerReplyDelayUseServiceRT:1;
            uint32 ucServerReplyDelayUseService:1;
            uint32 ucServerReplyDelayUseFlow:1;
            uint32 ulReserve3:4;
            uint32 ulServerReplyDelay:24;   /* 服务器响应时延（单位0.1毫秒），非零有效  */
        };
    };

    void *tcp_rsmb;
    void *pparselog_cur;
    struct timespec startTimer;  /* 流的首包绝对时间 */
    struct timespec stTimer;     /* 最后一次访问此会话的绝对时间 */
    
    uint8 ucTypeFlag;            /* 标识会话类型，以便老化线程使用 */
    uint8 flag;                  /* 是否命中采集策略 */
    uint16 interface;
    
    uint8 d_addr[MAC_ADDR_LEN];  /**< Destination address. */
    uint8 s_addr[MAC_ADDR_LEN];  /**< Source address. */

    uint8 ucStatClearFlag;  /* 此变量用于分析是否需要清除数据，当前统计和流日志分离在两个线程处理了 */
    uint8 ucForwardTupleBit;  /* 实时转发，命中规则bit，2bit位一个规则 */
    uint8 ucNetWorkCount;   /*网络的数量*/
    uint8 ucServiceCount;   /*服务的数量*/
    uint8 szNetWorkId[MAX_SUB_NETWORK_PER_NET];
    uint16 szServiceId[MAX_SESS_SERVICE_TYPE_NUM];
} st_session_head; /* 各种业务的公共会话部分 */

```


### st_mbuf_pktmsg

```c
typedef struct { 
    /* 注意，为了性能优化，修改此结构体大小，需要修改此结构体的清零初始化操作
       请查找 注释 "st_mbuf_pktmsg memset 0" 查找修改点 */
    union {
        uint16 usValue1;
        /* 以下所有变量为正在使用的值，如果有隧道，
                如果处理的是外层建会话，以下变量全部存外层
                如果处理的是内层建会话，以下变量全部存的是内层的 */
        struct {
            uint8  ucVersion:4;       /* V4或V6的版本识别 */
            uint8  ucFlagFrist:1;     /* 分片首片标记 */
            uint8  ucFlagAfter:1;     /* 分片后续片标记 */
            uint8  ucPacketFirst:1;   /* 首包标记，为当前流的首包打上1的标记 */
            uint8  ucDPIRes:1;        /* 是否已完成DPI检测，如果DPI已分析完成，这里将置为1 */
            uint8  ucAPPRuleFlag:1;   /* (应用存储功能)是否需要存储，如果不需要存储则置为0，需要存储为1。当ucDPIRes为1时，此变量才生效 */
            uint8  ucBPFCapture:1;    /* BPF分析后，是否需要捕获标记，为1时，需要捕获，0时由会话来决策是否需要捕获 */
            uint8  ucSessHit:2;       /* 是否命中会话 */
            uint8  ucLocalipHit:2;    /* 是否命中内网IP */
            uint8  ucFragReass:1;     /* IP分片重组标记*/
            uint8  ucARPFlag:1;     /* ARP报标记，此标记为1，则为ARP包 */
        };
    };
    uint16 usVlanId;        /*2+2=4*/
    uint16 usSPort;         /* 源端口 */
    uint16 usDPort;         /* 目的端口 *//*4+4=8*/
    union {
        struct {
            uint64 ullSIpAddr64[2];
            uint64 ullDIpAddr64[2];
        } stValue64;
        struct {
            uint32 ulSIpAddr[4];    /* V4只用第一个变量，V6使用整个数组 */   
            uint32 ulDIpAddr[4];    /* V4只用第一个变量，V6使用整个数组 */
        } stValue32;
        struct {
            uint8 ucSIpAddr8[16];
            uint8 ucDIpAddr8[16];
        } stValue8;
    } unValue;/*8+32=40*/
    
    uint8  ucProto;         /* 协议号 */
    uint8  eth_head_len;
    uint8  ip_head_len;     /* 当前使用的IP头长度，如果有tunel，代表内层，如果没有tunnel，则代表外层，反正当前使用IP头长度 */
    uint8  ucNetWorkId;      /* 网络ID，将接口划分网络 */
    uint8  ucGroupSN;       /* 报文序号对应的分组，与ulPacketSN配合使用，报文缓存时才需要使用 */
    uint8  ucTcpFlags;
    union {
        uint16 usValue2;
        struct {
            /* 以下所有位变量都是外层的情况，如果没有外层，将不进行处理 */
            uint8 ucOuterFlagFrist:1;
            uint8 ucOuterFlagAfter:1;
            uint8 ucOuterIPFlag:1;  /* 0为IPV4，1为IPV6 */
            uint8 ucMetadataFlag:1; /* metadata开关，1时此包需要送NPB，0不送NPB */
            uint8 ucDupAction:1;    /* 此包是否要进行去重处理，1去重 0不去重 */
            uint8 ucMPLSLabelFlag:1;    /* mpls的label是否有效，1为有效 */
            uint8 ucVXLANVnidFlag:1;    /* VXLANVnid是否有效，1为有效 */
            uint8 ucFlagNoLog:1;        /* 为1时：此包建会话时，此条流将不生成流日志 */
            
            uint8 ucDSCPType:6;          /* DSCP */
            uint8 ucGreKeyFlag:1;        /* GRE密钥标志位 */
            uint8 ucSessionVlanAction:1; /*“0”：“不区分vlan建流分析”，“1”：“区分vlan建流分析”*/
        };
    };  /*40+8=48*/
    
    union {
        uint32_t packet_type; /**< L2/L3/L4 and tunnel information. */
        struct {
            uint8_t l2_type:4; /**< (Outer) L2 type. */
            uint8_t l3_type:4; /**< (Outer) L3 type. */
            uint8_t l4_type:4; /**< (Outer) L4 type. */
            uint8_t tun_type:4; /**< Tunnel type. */
            uint8_t inner_l2_type:4;
            uint8_t inner_l3_type:4;
            uint8_t inner_l4_type:4; /**< Inner L4 type. */
            uint8_t inner_rese:4;
        };
    };

    union {
        uint64 ullAllHdrLen;       /**< combined for easy fetch */
        struct {
            uint16 l2_len:7;
            uint16 l3_len:9;    /**< L3 (IP) Header Length. */
            uint8  l4_len;      /**< L4 (TCP/UDP) Header Length. */
            uint8  inner_l4_len;
            uint16 inner_l2_len:7;
            uint16 inner_l3_len:9;
            uint16 tunnel_len;
        };
    };   /*48+16=64*/

    union { //不能超过32字节
        /* FPC使用 */
        struct {
            uint64 ulVXLANVnid:24;
            uint64 ulMPLSLabel:20;
            uint64 usMacOffset:16;
            uint64 ucStatisticsIdRT:2;      /* 实时统计写入ID */
            uint64 ucDHCPFlag:1;    /* 标记此为DHCP包，DHCP包，需要对MAC地址做sess key  */
            uint64 ucMacCopyFlag:1;         /* 是否向会话中拷贝MAC地址 */
            
            uint32 ulGreKey;        /* GRE隧道识别关键字，即密钥 */    
            uint16 usFragOffsetLen16;    /* 主机序,ip分片偏移 */
            uint16 usWindowSize;    /* TCP窗口大小，4层协议为TCP时才生效 */
            
            uint32 ulPacketSN;      /* 报文的序号，此线程此缓存区中收到的第几个报文 *//*88+24=112*/
            uint8  ucRecerve3[2];      /*112+16=128*/
            uint16 usFileBlockId;   /* 离线分析使用，用于获取该包属于哪个文件 */
            
            void  *pNextFragMbuf;   /* 下一个缓存分片 */
        };
        
        /* tcp重组时使用 */
        tcp_seq stTcpSeq;

        /* TCP重组使用 */
        struct {
            uint32 ulTCPReserve1[8];
        };

        /* NPB使用 */
        struct {
            st_uuid ucMainUUID; /* 离线分析主任务UUID */
            uint16 usLinkType;
        };
    };

    uint8  ucmbuf_type;
    uint8  ucSessRefresh;
    uint16 ulPayloadLen;        /* 应用层负载长度 */
    uint16 ipdatalen;       /* 包括IP头长度 */
    uint16 tcpdataoff;
    
    uint16 Layer4OffOri;        /* 从mubf的buf_addr开始算起 */
    uint16 Layer7OffOri;        /* 从mubf的buf_addr开始算起;NPB能解析元数据的偏移，不是严格的七层协议，如ICMP,ARP等就是例外。 */
    uint32 ulfraghash;  /*用作分片hash*/
    
    uint32 ulIdent;     /* IP头中的ID序号 */    
    uint32 pktlen;      /*报文实际长度，不包含pad*/
    uint32 layer7len;
    uint32 ulHashValue;     /* 五元组HASH值 */
    uint32 ulHashIpValue;   /* IP的HASH值 */      /*72+40=112*/
    uint8  ucForwardAction;
    uint8 ucReserve;
    uint16 usTruncLen;      /* 截断长度 */

    uint16 usDPIProtoId;
    uint16 usDPIAppId;       /* 应用ID, dpi的查询结果 */
    uint32 ulServerReplyDelay;/*NPB填*/
    
    uint8 *layer4data;
    uint8 *layer7data;
    uint64 ullfraghdr;
    uint64 ulSessionID;     /* 会话ID，报文命中会话后，将ID取出，后续挂接报文时，使用此ID挂接 */
    void  *pSess; /* 对应会话指针 FPC填，NPB修改*/
    void  *pFpcSess;/*fpc会话指针，NPB收到报文后将pSees保存到这里*/
    void  *pfrag_sess;/*分片会话指针*//*112+16+8=128+8=136*/
    union {
        st_lake_timer stTimer;
        uint64 ullTimeStamp;    /* 时间戳，mbuf结构中移入 */
    };
    packet pkt; /*转成packet结构*/      /*128+24+72=224*/
        
    union { /* 此共用体由NPB选择使用UUID还是使用ID，等切换到ID后，NPB不在对ucUUID进行赋值即可 */
        struct {
            uint8 ucNetWorkCount;   /*子网络,数量*/
            uint8 ucServiceCount;   /*服务的数量*/
            uint8 szNetWorkId[MAX_SUB_NETWORK_PER_NET];
            uint16 szServiceId[MAX_SESS_SERVICE_TYPE_NUM];
        };
        st_uuid   ucUUID;/*网络UUID*/
    };     /*224+24=248*/
} st_mbuf_pktmsg; /* 报文摘要信息，暂时存放在mbuf的headroom中 */
//mbuf预留区扩充到256字节
```