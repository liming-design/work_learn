### Makefile.am和makefile.in生成Makefile
https://www.cnblogs.com/bigbear1385/p/6604765.html


### Centos 7 离线安装 gcc 4.8.5
https://blog.csdn.net/xyy1028/article/details/103745702

```bash
chmod +x configure
rm -rf /opt/components/suricata
./configure --with-libhs-includes=/home/hyperscan/include --with-libhs-libraries=/home/hyperscan --enable-lua --enable-geoip --prefix=/opt/components/suricata --enable-dpdk  --with-libzmq-includes=/home/zmq/include --with-libzmq-libraries=/home/zmq
make -j32
make install
make install-conf
tar -zcvf suricata.tar.gz /opt/components/suricata
rm -rf suricata-install
mkdir suricata-install

```

gcc版本
```bash
Error: Package: rust-1.62.1-1.el7.x86_64 (epel)
           Requires: /usr/bin/cc
           Available: gcc-4.8.5-44.el7.x86_64 (base)
               Not found
           Installed: gcc-9.2.0-1.el7.centos.x86_64 (installed)
               Not found
 You could try using --skip-broken to work around the problem
** Found 4 pre-existing rpmdb problem(s), 'yum check' output follow  

[root@localhost suricata-6.0.3]# gcc -v
-bash: /usr/local/bin/gcc: No such file or directory
```
#### gcc 版本换回原来的版本
```
yum install centos-release-scl --可不做
yum install devtoolset-10-gcc*
scl enable devtoolset-10 bash
```


### 日志文件的位置
打开eve.log

```bash
vi /opt/components/suricata/etc/suricata/suricata.yaml
修改eve.log yes filename=eve.log filetype=regular
修改fast.log yes
重启suricata
日志输出路径
ls /opt/components/suricata/var/log/suricata/

vi /opt/components/suricata/suricata.rules
tail -F /opt/components/suricata/var/log/suricata/eve.log | grep 500000001
```


## 改动记录
output-json.c CreateEveHeader
```
    // if ((memcmp(event_type, "alert", 5) != 0) && (memcmp(event_type, "fileinfo", 8) != 0))
    //     return NULL;
```
## 调试结果
### ftp-full.pcapng
```bash
gdb ./src/.libs/suricata
b FTPParseRequest

run -r /root/Work/packet/ftp-full.pcapng  -l /root/Work/suricata_output/ -S /root/Work/suricata_rules/many_rules.rules
(gdb) bt
#0  FTPParseRequest (f=0x283f270, ftp_state=0x7fffc0543be0, pstate=0x7fffc053f410, input=0x7fffc0545320 "USER Admin\r\n", input_len=12, local_data=0x7fffc02824d0,
    flags=5 '\005') at app-layer-ftp.c:562
#1  0x00000000005c463c in AppLayerParserParse (tv=tv@entry=0x2a1db10, alp_tctx=0x7fffc02820e0, f=f@entry=0x283f270, alproto=2, flags=flags@entry=5 '\005',
    input=input@entry=0x7fffc0545320 "USER Admin\r\n", input_len=input_len@entry=12) at app-layer-parser.c:1269
#2  0x00000000005a2005 in TCPProtoDetect (tv=<optimized out>, ra_ctx=<optimized out>, app_tctx=app_tctx@entry=0x7fffc0281d50, p=p@entry=0x7fffe419c6c0,
    f=f@entry=0x283f270, ssn=ssn@entry=0x7fffc0344e70, stream=stream@entry=0x7fffd1ff9658, data=data@entry=0x7fffc0545320 "USER Admin\r\n", data_len=data_len@entry=12,
    flags=<optimized out>, flags@entry=5 '\005') at app-layer.c:456
#3  0x00000000005a2815 in AppLayerHandleTCPData (tv=tv@entry=0x2a1db10, ra_ctx=ra_ctx@entry=0x7fffc0281d20, p=p@entry=0x7fffe419c6c0, f=0x283f270,
    ssn=ssn@entry=0x7fffc0344e70, stream=stream@entry=0x7fffd1ff9658, data=0x7fffc0545320 "USER Admin\r\n", data_len=data_len@entry=12, flags=flags@entry=5 '\005')
    at app-layer.c:642
#4  0x000000000068fdb2 in ReassembleUpdateAppLayer (dir=UPDATE_DIR_OPPOSING, p=0x7fffe419c6c0, stream=0x7fffd1ff9658, ssn=0x7fffc0344e70, ra_ctx=0x7fffc0281d20,
    tv=0x2a1db10) at stream-tcp-reassemble.c:1173
#5  StreamTcpReassembleAppLayer (tv=tv@entry=0x2a1db10, ra_ctx=ra_ctx@entry=0x7fffc0281d20, ssn=ssn@entry=0x7fffc0344e70, stream=stream@entry=0x7fffc0344f00,
    p=p@entry=0x7fffe419c6c0, dir=dir@entry=UPDATE_DIR_OPPOSING) at stream-tcp-reassemble.c:1236
#6  0x0000000000690cbe in StreamTcpReassembleHandleSegmentUpdateACK (p=0x7fffe419c6c0, stream=0x7fffc0344f00, ssn=0x7fffc0344e70, ra_ctx=0x7fffc0281d20, tv=0x2a1db10)
    at stream-tcp-reassemble.c:1805
#7  StreamTcpReassembleHandleSegment (tv=tv@entry=0x2a1db10, ra_ctx=0x7fffc0281d20, ssn=ssn@entry=0x7fffc0344e70, stream=0x7fffc0344e80, p=p@entry=0x7fffe419c6c0,
    pq=<optimized out>) at stream-tcp-reassemble.c:1850
#8  0x000000000068a307 in StreamTcpPacketStateTimeWait (stt=<optimized out>, pq=<optimized out>, ssn=<optimized out>, p=<optimized out>, tv=<optimized out>)
    at stream-tcp.c:4321
#9  StreamTcpStateDispatch (tv=0x2a1db10, p=0x7fffe419c6c0, stt=0x7fffc0281a10, ssn=0x7fffc0344e70, pq=<optimized out>, state=<optimized out>) at stream-tcp.c:4739
#10 0x000000000068c2e8 in StreamTcpPacket (tv=0x2a1db10, p=0x7fffe419c6c0, stt=0x7fffc0281a10, pq=0x7fffc02808f0) at stream-tcp.c:4890
#11 0x000000000068ce60 in StreamTcp (tv=tv@entry=0x2a1db10, p=p@entry=0x7fffe419c6c0, data=<optimized out>, pq=pq@entry=0x7fffc02808f0) at stream-tcp.c:5226
#12 0x0000000000648d75 in FlowWorkerStreamTCPUpdate (detect_thread=0x7fffc0352cd0, p=0x7fffe419c6c0, fw=0x7fffc02808c0, tv=0x2a1db10) at flow-worker.c:364
#13 FlowWorker (tv=0x2a1db10, p=0x7fffe419c6c0, data=0x7fffc02808c0) at flow-worker.c:524
#14 0x00000000006980ae in TmThreadsSlotVarRun (tv=tv@entry=0x2a1db10, p=p@entry=0x7fffe419c6c0, slot=slot@entry=0x2a1dc30) at tm-threads.c:117
#15 0x0000000000699cac in TmThreadsSlotVar (td=0x2a1db10) at tm-threads.c:452
#16 0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#17 0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6

```
#### JsonFTPLogger
```bash
(gdb) bt
#0  JsonFTPLogger (tv=0x323b030, thread_data=0x7fffbc392150, p=0x7fffe427c2c0, f=0x31c09a0, state=0x7fffbc479020, vtx=0x7fffbc4790b0, tx_id=0)
    at output-json-ftp.c:151
#1  0x000000000085eb6f in OutputTxLog (tv=0x323b030, p=0x7fffe427c2c0, thread_data=0x7fffbc381fe0) at output-tx.c:298
#2  0x0000000000842e64 in OutputLoggerLog (tv=tv@entry=0x323b030, p=p@entry=0x7fffe427c2c0, thread_data=<optimized out>) at output.c:883
#3  0x0000000000834f67 in FlowWorker (tv=0x323b030, p=0x7fffe427c2c0, data=0x7fffbc2808c0) at flow-worker.c:545
#4  0x00000000008a724e in TmThreadsSlotVarRun (tv=tv@entry=0x323b030, p=p@entry=0x7fffe427c2c0, slot=slot@entry=0x323b150) at tm-threads.c:117
#5  0x00000000008a936f in TmThreadsSlotVar (td=0x323b030) at tm-threads.c:452
#6  0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#7  0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6

(gdb) p *f
$1 = {src = {address = {address_un_data32 = {2432696330, 0, 0, 0}, address_un_data16 = {10, 37120, 0, 0, 0, 0, 0, 0},
      address_un_data8 = "\n\000\000\221", '\000' <repeats 11 times>}}, dst = {address = {address_un_data32 = {1325400074, 0, 0, 0}, address_un_data16 = {10
        20224, 0, 0, 0, 0, 0, 0}, address_un_data8 = "\n\000\000O", '\000' <repeats 11 times>}}, {sp = 35578, icmp_s = {type = 250 '\372', code = 138 '\212'
    dp = 21, icmp_d = {type = 21 '\025', code = 0 '\000'}}, proto = 6 '\006', recursion_level = 0 '\000', vlan_id = {0, 0}, use_cnt = 1, vlan_idx = 0 '\000'
      ffr_ts = 0 '\000', ffr_tc = 0 '\000'}, ffr = 0 '\000'}, timeout_at = 1656487760, thread_id = {11, 11}, next = 0x0, livedev = 0x0, flow_hash = 19503450
  lastts = {tv_sec = 1656487160, tv_usec = 157626}, timeout_policy = 600, flow_state = 1, tenant_id = 0, probing_parser_toserver_alproto_masks = 0,
  probing_parser_toclient_alproto_masks = 0, flags = 14099227, file_flags = 4095, protodetect_dp = 0, parent_id = 0, m = {__data = {__lock = 1, __count = 0,
      __owner = 22372, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}},
    __size = "\001\000\000\000\000\000\000\000dW\000\000\001", '\000' <repeats 26 times>, __align = 1}, protoctx = 0x7fffbc35ac00, protomap = 0 '\000',
  flow_end_flags = 0 '\000', alproto = 2, alproto_ts = 2, alproto_tc = 31, alproto_orig = 0, alproto_expect = 0, de_ctx_version = 99, min_ttl_toserver = 64
  max_ttl_toserver = 64 '@', min_ttl_toclient = 128 '\200', max_ttl_toclient = 128 '\200', alparser = 0x7fffbc478fe0, alstate = 0x7fffbc479020,
  sgh_toclient = 0x0, sgh_toserver = 0x329d340, flowvar = 0x0, fb = 0x7fffef475680, startts = {tv_sec = 1656487153, tv_usec = 837798}, todstpktcnt = 4,
  tosrcpktcnt = 3, todstbytecnt = 260, tosrcbytecnt = 224, ulsessionID = 0, uuid = '\000' <repeats 20 times>, subtask_uuid = '\000' <repeats 20 times>}

(gdb) *(FTPTransaction*)vtx
Undefined command: "".  Try "help".
(gdb) p *(FTPTransaction*)vtx
$3 = {tx_id = 0, tx_data = {config = {log_flags = 0 '\000'}, logged = {flags = 0}, detect_flags_ts = 0, detect_flags_tc = 9223372036854775808},
  request_length = 0, request = 0x0, command_descriptor = 0x10bb5e0 <FtpCommands+1152>, dyn_port = 0, done = true, active = false, direction = 0 '\000',
  response_list = {tqh_first = 0x7fffbc479120, tqh_last = 0x7fffbc479130}, de_state = 0x0, next = {tqe_next = 0x7fffbc4791c0, tqe_prev = 0x7fffbc479038}}
(gdb)

```
##### 分析
0. JsonFTPLogger
1. OutputTxLog
2. OutputLoggerLog
3. FlowWorker
4. TmThreadsSlotVarRun
5. TmThreadsSlotVar
6. start_thread
7. clone ()


