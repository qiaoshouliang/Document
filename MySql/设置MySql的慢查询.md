


# 慢查询日志配置
### 1. 显示慢查询的配置项
```
show variables like '%slow%';
+---------------------------+------------------------------------------+
| Variable_name             | Value                                    |
+---------------------------+------------------------------------------+
| log_slow_admin_statements | OFF                                      |
| log_slow_slave_statements | OFF                                      |
| slow_launch_time          | 2                                        |
| slow_query_log            | ON                                       |
| slow_query_log_file       | /usr/local/mysql/data/localhost-slow.log |
+---------------------------+------------------------------------------+
5 rows in set (0.00 sec)

```
### 2. 修改其中的某一条数据
```
set global slow_query_log=on;
```



MySQL 慢查询的相关参数解释：

slow_query_log    ：是否开启慢查询日志，1表示开启，0表示关闭。

log-slow-queries  ：旧版（5.6以下版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件host_name-slow.log

slow-query-log-file：新版（5.6及以上版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件host_name-slow.log

long_query_time ：慢查询阈值，当查询时间多于设定的阈值时，记录日志。

log_queries_not_using_indexes：未使用索引的查询也被记录到慢查询日志中（可选项）。

log_output：日志存储方式。log_output='FILE'表示将日志存入文件，默认值是'FILE'。log_output='TABLE'表示将日志存入数据库，这样日志信息就会被写入到mysql.slow_log表中。MySQL数据库支持同时两种日志存储方式，配置的时候以逗号隔开即可，如：log_output='FILE,TABLE'。日志记录到系统的专用日志表中，要比记录到文件耗费更多的系统资源，因此对于需要启用慢查询日志，又需要能够获得更高的系统性能，那么建议优先记录到文件。