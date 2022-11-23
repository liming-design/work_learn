npb


mp_npb_connect()--创建一个socket客户端。

```c

```

```c

```

```
main()->mp_cli_create(cli_reg)
->rte_eal_remote(mp_cli_loop,,i)
->npb_pkt_parse(m, i)
->flow_process_packet(tmp, ullRdtscTemp1)
->npb_pkt_handle(pMbufpktmsg, mi);
->npb_proto_proc(pSess, pMbufpktmsg, tmp);
->strategy_judgment(pSess)

strategy_judgment(pSess)
命中返回-1，否则返回0

FlowPutPkt


```

## 数据包：ftp-1.pcapng
`tcpreplay -i vnet1 ftp-1.pcapng`
进入strategy_judgment(pMbufpktmsg->pSess)后的pSess变量;
### \*pSess
167772239{A00 004F}

```
(gdb) p *pSess
$1 = {lnode = {le_next = 0x0, le_prev = 0xb062118}, tnode = {tqe_next = 0x0, tqe_prev = 0x148cd040}, {fivepuple = {ucVersion = 4 '\004', ucDPISend = 0 '\000',
      ucDPIRes = 1 '\001', ucAPPRuleFlag = 0 '\000', ucFlowCapture = 0 '\000', ucProto = 6 '\006', usSPort = 65421, usDPort = 443, usVlanId = 0, unValue = {stValue64 = {
          ullSIpAddr64 = {167772239, 0}, ullDIpAddr64 = {1944877403, 0}}, stValue32 = {ulSIpAddr = {167772239, 0, 0, 0}, ulDIpAddr = {1944877403, 0, 0, 0}}}}, {
      ucVersion = 4 '\004', ucDPINoSend = 0 '\000', ucDPIRes = 1 '\001', ucAPPRuleFlag = 0 '\000', ucFlowCapture = 0 '\000', ucProto = 6 '\006', usSPort = 65421, usDPort = 443,
      usVlanId = 0, unValue = {stValue64 = {ullSIpAddr64 = {167772239, 0}, ullDIpAddr64 = {1944877403, 0}}, stValue32 = {ulSIpAddr = {167772239, 0, 0, 0}, ulDIpAddr = {
            1944877403, 0, 0, 0}}}}}, ulHashValue = 3247661340, ulLastTime = 807404, usAgeTimer = 600, time_wheeling_pos = 260, usDPIProtoId = 12, usServiceId = 0,
  ulServerReplyDelay = 0, interface = 0, usDPIAppId = 211, ulSessionID = 7117191324108849707, pTLSInfo = 0x0, session_stat = {ullupbytes = 0, ulldownbytes = 0,
    ulluppayloadbytes = 0, ulldownpayloadbytes = 0, ulluppackets = 0, ulldownpackets = 0}, tcp_rsmb = 0x0, pparselog_cur = 0x0, startTimer = {tv_sec = 1657101298,
    tv_nsec = 628897231}, stTimer = {tv_sec = 1657101298, tv_nsec = 628897231}, ucTypeFlag = 0 '\000', flag = 0 '\000', uscollection_info_id = 0,
  d_addr = "\004\214\026\202\217\274", s_addr = "ػ\301\216)\204", res_uint16 = 0, {{ucNetWorkCount = 1 '\001', ucServiceCount = 0 '\000', szNetWorkId = "\001\000\230\272",
      szServiceId = {65244, 13775, 9596, 23538}}, ucUUID = {ucUUID = "\001\000\001\000\230\272\334\376\317\065|%\362[\000\000\000\000\000\000"}}}

```
### g_st_collection_conf.pst_collection_conf
```
(gdb) p *g_st_collection_conf.pst_collection_conf
$3 = {pprevious = 0x0, id = "IAuGt4EBjNbcFrHhgtVx", '\000' <repeats 43 times>, name = "123", '\000' <repeats 28 times>, ulnamelen = 3, order_no = 0,
  ip_address = "10.0.0.1/24", '\000' <repeats 500 times>, ip_start = 167772160, ip_end = 167772415, protocol = 263174, level = 0, state = 0, deleted = 0,
  create_time = '\000' <repeats 29 times>, update_time = "2022-07-06 14:40:07.209+08\000\000\000", delete_time = '\000' <repeats 29 times>, netmask = 4294967040,
  ip_addr = 167772161, ip_point_addr = "10.0.0.1", '\000' <repeats 503 times>, pnext = 0x0}

```