### pop3-full
#### Pop3ParseResponse

```bash
步骤：
make clean
vi MakeFile
make
gdb ./src/.libs/suricata
b Pop3ParseRequest
run -r /root/Work/packet/pop3-full.pcap  -l /root/Work/suricata_output/ -S /root/Work/suricata_rules/many_rules.rules
b app-layer-pop3.c:342
Breakpoint 3, Pop3ParseResponse (f=<optimized out>, statev=0x7fffc4479d70,
    pstate=0x7fffc436aa90,
    input=0x7fffc4479560 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    input_len=54, local_data=<optimized out>, flags=9 '\t') at app-layer-pop3.c:342
342         TAILQ_FOREACH(ttx, &state->tx_list, next) {

(gdb) bt
#0  Pop3ParseResponse (f=<optimized out>, statev=0x7fffc4479d70,
    pstate=0x7fffc436aa90,
    input=0x7fffc4479560 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    input_len=54, local_data=<optimized out>, flags=9 '\t') at app-layer-pop3.c:342
#1  0x00000000005c47b6 in AppLayerParserParse (tv=tv@entry=0x2a6d2d0,
    alp_tctx=0x7fffc4282100, f=f@entry=0x29a79c0, alproto=26,
    flags=flags@entry=9 '\t',
    input=input@entry=0x7fffc4479560 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n", input_len=input_len@entry=54) at app-layer-parser.c:1270
#2  0x00000000005a20bd in TCPProtoDetect (tv=<optimized out>,
    ra_ctx=<optimized out>, app_tctx=app_tctx@entry=0x7fffc4281d70,
    p=p@entry=0x7fffe427d6c0, f=f@entry=0x29a79c0, ssn=ssn@entry=0x7fffc4352bf0,
    stream=stream@entry=0x7fffd37fc658,
    data=data@entry=0x7fffc4479560 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n", data_len=data_len@entry=54, flags=<optimized out>, flags@entry=9 '\t')
    at app-layer.c:456
#3  0x00000000005a28c5 in AppLayerHandleTCPData (tv=tv@entry=0x2a6d2d0,
    ra_ctx=ra_ctx@entry=0x7fffc4281d40, p=p@entry=0x7fffe427d6c0, f=0x29a79c0,
    ssn=ssn@entry=0x7fffc4352bf0, stream=stream@entry=0x7fffd37fc658,
    data=0x7fffc4479560 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    data_len=data_len@entry=54, flags=flags@entry=9 '\t') at app-layer.c:642
#4  0x0000000000690d72 in ReassembleUpdateAppLayer (dir=UPDATE_DIR_OPPOSING,
    p=0x7fffe427d6c0, stream=0x7fffd37fc658, ssn=0x7fffc4352bf0,
    ra_ctx=0x7fffc4281d40, tv=0x2a6d2d0) at stream-tcp-reassemble.c:1173
#5  StreamTcpReassembleAppLayer (tv=tv@entry=0x2a6d2d0,
    ra_ctx=ra_ctx@entry=0x7fffc4281d40, ssn=ssn@entry=0x7fffc4352bf0,
    stream=stream@entry=0x7fffc4352c00, p=p@entry=0x7fffe427d6c0,
    dir=dir@entry=UPDATE_DIR_OPPOSING) at stream-tcp-reassemble.c:1236
#6  0x0000000000691c7e in StreamTcpReassembleHandleSegmentUpdateACK (
    p=0x7fffe427d6c0, stream=0x7fffc4352c00, ssn=0x7fffc4352bf0,
    ra_ctx=0x7fffc4281d40, tv=0x2a6d2d0) at stream-tcp-reassemble.c:1805
---Type <return> to continue, or q <return> to quit---
#7  StreamTcpReassembleHandleSegment (tv=tv@entry=0x2a6d2d0, ra_ctx=0x7fffc4281d40,
    ssn=ssn@entry=0x7fffc4352bf0, stream=0x7fffc4352c80, p=p@entry=0x7fffe427d6c0,
    pq=<optimized out>) at stream-tcp-reassemble.c:1850
#8  0x000000000068b2c7 in StreamTcpPacketStateTimeWait (stt=<optimized out>,
    pq=<optimized out>, ssn=<optimized out>, p=<optimized out>, tv=<optimized out>)
    at stream-tcp.c:4321
#9  StreamTcpStateDispatch (tv=0x2a6d2d0, p=0x7fffe427d6c0, stt=0x7fffc4281a30,
    ssn=0x7fffc4352bf0, pq=<optimized out>, state=<optimized out>)
    at stream-tcp.c:4739
#10 0x000000000068d2a8 in StreamTcpPacket (tv=0x2a6d2d0, p=0x7fffe427d6c0,
    stt=0x7fffc4281a30, pq=0x7fffc42808f0) at stream-tcp.c:4890
#11 0x000000000068de20 in StreamTcp (tv=tv@entry=0x2a6d2d0,
    p=p@entry=0x7fffe427d6c0, data=<optimized out>, pq=pq@entry=0x7fffc42808f0)
    at stream-tcp.c:5226
#12 0x0000000000649a55 in FlowWorkerStreamTCPUpdate (detect_thread=0x7fffc4352d10,
    p=0x7fffe427d6c0, fw=0x7fffc42808c0, tv=0x2a6d2d0) at flow-worker.c:364
#13 FlowWorker (tv=0x2a6d2d0, p=0x7fffe427d6c0, data=0x7fffc42808c0)
    at flow-worker.c:524
#14 0x000000000069906e in TmThreadsSlotVarRun (tv=tv@entry=0x2a6d2d0,
    p=p@entry=0x7fffe427d6c0, slot=slot@entry=0x2a6d3f0) at tm-threads.c:117
#15 0x000000000069ac6c in TmThreadsSlotVar (td=0x2a6d2d0) at tm-threads.c:452
#16 0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#17 0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6

```
##### 分析
0. Pop3ParseResponse
1. AppLayerParserParse at app-layer-parser.c:1270
2. TCPProtoDetect at app-layer.c:456
3. AppLayerHandleTCPData at stream-tcp-reassemble.c:1173
4. ReassembleUpdateAppLayer 
5. StreamTcpReassembleAppLayer 
6. StreamTcpReassembleHandleSegmentUpdateACK
7. StreamTcpReassembleHandleSegment
8. StreamTcpPacketStateTimeWait
9. StreamTcpStateDispatch
10. StreamTcpPacket
11. StreamTcp
12. FlowWorkerStreamTCPUpdate
13. FlowWorker
14. TmThreadsSlotVarRun
15. TmThreadsSlotVar

