# Redis命令行
Redis启停：
```shell
./src/redis-server ./redis.conf
-------------------------------
./src/redis-cli shutdown
./src/redis-cli shutdown -a [password]
```



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
