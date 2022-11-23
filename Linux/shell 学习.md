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

## 批量删除数据
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
## 读取数据库中的数据
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

## if语句
```bash
整数变量表达式
-eq 等于
-ne 不等于
-gt 大于
-ge 大于等于
-lt 小于
-le 小于等于

```