#### Pop3ParseRequest
```bash
Breakpoint 2, Pop3ParseRequest (f=0x29a79c0, statev=0x7fffc4479d70,
    pstate=0x7fffc436aa90, input=0x7fffc4479d90 "user 812578452@qq.com\r\n",
    input_len=23, local_data=0x0, flags=5 '\005') at app-layer-pop3.c:246
246         SCLogNotice("Parsing pop3 request: len=%"PRIu32, input_len);
(gdb) bt
#0  Pop3ParseRequest (f=0x29a79c0, statev=0x7fffc4479d70, pstate=0x7fffc436aa90,
    input=0x7fffc4479d90 "user 812578452@qq.com\r\n", input_len=23, local_data=0x0,
    flags=5 '\005') at app-layer-pop3.c:246
#1  0x00000000005c47b6 in AppLayerParserParse (tv=tv@entry=0x2a6d2d0,
    alp_tctx=0x7fffc4282100, f=f@entry=0x29a79c0, alproto=26,
    flags=flags@entry=5 '\005',
    input=input@entry=0x7fffc4479d90 "user 812578452@qq.com\r\n",
    input_len=input_len@entry=23) at app-layer-parser.c:1270
#2  0x00000000005a20bd in TCPProtoDetect (tv=<optimized out>,
    ra_ctx=<optimized out>, app_tctx=app_tctx@entry=0x7fffc4281d70,
    p=p@entry=0x7fffe427c2c0, f=f@entry=0x29a79c0, ssn=ssn@entry=0x7fffc4352bf0,
    stream=stream@entry=0x7fffd37fc658,
    data=data@entry=0x7fffc4479d90 "user 812578452@qq.com\r\n",
    data_len=data_len@entry=23, flags=<optimized out>, flags@entry=5 '\005')
    at app-layer.c:456
#3  0x00000000005a28c5 in AppLayerHandleTCPData (tv=tv@entry=0x2a6d2d0,
    ra_ctx=ra_ctx@entry=0x7fffc4281d40, p=p@entry=0x7fffe427c2c0, f=0x29a79c0,
    ssn=ssn@entry=0x7fffc4352bf0, stream=stream@entry=0x7fffd37fc658,
    data=0x7fffc4479d90 "user 812578452@qq.com\r\n", data_len=data_len@entry=23,
    flags=flags@entry=5 '\005') at app-layer.c:642
#4  0x0000000000690d72 in ReassembleUpdateAppLayer (dir=UPDATE_DIR_OPPOSING,
    p=0x7fffe427c2c0, stream=0x7fffd37fc658, ssn=0x7fffc4352bf0,
    ra_ctx=0x7fffc4281d40, tv=0x2a6d2d0) at stream-tcp-reassemble.c:1173
#5  StreamTcpReassembleAppLayer (tv=tv@entry=0x2a6d2d0,
    ra_ctx=ra_ctx@entry=0x7fffc4281d40, ssn=ssn@entry=0x7fffc4352bf0,
    stream=stream@entry=0x7fffc4352c80, p=p@entry=0x7fffe427c2c0,
    dir=dir@entry=UPDATE_DIR_OPPOSING) at stream-tcp-reassemble.c:1236
#6  0x0000000000691c7e in StreamTcpReassembleHandleSegmentUpdateACK (
    p=0x7fffe427c2c0, stream=0x7fffc4352c80, ssn=0x7fffc4352bf0,
    ra_ctx=0x7fffc4281d40, tv=0x2a6d2d0) at stream-tcp-reassemble.c:1805
#7  StreamTcpReassembleHandleSegment (tv=tv@entry=0x2a6d2d0, ra_ctx=0x7fffc4281d40,
---Type <return> to continue, or q <return> to quit---
    ssn=ssn@entry=0x7fffc4352bf0, stream=0x7fffc4352c00, p=p@entry=0x7fffe427c2c0,
    pq=<optimized out>) at stream-tcp-reassemble.c:1850
#8  0x000000000068b2c7 in StreamTcpPacketStateTimeWait (stt=<optimized out>,
    pq=<optimized out>, ssn=<optimized out>, p=<optimized out>, tv=<optimized out>)
    at stream-tcp.c:4321
#9  StreamTcpStateDispatch (tv=0x2a6d2d0, p=0x7fffe427c2c0, stt=0x7fffc4281a30,
    ssn=0x7fffc4352bf0, pq=<optimized out>, state=<optimized out>)
    at stream-tcp.c:4739
#10 0x000000000068d2a8 in StreamTcpPacket (tv=0x2a6d2d0, p=0x7fffe427c2c0,
    stt=0x7fffc4281a30, pq=0x7fffc42808f0) at stream-tcp.c:4890
#11 0x000000000068de20 in StreamTcp (tv=tv@entry=0x2a6d2d0,
    p=p@entry=0x7fffe427c2c0, data=<optimized out>, pq=pq@entry=0x7fffc42808f0)
    at stream-tcp.c:5226
#12 0x0000000000649a55 in FlowWorkerStreamTCPUpdate (detect_thread=0x7fffc4352d10,
    p=0x7fffe427c2c0, fw=0x7fffc42808c0, tv=0x2a6d2d0) at flow-worker.c:364
#13 FlowWorker (tv=0x2a6d2d0, p=0x7fffe427c2c0, data=0x7fffc42808c0)
    at flow-worker.c:524
#14 0x000000000069906e in TmThreadsSlotVarRun (tv=tv@entry=0x2a6d2d0,
    p=p@entry=0x7fffe427c2c0, slot=slot@entry=0x2a6d3f0) at tm-threads.c:117
#15 0x000000000069ac6c in TmThreadsSlotVar (td=0x2a6d2d0) at tm-threads.c:452
#16 0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#17 0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6
```
##### 分析


#### Pop3ProbingParserTc
```bash

Breakpoint 2, Pop3ProbingParserTc (f=0x31c09d0, direction=9 '\t',
    input=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    input_len=54, rdir=0x7fffd1ff9350 "") at app-layer-pop3.c:247
247         if (input_len >= POP3_MIN_FRAME_LEN) {
(gdb) bt
#0  Pop3ProbingParserTc (f=0x31c09d0, direction=9 '\t',
    input=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    input_len=54, rdir=0x7fffd1ff9350 "") at app-layer-pop3.c:247
#1  0x00000000007623f1 in PPGetProto (rdir=0x7fffd1ff9350 "", alproto_masks=0x31c0a40,
    buflen=54,
    buf=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    flags=0 '\000', f=0x31c09d0, pe=0x1733d20) at app-layer-detect-proto.c:478
#2  AppLayerProtoDetectPPGetProto (reverse_flow=<synthetic pointer>, flags=9 '\t',
    ipproto=6 '\006', buflen=54,
    buf=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    f=0x31c09d0) at app-layer-detect-proto.c:588
#3  AppLayerProtoDetectGetProto (tctx=<optimized out>, f=f@entry=0x31c09d0,
    buf=buf@entry=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",                                                                  buflen=buflen@entry=54, ipproto=ipproto@entry=6 '\006', flags=flags@entry=9 '\t',
    reverse_flow=reverse_flow@entry=0x7fffd1ff942f) at app-layer-detect-proto.c:1563
#4  0x000000000075f1df in TCPProtoDetect (tv=tv@entry=0x323b840,
    ra_ctx=ra_ctx@entry=0x7fffc0281d40, app_tctx=app_tctx@entry=0x7fffc0281d70,
    p=p@entry=0x7fffe427d6c0, f=f@entry=0x31c09d0, ssn=ssn@entry=0x7fffc035ac00,
    stream=stream@entry=0x7fffd1ff9528,
    data=data@entry=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n"                                                                 , data_len=data_len@entry=54, flags=9 '\t') at app-layer.c:336
#5  0x00000000007601f2 in AppLayerHandleTCPData (tv=tv@entry=0x323b840,
    ra_ctx=ra_ctx@entry=0x7fffc0281d40, p=p@entry=0x7fffe427d6c0, f=0x31c09d0,
    ssn=ssn@entry=0x7fffc035ac00, stream=stream@entry=0x7fffd1ff9528,
    data=0x7fffc0481580 "+OK XMail POP3 Server v1.0 Service Ready(XMail v1.0)\r\n",
    data_len=54, flags=flags@entry=9 '\t') at app-layer.c:642
#6  0x000000000089cebc in ReassembleUpdateAppLayer (dir=UPDATE_DIR_OPPOSING,
    p=0x7fffe427d6c0, stream=0x7fffd1ff9528, ssn=0x7fffc035ac00, ra_ctx=0x7fffc0281d40,
    tv=0x323b840) at stream-tcp-reassemble.c:1173
#7  StreamTcpReassembleAppLayer (tv=tv@entry=0x323b840, ra_ctx=ra_ctx@entry=0x7fffc0281d40,
    ssn=ssn@entry=0x7fffc035ac00, stream=stream@entry=0x7fffc035ac10,
    p=p@entry=0x7fffe427d6c0, dir=dir@entry=UPDATE_DIR_OPPOSING)
---Type <return> to continue, or q <return> to quit---
    at stream-tcp-reassemble.c:1236
#8  0x000000000089ea63 in StreamTcpReassembleHandleSegmentUpdateACK (p=0x7fffe427d6c0,
    stream=0x7fffc035ac10, ssn=0x7fffc035ac00, ra_ctx=0x7fffc0281d40, tv=0x323b840)
    at stream-tcp-reassemble.c:1805
#9  StreamTcpReassembleHandleSegment (tv=tv@entry=0x323b840, ra_ctx=0x7fffc0281d40,
    ssn=ssn@entry=0x7fffc035ac00, stream=stream@entry=0x7fffc035ac98,
    p=p@entry=0x7fffe427d6c0, pq=pq@entry=0x7fffc0281a38) at stream-tcp-reassemble.c:1850
#10 0x000000000087f02d in HandleEstablishedPacketToServer (tv=tv@entry=0x323b840,
    ssn=ssn@entry=0x7fffc035ac00, p=p@entry=0x7fffe427d6c0, pq=pq@entry=0x7fffc0281a38,
    stt=0x7fffc0281a30) at stream-tcp.c:2318
#11 0x00000000008838d5 in StreamTcpPacketStateEstablished (tv=tv@entry=0x323b840,
    p=p@entry=0x7fffe427d6c0, stt=stt@entry=0x7fffc0281a30, ssn=ssn@entry=0x7fffc035ac00,
    pq=pq@entry=0x7fffc0281a38) at stream-tcp.c:2688
#12 0x00000000008967a4 in StreamTcpStateDispatch (state=4 '\004', pq=0x7fffc0281a38,
    ssn=0x7fffc035ac00, stt=0x7fffc0281a30, p=0x7fffe427d6c0, tv=0x323b840)
    at stream-tcp.c:4703
#13 StreamTcpPacket (tv=tv@entry=0x323b840, p=p@entry=0x7fffe427d6c0,
    stt=stt@entry=0x7fffc0281a30, pq=pq@entry=0x7fffc02808f0) at stream-tcp.c:4890
#14 0x0000000000897980 in StreamTcp (tv=tv@entry=0x323b840, p=p@entry=0x7fffe427d6c0,
    data=0x7fffc0281a30, pq=pq@entry=0x7fffc02808f0) at stream-tcp.c:5226
#15 0x0000000000834e50 in FlowWorkerStreamTCPUpdate (detect_thread=0x7fffc035ad30,
    p=0x7fffe427d6c0, fw=0x7fffc02808c0, tv=0x323b840) at flow-worker.c:364
#16 FlowWorker (tv=0x323b840, p=0x7fffe427d6c0, data=0x7fffc02808c0) at flow-worker.c:524
#17 0x00000000008a727e in TmThreadsSlotVarRun (tv=tv@entry=0x323b840,
    p=p@entry=0x7fffe427d6c0, slot=slot@entry=0x323b960) at tm-threads.c:117
#18 0x00000000008a939f in TmThreadsSlotVar (td=0x323b840) at tm-threads.c:452
#19 0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#20 0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6

```

