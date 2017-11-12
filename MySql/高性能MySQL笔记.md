## ACID
ACID，指数据库事务正确执行的四个基本要素的缩写。包含：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）

## LOCK TABLES 的注意事项

除了事物中禁用了AUTOCOMMIT，可以使用LOCK TABLES之外，其他任何时候都不要显式执行LOCK TABLES，不管是使用什么存储引擎。