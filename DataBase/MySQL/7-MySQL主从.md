# 主从同步

## 主从复制原理

主要依靠

- 主库的bin log文件（记录所有的操作的SQL语句）

- 从库的relay log中继日志；

一共需要**三个线程**完成：

1、配置主从

2、主库启动bin log**输出线程**

- 将bin log复制给从库的relay log

3、从库启动两个线程：**IO线程**、**SQL线程；**

- IO线程：用于复制主库的binlog语句，写入从库的relay log
- SQL线程：用于执行relay log的SQL语句，完成数据同步；