#### JsonPop3Logger
```bash
(gdb) bt
#0  JsonPop3Logger (tv=0x3299a60, thread_data=0x7fffd84525d0, p=0x7fffe427aec0, f=0x31c09a0, state=0x7fffd8478810, tx=0x7fffd8479080, tx_id=0)
    at output-json-pop3.c:69
#1  0x000000000085eb6f in OutputTxLog (tv=0x3299a60, p=0x7fffe427aec0, thread_data=0x7fffd8381fe0) at output-tx.c:298
#2  0x0000000000842e64 in OutputLoggerLog (tv=tv@entry=0x3299a60, p=p@entry=0x7fffe427aec0, thread_data=<optimized out>) at output.c:883
#3  0x0000000000834f67 in FlowWorker (tv=0x3299a60, p=0x7fffe427aec0, data=0x7fffd82808c0) at flow-worker.c:545
#4  0x00000000008a724e in TmThreadsSlotVarRun (tv=tv@entry=0x3299a60, p=p@entry=0x7fffe427aec0, slot=slot@entry=0x3299b80) at tm-threads.c:117
#5  0x00000000008a936f in TmThreadsSlotVar (td=0x3299a60) at tm-threads.c:452
#6  0x00007ffff46ecea5 in start_thread () from /lib64/libpthread.so.0
#7  0x00007ffff3ffb9fd in clone () from /lib64/libc.so.6


Breakpoint 2, JsonPop3Logger (tv=0x3299a60, thread_data=0x7fffd84525d0, p=0x7fffe427aec0, f=0x31c09a0, state=0x7fffd8478810, tx=0x7fffd8479080, tx_id=0)
    at output-json-pop3.c:69
69      {
(gdb) p *f
$4 = {src = {address = {address_un_data32 = {2265360700, 0, 0, 0}, address_un_data16 = {43324, 34566, 0, 0, 0, 0, 0, 0},
      address_un_data8 = "<\251\006\207", '\000' <repeats 11 times>}}, dst = {address = {address_un_data32 = {719389623, 0, 0, 0}, address_un_data16 = {951,
        10977, 0, 0, 0, 0, 0, 0}, address_un_data8 = "\267\003\341*", '\000' <repeats 11 times>}}, {sp = 42492, icmp_s = {type = 252 '\374', code = 165 '\245'}}, {
    dp = 110, icmp_d = {type = 110 'n', code = 0 '\000'}}, proto = 6 '\006', recursion_level = 0 '\000', vlan_id = {0, 0}, use_cnt = 1, vlan_idx = 0 '\000', {{
      ffr_ts = 0 '\000', ffr_tc = 0 '\000'}, ffr = 0 '\000'}, timeout_at = 1598857528, thread_id = {5, 5}, next = 0x0, livedev = 0x0, flow_hash = 3864012143,
  lastts = {tv_sec = 1598856928, tv_usec = 114399}, timeout_policy = 600, flow_state = 1, tenant_id = 0, probing_parser_toserver_alproto_masks = 0,
  probing_parser_toclient_alproto_masks = 0, flags = 1123099, file_flags = 4095, protodetect_dp = 0, parent_id = 0, m = {__data = {__lock = 1, __count = 0,
      __owner = 32256, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}},
    __size = "\001\000\000\000\000\000\000\000\000~\000\000\001", '\000' <repeats 26 times>, __align = 1}, protoctx = 0x7fffd835ac00, protomap = 0 '\000',
  flow_end_flags = 0 '\000', alproto = 26, alproto_ts = 2, alproto_tc = 26, alproto_orig = 0, alproto_expect = 0, de_ctx_version = 99, min_ttl_toserver = 64 '@',
  max_ttl_toserver = 64 '@', min_ttl_toclient = 53 '5', max_ttl_toclient = 53 '5', alparser = 0x7fffd84787d0, alstate = 0x7fffd8478810, sgh_toclient = 0x0,
  sgh_toserver = 0x329d340, flowvar = 0x0, fb = 0x7fffeb56dc00, startts = {tv_sec = 1598856916, tv_usec = 355567}, todstpktcnt = 5, tosrcpktcnt = 4,
  todstbytecnt = 313, tosrcbytecnt = 294, ulsessionID = 0, uuid = '\000' <repeats 20 times>, subtask_uuid = '\000' <repeats 20 times>}

(gdb) p *(Pop3Transaction*) tx
$5 = {tx_id = 0, decoder_events = 0x0, request_buffer = 0x7fffd84790f0 "user 812578452@qq.com\r\n", request_buffer_len = 23,
  response_buffer = 0x7fffd847a120 "+OK\r\n", response_buffer_len = 5, response_done = 1 '\001', de_state = 0x0, tx_data = {config = {log_flags = 0 '\000'},
    logged = {flags = 0}, detect_flags_ts = 9223372036854775808, detect_flags_tc = 0}, next = {tqe_next = 0x0, tqe_prev = 0x7fffd8478810}}
(gdb)

```


## pop3-manyCC.pcap
### Pop3GetStateProgress
```bash
Pop3GetStateProgress(void * txv, uint8_t direction) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-pop3.c:755)
AppLayerParserGetStateProgress(uint8_t ipproto, AppProto alproto, void * alstate, uint8_t flags) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-parser.c:1054)
AppLayerParserSetTransactionInspectId(const Flow * f, AppLayerParserState * pstate, void * alstate, const uint8_t flags, _Bool tag_txs_as_inspected) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-parser.c:764)
DeStateUpdateInspectTransactionId(Flow * f, const uint8_t flags, const _Bool tag_txs_as_inspected) (\tmp\share\code\Suricata\suricata-6.0.3\src\detect-engine-state.c:256)
DetectRunPostRules(ThreadVars * tv, DetectEngineCtx * de_ctx, DetectEngineThreadCtx * det_ctx, Packet * const p, Flow * const pflow, DetectRunScratchpad * scratch) (\tmp\share\code\Suricata\suricata-6.0.3\src\detect.c:931)
DetectRun(ThreadVars * th_v, DetectEngineCtx * de_ctx, DetectEngineThreadCtx * det_ctx, Packet * p) (\tmp\share\code\Suricata\suricata-6.0.3\src\detect.c:141)
DetectFlow(ThreadVars * tv, DetectEngineCtx * de_ctx, DetectEngineThreadCtx * det_ctx, Packet * p) (\tmp\share\code\Suricata\suricata-6.0.3\src\detect.c:1599)
Detect(ThreadVars * tv, Packet * p, void * data) (\tmp\share\code\Suricata\suricata-6.0.3\src\detect.c:1673)
FlowWorker(ThreadVars * tv, Packet * p, void * data) (\tmp\share\code\Suricata\suricata-6.0.3\src\flow-worker.c:540)
TmThreadsSlotVarRun(ThreadVars * tv, Packet * p, TmSlot * slot) (\tmp\share\code\Suricata\suricata-6.0.3\src\tm-threads.c:117)
TmThreadsSlotVar(void * td) (\tmp\share\code\Suricata\suricata-6.0.3\src\tm-threads.c:452)
libpthread.so.0!start_thread (Unknown Source:0)
libc.so.6!clone (Unknown Source:0)
```

### Pop3TxFree
```bash
Pop3TxFree(void * txv) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-pop3.c:116)
Pop3StateTxFree(void * statev, uint64_t tx_id) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-pop3.c:177)
AppLayerParserTransactionsCleanup(Flow * f) (\tmp\share\code\Suricata\suricata-6.0.3\src\app-layer-parser.c:1002)
FlowWorker(ThreadVars * tv, Packet * p, void * data) (\tmp\share\code\Suricata\suricata-6.0.3\src\flow-worker.c:562)
TmThreadsSlotVarRun(ThreadVars * tv, Packet * p, TmSlot * slot) (\tmp\share\code\Suricata\suricata-6.0.3\src\tm-threads.c:117)
TmThreadsSlotVar(void * td) (\tmp\share\code\Suricata\suricata-6.0.3\src\tm-threads.c:452)
libpthread.so.0!start_thread (Unknown Source:0)
libc.so.6!clone (Unknown Source:0)
```


## 测试规则
```bash
/opt/components/suricata/bin/suricata -r /tmp/share/Work/packet/ftp-full.pcapng  -l /tmp/share/Work/suricata_output/ -S /tmp/share/Work/suricata_rules/manyrules.rules


mkdir -p /opt/components/suricata/var/lib/suricata/rules/
```

