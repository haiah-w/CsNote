# Redis基本命令

1. exists：查看是否存在
   
   返回1：存在；0：不存在
   
   ```shell
   master:6379> exists key1
   (integer) 1
   ```

2. ttl：查看key剩余有效时间
   
   -1：永久有效；
   
   -2：已经失效；
   
   ```shell
   master:6379> ttl key1
   (integer) -1
   ```

3. expire：设置key的过期时间
   
   ```shell
   master:6379> expire key1 10
   (integer) 1    # 设置成功
   master:6379> ttl key1
   (integer) 8    # 还剩8秒
   master:6379> ttl key1
   (integer) -2    # 永久失效
   master:6379> get key1
   (nil)
   ```

4. PX：set的同时，设置过期时间

5. NX：只在键不存在时，才对键进行设置操作。 

6. persist：移除key的过期时间；变成永久key

7. del key：删除key

8. **keys：匹配正则，查找key，但是会阻塞服务；**

9. **scan：无阻塞的匹配正则；花费时间长，但是不会阻塞服务；**

# Redis内部结构

## Redis服务器

Redis服务器的16个库由redisServer结构体来存储：

```c
struct redisServer{
    //...
    redisDb *db;
}
```

## Redis客户端

Redis客户端，通过修改指向的Redis服务器的db指针，来切换数据库；

<img title="" src=".../images/image-20200812225722470.png" alt="image-20200812225722470" style="zoom: 67%;" width="345" data-align="center">

## Redis键空间

Redis中的一个库下所有k-v全都保存在一个字典内部：

一个库一个键空间：

```c
typedef struct redisDb{
    //...
    // 键空间
    dict *dict;
    // 过期字典
    dict *expires;
    //...
} redisDb;
```

- dict：存储了所有的key-value；

- expires：
  
  存储了设置了过期时间的key，value为计算出的过期时间点Unix时间戳；
  
  value是long类型；
  
  时间戳，精确到毫秒；
  
  （1）添加过期时间：将key添加进过期字典；value设置为当前时间戳；
  
  （2）删除过期时间：将key从过期字典中删除；
  
  （3）判断过期：value>当前时间戳，则没有过期；小于，则说明已经过期；
  
  （4）惰性删除/定期删除；

![image-20200812230305321](.../images/image-20200812230305321.png)

## RedisObject

Redis五种数据结构底层都是一个C语言下的redisObject结构体：

```c
typedef struct redisObject{
     //类型
     unsigned type:4;
     //编码
     unsigned encoding:4;
     //指向底层数据结构的指针
     void *ptr;
     //引用计数
     int refcount;
     //记录最后一次被程序访问的时间
     unsigned lru:22;

}robj
```

Redis中创建一个key-value，至少会创建两个redisObject：

键对象：一定是一个REDIS_STRING类型；

值对象；

- type：表明redisObject对象类型：
  
  - REDIS_STRING：字符串对象
  - REDIS_LIST：列表对象
  - REDIS_HASH：哈希对象
  - REDIS_SET：集合
  - REDIS_ZSET：有序集合

- ptr指针：指向具体的数据；
  
  - 如果是key，指向value（redisObject）
  - 如果是value，指向value的真正的值；

- refcount：引用计数；
  
  用于内存回收；
  
  - 当创建一个对象，引用计数初始化为1；
  - 当对象被一使用一次，计数+1；

## String

