# MySQL Binlog

# 一、简介

MySQL Binlog（MySQL Binary Log，MySQL的二进制日志文件）,它记录了数据库上的所有改变，并以二进制的形式保存在磁盘中；它的作用：

- 查看数据库的变更历史
- 数据库增量备份和恢复
- MySQL的复制（主从数据库的复制）

# 二、Binglog格式

Binlog有`STATEMENT、ROW、MIXED`三种格式

## 1、STATEMENT格式：基于SQL语句的复制

- 每一条会修改数据的sql都会记录在binlog中。

- 优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。

- 缺点：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同 的结果。另外mysql 的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题。

- 注意：相比row能节约多少性能与日志量，这个取决于应用的SQL情况，正常同一条记录修改或者插入row格式所产生的日志量还小于Statement产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter表等操作，ROW格式会产生大量日志，因此在考虑是否使用ROW格式日志时应该跟据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题。



## 2、ROW格式：基于行的复制

- 5.1.5版本的MySQL才开始支持row level的复制,它不记录sql语句上下文相关信息，仅保存哪条记录被修改。
- 优点： binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题.
- 缺点:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容。
- ps:新版本的MySQL中对row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录，如果sql语句确实就是update或者delete等修改数据的语句，那么还是会记录所有行的变更。



## 3、MIXED格式：混合模式复制

- 从5.1.8版本开始，MySQL提供了Mixed格式，实际上就是Statement与Row的结合。

- 在Mixed模式下，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog，MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种。

# 三、配置Binlog



```bash
show variables like '%binlog%'
# 检查MySQL是否已经开启binlog
show variables like 'log_bin'

# 查看binlog文件列表
show binary logs

# 查看binlog内容
show binlog events

# 查看当前最新一个binlog日志文件的状态信息，显示正在写入的二进制文件，及当前position
show master status;

#查看所有binlog日志列表
show master logs;
```





```bash
log-bin=/data/mysql/logs/binlogs/mysql-bin.log
expire-logs-days=14
max-binlog-size=500M
server-id=1
binlog_format=ROW
binlog_row_image=FULL
relay_log_info_repository=TABLE
```

# 三、mysqlbinlog命令查看Binlog

因为binlog日志文件是二进制文件`，没法用vi等打开，这时就`需要mysql的自带的mysqlbinlog工具进行解码