## 调试
```shell
pop3
gdbserver @localhost:2000 ./src/.libs/suricata -c ./suricata.yaml -l /tmp/share/Work/suricata_output/ -S /tmp/share/Work/suricata_rules/manyrules.rules  --runmode single -r /tmp/share/Work/packet/pop3/pop3-manyCC.pcap

smtp
gdbserver @localhost:2000 ./src/.libs/suricata -c ./suricata.yaml   -l /tmp/share/Work/suricata_output/ -S /tmp/share/Work/suricata_rules/manyrules.rules -r /tmp/share/Work/packet/smtp/smtp-manyCC.pcap

debug方法
https://blog.inliniac.net/2010/01/04/suricata-debugging/

SC_LOG_LEVEL=Debug
SC_LOG_OP_FILTER=<regex>


```

==邮件ID==
协议
==发送日期==
==邮件主题==
==发件人==
==收件人==
==抄送==
密送
加密方式
附件名称
级别
策略名称
## pop3邮件格式
### 邮件mime格式
```
https://blog.csdn.net/u014608280/article/details/89380417
3.1.1.邮件头
邮件头包含了发件人、收件人、主题、时间、MIME版本、邮件内容的类型等重要信息。每条信息称为一个域，由域名后加“: ”和信息内容构成，可以是一行，较长的也可以占用多行。域的首行必须“顶头”写，即左边不能有空白字符（空格和制表符）；续行则必须以空白字符打头，且第一个空白字符不是信息本身固有的，解码时要过滤掉。

邮件头中不允许出现空行。有一些邮件不能被邮件客户端软件识别，显示的是原始码，就是因为首行是空行。

```
- Content-Disposition: 
Content- Disposition头字段用于指定邮件阅读程序处理数据内容的方式，有inline和attachment两种标准方式，inline表示直接处理， 而attachment表示当做附件处理。如果将Content-Disposition设置为attachment，在其后还可以指定filename 属性，


- Content-Type:
”为“multipart/mixed”等时，表示数据为多种内容的混合，此时会有类似boundary="652b8c4dcb00cdcdda1e16af36781caf"的分隔线描述，分隔线会将数据内容分隔成各自独立的部分，在各部分中，分别有独立的数据内容描述。分隔线的前后，会有“--”，处理过程中过滤即可。


## smtp 解析
```
解析data命令后的数据中的MIME头部字段：
SMTPParse-> SMTPProcessRequest-> SMTPProcessCommandDATA-> MimeDecParseLine-> ProcessMimeEntity-> ProcessMimeHeaders

解析data命令后的数据中的MIME的body数据：
SMTPParse-> SMTPProcessRequest-> SMTPProcessCommandDATA-> MimeDecParseLine-> ProcessMimeEntity-> ProcessMimeBody
```

## json_builder
```c
struct JsonBuilder *jb_new_object(void);

struct JsonBuilder *jb_new_array(void);

struct JsonBuilder *jb_clone(struct JsonBuilder *js);

void jb_free(struct JsonBuilder *js);

uintptr_t jb_capacity(struct JsonBuilder *jb);

void jb_reset(struct JsonBuilder *jb);

bool jb_open_object(struct JsonBuilder *js, const char *key);

bool jb_start_object(struct JsonBuilder *js);

bool jb_open_array(struct JsonBuilder *js, const char *key);

bool jb_set_string(struct JsonBuilder *js, const char *key, const char *val);

bool jb_set_string_from_bytes(struct JsonBuilder *js,
                              const char *key,
                              const uint8_t *bytes,
                              uint32_t len);

bool jb_set_formatted(struct JsonBuilder *js, const char *formatted);

bool jb_append_object(struct JsonBuilder *jb, const struct JsonBuilder *obj);

bool jb_set_object(struct JsonBuilder *js,
                   const char *key,
                   struct JsonBuilder *val);

bool jb_append_string(struct JsonBuilder *js, const char *val);

bool jb_append_string_from_bytes(struct JsonBuilder *js,
                                 const uint8_t *bytes,
                                 uint32_t len);

bool jb_append_uint(struct JsonBuilder *js, uint64_t val);

bool jb_append_float(struct JsonBuilder *js, double val);

bool jb_set_uint(struct JsonBuilder *js, const char *key, uint64_t val);

bool jb_set_float(struct JsonBuilder *js, const char *key, double val);

bool jb_set_bool(struct JsonBuilder *js, const char *key, bool val);

bool jb_close(struct JsonBuilder *js);

uintptr_t jb_len(const struct JsonBuilder *js);

const uint8_t *jb_ptr(struct JsonBuilder *js);

void jb_get_mark(struct JsonBuilder *js, struct JsonBuilderMark *mark);

bool jb_restore_mark(struct JsonBuilder *js, struct JsonBuilderMark *mark);

```

## suricata 命令
```bash
[root@localhost bin]# ./suricatasc --help
usage: suricatasc [-h] [-v] [-c COMMAND] [socket]

Client for Suricata unix socket

positional arguments:
  socket                socket file to connect to

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbose output (including JSON dump)
  -c COMMAND, --command COMMAND
                        execute on single command and return JSON
[root@localhost bin]# ./suricatasc -c
usage: suricatasc [-h] [-v] [-c COMMAND] [socket]
suricatasc: error: argument -c/--command: expected one argument
[root@localhost bin]# ./suricatasc -c help
{"message": {"count": 33, "commands": ["shutdown", "command-list", "help", "version", "uptime", "running-mode", "capture-mode", "conf-get", "dump-counters", "reload-rules", "ruleset-reload-rules", "ruleset-reload-nonblocking", "ruleset-reload-time", "ruleset-stats", "ruleset-failed-rules", "register-tenant-handler", "unregister-tenant-handler", "register-tenant", "reload-tenant", "unregister-tenant", "add-hostbit", "remove-hostbit", "list-hostbit", "reopen-log-files", "memcap-set", "memcap-show", "memcap-list", "dataset-add", "dataset-remove", "iface-stat", "iface-list", "iface-bypassed-stat", "ebpf-bypassed-stat"]}, "return": "OK"}
[root@localhost bin]# ls
suricata  suricatactl  suricatasc  suricata-update
[root@localhost bin]# pwd
/opt/components/suricata/bin

```

## 程序架构
### 线程模块
线程模块是对packet处理任务的抽象。线程模块大概有以下几种：

```
1.Receive模块：收集网络数据包，封装成Packet对象后将其传递给Decode线程模块2.

2.Decode模块：对Packet按协议4层模型(数据链路层、网络层、传输层、应用层)进行解码，获取协议和负荷信息，解码完成后将 Packet传递给FlowWorker线程模块。该模块主要是进行packet解码，不处理应用层。应用层由专门的应用层解码模块处理

3.FlowWorker模块：对packets进行分配flow，Tcp会话管理，TCP重组，应用层数据解析处理，Detect规则检测

4.Verdict模块：根据Detect模块检测的结果，对drop标记的包需要做丢弃处理

5.RespondReject模块：根据detect检测后的结果，对于reject的包需要向双端发送reset包

6.logs模块：将处理结果记录在日志中。
```

```c
TmThreadSetSlots(ThreadVars *tv, const char *name, void *(*fn_p)(void *))
TmThreadSetSlots
根据name不同，向tv->tm_func 赋值函数，tm_func 为线程主函数
pktacqloop 赋值 TmThreadsSlotPktAcqLoop

TmThreadCreate 函数调用 TmThreadSetSlots 
TmThreadCreatePacketHandler 函数调用 TmThreadCreate
RunModeFilePcapSingle 函数调用 TmThreadCreatePacketHandler

RunModeFilePcapSingle 中

TmThreadCreatePacketHandler 创建tv
TmSlotSetFuncAppend 向tv中添加receive、decode、flowworker 模块
创建线程
```