### temp
```
(gdb) p *temp
$4 = {pprevious = 0x0, id = "IAuGt4EBjNbcFrHhgtVx", '\000' <repeats 43 times>, name = "123", '\000' <repeats 28 times>, ulnamelen = 3, order_no = 0,
  ip_address = "10.0.0.1/24", '\000' <repeats 500 times>, ip_start = 167772160, ip_end = 167772415, protocol = 263174, level = 0, state = 0, deleted = 0,
  create_time = '\000' <repeats 29 times>, update_time = "2022-07-06 14:40:07.209+08\000\000\000", delete_time = '\000' <repeats 29 times>, netmask = 4294967040,
  ip_addr = 167772161, ip_point_addr = "10.0.0.1", '\000' <repeats 503 times>, pnext = 0x0}

```
## 数据包v6.pcap
`tcpreplay -i vnet1 v6.pcap`
### pSess
72057594155761215         {0x010000000705fe3f}
15744590890054057986 {0xda8005feff860002}
```
$1 = {lnode = {le_next = 0x0, le_prev = 0x17fa550}, tnode = {tqe_next = 0x0, tqe_prev = 0x148d2ed0}, {fivepuple = {ucVersion = 6 '\006', ucDPISend = 0 '\000',
      ucDPIRes = 1 '\001', ucAPPRuleFlag = 0 '\000', ucFlowCapture = 0 '\000', ucProto = 6 '\006', usSPort = 1022, usDPort = 22, usVlanId = 0, unValue = {stValue64 = {
          ullSIpAddr64 = {72057594155761215, 15744590890054057986}, ullDIpAddr64 = {17609383083583, 4468494415821783042}}, stValue32 = {ulSIpAddr = {117833279, 16777216,
            4286971906, 3665823230}, ulDIpAddr = {17169983, 4100, 4292853762, 1040402430}}}}, {ucVersion = 6 '\006', ucDPINoSend = 0 '\000', ucDPIRes = 1 '\001',
      ucAPPRuleFlag = 0 '\000', ucFlowCapture = 0 '\000', ucProto = 6 '\006', usSPort = 1022, usDPort = 22, usVlanId = 0, unValue = {stValue64 = {ullSIpAddr64 = {
            72057594155761215, 15744590890054057986}, ullDIpAddr64 = {17609383083583, 4468494415821783042}}, stValue32 = {ulSIpAddr = {117833279, 16777216, 4286971906,
            3665823230}, ulDIpAddr = {17169983, 4100, 4292853762, 1040402430}}}}}, ulHashValue = 1220707621, ulLastTime = 807982, usAgeTimer = 600, time_wheeling_pos = 237,
  usDPIProtoId = 8, usServiceId = 0, ulServerReplyDelay = 0, interface = 0, usDPIAppId = 203, ulSessionID = 7117198363560248002, pTLSInfo = 0x0, session_stat = {ullupbytes = 0,
    ulldownbytes = 0, ulluppayloadbytes = 0, ulldownpayloadbytes = 0, ulluppackets = 0, ulldownpackets = 0}, tcp_rsmb = 0x0, pparselog_cur = 0x0, startTimer = {
    tv_sec = 1657101877, tv_nsec = 42700706}, stTimer = {tv_sec = 1657101877, tv_nsec = 42700706}, ucTypeFlag = 0 '\000', flag = 0 '\000', uscollection_info_id = 0,
  d_addr = "\000`\227\a", <incomplete sequence \352>, s_addr = "\000\000\206\005\200", <incomplete sequence \332>, res_uint16 = 0, {{ucNetWorkCount = 1 '\001',
      ucServiceCount = 0 '\000', szNetWorkId = "\001\000\230\272", szServiceId = {65244, 36770, 651, 24117}}, ucUUID = {
      ucUUID = "\001\000\001\000\230\272\334\376\242\217\213\002\065^\000\000\000\000\000\000"}}}

```

### g_st_collection_conf.pst_collection_conf
```
(gdb) p *g_st_collection_conf.pst_collection_conf
$2 = {pprevious = 0x0, id = "IAuGt4EBjNbcFrHhgtVx", '\000' <repeats 43 times>, name = "123", '\000' <repeats 28 times>, ulnamelen = 3, order_no = 0,
  ip_address = "10.0.0.1/24", '\000' <repeats 500 times>, ip_start = 167772160, ip_end = 167772415, protocol = 263174, level = 0, state = 0, deleted = 0,
  create_time = '\000' <repeats 29 times>, update_time = "2022-07-06 14:40:07.209+08\000\000\000", delete_time = '\000' <repeats 29 times>, netmask = 4294967040,
  ip_addr = 167772161, ip_point_addr = "10.0.0.1", '\000' <repeats 503 times>, pnext = 0x0}

```
### temp
```
(gdb) p *temp
$3 = {pprevious = 0x0, id = "IAuGt4EBjNbcFrHhgtVx", '\000' <repeats 43 times>, name = "123", '\000' <repeats 28 times>, ulnamelen = 3, order_no = 0,
  ip_address = "10.0.0.1/24", '\000' <repeats 500 times>, ip_start = 167772160, ip_end = 167772415, protocol = 263174, level = 0, state = 0, deleted = 0,
  create_time = '\000' <repeats 29 times>, update_time = "2022-07-06 14:40:07.209+08\000\000\000", delete_time = '\000' <repeats 29 times>, netmask = 4294967040,
  ip_addr = 167772161, ip_point_addr = "10.0.0.1", '\000' <repeats 503 times>, pnext = 0x0}

```
修改
ICMPv6 界面
![image](https://img2022.cnblogs.com/blog/1865959/202207/1865959-20220710234010484-602574321.png)

![image](https://img2022.cnblogs.com/blog/1865959/202207/1865959-20220710235253984-584726878.png)