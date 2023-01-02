# 定义pod对象

通过yaml文件定义pod对象：

```yaml

```

查看集群所有节点：

```shell
kubectl get nodes
------------------
NAME     STATUS     ROLES           AGE     VERSION
master   NotReady   control-plane   3d11h   v1.26.0
```

根据配置文件，创建集群资源

```shell
kubectl apply -f [config.yaml]
```

查看集群部署的应用：

```shell
kubectl get pods -A
```

# 日志相关

查看指定pod日志：

```shell
kubectl logs [pods_name] [namespace] 
```