## packet
```c
/* sizes of the members:
 * src: 17 bytes
 * dst: 17 bytes
 * sp/type: 1 byte
 * dp/code: 1 byte
 * proto: 1 byte
 * recurs: 1 byte
 *
 * sum of above: 38 bytes
 *
 * flow ptr: 4/8 bytes
 * flags: 1 byte
 * flowflags: 1 byte
 *
 * sum of above 44/48 bytes
 */
typedef struct Packet_
{
    /* Addresses, Ports and protocol
     * these are on top so we can use
     * the Packet as a hash key */
    Address src;
    Address dst;
    union {
        Port sp;
        // icmp type and code of this packet
        struct {
            uint8_t type;
            uint8_t code;
        } icmp_s;
    };
    union {
        Port dp;
        // icmp type and code of the expected counterpart (for flows)
        struct {
            uint8_t type;
            uint8_t code;
        } icmp_d;
    };
    uint8_t proto;
    /* make sure we can't be attacked on when the tunneled packet
     * has the exact same tuple as the lower levels */
    uint8_t recursion_level;

    uint16_t vlan_id[2];
    uint8_t vlan_idx;

    /* flow */
    uint8_t flowflags;
    /* coccinelle: Packet:flowflags:FLOW_PKT_ */

    /* Pkt Flags */
    uint32_t flags;

    struct Flow_ *flow;

    /* raw hash value for looking up the flow, will need to modulated to the
     * hash size still */
    uint32_t flow_hash;

    struct timeval ts;

    union {
        /* nfq stuff */
#ifdef HAVE_NFLOG
        NFLOGPacketVars nflog_v;
#endif /* HAVE_NFLOG */
#ifdef NFQ
        NFQPacketVars nfq_v;
#endif /* NFQ */
#ifdef IPFW
        IPFWPacketVars ipfw_v;
#endif /* IPFW */
#ifdef AF_PACKET
        AFPPacketVars afp_v;
#endif
#ifdef HAVE_NETMAP
        NetmapPacketVars netmap_v;
#endif
#ifdef HAVE_PFRING
#ifdef HAVE_PF_RING_FLOW_OFFLOAD
        PfringPacketVars pfring_v;
#endif
#endif
#ifdef WINDIVERT
        WinDivertPacketVars windivert_v;
#endif /* WINDIVERT */

        /* A chunk of memory that a plugin can use for its packet vars. */
        uint8_t plugin_v[PLUGIN_VAR_SIZE];

        /** libpcap vars: shared by Pcap Live mode and Pcap File mode */
        PcapPacketVars pcap_v;
    };

    /** The release function for packet structure and data */
    void (*ReleasePacket)(struct Packet_ *);
    /** The function triggering bypass the flow in the capture method.
     * Return 1 for success and 0 on error */
    int (*BypassPacketsFlow)(struct Packet_ *);

    /* pkt vars */
    PktVar *pktvar;

    /* header pointers */
    EthernetHdr *ethh;

    /* Checksum for IP packets. */
    int32_t level3_comp_csum;
    /* Check sum for TCP, UDP or ICMP packets */
    int32_t level4_comp_csum;

    IPV4Hdr *ip4h;

    IPV6Hdr *ip6h;

    /* IPv4 and IPv6 are mutually exclusive */
    union {
        IPV4Vars ip4vars;
        struct {
            IPV6Vars ip6vars;
            IPV6ExtHdrs ip6eh;
        };
    };
    /* Can only be one of TCP, UDP, ICMP at any given time */
    union {
        TCPVars tcpvars;
        ICMPV4Vars icmpv4vars;
        ICMPV6Vars icmpv6vars;
    } l4vars;
#define tcpvars     l4vars.tcpvars
#define icmpv4vars  l4vars.icmpv4vars
#define icmpv6vars  l4vars.icmpv6vars

    TCPHdr *tcph;

    UDPHdr *udph;

    SCTPHdr *sctph;

    ICMPV4Hdr *icmpv4h;

    ICMPV6Hdr *icmpv6h;

    PPPHdr *ppph;
    PPPOESessionHdr *pppoesh;
    PPPOEDiscoveryHdr *pppoedh;

    GREHdr *greh;

    /* ptr to the payload of the packet
     * with it's length. */
    uint8_t *payload;
    uint16_t payload_len;

    /* IPS action to take */
    uint8_t action;

    uint8_t pkt_src;

    /* storage: set to pointer to heap and extended via allocation if necessary */
    uint32_t pktlen;
    uint8_t *ext_pkt;

    /* Incoming interface */
    struct LiveDevice_ *livedev;

    PacketAlerts alerts;

    struct Host_ *host_src;
    struct Host_ *host_dst;

    /** packet number in the pcap file, matches wireshark */
    uint64_t pcap_cnt;


    /* engine events */
    PacketEngineEvents events;

    AppLayerDecoderEvents *app_layer_events;

    /* double linked list ptrs */
    struct Packet_ *next;
    struct Packet_ *prev;

    /** data linktype in host order */
    int datalink;

    /* tunnel/encapsulation handling */
    struct Packet_ *root; /* in case of tunnel this is a ptr
                           * to the 'real' packet, the one we
                           * need to set the verdict on --
                           * It should always point to the lowest
                           * packet in a encapsulated packet */

    /** mutex to protect access to:
     *  - tunnel_rtv_cnt
     *  - tunnel_tpr_cnt
     */
    SCMutex tunnel_mutex;
    /* ready to set verdict counter, only set in root */
    uint16_t tunnel_rtv_cnt;
    /* tunnel packet ref count */
    uint16_t tunnel_tpr_cnt;

    /** tenant id for this packet, if any. If 0 then no tenant was assigned. */
    uint32_t tenant_id;

    /* The Packet pool from which this packet was allocated. Used when returning
     * the packet to its owner's stack. If NULL, then allocated with malloc.
     */
    struct PktPool_ *pool;

    /* count decoded layers of packet : too many layers
     * cause issues with performance and stability (stack exhaustion)
     */
    uint8_t nb_decoded_layers;

#ifdef PROFILING
    PktProfiling *profile;
#endif
#ifdef HAVE_NAPATECH
    NapatechPacketVars ntpv;
#endif
} Packet;
```

-   src
-   dst  
    src和dst保存了三层地址的地址族类型和地址数据。
-   sp、dp  
    sp和dp保存了四层端口号。
-   type、code  
    type和code保存了icmpv4协议的type和code。其中type和sp共用一个联合体，code和dp共用一个联合体。
-   proto  
    三层数据包载荷的协议类型。一般来说是一个四层协议类型，但如果是一个隧道包那这里会是隧道内层数据包的类型，就不是四层协议类型了。
-   recursion_level  
    指示了当前Packet经历了几次隧道封装。普通数据包这个值是0.
-   vlan_id  
    长度为2的数组，存储了vlan的id，允许vlan嵌套，数组中的两项分别代表该嵌套层的vlan id。
-   vlan_idx  
    标记了当前的vlan嵌套层数。最多2层。
-   flowflags  
    标记了当前packet与flow相关的一些标识。
    -   FLOW_PKT_TOSERVER
    -   FLOW_PKT_TOCLIENT
    -   FLOW_PKT_ESTABLISHED
    -   FLOW_PKT_TOSERVER_IPONLY_SET
    -   FLOW_PKT_TOCLIENT_IPONLY_SET
    -   FLOW_PKT_TOSERVER_FIRST
    -   FLOW_PKT_TOCLIENT_FIRST
-   flags  
    标识了当前Packet的flag，后面列举了各个flag。
-   flow  
    关联的flow。
-   flow_hash  
    数据包匹配相应的flow时使用的hash值，tcp/udp/sctp/icmp会设定该项。使用源目的地址、源目的端口、协议号、隧道递归值、vlan_id数组、系统初始化随机数计算得来。
-   ts  
    记录了数据包的时间。
-   nflog_v
-   nfq_v
-   ipfw_v
-   afp_v
-   mpipe_v
-   netmap_v
-   pcap_v  
    上面几个用于记录收包阶段时各个收包模块特有的数据。
-   ReleasePacket  
    函数指针，当数据包需要被释放时调用。可能是释放回原线程的PktPool中，也可能是直接释放内存。
-   BypassPacketsFlow  
    TODO
-   pktvar  
    TODO
-   ethh  
    二层以太网包头指针。
-   level3_comp_csum
-   level4_comp_csum  
    TODO
-   ip4h  
    三层ipv4包头指针。
-   ip6h  
    三层ipv6包头指针。
-   ip4vars  
    保存了ipv4的可选项信息，如果存在可选项则写入该成员。
    -   opt_cnt  
        保存了可选项的数量。
    -   opts_set  
        保存了有哪些可选项的标记。以`IPV4_OPT_FLAG_`开头的一组枚举型标识。
-   ip6vars
-   ip6eh  
    这两个成员组成了一个结构体，这个结构体与ip4vars组成联合体，可以看出是ipv6的扩展选项，目前不关注。
-   tcpvars
-   icmpv4vars
-   icmpv6vars  
    四层协议的扩展项。
-   tcph  
    四层tcp协议头。
-   udph  
    四层udp协议头。
-   sctph  
    四层sctp协议头。
-   icmpv4h  
    四层icmpV4协议头。
-   icmpv6h  
    四层icmpV6协议头。
-   ppph  
    二层ppp协议头。上层支持ipv4与ipv6协议。
-   pppoesh  
    pppoe会话协议头。位于二层以太网之上，内部封装支持ipv4和ipv6。
-   pppoedh  
    pppoe发现协议头。位于二层以太网之上。
-   greh  
    gre封装协议头。位于三层ipv4之上，内部封装支持的比较多。
-   vlanh  
    长度为2的数组，vlan头指针，与上面的vlan_idx组合，与vlan_id数组对应。
-   payload  
    四层数据包的载荷，比如tcp或udp协议内的载荷。
-   payload_len  
    四层数据包的载荷长度。
-   action  
    IPS模式下，标识了要对这个数据包做的处理动作。动作定义在`action-globals.h`中。
-   pkt_src  
    标识数据包的来源。比如PKT_SRC_WIRE表示直接收到的包，PKT_SRC_DECODER_GRE表示是由GRE解封装得到的伪造数据包。
-   pktlen  
    收取的数据包字节长度
-   ext_pkt  
    这个成员有一点特殊。Packet结构体在创建并分配内存时预分配了更多的default_packet_size字节的内存，也就是说当收取的数据包长度不大于default_packet_size时可以使用Packet结构体后的内存空间用以拷贝存储数据包数据。但是当数据包长度超出了default_packet_size时，将分配一块MAX_PAYLOAD_SIZE大小的内存块，地址赋值给ext_pkt成员，用以拷贝存储数据包数据，当数据包内存释放或归还到池子后重用时ext_pkt指向的内存将释放。另外如果数据包是使用零拷贝映射的，那么ext_pkt成员用于指向数据包内存地址，这时该指向的内存不能由用户释放。
-   livedev  
    指向数据包来源的一个LiveDevice结构。
-   alerts  
    TODO
-   host_src
-   host_dst  
    TODO
-   pcap_cnt  
    pcap文件中的数据包序号。
-   events  
    记录解码、分片重组、和stream时的event。event类型记录在`decode-events.h`中。
    -   cnt  
        当前记录了的event的数量。
    -   events  
        PACKET_ENGINE_EVENT_MAX长度的数组，顺序记录event
-   app_layer_events  
    TODO
-   next
-   prev  
    next和prev两个成员可以将Packet链接起来，PktPool和PacketQueue结构都有使用。
-   datalink  
    标识了收包设备的二层类型。决定了收取数据包首部的header结构类型。
-   root  
    如果一个数据包是隧道数据包或封装数据包，解封装后会生成一个新的伪造数据包进入后续模块处理，这时新的伪造数据包的root成员会设置为指向最原始的那个数据包，也就是说如果是多层封装，那么后续的解封装后的伪造数据包root成员们都会指向最底层的那一个原始数据包。