<mark>以下的sds结构已经过时</mark>，详见 ：[Redis进阶 - 数据结构：底层数据结构详解](https://pdai.tech/md/db/nosql-redis/db-redis-x-redis-ds.html#%E7%AE%80%E5%8D%95%E5%8A%A8%E6%80%81%E5%AD%97%E7%AC%A6%E4%B8%B2---sds)

![image-20200705172142405](.../images/image-20200705172142405.png)

Redis的字符串底层使用SDS（动态字符串），而不是C语言的字符串；

```c
// SDS的定义
strcut sdshdr{
    // 记录字符串长度
    int len;
    // 记录未使用的buf；
    int free;
    // 字符数组，存放实际的字符串
    char buf[];
}
```

![image-20200705163504687](.../images/image-20200705163504687.png)

为什么不使用C的字符串？

| C字符串                | SDS                  |
|:-------------------:|:--------------------:|
| 获取字符串长度复杂度：O(N)     | 获取字符串长度复杂度：O(1)      |
| 不安全，可能缓冲区溢出         | 安全，不会溢出，不会泄露         |
| 修改字符串，必须执行N次内存的重新分配 | 修改字符串，除非是全部改需要N次内存分配 |

![image-20200705163837635](.../images/image-20200705163837635.png)

## List

底层双向链表

```c
typedef strcut list{
    // 头节点
    listNode *head;
    // 尾节点
    listNode *tail;
    // 节点数量
    unsigned long len;
    // 节点复制函数
    void *(*dup)(void *ptr);
    // 节点释放函数
    void *(*free)(void *ptr);

}
```

```c
typedef strcut listNode{
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点值
    void *value;
}
```

![image-20200814112944749](.../images/image-20200814112944749.png)

链表特点：

- 双向链表；
- 带有头尾指针；
- 记录链表长度；
- 多态：同样是listNode，但是可以通过void*指针，保存不同类型的值；（泛型的感觉）

## set（dict）

集合对象，有两种编码：`intset`、`hashtable`；

当集合对象，可以同时满足下面两条，则使用`intset`编码：

- 所有元素都为整数数值；
- 元素数量不超过512；

否则：使用`hashtable`编码（value为null的字典）

两种编码的存储方式：

<img src=".../images/image-20200814110518399.png" alt="image-20200814110518399" style="zoom: 67%;" />

## Zset跳跃表

是一种有序数据结构，通过每个节点中维持多个指向其他节点指针，达到快速访问；

复杂度：增删改查O（logn），最坏O（n）

- **通过节点的前进指针的个数（层），以及每个前进指针的跨度，来实现跳跃表；**

![image-20200705170215999](.../images/image-20200705170215999.png)

- 层：每个节点的多个指针，是一个指针集合List；同层之前指针关联；
- 前进指针：每一层的从头指向后面的指针；
- 跨度：记录两个节点间的距离；
- 后退指针：每个节点，只有一个后退指针，也就是全部后退指针相当于一个单链表；
- 分值Score：跳跃表，按照Score大小排序；
- 成员：也就是此节点的value；

Zset插入节点，跳跃表的结构：

（1）每插入一个节点，随机为当前节点设置一个层数；

结构源码：

```c
// zset结构（跳跃表外观）
typedef struct zskiplist{
    // 头节点，尾节点
    structz skiplisNode *header , *tail;
    // 节点数量
    unsigned long length;
    // 最大的节点层数；
    int level;
}

// 跳跃表的结构
typedef struct skiplisNode{
    // 层：是一个指针集合
    struct zskiplistLevel{
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;

}
```

按照上面两个结构，形成的总体为：

- zskiplist：元数据；
- zskiplistNode：每一个节点；

![image-20200717094520284](.../images/image-20200717094520284.png)

## Hash表

RedisObject中的ptr指针，指向的就是哈希对象：

```c
typedef struct dict{
    // 指向对哈希表操作的函数
    dictType *type;
    // 私有数据
    void *privdata;
    // ht[1]指向真正的哈希表结构,ht[2]用于备用扩容，指向正在扩容的哈希表
    dictht ht[2];
    // 是否在rehash：如果不在rehash，此值为-1；
    // 当rehash开始，用trehashidx来记录索引
    int trehashidx;
}
```

哈希表：

```c
typedef struct dictht{
    // 哈希数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 记录尾部：size-1
    unsigned long sizemask;
    // 已使用的大小
    unsigned long used;
}
```

哈希节点对象：

```c
typedef struct dictEntry{
    // key
    void *key;
    // value
    union{
        void *val;
        unit64_tu64;
        int64_ts64;
    } v;
    // next指针
    struct dictEntry *next;
} dictEntry;
```

![image-20200705180105601](.../images/image-20200705180105601.png)

- 上面的整个哈希表，出现了哈希冲突，使用链地址法解决哈希冲突；

### 渐进式ReHash

![image-20200705180817080](.../images/image-20200705180817080.png)

1、为ht[1]分配空间，`rehashidx`置为0，表示正在rehash，并以此记录索引；

2、采用`渐进式rehash`，不是rehash开始就一次性的把ht[0]都移动到ht[1]；

而是：保持rehash的状态，之后每次对此hash表的元素进行添加、删除、查找、更新时，除了执行相应的操作外，将`rehashidx`索引处的key-value，移动到ht[1]中；

3、随着对哈希表的操作，终会在某一次操作的时候，rehash完成；把h[1]整个放回ht[0]，清空ht[1]；

4、`渐进式rehash`：避免了集中rehash的计算量；

缺点：查找的时候，可能要找的键已经rehash到ht[1]中去了，所以，每次查找，两个ht[0]、ht[1]都要进行查找；

# Redis数据类型

reids key数量：能存2的32次方个key

### String

最大value：512MB

set/get

```bash
127.0.0.1:6379> select 0
OK
127.0.0.1:6379> set key_1 value_1
OK
127.0.0.1:6379> get key_1
"value_1"
```

批处理mget/mset

```bash
127.0.0.1:6379> mset name gdx age 17 data 9102-1-1
OK
127.0.0.1:6379> mget name age data
1) "gdx"
2) "17"
3) "9102-1-1"
```

incr/decr

```bash
127.0.0.1:6379> incr age   # 自增1
(integer) 18
127.0.0.1:6379> decr age   # 自减1
(integer) 17
```

#### 场景

String应用于常规key-value缓存应用； 常规计数：微博数，粉丝数等。

### Hash

（无序散列表）

【key field value】 存储结构化数据（相当于mysql中的一个table）

key：表名；

field：字段名；

value：值；

hget / hset

```bash
127.0.0.1:6379> hset user name ly
(integer) 1
127.0.0.1:6379> hget user name
"ly"
```

hmset  /  hmget  /  hgetall

```bash
127.0.0.1:6379> hmset user age 17 gender male
OK
127.0.0.1:6379> hmget user age gender    # fields
1) "17"
2) "male"
127.0.0.1:6379> hgetall user    # key
1) "age"
2) "17"
3) "gender"
4) "male"
```

hexists(查看是否存在某key下的field)  /  hlen（查看user表数据量）

```bash
127.0.0.1:6379> hexists user age
(integer) 1
127.0.0.1:6379> hlen user
(integer) 2
```

#### 场景

- hash 特别适合用于存储对象、
- 比如我们可以使用hash 数据结构来存储用户信息，商品信息等等。
- 可以在单点登陆的时候，用hash存储用户信息，模拟session

### List

List的实现为一个双向链表，即可以支持反向查找和遍历；

多用于实现双端出入的队列；

【list key vlaue】

**rpush（**从list右侧依次插入）

**lpush**（从左侧依次插入）

**lrange**（输出所有list下的所有key）

**rpop**（从右侧弹出一个key）

```bash
127.0.0.1:6379> rpush fruit apple banana pear
(integer) 3
127.0.0.1:6379> lpush fruit orange  
(integer) 4
127.0.0.1:6379> lrange fruit 0 -1    # 遍历全部
1) "orange"
2) "apple"
3) "banana"
4) "pear"
127.0.0.1:6379> rpop fruit            # 弹出pear
"pear"
127.0.0.1:6379> lrange fruit 0 -1
1) "orange"
2) "apple"
3) "banana"
```

#### 场景

- 多数的列表场景，粉丝列表，消息列表，关注列表，商品列表等结构，都可以使用List实现
- 可以用range命令，做基于Redis的分页，性能极好；
- **可以做队列；rpush生产消息，lpop消费消息；**

### set

无序集合，用于全局去重，通过交集，差集，并集等计算公共喜好；

【set1  member 】

无序集合 集合成员唯一 

```bash
127.0.0.1:6379> sadd nums_1 1 2 1
(integer) 2
127.0.0.1:6379> smembers nums_1        # 查看成员
1) "1"
2) "2"
127.0.0.1:6379> sinter nums_1 nums_2   # 查看交集
1) "1"
127.0.0.1:6379> sunion nums_1 nums_2     # 求并集
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> sdiff nums_1 nums_2     # 求差集 nums_1有，nums_2 没有
1) "2"
```

#### 场景

和List的场景类似，但是set自动去重；

- 当需要存储列表数据，又不希望有重复数据的时候，使用set；
- 可以通过set的交集，实现共同关注，共同喜好等功能；

### sorted set

（有序集合类型，增加了score权重参数）底层：跳跃表实现；

有序集合 集合成员唯一 （按score排序）

```bash
127.0.0.1:6379> zadd nums 100 a   # 添加'a'，score为100
(integer) 1
127.0.0.1:6379> zadd nums 99 b
(integer) 1
127.0.0.1:6379> zrange nums 0 -1
1) "b"
2) "a"
127.0.0.1:6379> zrange nums 0 -1 withscores
1) "b"
2) "99"
3) "a"
4) "100"
127.0.0.1:6379> zrangebyscore nums 99 100  # 查找分数在【99-100】内的成员
1) "b"
2) "a"
```

#### 场景

- 在直播场景中，实时的粉丝排行，礼物排行等等需要排序的场景，可以使用Sorted Set；
- 可以做延时队列；用时间戳做score，消息内容作为key，消费者就可以通过zrangebyscore获取N秒之前的数据；

# Redis网络模型

Redis使用单线程+多进程的方案；

- 支持高并发连接
  
  体现在I/O多路复用程序，可以并发的监听多个Socket，为其绑定不同的事件

- (worker)工作是单线程（一个Redis进程同时只能处理一个IO事件）；

### 模型

![](.../images/redissingle.png)

- Redis每个客户端都有一个指令队列，同一队列的指令顺序执行，响应类似；
- 不同队列之间由Redis的单线程Epoll多路复用切换处理；
- 一个Redis实例同一时刻只会处理一个命令或响应；

## 网络模型

Redis基于Reactor模型

1、通过`I/O多路复用程序`来同时监听多个Socket；（**并发处理**）

2、`I/O多路复用程序`按照Socket的执行事件（连接、读取、写入等），为其关联不同的事件；

3、然后`I/O多路复用程序`，就会将Socket加入一个队列中，同步地，一个个地向`文件事件分派器`发送Socket（**这里是串行执行**）

4、`文件事件分派器`根据Socket的事件，调用不同的事件处理器（就是一个个函数）；

![](.../images/redismodule.PNG)

## I/O多路复用程序

Redis的IO多路复用程序有多种：

- select
- epoll
- evport
- kqueue

Redis对这四种IO多路复用函数库，都进行了封装实现，然后根据操作系统，来选择性能最高的程序；

<img src=".../images/image-20200814203840393.png" alt="image-20200814203840393" style="zoom:80%;" />

### 高性能

Redis的单线程如何保证其高性能？

#### 1. 纯内存访问

正是因为Redis是基于内存，CPU不是限制Redis速度的瓶颈，内存大小才是限制Redis的瓶颈，所以直接采用单线程即可，而且实现简单；

#### 2. 避免了线程切换

线程切换一定有性能损耗，避免线程切换；

避免竞态（加锁降低性能）的资源消耗；

#### 3. I/O多路复用

采用非阻塞I/O的Epoll多路复用技术—即：让单线程处理多个连接请求；

多路-指的是多个socket连接，复用-指的是复用一个线程 

# Redis过期策略

Redis采用：**定期删除+惰性删除**来对过期的key进行处理；

### 定期删除

定期删除：在设置key的时候，给定一个expire time（过期时间），指定key的存活时间；

但是：Redis并不会监控这些key，到期就删除；

而是：Redis回每隔100ms**随机抽取**一些具有过期时间的key，进行检测是否过期，如果过期就删除；

存在问题：既然是随机抽取，就会出现很多到期的key，并没有被抽到，从而没有被删除；

### 惰性删除

每当获取某个key的时候，Redis都会进行检查是否设置了过期时间，是否过期；

存在问题：仍然有可能过期的key，既没有定期删除，也没有被请求访问，导致过期key一直占用内存，就会有内存耗尽的可能；

这就需要内存淘汰机制了；

# Redis内存淘汰机制—热点数据

过期策略存在无法解决的key，可能导致redis内存耗尽；

这时候就需要：内存淘汰策略（多种淘汰策略）

```shell
# 配置内存淘汰策略
maxmemory-policy volatile-lru
```

六种：（三类：报错，AllKeys，volatile）

前提：当内存不足以容纳新写入数据时

- noeviction：新写入操作会报错；（不推荐）
- allkeys-lru：在键空间中，移除最近最少使用的key；（推荐）
- volatile-lru：在设置了过期时间的键空间中，移除最近最少使用的key（不推荐）
- allkeys-random：在键空间中，随机移除某个key；（不推荐）
- volatile-random：在设置了过期时间的键空间中，随机移除某个key（不推荐）
- volatile-ttl：在设置了过期时间的键空间中，有更早过期时间的key优先移除；（不推荐）

# Redis持久化

Redis宕机，内存是会清空的；

如果无法及时恢复数据，是容易造成缓存雪崩，关系数据库就会承受大量请求！

如何保证，宕机，重启，仍能快速恢复数据？——即：**故障恢复**

Redis提供两种持久化机制：

- RDB (快照)
- AOF (日志)
- 混合持久化 (4.0后引入)

## RDB

将某个时间点的全量数据，写入一个临时dump.rdb文件；

两个命令实现：

- SAVE：阻塞Redis服务器进程，知道RDB文件创建完成；
- BGSAVE：fork一个子进程进行创建RDB文件，主进程可以继续响应客户端；

RDB的创建：

（1）手动使用命令，创建dump.rdb文件（压缩的二进制文件）

（2）自动定期执行；（save配置）

RDB的载入：

（1）会优先执行AOF的载入，如果没有开启AOF，才会载入RDB文件；

（2）载入过程服务器一直处于阻塞状态；

RDB的配置：

```shell
# (默认配置)
save 900 1 # 900s内，修改了1次，就会执行BGSAVE
save 300 10
save 60 100
```

### 优点

1、只有一个dump.rdb文件。二进制序列，节奏紧凑，文件小；

2、性能好；

父进程fork一个子进程来进行写操作，主进程可以继续响应客户端，子进程进行持久化；

### 缺点

1、会丢数据，RDB每隔一定时间进行一次持久化；

2、时间间隔小，就会引起大量的磁盘IO，影响性能；

3、间隔大，就会存在宕机，丢失数据的风险；

## AOF

本质是：将**写命令**以纯文本的方式追加写入文件；

### AOF执行流程

1、命令追加到缓冲区；

Redis服务器，每执行一个写命令，就追加到缓冲区内；

2、缓存区命令，写入、同步到AOF文件

同步的操作，是一个循环事件，由配置的`appendonly`的属性来决定怎么执行：

```shell
# 不开启，修改同步，每秒
no，  always，  everysec 
```

### 特点

- 日志是连续的增量备份，写入操作时以`append-only`模式，写入一个日志文件；
- 日志记录的触发策略：（1）每秒同步（2）每次修改同步—修改自动触发同步，安全，效率低；
- Redis宕机，重启则重放日志的指令，完成数据恢复；

### 优点

1、数据安全，可以每次操作，都同步到aof文件；

### 缺点

1、体积大，文件比较大；（重写AOF）

2、效率低，数据量大，启动速度比较慢；

## 对比

1. RDB文件紧凑，体积小，数据恢复速度快；AOF体积大，数据恢复速度慢；

2. RDB基本不影响Redis的主进程性能；频繁持久化AOF对Redis性能影响较大；

3. 如果发生故障，RDB丢失数据较多，因为快照的间隔时间比较长；
   
   AOF的数据则更完整，触发写日志的时间间隔是秒级；

## 集群策略

- Master不要做持久化工作，专注于处理客户端任务；
  
  让某个Slave进行AOF备份数据，策略：每秒同步一次；

- 最好是Master，Slave通过内网链接，速度和稳定性有保障；

# Redis事务

### 概念

Redis事务本质是一系列命令的集合；

一个事务的所有命令都会序列化，事务执行过程中，会串行化执行命令；

- 一次性，顺序性，排他性的执行一个队列中的一系列命令；
- Redis事务没有隔离级别的概念；
- 单个Redis实例在执行事务期间，不会响应其他客户端；

### 执行过程

1. 开始事务multi
2. 命令入队多条命令（exec）
3. 放弃事务discard
4. 执行事务exec

### 执行成功条件

- 命令正确—执行成功

- 命令性错误（不存在此命令）—exec执行失败
  
  ```shell
  127.0.0.1:6379> multi
  OK
  127.0.0.1:6379> set k2 v2
  QUEUED
  127.0.0.1:6379> setget
  (error) ERR unknown command 'setget'
  127.0.0.1:6379> exec
  (error) EXECABORT Transaction discarded because of previous errors.
  ```

- 命令语法错误（命令正确，无法执行）—执行成功
  
  ```shell
  127.0.0.1:6379> multi
  OK
  127.0.0.1:6379> set name kit
  QUEUED
  127.0.0.1:6379> incr name    # 此命令不会执行，自动被忽略
  QUEUED
  127.0.0.1:6379> exec
  1) OK
  2) (error) ERR value is not an integer or out of range
  127.0.0.1:6379> get name    # 事务依然成功
  "kit"
  ```

# Redis+MySql

以查询员工信息为例，做Redis二级缓存；

流程：

1. 将数据库查询的员工信息，缓存进Redis；
2. 查询的时候，先进Redis查询，如果没有，则进数据库查询，并更新Redis缓存；

需要考虑的问题：

1. Key长时间存储与Redis，给缓存容量造成压力
   
   - 设置TTL缓存失效时间；

2. **缓存雪崩**：同一时间大面积的Key失效，导致大量请求进入数据库
   
   - 设置**随机的TTL**，防止大面积Key，同时失效；

3. **缓存穿透**：恶意用户模拟大小缓存不存在的Key，导致大量请求进入数据库；
   
   - 修改查询逻辑：Redis里没有数据，那么去数据库查
     
     查到塞入缓存
     
     查不到也塞入缓存，value设置为`""`(空)；

优化：

- 缓存预热；
- 主动推送MySql数据到redis缓存；

# 双写一致性

## Cache-Aside

缓存+数据库

- 读请求：
  
  先读Redis，有数据，直接返回；
  
  没有数据，查询数据库，数据放入缓存，返回；

- 更新：(失效模式)
  
  先更新数据库，再**删除Redis缓存**；不更新Redis缓存；（**懒加载思想**：真正要读的时候才去加载）

### 问题一：为什么删缓存，而不是更新？

如果某个数据在一段时间内，要修改100次；但是可能只读取几次；

每次都更新缓存，浪费性能；

因为每次的更新，并不一定要被读取，是没有意义的；

只需要在真正读的时候，查询数据库，放入Redis缓存即可；（懒加载）

### 问题二：为什么先更新数据库，不是先删缓存？

其实无论哪个先，都会造成脏数据；

比如：

**先删缓存：**

现在有请求A，更新数据，先删除了缓存，然后去修改数据库；

在修改数据库还没有完成的时候

来了读请求B，发现Redis中，没有数据，然后查询了数据库，将旧值又放进了Redis；

那么后续的读请求，读到的总是旧值；

**先更新数据库：**

缓存刚好失效
请求A查询数据库，得一个旧值
请求B将新值写入数据库
请求B删除缓存
请求A将查到的旧值写入缓存

- 只不过先更新数据库，这种方式，可以扩展补救措施；
- 并且，在并发不是很高的情况下，先更新数据库是比较好的；

### 问题三：删除缓存失败的补救措施

每次更新的操作，修改数据库之后，如果删除缓存失败；

借助**消息队列**：

在删除缓存失败后，需要删除的key入队，经过一段时间再次尝试删除；

失败继续入队，循环执行；

### binlog同步组件更新缓存

可以通过读取binlog的方式，解耦业务代码，即不在业务代码中，将key入队；

而是在非业务代码中，通过读取binlog，拿到执行了更新操作的数据和key，入队消息队列，尝试删除key；

## Cache-As-Sor

所有业务代码的增删改查，都是直接对缓存操作；

具体有没有数据，由缓存框架去对数据库完成操作；

优点：

- 业务代码简洁，只针对缓存操作；

缺点：

- 屏蔽了对数据库的操作；
- 操作性受限，可能一些特别的操作，无法实现；

# 缓存雪崩问题

缓存雪崩：同一时间缓存大面积失效，导致大量请求进入数据库，导致数据库崩溃；

解决方法：

1. 不同的key，在设置缓存的失效时间，加上一个随机值，避免集体失效；
2. 设置热点的key，永不过期；

# 缓存穿透

请求访问了一定不存在的Key，导致请求进入数据库，数据库也不存在的数据；

- 修改查询逻辑：Redis里没有数据，那么去数据库查
  
  查到塞入缓存
  
  查不到也塞入缓存，value设置为`""`(空)；
  
  缓存时间可以设置的稍微短一点；

- 布隆过滤解决
  
  判断是否有这个key，没有直接返回；（布隆过滤器能完全确定某key不存在）
  
  一定不会存在的key，就会被布隆过滤器拦截掉；

# 缓存击穿

请求访问了缓存不存在，但是数据库存在的key（可能缓存到期了）

- 热点的数据，设置永不过期；

- 接口限流，限制QPS；
  
  重要接口，做好限流，防止一时间请求过大，打入数据库，撑不住；

- 加互斥锁；（分布式锁）SetNX
  
  这样，只有一个线程进到数据库，查询到热点数据，再把数据放入缓存；
  
  让其他线程等待一定时间，再重新去缓存查询；就可以直接从缓存获取数据了；

# reboom

可以统计网站UV（日访问量）

https://segmentfault.com/a/1190000016721700

#### 目的

通过**布隆过滤器**判断一个集合中是否存在某个元素；

#### 结构

是一个**位数组+多个hash算法**

#### 实现原理

添加元素时，对某个元素，使用n个hash算法，求出n个值h(1)，h(2).....，将**位数组**中arr[h(1)]，arr[h(2)]....都置1；

判断的时候，只需要判断此元素，求出的hash值，对应位是否**都为1**，

- 如果不是，**一定不存在**；

- 如果是，可能存在；

#### 为什么有set还需要布隆过滤器？

- 布隆过滤器占用空间小：只需要维护一个**位数组**

- 查找快：hash算法+数组的随机访问；

#### 缺点

有误报率；

- 不能完全判定元素存在；
- 能完全判定元素的不存在；

# Redis分布式锁

`setNX`命令：

设置一个key，如果key存在，则返回false；key不存在，则设置成功；

一个线程如果设置key成功，则相当于拿到锁；

失败，则拿锁失败，不能继续后续逻辑；

Java API可以用：`valueOperations.setIfAbsent(key,value)`

- key不存在，创建，即拿到锁；

- key已存在，则不会创建新的key，返回false；

Jedis：`String lock = jedis.set(key, value, SetParams.setParams().ex(3).nx());`

- 加锁成功，返回"OK"

## 底层

猜想：事务执行

先判断key是否存在；

- 不存在，设置key-value，expire，返回1；
- 存在，直接返回0；

# Redisson

[参考文档]: https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器

封装了上面设置UUID，原子解锁的逻辑；

只需要：

```java
RLock rlock = Redisson.getLock("redisson-product-1");
rlock.lock(3,TimeUnit.SECONDS);
// 业务
rlock.unlock();
```

- Redisson锁，自动续时；
- 解锁，只解当前线程的锁；

### lock()

是一个阻塞方法，其他线程会阻塞住；

拿锁线程，如果解锁，会通过发布订阅模型，通知其他线程；

`lock(10,TimeUnit.SECONDS)`：10秒解锁；

### tryLock()

非阻塞；

`tryLock()`：没拿到锁的线程，立即返回`false`；

`tryLock(100,10,TimeUnit.SECONDS)`：尝试100s，如果拿到锁，设置10s超时时间；

### 读写锁

```java
RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
// 最常见的使用方法
rwlock.readLock().lock();
// 或
rwlock.writeLock().lock();
```

### 信号量

```java
RSemaphore semaphore = redisson.getSemaphore("lock-semaphore");
// 获取,可设置过期时间
semaphore.acquire();
String permitId = semaphore.acquire(2, TimeUnit.SECONDS);
// 释放
semaphore.release();
// 非阻塞获取
semaphore.tryAcquire();
```

- acquire()：阻塞方法；
- release()：释放

### 闭锁（CountDownLatch）

await的线程必须等到count为0；

```java
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(10);
latch.await();
// ...
latch.countDown();
```

# Redis主从

## Redis复制过程

分为两个步骤：

1、同步：全量复制；

2、命令传播：增量复制；

### 同步

1、从服务器向主服务器发送`PSYNC`命令，告知要进行全量复制；

2、主服务器执行`BGSAVE`，生成一个RDB文件，发送给从服务器，并开启一个缓冲区，记录之后执行的命令；

3、从服务器收到RDB文件，进行载入；

4、主服务器，向从服务器，发送缓冲区内的命令，从服务器执行接受的命令；

5、直到两个服务器达到数据一致，同步完成；

### 命令传播

如果出现，主从数据不一致，那么主服务器就会发送命令到从服务器执行，确保数据一致；

如何检测到数据不一致：

主从都会维护一个复制偏移量；（偏移量按字节计数）

![image-20200814205526185](.../images/image-20200814205526185.png)

- 因为偏移量的存在，即使从服务器下线，也能根据偏移量，重新同步到数据一致性的状态；
- 通过心跳，从服务器默认1s，发送一个心跳给主服务器，并携带偏移量；

缺陷：

- 一旦主节点宕机，需要手动将slave提升为master，各个应用要更新主节点地址；（致命）
- 简单的主从，只是备份数据，还是无法达到高可用，需要集群或Sentinal；

# Redis集群

<img src="image\image-20200522175607759.png" alt="image-20200522175607759" style="zoom: 67%;" />

## 集群实现

1、启动节点

节点启动，会根据配置文件的：`cluster-enabled`来决定是否启动集群模式；

几乎启动方式都跟普通启动一样，只不过会新增2个结构：

`clusterState`：集群状态信息；

`clusterNode`：保存集群中所有节点的信息；（包括自己）

- 包括：IP、端口、节点名、槽信息等；

<img src=".../images/image-20200816222019531.png" alt="image-20200816222019531" style="zoom: 80%;" />

2、节点握手

master之间，通过cluster meet命令，完成节点握手；

3、槽指派

Redis集群通过分片方式，保存数据；

槽：16384个（是一个bit数组），共16384/8 = 2048字节；

![image-20200816223558731](.../images/image-20200816223558731.png)

- 当前节点所使用的槽的范围，其值置为1；

当所有槽全部指派给所有集群完毕，整个集群才能上线；

存储的key的落点，一般是哈希进行索引；

## 集群下的查询操作

<img src=".../images/image-20200816223919975.png" alt="image-20200816223919975" style="zoom: 67%;" />

1、客户端发送键命令；（get / set）

2、计算槽

接受命令的服务器，会计算出key所在的槽；

```c
def slot_number(key):
    return CRC16(key) & 16383
```

3、判断槽，是否在当前节点：（查看槽数组的值是否为1）

- 在当前节点，直接进行键命令操作；

- 不再当前节点，查询出槽在哪个节点，返回给客户端；

4、客户端重新发送键命令；

## 集群下优化

- Master不要做持久化工作，专注于处理客户端任务；
  
  让某个Slave进行AOF备份数据，策略：每秒同步一次；

- 最好是Master，Slave通过内网连接，速度和稳定性有保障；

# Sentinel

哨兵是一个独立的进程，哨兵会实时监控master节点的状态，当master不可用时会从slave节点中选出一个作为新的master，并修改其他节点的配置指向到新的master。 

### 架构：

![](C:/Users/whr/Desktop/notes/数据库/.../images/Sentinel.PNG)

- 哨兵节点：哨兵节点是特殊的redis节点，不存储数据，Sentinel之间会相互通信；
- 数据节点：redis主从架构，主节点和从节点都是数据节点。 
- 每个Sentinel都会监控所有的redis实例；

## Sentinel的启动

1、初始化Sentinel服务器

Sentinel本质是一个特殊的Redis服务器，但是启动不会加载RDB、AOF文件；

2、使用Sentinel的专用代码

Sentinel不存数据，所以不需要get、set等命令；

但是需要各种检测Redis服务器状态、通信的代码：

比如：INFO、Ping

3、初始化Sentinel

这里主要是Sentinel的数据结构中的字典，也不再存储数据，而是存储Redis节点和其他Sentinel的节点信息数据；

<img src=".../images/image-20200817085750940.png" alt="image-20200817085750940" style="zoom:67%;" />

4、创建网络连接

此时Sentinel相当于Redis的客户端，需要创建连向Redis主服务器的连接：（两个连接）

- 命令连接：用以发送各种命令：INFO、Ping
- 订阅连接：

5、获取主服务器信息

通过INFO命令，获取主服务器信息：（两个信息）

- 主服务器的信息
- 主服务器下的所有从服务器的信息；

## 故障转义功能

核心功能：**主节点的自动故障转移**

1、检测主观下线

Sentinel每隔1s向所有Redis实例（包括其他Sentinel），发送Ping命令，来判断是否下线；

2、检测客观下线

如果一个Sentinel发现存在实例下线了，为了确认是否真的下线，会向其他Sentinel进行询问；

当从其他Sentinel接受到的确定下线的数量超过`<quorum>`个数，那么就确定客观下线；

开始执行故障转移；

3、选取领头Sentinel

各个Sentinel进行投票，选举一个领头Sentinel，完成故障转移；（投票超过半数）

4、开始故障转移

（1）领导者Sentinel来选取一个slave作为新的master，选取准则是取相似度最高的slave（偏移量最接近的）

（2）修改其他slave的复制目标为新的master

（3）所有Sentinel更新主节点信息；

（4）如果下线的master重新上线，就将其变成一个slave；

### 配置

在主从的基础上，还需要配置Sentinel：`sentinel.conf`；

```properties
port 26379
sentinel myid a780167ff20ae409c7803e2f7d5d55fbf120018f
daemonize yes
# Generated by CONFIG REWRITE
dir "D:\\Redis-6379"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel current-epoch 0
```

### 缺陷

- 写操作无法均衡负载

## 脑裂

脑裂：同时存在了多个Master

# Redis实现幂等
