# Redis事务

Redis事务本质是一系列命令的集合；

一个事务的所有命令都会序列化，事务执行过程中，会串行化执行命令；

- 一次性，顺序性，排他性的执行一个队列中的一系列命令；
- Redis事务没有隔离级别的概念；
- 单个Redis实例在执行事务期间，不会响应其他客户端；

# 执行过程

1. 开始事务：multi
2. 命令入队多条命令
3. 放弃事务：discard
4. 执行事务：exec

# 事务执行场景

1、命令全部正确，则执行成功

2、命令在入队时，出现错误(命令不存在)，则全部命令不执行；

```shell
> multi
OK
> set k2 v2
QUEUED
> setget # 错误命令，报错
(error) ERR unknown command 'setget'
> exec # 全部命令不会执行
(error) EXECABORT Transaction discarded because of previous errors.
```

3、命令都成功入队，但是执行时，有命令失败，则继续执行，不会回滚；

```shell
> multi
OK
> set name kit
QUEUED
> incr name
QUEUED
-------------------命令全部入队成功-------------------
> exec
1) OK
2) (error) ERR value is not an integer or out of range
-------事务执行完毕，其中incr name命令执行失败，其他命令正常执行----------

> get name    # 事务依然成功
"kit"
```

# Lua脚本

Redis会将Lua脚本命令，作为整体执行，只能保证一起执行，但不保证原子性；

脚本的执行，满足上面的三种场景；
