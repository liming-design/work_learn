## postgresql 学习

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
```
psql -d fpc -U machloop -p 5432

Machloop@123

``` 

使用\\d命令查看postgresql中所有表

\\d+ 命令更加详细的显示表