```
mysqlbinlog [参数] binlog文件路径

参数：
  -?, --help          Display this help and exit.
  --base64-output=name
                      Determine when the output statements should be
                      base64-encoded BINLOG statements: 'never' disables it and
                      works only for binlogs without row-based events;
                      'decode-rows' decodes row events into commented
                      pseudo-SQL statements if the --verbose option is also
                      given; 'auto' prints base64 only when necessary (i.e.,
                      for row-based events and format description events).  If
                      no --base64-output[=name] option is given at all, the
                      default is 'auto'.
  --bind-address=name IP address to bind to.
  --character-sets-dir=name
                      Directory for character set files.
  -d, --database=name List entries for just this database (local log only).
  --debug-check       Check memory and open file usage at exit .
  --debug-info        Print some debug info at exit.
  --default-auth=name Default authentication client-side plugin to use.
  -D, --disable-log-bin
                      Disable binary log. This is useful, if you enabled
                      --to-last-log and are sending the output to the same
                      MySQL server. This way you could avoid an endless loop.
                      You would also like to use it when restoring after a
                      crash to avoid duplication of the statements you already
                      have. NOTE: you will need a SUPER privilege to use this
                      option.
  -F, --force-if-open Force if binlog was not closed properly.
                      (Defaults to on; use --skip-force-if-open to disable.)
  -f, --force-read    Force reading unknown binlog events.
  -H, --hexdump       Augment output with hexadecimal and ASCII event dump.
  -h, --host=name     Get the binlog from server.
  -l, --local-load=name
                      Prepare local temporary files for LOAD DATA INFILE in the
                      specified directory.
  -o, --offset=#      Skip the first N entries.
  -p, --password[=name]
                      Password to connect to remote server.
  --plugin-dir=name   Directory for client-side plugins.
  -P, --port=#        Port number to use for connection or 0 for default to, in
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
                      /etc/services, built-in default (3306).
  --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
  -R, --read-from-remote-server
                      Read binary logs from a MySQL server. This is an alias
                      for read-from-remote-master=BINLOG-DUMP-NON-GTIDS.
  --read-from-remote-master=name
                      Read binary logs from a MySQL server through the
                      COM_BINLOG_DUMP or COM_BINLOG_DUMP_GTID commands by
                      setting the option to either BINLOG-DUMP-NON-GTIDS or
                      BINLOG-DUMP-GTIDS, respectively. If
                      --read-from-remote-master=BINLOG-DUMP-GTIDS is combined
                      with --exclude-gtids, transactions can be filtered out on
                      the master avoiding unnecessary network traffic.
  --raw               Requires -R. Output raw binlog data instead of SQL
                      statements, output is to log files.
  -r, --result-file=name
                      Direct output to a given file. With --raw this is a
                      prefix for the file names.
  --secure-auth       Refuse client connecting to server if it uses old
                      (pre-4.1.1) protocol.
                      (Defaults to on; use --skip-secure-auth to disable.)
  --server-id=#       Extract only binlog entries created by the server having
                      the given id.
  --server-id-bits=#  Set number of significant bits in server-id
  --set-charset=name  Add 'SET NAMES character_set' to the output.
  -s, --short-form    Just show regular queries: no extra info and no row-based
                      events. This is for testing only, and should not be used
                      in production systems. If you want to suppress
                      base64-output, consider using --base64-output=never
                      instead.
  -S, --socket=name   The socket file to use for connection.
  --start-datetime=name
                      Start reading the binlog at first event having a datetime
                      equal or posterior to the argument; the argument must be
                      a date and time in the local time zone, in any format
                      accepted by the MySQL server for DATETIME and TIMESTAMP
                      types, for example: 2004-12-25 11:25:56 (you should
                      probably use quotes for your shell to set it properly).
  -j, --start-position=#
                      Start reading the binlog at position N. Applies to the
                      first binlog passed on the command line.
  --stop-datetime=name
                      Stop reading the binlog at first event having a datetime
                      equal or posterior to the argument; the argument must be
                      a date and time in the local time zone, in any format
                      accepted by the MySQL server for DATETIME and TIMESTAMP
                      types, for example: 2004-12-25 11:25:56 (you should
                      probably use quotes for your shell to set it properly).
  --stop-never        Wait for more data from the server instead of stopping at
                      the end of the last log. Implicitly sets --to-last-log
                      but instead of stopping at the end of the last log it
                      continues to wait till the server disconnects.
  --stop-never-slave-server-id=#
                      The slave server_id used for --read-from-remote-server
                      --stop-never.
  --stop-position=#   Stop reading the binlog at position N. Applies to the
                      last binlog passed on the command line.
  -t, --to-last-log   Requires -R. Will not stop at the end of the requested
                      binlog but rather continue printing until the end of the
                      last binlog of the MySQL server. If you send the output
                      to the same MySQL server, that may lead to an endless
                      loop.
  -u, --user=name     Connect to the remote server as username.
  -v, --verbose       Reconstruct pseudo-SQL statements out of row events. -v
                      -v adds comments on column data types.
  -V, --version       Print version and exit.
  --open-files-limit=#
                      Used to reserve file descriptors for use by this program.
  -c, --verify-binlog-checksum
                      Verify checksum binlog events.
  --binlog-row-event-max-size=#
                      The maximum size of a row-based binary log event in
                      bytes. Rows will be grouped into events smaller than this
                      size if possible. This value must be a multiple of 256.
  --skip-gtids        Do not print Global Transaction Identifier information
                      (SET GTID_NEXT=... etc).
  --include-gtids=name
                      Print events whose Global Transaction Identifiers were
                      provided.
  --exclude-gtids=name
                      Print all events but those whose Global Transaction
                      Identifiers were provided.

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
base64-output                     (No default value)
bind-address                      (No default value)
character-sets-dir                (No default value)
database                          (No default value)
debug-check                       FALSE
debug-info                        FALSE
default-auth                      (No default value)
disable-log-bin                   FALSE
force-if-open                     TRUE
force-read                        FALSE
hexdump                           FALSE
host                              (No default value)
local-load                        (No default value)
offset                            0
plugin-dir                        (No default value)
port                              0
read-from-remote-server           FALSE
read-from-remote-master           (No default value)
raw                               FALSE
result-file                       (No default value)
secure-auth                       TRUE
server-id                         0
server-id-bits                    32
set-charset                       (No default value)
short-form                        FALSE
socket                            /data/mysql/data/mysqld.sock
start-datetime                    (No default value)
start-position                    4
stop-datetime                     (No default value)
stop-never                        FALSE
stop-never-slave-server-id        -1
stop-position                     18446744073709551615
to-last-log                       FALSE
user                              (No default value)
open-files-limit                  64
verify-binlog-checksum            FALSE
binlog-row-event-max-size         4294967040
skip-gtids                        FALSE
include-gtids                     (No default value)
exclude-gtids                     (No default value)

```



# 参考：

1. https://blog.csdn.net/ouyang111222/article/details/50300851