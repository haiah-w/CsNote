# 安装

## 编译安装

1、下载对应版本：wget http://download.redis.io/releases/redis-5.0.14.tar.gz

2、解压到 `/opt` 下

```shell
tar -zxvf redis-5.0.14.tar.gz -C /opt/redis/
```

3、编译
```shell
make && make install
----------------------
# 安装编译环境
apt install make build-essential gcc
```

4、更改配置，启动
```conf
# 关闭保护模式，开启外部访问
protected-mode no
# 注释
# bind 127.0.0.1
# 后台启动
daemonize yes
```

启动
```shell
./src/redis-server redis.conf
```

## 包安装
```shell
apt-get install redis-server
---------------------------
# 以服务的方式启动：
service start redis-server
```

## docker
```shell
# 拉镜像
docker pull redis:5.7
# 直接启动不挂载
docker run --name redis-test -p 6379:6379 -d redis
# 配置启动
docker run `
--name redis-test `
-p 6379:6379 `
-v /d/data/redis/data:/data `
-v /d/data/redis/conf:/etc/redis/redis.conf `
-d redis redis-server /etc/redis/redis.conf `
redis:5.7
```

