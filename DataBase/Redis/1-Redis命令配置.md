# Redis启停

```shell
./src/redis-server ./redis.conf
-------------------------------
./src/redis-cli shutdown
./src/redis-cli shutdown -a [password]
```

# Redis基本命令

Sheet：[Redis Cheat Sheet & Quick Reference](https://quickref.me/redis#redis-string-command)

1. exists [key]：查看是否存在

2. ttl [key]：查看key剩余有效时间
   
   -1：永久有效；
   
   -2：已经失效；

3. expire [key] [time]：设置key的过期时间

4. PX [key]：set的同时，设置过期时间

5. NX [key]：只在键不存在时，才对键进行设置操作。

6. persist [key]：移除key的过期时间；变成永久key

7. del [key]：删除key

8. keys：匹配正则，查找key，但是会阻塞服务；

9. scan [partten]：无阻塞的匹配正则；花费时间长，但是不会阻塞服务；

# Redis配置文件

```conf
####################### NETWORK #######################


####################### GENERAL #######################
# 守护进程启动
daemonize yes

# 日志：debug、verbose、notice、warning
loglevel notice
logfile "/opt/redis/logs/redis.log"

####################### SNAPSHOTTING #######################

save 900 1
save 300 10
save 60 10000
```
