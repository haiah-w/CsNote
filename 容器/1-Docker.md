- image
- container
- repository

# 镜像命令

- search：查询镜像，如果需要tag，只能去官网查：[https://hub.docker.com/](https://hub.docker.com/)
  
  ```shell
  docker search [REPOSITORY]
  ```

- pull：拉取镜像
  
  ```shell
  docker pull [repository]:[tag]
  ```

- ls：所有镜像
  
  ```shell
  docker images ls
  ```

- build：从Dockerfile创建
- push
- rm：删除镜像
  
  ```shell
  docker rmi [image id]
  ```

# 容器命令

- create：创建
- start：启动已存在容器
  
  ```shell
  docker start [container id]
  ```

- run = create + start 创建并启动
  
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

- exec：
  
  ```shell
  docker exec -it [container id] bash
  ```

- restart：重启容器
  
  ```shell
  docker restart [container id]
  ```

- stop/kill：停止容器
  
  ```shell
  docker stop [container id/name] # 优雅停止
  docker kill [container id/name] # 强制停止
  ```

- rm：删除容器(需停止)
  
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

创建容器并使用com.whr网络，添加 --net 参数

```shell
docker run -d `

--name elasticsearch `

--net com.whr `

...
```