-   tunnel_mutex  
    锁，用于保护另外两个成员tunnel_rtv_cnt和tunnel_tpr_cnt的并发访问。
-   tunnel_rtv_cnt  
    标识这个数据包做为其他数据包的root时，生成的新伪造数据包处理完成被回收(retrieve)的个数。只增不减。
-   tunnel_tpr_cnt  
    标识这个数据包作为其他数据包的root被引用的次数，与root和tunnel_rtv_cnt配置实用。只增不减。
-   tenant_id  
    TODO
-   pool  
    这个数据包所属的PktPool，在释放时会返回挂载到这个池子中。
-   profile
-   cuda_pkt_vars
-   ntpv  
    TODO

## pcre 函数库
```c
pcre *reCM, *reUN, *reTC, *reCDMA; 
int rcCM, rcUN, rcTC, rcCDMA, i; 
scanf("%s", src);
char pattern_CM[] = "^1(3[4-9]|5[012789]|8[78])\\d{8}$";
reCM = pcre_compile(pattern_CM, 0, &error, &erroffset, NULL); 
rcCM = pcre_exec(reCM, NULL, src, strlen(src), 0, 0, ovector, OVECCOUNT); 
if (rcCM > 0) 
{
    printf("Pattern_CM: \"%s\"\n", pattern_CM);
    printf("String : %s\n", src);
}
```

# detect

## 数据结构
### DetectEngineCtx
```c
/** \brief main detection engine ctx */
typedef struct DetectEngineCtx_ {
    uint8_t flags;
    int failure_fatal;

    int tenant_id;

    Signature *sig_list;
    uint32_t sig_cnt;

    /* version of the srep data */
    uint32_t srep_version;

    /* reputation for netblocks */
    SRepCIDRTree *srepCIDR_ctx;

    Signature **sig_array;
    uint32_t sig_array_size; /* size in bytes */
    uint32_t sig_array_len;  /* size in array members */

    uint32_t signum;

    /** Maximum value of all our sgh's non_mpm_store_cnt setting,
     *  used to alloc det_ctx::non_mpm_id_array */
    uint32_t non_pf_store_cnt_max;

    /* used by the signature ordering module */
    struct SCSigOrderFunc_ *sc_sig_order_funcs;

    /* hash table used for holding the classification config info */
    HashTable *class_conf_ht;
    /* hash table used for holding the reference config info */
    HashTable *reference_conf_ht;

    /* main sigs */
    DetectEngineLookupFlow flow_gh[FLOW_STATES];

    uint32_t gh_unique, gh_reuse;

    /* init phase vars */
    HashListTable *sgh_hash_table;

    HashListTable *mpm_hash_table;

    /* hash table used to cull out duplicate sigs */
    HashListTable *dup_sig_hash_table;

    DetectEngineIPOnlyCtx io_ctx;
    ThresholdCtx ths_ctx;

    uint16_t mpm_matcher; /**< mpm matcher this ctx uses */
    uint16_t spm_matcher; /**< spm matcher this ctx uses */

    /* spm thread context prototype, built as spm matchers are constructed and
     * later used to construct thread context for each thread. */
    SpmGlobalThreadCtx *spm_global_thread_ctx;

    /* Config options */

    uint16_t max_uniq_toclient_groups;
    uint16_t max_uniq_toserver_groups;

    /* specify the configuration for mpm context factory */
    uint8_t sgh_mpm_ctx_cnf;

    /* max flowbit id that is used */
    uint32_t max_fb_id;

    uint32_t max_fp_id;

    MpmCtxFactoryContainer *mpm_ctx_factory_container;

    /* maximum recursion depth for content inspection */
    int inspection_recursion_limit;

    /* conf parameter that limits the length of the http request body inspected */
    int hcbd_buffer_limit;
    /* conf parameter that limits the length of the http response body inspected */
    int hsbd_buffer_limit;

    /* array containing all sgh's in use so we can loop
     * through it in Stage4. */
    struct SigGroupHead_ **sgh_array;
    uint32_t sgh_array_cnt;
    uint32_t sgh_array_size;

    int32_t sgh_mpm_context_proto_tcp_packet;
    int32_t sgh_mpm_context_proto_udp_packet;
    int32_t sgh_mpm_context_proto_other_packet;
    int32_t sgh_mpm_context_stream;

    /* the max local id used amongst all sigs */
    int32_t byte_extract_max_local_id;

    /** version of the detect engine. The version is incremented on reloads */
    uint32_t version;

    /** sgh for signatures that match against invalid packets. In those cases
     *  we can't lookup by proto, address, port as we don't have these */
    struct SigGroupHead_ *decoder_event_sgh;

    /* Maximum size of the buffer for decoded base64 data. */
    uint32_t base64_decode_max_len;

    /** Store rule file and line so that parsers can use them in errors. */
    char *rule_file;
    int rule_line;
    bool sigerror_silent;
    bool sigerror_ok;
    const char *sigerror;

    /** list of keywords that need thread local ctxs */
    DetectEngineThreadKeywordCtxItem *keyword_list;
    int keyword_id;

    struct {
        uint32_t content_limit;
        uint32_t content_inspect_min_size;
        uint32_t content_inspect_window;
    } filedata_config[ALPROTO_MAX];
    bool filedata_config_initialized;

#ifdef PROFILING
    struct SCProfileDetectCtx_ *profile_ctx;
    struct SCProfileKeywordDetectCtx_ *profile_keyword_ctx;
    struct SCProfilePrefilterDetectCtx_ *profile_prefilter_ctx;
    struct SCProfileKeywordDetectCtx_ **profile_keyword_ctx_per_list;
    struct SCProfileSghDetectCtx_ *profile_sgh_ctx;
    uint32_t profile_match_logging_threshold;
#endif
    uint32_t prefilter_maxid;

    char config_prefix[64];

    enum DetectEngineType type;

    /** how many de_ctx' are referencing this */
    uint32_t ref_cnt;
    /** list in master: either active or freelist */
    struct DetectEngineCtx_ *next;

    /** id of loader thread 'owning' this de_ctx */
    int loader_id;

    /** are we using just mpm or also other prefilters */
    enum DetectEnginePrefilterSetting prefilter_setting;

    HashListTable *dport_hash_table;

    DetectPort *tcp_whitelist;
    DetectPort *udp_whitelist;

    /** table for storing the string representation with the parsers result */
    HashListTable *address_table;

    /** table to store metadata keys and values */
    HashTable *metadata_table;

    DetectBufferType **buffer_type_map;
    uint32_t buffer_type_map_elements;

    /* hash table with rule-time buffer registration. Start time registration
     * is in detect-engine.c::g_buffer_type_hash */
    HashListTable *buffer_type_hash;
    int buffer_type_id;

    /* list with app inspect engines. Both the start-time registered ones and
     * the rule-time registered ones. */
    DetectEngineAppInspectionEngine *app_inspect_engines;
    DetectBufferMpmRegistery *app_mpms_list;
    uint32_t app_mpms_list_cnt;
    DetectEnginePktInspectionEngine *pkt_inspect_engines;
    DetectBufferMpmRegistery *pkt_mpms_list;
    uint32_t pkt_mpms_list_cnt;

    uint32_t prefilter_id;
    HashListTable *prefilter_hash_table;

    /** time of last ruleset reload */
    struct timeval last_reload;

    /** signatures stats */
    SigFileLoaderStat sig_stat;

    /** per keyword flag indicating if a prefilter has been
     *  set for it. If true, the setup function will have to
     *  run. */
    bool sm_types_prefilter[DETECT_TBLSIZE];
    bool sm_types_silent_error[DETECT_TBLSIZE];

} DetectEngineCtx;
```


