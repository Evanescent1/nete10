# nete10
&ensp;&ensp&ensp;&ensp;查看索引的信息 show indexes from testlog\G
利用索引查询数据select * from testlog where id=99999;
查看命令是否利用索引 explain select * from testlog where id=199999;
possible_keys可能用到的索引，key用的索引
创建索引
create index idx_name索引的名字 on指定表 testlog(name(10)定义前多少个字节做索引);
利用定义的字段左前缀查询
explain select * from testlog where name like 'wang1000%'\G
删除索引
drop index idx_name索引名 on testlog表;
创建复合索引（单独查name可以利用索引，查age不可以）
 create index idx_name_age on testlog(name,age);
Non_unique: 1表示不是唯一键索引
当利用左前缀有多条记录将不利用索引
explain select * from students where name like 'x%'\G
添加记录
insert students表 (name,age) values('xiren',30);
利用组合索引查询
explain select * from students where name='x%' and age=30\G
创建唯一键索引
create unique唯一键 index uni_name on students(name);
定义是否有冗余索引（默认是不启用的）
show variables like 'userstat';   
tbl_name [[AS] alias] lock_type
[, tbl_name [[AS] alias] lock_type] ...
lock_type: READ 读锁， WRITE写锁
UNLOCK TABLES 解锁
FLUSH TABLES [tb_name[,...]] [WITH READ LOCK]对数据库加读锁，通常在备份前加全局读锁
flush tables关闭正在打开的表（清除查询缓存）
SELECT clause [FOR UPDATE | LOCK IN SHARE MODE]
查询时加写或读锁
添加读锁 (会导致访问页面卡顿）  
lock  tables students read;
unlock tables;退出解读锁
添加写锁
lock tables students write;
修改表的记录
 update students表 set classid字段=3 where stuid=25id等于25的;
对整个数据库(实例）加上读锁（备份，温备）
flush tables  with  read lock;
当执行完一条单独命令创建标签savepoint sp_a标签名;指定要撤回的位置rollback to sp_a;只能撤销一次。
释放某个保存点release savepoint sp_b;
事务的死锁：当两个事务同时执行，修改不同表，执行事务的时候，会自动添加锁。如果同时事务A去修改B的表
，而B修改A的表，会形成死锁。当系统发现死锁，会自动结束一个执行时间短的事务。
事务隔离级别
事务隔离级别：从上至下更加严格
READ UNCOMMITTED 可读取到未提交数据，产生脏读（看到事务的修改数据）
READ COMMITTED 可读取到提交数据，但未提交数据不可读，产生不可重复读，即可读取到多个提交数据
，导致每次读取数据不一致（看到的是提交的事务数据，看不到修改的数据，在同一事务中看到的数据不同，）支持MVCC
REPEATABLE READ 可重复读，多次读取数据都一致，产生幻读，即读取过程中，即使有其它提交的事务修改数据，
仍只能读取到未修改前的旧数据。此为MySQL默认设置（看到的数据永远是开始事务的数据的那一瞬间的状态）做备份
，系统默认的隔离级别       支持MVCC
SERIALIZABILE 可串行化，未提交的读事务阻塞修改事务，或者未提交的修改事务阻塞读事务。导致并发性能差
查看慢查询是否启用
 show variables like 'slow_query_log';
在配置文件里添加(启用慢查询）
slow_query_log
查看慢查询记录日志定义的时间
select @@long_query_time;
在配置文件定义写慢查询日志的时间long_query_time=3 
定义表的每一条记录休眠一秒
select sleep(1) from teachers;
默认不启用
select @@log_queries_not_using_indexes;
在配置文件里加上log_queries_not_using_insexes  启用，如果没有利用索引将记录慢查询日志
select @@profiling;0是不启用，需要启用set profiling=ON;
查看启用的后的命令列表编号show profiles;
利用编号show profile for query 5;查看这条命令执行详细内容，分析慢的原因
完全备份：服务器性能会受影响，不会经常做，在完全备份基础上做差异本分，
差异备份;做完全备份到差异备份之间数据的变化，下一个差异本分做完全本备份到这个时间点的数据变化，
每一个差异备份都可能包含前一个差异备份基础之上的内容，
如果利用差异备份还原数据，先还原完全备份，在利用数据丢失的最后一次差异备份，恢复到最后一次做差异备份的状态，
然后利用二进制还原到数据破坏的那个时间段。
在完全备份的基础上做增量备份
增量备份（二进制）：在完全备份到做增量备份的数据变化，这次增量备份做上一次增量到这次的之间的数据变化，
每个增量备份都在上一个增量备份的基础上做备份。
还原数据，先还原完全备份，在依次按照做增量备份的顺序来还原，还原到最后一次做备份，
利用二进制数据做还原到出错那一刻的状态
差异备份还原快，备份慢，增量备份备份快，还原慢
mysqldump备份
第一种语法mysqldump hellodb  > hellodb_bak.sql（不建议使用）
mysqldump hellodb（数据库）  只对一个数据库做备份，不能跟多个，跟上第二个，会认为是前一个数据库的表，
只备份表的内容，没有数据库内容和结构。如果要还原，要创建数据库，指定数据库导入  mysql hello指定的数据库 
< hellodb_bak.sql
备份表mysql hello库 students表 > students.sql
第二种语法mysqldump -B指定 hello库 > hello_bak_B.sql
会备份数据库结构，字符集，但不备份函数，视图，触发器，只备份表。数据库的信息不全部放在数据库目录下，
一部分资源放在mysql的系统数据库中，不要用rm -f命令直接删，在数据库中用drop命令删除。恢复数据不需要写库名，
直接导入mysql  <  hello_bak_B.sql，把数据库的文件压缩xz  hello_bak2_B.sql 
解压gzip  -d hello.sql.gz
备份压缩：mysqldump -B hello | gzip > hello_bak.sql.gz
                mysqldump -B hello | xz  > hello_bak.sql.xz
查看存储过程：show procedure status\G
第三种语法：备份所有mysqldump -A  > all_bak.sql
备份所有的数据库创建，包括函数，视图，触发器
--default-character-set=utf8备份默认的字符集，备份前确定系统的字符集，可以指定
--master-data（确保启用二进制日志）在备份文件添加一个指令change  master to是否存在注释 ，1表示不注释，
用主从复制，2表示在备份文件里注释掉（定义了备份时候二进制日志的位置，
还原数据可以根据这行定义查看二进制日志的位置）
备份数据：mysqldump --master-data=1 -A > all_bak4.sql  记录二进制的位置，从这个位置之前备份了，
位置以后没备份
CHANGE MASTER TO MASTER_LOG_FILE='mariadb-bin.000001', MASTER_LOG_POS=245;                                                          
刷新日志,生成新的二进制：mysql  -e   ‘flush  logs'
查看二进制；mysql -e 'show master logs'
实验：恢复删除表（没有明确时间）
1把数据库锁住，禁止用户访问
2关闭二进制日志set   sql_log_bin=off;
3把所有内容还原，先利用完全备份还原到最后一次做备份的状态
source /root/all_bak.sql;
4另开一个新终端先找到备份时间，分析没有备份后的二进制日志，找到删除表的时间
导入文件mysqlbinlog /data/binlog/mysql-bin.000007 > incr.sql
5搜索DROP命令（大小写敏感）  找到所在的行，注释掉删除命令。
6回到原来的终端导入文件：source /root/incr.sql
7开启二进制日志set sql_log_bin=on;
日志序列号LSN：每个数据文件分成一块块排列，更改数据的时候，在每个数据块上加上编号（更改数据时间的值），
做增量备份根据数据块上的序列号（时间值），定义了数据是否备份，实现增量备份。每块单位（16k）
示例：旧版xtrabackup完全备份及还原
1 在原主机
innobackupex --user=root /backups指定数据库的内容备份到哪个目录下
scp -r /backups/2018-02-23_11-55-57/文件自动生成 目标主机:/data/复制到新主机还原
2 在目标主机
innobackupex --apply-log /data/2018-02-23_11-55-57/完全备份(最后一次，用log)整理数据
systemctl stop mariadb
rm -rf /var/lib/mysql/*  清空数据库
innobackupex --copy-back /data/2018-02-23_11-55-57/自动找到配置文件复制到数据
chown -R mysql.mysql /var/lib/mysql/  改属性
systemctl start mariadb

示例：新版xtrabackup完全备份及还原
1 在原主机做完全备份到/data/backups
xtrabackup --backup --target-dir=/backups/
scp -r /backups/*不会生成新的文件，只拷贝到目录下 目标主机:/backups
2 在目标主机上
1）预准备：确保数据一致，提交完成的事务，回滚未完成的事务
xtrabackup --prepare --target-dir=/backups/
2）复制到数据库目录
注意：数据库目录必须为空，MySQL服务不能启动
xtrabackup --copy-back --target-dir=/backups/
3）还原属性
chown -R mysql:mysql /var/lib/mysql
4）启动服务
systemctl start mariadb
