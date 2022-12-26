# Install

[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- image
- container
- repository

# Docker配置

配置文件：/etc/docker/daemon.json

```shell
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors":["https://xmgeb39x.mirror.aliyuncs.com"]
}
```

# 非root管理docker

1、赋予用户root权限

2、添加用户到docker组

docker安装时默认创建docker组

```shell
# 查看docker组
sudo cat /etc/group  |grep docker 

# 查看docker.sock所在组
ll /var/run/docker.sock 

# 添加will用户到docker组
sudo gpasswd -a will docker  

# 刷新用户组
newgrp docker

# 查看当前用户所在群组
id
```

# 启停Docker

```shell
sudo systemctl daemon-reload && sudo systemctl restart docker
```

[Ubuntu下搭建Kubernetes集群(3)--k8s部署 - 走看看 (zoukankan.com)](http://t.zoukankan.com/xl2432-p-10933022.html)

# 镜像命令

镜像查询：[https://hub.docker.com/](https://hub.docker.com/)

本地查询：

```shell
docker search [REPOSITORY]
```

pull：拉取镜像

```shell
docker pull [repository]:[tag]
```

查看本地镜像：

```shell
docker images
```

打包/导入镜像：

```shell
docker save -o [输出文件] [image_id/image_name:tag]
docker save -o [输出文件] image_id1 image_id2 # 打包多个镜像
-------------------------------------------------
docker save -o test.tar 5eeff402b659
docker save -o test.tar k8simage/kube-apiserver:v1.14.2

docker load -i test.tar
```

build：从Dockerfile创建

rm：删除镜像

```shell
docker rmi [image id]
# 如果修改过镜像，镜像可能指向多个名称，需要指定tag删除
docker rmi [REPOSITORY:TAG]
```

# 容器命令

create：创建

start：启动已存在容器

```shell
docker start [container id]
```

run = create + start 创建并启动

```shell
docker run `

-d ` 后台运行 
--name kibana ` 指定容器名  
--net com.whr ` 指定网络 
--ip 172.18.0.2 ` 在com.whr网络中指定IP  
-e ES_JAVA_OPTS="-Xms64m -Xmx128m" ` 配置环境变量 
--link elasticsearch:elasticsearch ` 关联容器（必须在同一网络）  
-p 5601:5601 ` 端口映射宿主机 
-v /elk/kibana.yml:/usr/share/kibana/config/kibana.yml ` 挂载目录：文件、目录  
-v /d/elk/kibana.yml:/usr/share/kibana/config/kibana.yml ` windows挂载目录 
kibana:7.8.0 ` 使用的镜像
```

exec：

```shell
docker exec -it [container id] bash
```

restart：重启容器

```shell
docker restart [container id]
```

stop/kill：停止容器

```shell
docker stop [container id/name] # 优雅停止
docker kill [container id/name] # 强制停止
```

rm：删除容器(需停止)

```shell
docker rm [container id/name]
```

# network

配置网络，分配容器IP

- bridge：可以自定义网段，固定容器ip；不指定则重启后重新分配ip给容器；可创建网桥，为容器指定ip；
- none
- host：使用主机端口，占用主机端口

查看所有network：

```shell
docker network ls
------------------------------------
NETWORK ID NAME DRIVER SCOPE
5c48fb70b629 elk-net bridge local
```

创建桥接网络：

```shell
docker network create [网络名称]
docker network create com.whr
```

# 常用镜像

MySQL

```shell
docker pull mysql:5.7

docker run -d `
-p 3306:3306 `
--name mysql `
--net=haiah `
--ip=172.18.0.100 `
-v /d/data/mysql/data:/var/lib/mysql `
-e MYSQL_ROOT_PASSWORD=root `
mysql:5.7
```

Redis

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

nginx

```shell
docker pull nginx:latest

docker run -d --name nginx -p 80:80 nginx:latest

docker run -d `
--name nginx `
-p 80:80 `
-v /d/data/nginx/nginx.conf:/etc/nginx/nginx.conf `
-v /d/data/nginx/log:/var/log/nginx `
-v /d/data/nginx/html:/usr/share/nginx/html `
-v /d/data/nginx/conf:/etc/nginx/conf.d `
nginx:latest
```

Etcd

```shell
docker pull bitnami/etcd:latest

docker run -d --name etcd-server `
--network haiah `
--publish 2379:2379 `
--publish 2380:2380 `
--env ALLOW_NONE_AUTHENTICATION=yes `
--env ETCD_ADVERTISE_CLIENT_URLS=http://etcd-server:2379 `
bitnami/etcd:latest
```
