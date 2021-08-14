# mysql命令



### 常用命令

- 查看master状态：SHOW MASTER STATUS;
- 查看binlog列表：show binary logs;  或者  SHOW BINARY LOGS;
- 查看binlog模式：show global variables like '%binlog_format%';
- 查看binlog相关变量：show global variables like '%binlog%';
- 设置binlog的row模式下为增量模式（global为全局生效，不用global为会话生效）：set global binlog_row_image ='MINIMAL';
- 设置binlog的row模式下为全量模式（global为全局生效，不用global为会话生效）：set global binlog_row_image ='full';
- flush 刷新log日志，自此刻开始产生一个新编号的binlog日志文件：flush logs;
- 重置（清空）所有binlog日志：reset master;
- 查询用户权限：show grants for 'bill_test'@'%';
- 分配所有权限：grant  all on *.* to 'bill_test'@'%';
- 分配特定权限：grant  RELOAD, REPLICATION SLAVE, REPLICATION CLIENT, select,insert,update,delete on *.* to 'bill_test'@'%';
- 撤销用户所有权限：revoke all on *.* from 'bill_test'@'%';
- 撤销用户特定权限：revoke RELOAD, delete  on *.* from 'bill_test'@'%';
- 设置

### mysqlbinlog

##### 参数

- --base64-output=# 确定输出语句何时应为base64编码的BINLOG语句。有三种情况：
  1. never禁用它并仅适用于没有基于行的事件的BINLOG。
  2. decode rows 如果还提供了--verbose选项，则将行事件解码为带注释的伪SQL语句，一般采用此选项。
  3. auto 仅在必要时打印base64（即对于基于行的事件和格式描述事件），默认值。

- --start-datetime=#  转储日志的起始时间。
- --stop-datetime=#  转储日志的结束时间。
- --start-position =# 转储日志的起始位置。
- --stop-position=#  转储日志的结束位置。
- --database=#   仅显示指定数据库的转储内容。
- --offset=#      跳过前N行的日志条目。
- -S, --socket=name  用于连接的套接字文件。
- --user=# 复制的MySQL用户，只需要授予REPLICATION SLAVE权限。
- -h, --host=name 指定mysql服务器地址。
- -P, --port=3306 指定mysql服务器端口。
- -u, --user=name 指定mysql服务器用户名。
- -p, --password 指定mysql服务器用户密码。
- -v,  --verbose 从行事件中重构伪SQL语句-v-v在列数据类型上添加注释。
- --read-from-remote-server  从MySQL服务器读取二进制日志。如果不指定此参数，则会查找本地的binlog文件。
- --raw binlog日志会以二进制格式存储在磁盘中，如果不指定该选项，则会以文本形式保存。
- --stop-never：mysqlbinlog可以只从远程服务器获取指定的几个binlog，也可将不断生成的binlog保存到本地。指定此选项，代表只要远程服务器不关闭或者连接未断开，mysqlbinlog就会不断的复制远程服务器上的binlog。
- --result-file：用于设置远程服务器的binlog，保存到本地的前缀。譬如对于mysql-bin.000001，如果指定--result-file=/test/backup-，则保存到本地后的文件名为/test/backup-mysql-bin.000001。注意：如果将--result-file设置为目录，则一定要带上目录分隔符“/”。譬如--result-file=/test/，而不是--result-file=/test，不然保存到本地的文件名为/testmysql-bin.000001。

