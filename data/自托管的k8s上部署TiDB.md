# TiDB在k8s上的部署

> [官方文档链接](https://docs.pingcap.com/zh/tidb-in-kubernetes/dev/deploy-tidb-operator#%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85-tidb-operator)
>
> - 建议先看官方文档，再看本篇文章。本篇文章步骤部分和官方文档中不同！！！
>
> - 这三个从范式文件服务器下载文件，可以和官方下载文件比较着看，改动了那些地方。
>
>   - local-volume-provisioner.yaml
>
>   - tidb-operator-v1.4.0-modified.tar.gz
>
>   - tidb-cluster.yaml

## 集群环境要求

**参考官方文档**

## 配置Storage Class

### 本地PV配置

#### 挂载和绑定

- `mout_bind_tidb.sh`

  ```shell
  for i in $(seq 1 9); do
    sudo mkdir -p /mnt/disk4/vol${i} /mnt/ti/vol${i}
    sudo mount --bind /mnt/disk4/vol${i} /mnt/ti/vol${i}
  done
  ```

- Persistent bind mount entries into /etc/fstab（确保开关机不影响绑定）

  ```shell
  for i in $(seq 1 9); do
    echo /mnt/disk4/vol${i} /mnt/ti/vol${i} none bind 0 0 | sudo tee -a /etc/fstab
  done
  ```

#### 部署local-volume-provisioner

1. 下载配置文件

   ```sh
   # 这里暂时不要从他们官网下载，有个修改版的放在了范式的文件服务器上
   wget https://raw.githubusercontent.com/huang3eng/mdpic/master/file/local-volume-provisioner.yaml
   ```

2. 部署 local-volume-provisioner 

   ```sh
   kubectl apply -f local-volume-provisioner.yaml
   ```

3. 检查 Pod 和 PV 状态

   ```sh
   kubectl get po -n kube-system -l app=local-volume-provisioner && \
   kubectl get pv 
   ```
   
   ![kube-system](https://raw.githubusercontent.com/huang3eng/mdpic/master/uPic/image2023-1-18_11-37-1.png)
   
   ![pv](https://raw.githubusercontent.com/huang3eng/mdpic/master/uPic/image2023-1-18_11-36-29.png)

## 部署TiDB Operator

### 准备环境

- Kubernetes v1.12 或者更高版本
- [DNS 插件](https://kubernetes.io/docs/tasks/access-application-cluster/configure-dns-cluster/)
- [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [RBAC](https://kubernetes.io/docs/admin/authorization/rbac) 启用（可选）
- [Helm 3](https://helm.sh/)

### 安装helm

- 参考 [使用 Helm](https://docs.pingcap.com/zh/tidb-in-kubernetes/stable/tidb-toolkit#使用-helm) 安装 Helm 并配置 PingCAP 官方 chart 仓库

### 部署TiDB Operator

#### 创建CRD

```shell
# 下载crd.yaml文件并安装
wget https://raw.githubusercontent.com/huang3eng/mdpic/master/file/crd.yaml
kubectl create -f ./crd.yaml

# 查看是否安装成功
kubectl get crd
```

#### 安装TiDB Operator

1. 下载 `tidb-operator` chart

   ```shell
   # 这里暂时不要从他们官网下载，有个修改版的放在了范式的文件服务器上
   wget https://raw.githubusercontent.com/huang3eng/mdpic/master/file/tidb-operator-v1.4.0-modified.tar.gz
   tar zxvf tidb-operato-v1.4.0-modified.tar.gz # 解压后的文件夹是tidb-operato
   ```

2. 下载所需要的相关镜像

   > 注意：后面所有需要外网下载的镜像都参考以下步骤。不再赘述！！

   ```sh
   # 在一台能够访问外网的机器上下载相关镜像
   docker pull pingcap/tidb-operator:v1.4.0
   docker pull pingcap/tidb-backup-manager:v1.4.0
   docker pull bitnami/kubectl:latest
   docker pull pingcap/advanced-statefulset:v0.3.3
   
   # 登录docker.4pd.io
   # username:deploy
   # password:GlW5SRo1TC3q
   docker login docker.4pd.io
   
   # 修改已下载镜像的相关tag
   docker tag pingcap/tidb-operator:v1.4.0 docker.4pd.io/pingcap/tidb-operator:v1.4.0
   docker tag pingcap/tidb-backup-manager:v1.4.0 docker.4pd.io/pingcap/tidb-backup-manager:v1.4.0
   docker tag bitnami/kubectl:latest docker.4pd.io/bitnami/kubectl:latest
   docker tag pingcap/advanced-statefulset:v0.3.3 docker.4pd.io/pingcap/advanced-statefulset:v0.3.3
   
   # 上传到docker.4pd.io
   docker push docker.4pd.io/pingcap/tidb-operator:v1.4.0
   docker push docker.4pd.io/pingcap/tidb-backup-manager:v1.4.0
   docker push docker.4pd.io/bitnami/kubectl:latest
   docker push docker.4pd.io/pingcap/advanced-statefulset:v0.3.3
   
   # 最后要记得在服务器上pull下来，如果有必要，可能要需要把tag变回去
   # 例如docker tag docker.4pd.io/pingcap/tidb-operator:v1.4.1 pingcap/tidb-operator:v1.4.1
   
   ```

3. 配置 TiDB Operator

   如果需要部署 `tidb-scheduler`，请修改 `./tidb-operator/values.yaml` 文件来配置内置 `kube-scheduler` 组件的 Docker 镜像名字和版本，例如你的 Kubernetes 集群中的 `kube-scheduler` 使用的镜像为 `k8s.gcr.io/kube-scheduler:v1.16.9`，请这样设置 `./tidb-operator/values.yaml`：

   ```yaml
   ...
   scheduler:
     serviceAccount: tidb-scheduler
     logLevel: 2
     replicas: 1
     schedulerName: tidb-scheduler
     resources:
       limits:
         cpu: 250m
         memory: 150Mi
       requests:
         cpu: 80m
         memory: 50Mi
     kubeSchedulerImageName: k8s.gcr.io/kube-scheduler
     kubeSchedulerImageTag: v1.16.9
   ...
   ```

4. 创建`namespace`

   ```sh
   kubectl create namespace tidb-admin
   ```

   

5. 安装 TiDB Operator

   ```shell
   helm install tidb-operator ./tidb-operator --namespace=tidb-admin
   ```

6. 验证是否安装成功

   ```sh
   kubectl -n tidb-admin get all # 应该包含两个组件tidb-controller-manager和tidb-scheduler
   ```

   ![tidb-admin](https://raw.githubusercontent.com/huang3eng/mdpic/master/uPic/image2023-1-18_11-35-1.png)

## 部署TiDB集群

1. 下载配置文件

   ```sh
   wget https://raw.githubusercontent.com/huang3eng/mdpic/master/file/tidb-cluster.yaml
   ```

2. 下载相关镜像（参考上面下载镜像的方式）

   ```
   pingcap/pd:v6.5.0
   pingcap/tikv:v6.5.0
   pingcap/tidb:v6.5.0
   pingcap/tidb-binlog:v6.5.0
   pingcap/ticdc:v6.5.0
   pingcap/tiflash:v6.5.0
   pingcap/tidb-monitor-reloader:v1.0.1
   pingcap/tidb-monitor-initializer:v6.5.0
   grafana/grafana:6.0.1
   prom/prometheus:v2.18.1
   busybox:1.26.2
   ```

3. 创建 `namespace`

   ```sh
   kubectl create namespace tidb-cluster
   ```

4. 部署 TiDB 集群

   ```sh
   kubectl -n tidb-cluster apply  -f ./tidb-cluster.yaml
   ```

5. 验证是否安装成功

   ```sh
   kubectl -n tidb-cluster get all
   ```

   ![image-20230117182622931](https://raw.githubusercontent.com/huang3eng/mdpic/master/uPic/image-20230117182622931.png)

## References

- [TiDB问答社区](https://tidb.net/?_gl=1*dorv8n*_ga*MTg2MzM4MjI2Ny4xNjczMjQ2NjM3*_ga_D02DELFW09*MTY3NDAwOTUxNi4xNS4xLjE2NzQwMDk2NzEuMC4wLjA.)
- [k8s中RBAC机制](https://blog.yingchi.io/posts/2020/7/k8s-rbac.html)
- [k8s中Pod Topology Spread Constraints介绍](https://cloud.tencent.com/developer/article/1639217)