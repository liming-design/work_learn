```bash
//查询端口
docker ps | grep click

docker exec -it clickhouse clickhouse-client -dfpc -u                                             clickhouse --password Machloop@123

select * from t_fpc_protocol_http_log_record order by start_time desc limit 1\G

--查看所有表
show tables  

-- 查看表结构
desc test.t1

--查找告警信息
select * from t_fpc_analysis_suricata_alert_message order by timestamp desc limit 1\G


```

### 添加字段
```
alter table product_detail add column `remark` String DEFAULT '' COMMENT '备注';

alter table product_detail modify column `remark` Nullable(String) DEFAULT NULL COMMENT '备注';

alter table t_fpc_protocol_mail_log_record add column `url_list`  Array(String)

```


## 日志文件
```bash

/opt/machloop/log/fpc-manager
```

## 搜索Linux下命令的路径
```
which gdb --返回正在执行的命令
whereis gdb  --把所有包含gdb（不管是文件还是文件夹）的路径都列了出来。
```