> 注：
>
> 1. 如果指定了--raw，mysqlbinlog获取事件后，并不会实时落盘，而是先保存在本地服务器的内存中，每4K刷盘一次。这也就减少了频繁的日志写操作。如果此时mysqlbinlog和主服务器之间的连接断开了，则内存中的binlog会马上刷新到磁盘中。
>
> 2. 尽管mysqlbinlog类似于从服务器，但从服务器上的relaylog却是实时存盘的，即从服务器获取主服务器产生的事件后，会实时写入到relaylog中。
>
> 3. 如果不指定--raw，这个时候会以文本格式存盘，此时，--result-file=/test/不能指定为目录，必须明确写上文件名，譬如--result-file=/test/1.sql，此时，mysqlbinlog获取事件后，是实时落盘的，不会每4K刷盘一次。
> 4. 对于mysqlbinlog，如果断开了，并不会自动连接，可通过脚本来弥补上述不足：
>
> ```BASH
> #!/bin/bash
> BACKUP_BIN=/usr/bin/mysqlbinlog
> LOCAL_BACKUP_DIR=/backup/binlog/BACKUP_LOG=/backup/binlog/backuplog
> REMOTE_HOST=192.168.244.145REMOTE_PORT=3306REMOTE_USER=repl
> REMOTE_PASS=repl
> FIRST_BINLOG=mysql-bin.000001#time to wait before reconnecting after failure
> SLEEP_SECONDS=10##create local_backup_dir if necessarymkdir -p ${LOCAL_BACKUP_DIR}
> cd ${LOCAL_BACKUP_DIR}
> ## 运行while循环，连接断开后等待指定时间，重新连接while :do
> if [ `ls -A "${LOCAL_BACKUP_DIR}" |wc -l` -eq 0 ];then
>    LAST_FILE=${FIRST_BINLOG}  else
>    LAST_FILE=`ls -l ${LOCAL_BACKUP_DIR} | grep -v backuplog |tail -n 1 |awk '{print $9}'`  
> fi
> ${BACKUP_BIN} --raw --read-from-remote-server --stop-never --host=${REMOTE_HOST} --port=${REMOTE_PORT} --user=${REMOTE_USER} --password=${REMOTE_PASS} ${LAST_FILE}  echo "`date +"%Y/%m/%d %H:%M:%S"` mysqlbinlog停止，返回代码：$?" | tee -a ${BACKUP_LOG}  echo "${SLEEP_SECONDS}秒后再次连接并继续备份" | tee -a ${BACKUP_LOG}
> sleep ${SLEEP_SECONDS}done
> ```

#### 栗子

1. 远程读取binlog.000033文件：mysqlbinlog -uroot -p -h192.168.0.12 -P3307 --database=gamers --read-from-remote-server --base64-output=decode-rows -v -- start-position=335826506 binlog.000033
2. 读取本地binlog日志：mysqlbinlog --stop-datetime="2017-08-16 15:00:00" mysqld-bin.000001
3. 恢复binlog到数据库：mysqlbinlog mysql-bin.0000xx | mysql -u用户名 -p密码 数据库名

### mysqldump

#### mysqldump 简介

`mysqldump` 是 `MySQL` 自带的逻辑备份工具。

它的备份原理是通过协议连接到 `MySQL` 数据库，将需要备份的数据查询出来，将查询出的数据转换成对应的`insert` 语句，当我们需要还原这些数据时，只要执行这些 `insert` 语句，即可将对应的数据还原。

#### 备份命令

##### 命令格式

```java
mysqldump [选项] 数据库名 [表名] > 脚本名
```

或

```java
mysqldump [选项] --数据库名 [选项 表名] > 脚本名
```

或

```java
mysqldump [选项] --all-databases [选项]  > 脚本名
```

##### 选项说明

| 参数名                          | 缩写 | 含义                          |
| ------------------------------- | ---- | ----------------------------- |
| --host                          | -h   | 服务器IP地址                  |
| --port                          | -P   | 服务器端口号                  |
| --user                          | -u   | MySQL 用户名                  |
| --pasword                       | -p   | MySQL 密码                    |
| --databases                     |      | 指定要备份的数据库            |
| --all-databases                 |      | 备份mysql服务器上的所有数据库 |
| --compact                       |      | 压缩模式，产生更少的输出      |
| --comments                      |      | 添加注释信息                  |
| --complete-insert               |      | 输出完成的插入语句            |
| --lock-tables                   |      | 备份前，锁定所有数据库表      |
| --no-create-db/--no-create-info |      | 禁止生成创建数据库语句        |
| --force                         |      | 当出现错误时仍然继续备份操作  |
| --default-character-set         |      | 指定默认字符集                |
| --add-locks                     |      | 备份数据库表时锁定数据库表    |

##### 实例

备份所有数据库：

```java
mysqldump -uroot -p --all-databases > /backup/mysqldump/all.db
```

备份指定数据库：

```java
mysqldump -uroot -p test > /backup/mysqldump/test.db
```

备份指定数据库指定表(多个表以空格间隔)

```java
mysqldump -uroot -p  mysql db event > /backup/mysqldump/2table.db
```

备份指定数据库排除某些表

```java
mysqldump -uroot -p test --ignore-table=test.t1 --ignore-table=test.t2 > /backup/mysqldump/test2.db
```

#### 还原命令

##### 系统行命令

```java
mysqladmin -uroot -p create db_name 
mysql -uroot -p  db_name < /backup/mysqldump/db_name.db

注：在导入备份数据库前，db_name如果没有，是需要创建的； 而且与db_name.db中数据库名是一样的才可以导入。
```

##### soure 方法

```java
mysql > use db_name
mysql > source /backup/mysqldump/db_name.db
```