DetectEngineThreadCtx
```c
/**
  * Detection engine thread data.
  */
typedef struct DetectEngineThreadCtx_ {
    /** \note multi-tenant hash lookup code from Detect() *depends*
     *        on this being the first member */
    uint32_t tenant_id;

    /** ticker that is incremented once per packet. */
    uint64_t ticker;

    /* the thread to which this detection engine thread belongs */
    ThreadVars *tv;

    /** Array of non-prefiltered sigs that need to be evaluated. Updated
     *  per packet based on the rule group and traffic properties. */
    SigIntId *non_pf_id_array;
    uint32_t non_pf_id_cnt; // size is cnt * sizeof(uint32_t)

    uint32_t mt_det_ctxs_cnt;
    struct DetectEngineThreadCtx_ **mt_det_ctxs;
    HashTable *mt_det_ctxs_hash;

    struct DetectEngineTenantMapping_ *tenant_array;
    uint32_t tenant_array_size;

    uint32_t (*TenantGetId)(const void *, const Packet *p);

    /* detection engine variables */

    uint64_t raw_stream_progress;

    /** offset into the payload of the last match by:
     *  content, pcre, etc */
    uint32_t buffer_offset;
    /* used by pcre match function alone */
    uint32_t pcre_match_start_offset;

    /* counter for the filestore array below -- up here for cache reasons. */
    uint16_t filestore_cnt;

    /** id for alert counter */
    uint16_t counter_alerts;
#ifdef PROFILING
    uint16_t counter_mpm_list;
    uint16_t counter_nonmpm_list;
    uint16_t counter_fnonmpm_list;
    uint16_t counter_match_list;
#endif

    int inspect_list; /**< list we're currently inspecting, DETECT_SM_LIST_* */

    struct {
        InspectionBuffer *buffers;
        uint32_t buffers_size;          /**< in number of elements */
        uint32_t to_clear_idx;
        uint32_t *to_clear_queue;
    } inspect;

    struct {
        /** inspection buffers for more complex case. As we can inspect multiple
         *  buffers in parallel, we need this extra wrapper struct */
        InspectionBufferMultipleForList *buffers;
        uint32_t buffers_size;                      /**< in number of elements */
        uint32_t to_clear_idx;
        uint32_t *to_clear_queue;
    } multi_inspect;

    /* used to discontinue any more matching */
    uint16_t discontinue_matching;
    uint16_t flags;

    /* bool: if tx_id is set, this is 1, otherwise 0 */
    uint16_t tx_id_set;
    /** ID of the transaction currently being inspected. */
    uint64_t tx_id;
    Packet *p;

    SC_ATOMIC_DECLARE(int, so_far_used_by_detect);

    /* holds the current recursion depth on content inspection */
    int inspection_recursion_counter;

    /** array of signature pointers we're going to inspect in the detection
     *  loop. */
    Signature **match_array;
    /** size of the array in items (mem size if * sizeof(Signature *)
     *  Only used during initialization. */
    uint32_t match_array_len;
    /** size in use */
    SigIntId match_array_cnt;

    RuleMatchCandidateTx *tx_candidates;
    uint32_t tx_candidates_size;

    SignatureNonPrefilterStore *non_pf_store_ptr;
    uint32_t non_pf_store_cnt;

    /** pointer to the current mpm ctx that is stored
     *  in a rule group head -- can be either a content
     *  or uricontent ctx. */
    MpmThreadCtx mtc;   /**< thread ctx for the mpm */
    MpmThreadCtx mtcu;  /**< thread ctx for uricontent mpm */
    MpmThreadCtx mtcs;  /**< thread ctx for stream mpm */
    PrefilterRuleStore pmq;

    /** SPM thread context used for scanning. This has been cloned from the
     * prototype held by DetectEngineCtx. */
    SpmThreadCtx *spm_thread_ctx;

    /** ip only rules ctx */
    DetectEngineIPOnlyThreadCtx io_ctx;

    /* byte_* values */
    uint64_t *byte_values;

    /* string to replace */
    DetectReplaceList *replist;
    /* vars to store in post match function */
    DetectVarList *varlist;

    /* Array in which the filestore keyword stores file id and tx id. If the
     * full signature matches, these are processed by a post-match filestore
     * function to finalize the store. */
    struct {
        uint32_t file_id;
        uint64_t tx_id;
    } filestore[DETECT_FILESTORE_MAX];

    DetectEngineCtx *de_ctx;
    /** store for keyword contexts that need a per thread storage. Per de_ctx. */
    void **keyword_ctxs_array;
    int keyword_ctxs_size;
    /** store for keyword contexts that need a per thread storage. Global. */
    int global_keyword_ctxs_size;
    void **global_keyword_ctxs_array;

    uint8_t *base64_decoded;
    int base64_decoded_len;
    int base64_decoded_len_max;

    AppLayerDecoderEvents *decoder_events;
    uint16_t events;

#ifdef DEBUG
    uint64_t pkt_stream_add_cnt;
    uint64_t payload_mpm_cnt;
    uint64_t payload_mpm_size;
    uint64_t stream_mpm_cnt;
    uint64_t stream_mpm_size;
    uint64_t payload_persig_cnt;
    uint64_t payload_persig_size;
    uint64_t stream_persig_cnt;
    uint64_t stream_persig_size;
#endif
#ifdef PROFILING
    struct SCProfileData_ *rule_perf_data;
    int rule_perf_data_size;
    struct SCProfileKeywordData_ *keyword_perf_data;
    struct SCProfileKeywordData_ **keyword_perf_data_per_list;
    int keyword_perf_list; /**< list we're currently inspecting, DETECT_SM_LIST_* */
    struct SCProfileSghData_ *sgh_perf_data;

    struct SCProfilePrefilterData_ *prefilter_perf_data;
    int prefilter_perf_size;
#endif
} DetectEngineThreadCtx;

```

```
Pop3ProbingParserTs(Flow * f, uint8_t direction, const uint8_t * input, uint32_t input_len, uint8_t * rdir) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer-pop3.c:305)
PPGetProto(const AppLayerProtoDetectProbingParserElement * pe, Flow * f, uint8_t flags, const uint8_t * buf, uint32_t buflen, uint32_t * alproto_masks, uint8_t * rdir) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer-detect-proto.c:476)
AppLayerProtoDetectPPGetProto(Flow * f, const uint8_t * buf, uint32_t buflen, uint8_t ipproto, const uint8_t flags, _Bool * reverse_flow) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer-detect-proto.c:588)
AppLayerProtoDetectGetProto(AppLayerProtoDetectThreadCtx * tctx, Flow * f, const uint8_t * buf, uint32_t buflen, uint8_t ipproto, uint8_t flags, _Bool * reverse_flow) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer-detect-proto.c:1563)
TCPProtoDetect(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, AppLayerThreadCtx * app_tctx, Packet * p, Flow * f, TcpSession * ssn, TcpStream ** stream, uint8_t * data, uint32_t data_len, uint8_t flags) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer.c:336)
AppLayerHandleTCPData(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, Packet * p, Flow * f, TcpSession * ssn, TcpStream ** stream, uint8_t * data, uint32_t data_len, uint8_t flags) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\app-layer.c:642)
ReassembleUpdateAppLayer(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, TcpSession * ssn, TcpStream ** stream, Packet * p, enum StreamUpdateDir dir) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp-reassemble.c:1173)
StreamTcpReassembleAppLayer(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, TcpSession * ssn, TcpStream * stream, Packet * p, enum StreamUpdateDir dir) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp-reassemble.c:1236)
StreamTcpReassembleHandleSegmentUpdateACK(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, TcpSession * ssn, TcpStream * stream, Packet * p) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp-reassemble.c:1805)
StreamTcpReassembleHandleSegment(ThreadVars * tv, TcpReassemblyThreadCtx * ra_ctx, TcpSession * ssn, TcpStream * stream, Packet * p, PacketQueueNoLock * pq) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp-reassemble.c:1850)
HandleEstablishedPacketToClient(ThreadVars * tv, TcpSession * ssn, Packet * p, StreamTcpThread * stt, PacketQueueNoLock * pq) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp.c:2469)
StreamTcpPacketStateEstablished(ThreadVars * tv, Packet * p, StreamTcpThread * stt, TcpSession * ssn, PacketQueueNoLock * pq) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp.c:2702)
StreamTcpStateDispatch(ThreadVars * tv, Packet * p, StreamTcpThread * stt, TcpSession * ssn, PacketQueueNoLock * pq, const uint8_t state) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp.c:4703)
StreamTcpPacket(ThreadVars * tv, Packet * p, StreamTcpThread * stt, PacketQueueNoLock * pq) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp.c:4890)
StreamTcp(ThreadVars * tv, Packet * p, void * data, PacketQueueNoLock * pq) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\stream-tcp.c:5226)
FlowWorkerStreamTCPUpdate(ThreadVars * tv, FlowWorkerThreadData * fw, Packet * p, void * detect_thread) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\flow-worker.c:364)
FlowWorker(ThreadVars * tv, Packet * p, void * data) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\flow-worker.c:524)
TmThreadsSlotVarRun(ThreadVars * tv, Packet * p, TmSlot * slot) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\tm-threads.c:117)
TmThreadsSlotProcessPkt(ThreadVars * tv, TmSlot * s, Packet * p) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\tm-threads.h:192)
PcapFileCallbackLoop(char * user, struct pcap_pkthdr * h, u_char * pkt) (\tmp\share\code\Suricata\sftp\Suricata\suricata-6.0.3\src\source-pcap-file-helper.c:101)
```

## detect-dns-query.c

```c
DetectAppLayerMpmRegister2

DetectAppLayerInspectEngineRegister2
DetectBufferTypeSetDescriptionByName 
DetectAppLayerInspectEngineRegister
```

### DetectAppLayerMpmRegister2
registering 
name : dns_query  
direction : SIG_FLAG_TOSERVER
priority:  2
PrefilterRegister :  PrefilterMpmDnsQueryRegister
GetData:  NULL
alproto：ALPROTO_DNS
tx_min_progress：1

新申请一个结构体 DetectBufferMpmRegistery，将它挂在g_mpm_list[DETECT_BUFFER_MPM_TYPE_APP] 全局变量中
am -> PrefilterRegisterWithListId = PrefilterMpmDnsQueryRegister
### DetectAppLayerInspectEngineRegister2
name : dns_query  
alproto：ALPROTO_DNS
dir ：SIG_FLAG_TOSERVER
int progress :1
InspectEngineFuncPtr2 Callback2 : DetectEngineInspectDnsQuery
InspectionBufferGetDataPtr GetData : NULL

在g_buffer_type_hash 中注册 dns_query
新申请一个结构体 DetectEngineAppInspectionEngine ，将它挂在 g_app_inspect_engines
new_engine->v2.Callback = DetectEngineInspectDnsQuery
### DetectAppLayerInspectEngineRegister

新申请一个结构体 DetectEngineAppInspectionEngine ，将它挂在 g_app_inspect_engines
new_engine->Callback = DetectEngineInspectDnsRequest，DetectEngineInspectDnsResponse

### PrefilterMpmDnsQueryRegister
新申请一个结构体 PrefilterMpmDnsQuery pectx
DetectEngineCtx *de_ctx
SigGroupHead *sgh
MpmCtx *mpm_ctx
const DetectBufferMpmRegistery *mpm_reg
int list_id

de_ctx
sgh
PrefilterTxDnsQuery 
mpm_reg->app_v2.alproto
mpm_reg->app_v2.tx_min_progress
pectx, 
PrefilterMpmDnsQueryFree
mpm_reg->pname

return PrefilterAppendTxEngine

### PrefilterAppendTxEngine

新申请一个结构体 PrefilterEngineList *e 
e->PrefilterTx = PrefilterTxFunc (PrefilterTxDnsQuery) 
将 e 插入到 sgh->init->tx_engines 链表中