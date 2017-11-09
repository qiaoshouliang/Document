
# 修改提示符的命令如下所示：

```
mysql> PROMPT \u@\h \d>
PROMPT set to '\u@\h \d>'
root@127.0.0.1 (none)>USE test
Database changed
root@127.0.0.1 test>
```
---
# 查询表列的信息

```
root@localhost test>show columns from tdb_goods;
+-------------+------------------------+------+-----+---------+----------------+
| Field       | Type                   | Null | Key | Default | Extra          |
+-------------+------------------------+------+-----+---------+----------------+
| goods_id    | smallint(5) unsigned   | NO   | PRI | NULL    | auto_increment |
| goods_name  | varchar(150)           | NO   |     | NULL    |                |
| goods_cate  | varchar(40)            | NO   |     | NULL    |                |
| brand_name  | varchar(40)            | NO   |     | NULL    |                |
| goods_price | decimal(15,3) unsigned | NO   |     | 0.000   |                |
| is_show     | smallint(6)            | NO   |     | 1       |                |
| is_saleoff  | smallint(6)            | NO   |     | 0       |                |
+-------------+------------------------+------+-----+---------+----------------+
```

---

# 修改mysql的结束符
```
DELIMITER //
```
将 ；结束符改为了 //

---

# 分配guru2b用户在firstdb数据库下的所有权限
```
grant all on firstdb.* to guru2b@localhost identified by '112233';
```

# 用CONCAT连接列

```
SELECT concat(first_name," ",surnname) from sales_rep;

'Sol Rive'
'charlene Gordimer'
'Mike Serote'
'Mongane Rive'
'Mike Smith'
```

#批量存数数据库

数据的格式入下：

数据之间使用tab键来分割，使用回车来换行表明改行数据已经数据完毕
```
/Users/qiaoshouliang/Desktop/sales.sql

1	1	1	2000
2	4	3	250
3	2	3	500
4	1	4	450
5	3	1	3800
6	1	2	500
```

```
load data local infile '/Users/qiaoshouliang/Desktop/sales.sql' into table sales;
```


