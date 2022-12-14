# Explain执行计划

```sql
-- 使用
Explain + SQL
```

分析执行计划：（重点看Type，key，Extra）

- id：表示此表的执行优先级
  
  - id相同，表的执行顺序依次从上往下；
    
    ![](../.images/id_1.png)
  
  - id不同，并且递增，id值越大执行优先级越高，越先被执行；
    
    ![](../.images/id_2.png)

- select_type：读取一张表的操作类型
  
  - SIMPLE：简单SELECT查询，并且整个语句中的查询不包含<子查询>和<联合查询UNION>
  - PRIMARY：复杂查询中最外层查询，即PRIMARY
  - SUBQUERY：子查询
  - DERIVED：衍生，FROM后面跟的子查询；`select * from (select * from t1 where id = 1) d1`
  - UNION：联合查询
  - UNION RESULT：UNION合并的结果集；

- table：表示这一行操作，是哪一张表的

- **type**：表示此条查询的检索方式，看是否使用了索引（7种，下面是按照效率由高到低排列）
  
  - system：基本用不到，表示一张表，只有一条记录；
  
  - const：`where id = 1`指定的常量查询；（主键索引，走聚簇索引）
  
  - eq_ref：唯一索引扫描；对于某个索引键，表中只有一条记录与之匹配；（比如id主键索引）
  
  - ref：非唯一索引扫描；`where name = 'ZhangSan'`，如果ZhangSan不止一个人，且name字段建有索引，那么就是ref检索；（部分扫描）
    
    需要涉及两个B+树；
  
  - range：`where id between 30 and 60`或者类似`where id in (1,3,5)`或者`comments  > 1`
  
  - index：`select id from user`其中id是主键索引；（仍然全表扫描）
  
  - ALL：表明检索方式为全表扫描；出ALL类型检索，需要优化。
  
  - NULL：（效率最高）
    
    使用`is null`的时候；
    
    执行时甚至不用访问表或索引：如从一个索引列里选取最小值可以通过单独索引查找完成

- possible_keys：可能用到的索引

- **key**：实际使用的索引

- key_len：索引中使用的字节数
  
  ![](../.images/key_len.png)

- ref：ref是指用来与key中所选索引列比较的常量(const)或者连接查询列；
  
  可以看上面key_len的例子；

- rows：大致估算出每张表有多少行记录被查询了

- **Extra**：额外信息
  
  - Using filesort：文件排序，在无法通过索引来进行排序的情况下，就会默认使用文件排序；如果出现此信息，表示需要优化；
  
  - Using temporary：使用临时表，表示在排序时，使用了临时表；也是提示我们，此语句需要优化；
    
    常见于：排序，分组查询（group by，order by）
  
  - Using index：表明相应的查询操作中，使用了**覆盖索引**；
    
    如果同时伴有Using where：表明索引被用来执行主键的查找；
    
    （就是说通过col1的索引B+树，找到了对应的主键，再通过主键索引的B+树，找到了全部数据）
    
    ![](../.images/usingindex.png)
    
    如果不带有Using where：表明索引用来读取数据，而非查找动作；
    
    ![](../.images/usingindex_2.png)
  
  - impossible where：表示错误的where，比如`where name = 'lisi' and name = 'zhangsan'`
  
  - distinct：自动优化distinct关键字，在找到第一条匹配项后停止

Explain例子

![](../.images/explain.png)

表的执行顺序：

1. 【select name, id from t2】
2. 【select id, name from t1 where other_column = ' '】子查询
3. 【select id, from t3 】衍生查询
4. 【select d1.name ...】最外层查询
5. UNION操作

# 优化思路

## SQL优化

1、定位需要优化的SQL，也就是响应时间比较长的SQL

首先要开启慢查询日志（不是为了调优，不建议开启，影响性能）

慢查询日志：会记录响应时间超过阈值的SQL语句；（默认阈值10s）

2、定位到SQL，通过Explain执行计划，查看SQL的执行过程；

主要看：

（1）Type：是否使用了索引；

- All：全表扫描
- ref：走了非唯一索引（存在回表）；
- eq_ref：走了唯一索引；
- const：表示一次索引，就找到了数据（覆盖了索引），不需要优化；

（2）Key：具体使用了哪个索引；

（3）Extra：判断回表的情况，最好是使用覆盖索引（）

- Using Index：使用了覆盖索引，是最好的；

- Using filesort：表明使用了一次额外的排序，即没有走唯一索引；

- Using temporary：使用了临时表；比如子查询，建了临时表，用完还会删除，降低性能；
  
  （临时表：自动创建的用来存储中间结果）

3、使用show profile查看SQL的执行耗时；

首先要开启:

```sql
set profiling=1; -- 开启profile
```

直接使用`show profile`查看最近几条SQL的执行耗时；

```sql
> show profiles; -- 获取执行sql的时间
+----------+------------+-----------------------+
| Query_ID | Duration   | Query                 |
+----------+------------+-----------------------+
|        1 | 0.00054100 | show databases        |
|        2 | 0.02368350 | SELECT DATABASE()     |
|        3 | 0.00561150 | show tables           |
|        4 | 0.03310350 | select * from user    |
|        5 | 0.00008600 | explain show profiles |
+----------+------------+-----------------------+
```

继续查询某个SQL的具体，每一步的耗时；

```sql
mysql> show profile for query 4;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000037 |
| checking permissions | 0.000004 |
| Opening tables       | 0.022872 |
| init                 | 0.000019 |
| System lock          | 0.000019 |
| optimizing           | 0.000004 |
| statistics           | 0.000007 |
| preparing            | 0.000006 |
| executing            | 0.000002 |
| Sending data         | 0.010039 |
| end                  | 0.000009 |
| query end            | 0.000008 |
| closing tables       | 0.000007 |
| freeing items        | 0.000060 |
| cleaning up          | 0.000012 |
+----------------------+----------+
```

## 使用缓存

- 缓存中间件：Redis、MongoDb

- 本地缓存：MyBatis缓存等