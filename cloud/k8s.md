# k8s

## cli

```shell
# 应用
k create -f master-deployment.yaml
k apply -f autocv_deployment.yaml

# 获取所有命名空间
k get ns 

# 创建并运行一个或多个容器镜像
k run NAME --image=<imagename> --namespace=<namespace>

k -n <namespace> describe pod <podname>

k -n <namespace> logs <podname>

# 获取命名空间下所有的pod
k -n <namespace> get pods
# 删除指定命名空间下的pod
k -n <namespace> delete pod <pod_name>

# 获取命名空间下所有的deployment
k -n <namespace> get deployment
# 删除指定命名空间下的deployment
k -n <namespace> delete deployment <deployment_name>

# 进入k8s容器中
k -n <namespace> exec -it <podname> -- bash # 只有一个container
k -n <namespace> exec -it <podname> --container <containername> -- bash # 多个container

# k8s pod中多进程通信使用共享内存，共享内存不足报错
RuntimeError: DataLoader worker (pid 2479) is killed by signal: Bus error. It is possible that dataloader's workers are out of shared memory. Please try to raise your shared memory limit.

# k8s的端口转发，如果不指定address，默认转发的主机是127.0.0.1
k port-forward -n <namespace> <podname> --address 0.0.0.0 8080:8080
```

## helm

> [helm入门教学](https://zhuanlan.zhihu.com/p/350328164)

## 其他

- 教学视频
  - [一小时入门k8s](https://www.youtube.com/watch?v=s_o8dwzRlu4)
- [中文网站](https://www.kubernetes.org.cn/doc-48)
- [Kubernetes in Action(Second Edition)](https://wangwei1237.github.io/Kubernetes-in-Action-Second-Edition/)

