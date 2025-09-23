# 4. 持久卷的相关信息

持久卷允许多个pod 读取和写入同一个文件夹。

## 基本介绍

### local 本地路径

local 持久卷， 官方文档： https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#local     
使用本地的一个目录来做存储， 允许挂载到不同的pod

下面看一个示例的配置文件
```yaml
# pv-example.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
  labels:
    release: "test"
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /home/vagrant/storage1/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - minikube
```

- `kubectl apply -f pv-example.yaml` 即可创建一个 local 存储。 
- `kubectl get pv`   可以查看
- `spec.local.path`  指示了一个本地文件夹， 需要提前存在。 如果使用的是 minikube的话， 这里指示的是**一个docker 容器里面的路径**

访问模式有：
- ReadWriteOnce
  - 卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。
- ReadOnlyMany
  - 卷可以被多个节点以只读方式挂载。
- ReadWriteMany
  - 卷可以被多个节点以读写方式挂载。
- ReadWriteOncePod
  - 特性状态： Kubernetes v1.27 [beta]
  - 卷可以被单个 Pod 以读写方式挂载。 如果你想确保整个集群中只有一个 Pod 可以读取或写入该 PVC， 请使用 ReadWriteOncePod 访问模式。这只支持 CSI 卷以及需要 Kubernetes 1.22 以上版本。

nodeAffinity
- `kubernetes.io/hostname` 的值需要填写一个节点的 hostname， 
- 使用 `kubectl get node`  应该是可以获取到全部的节点列表。 

*使用 `kubectl describe pods $POD_NAME`   可以查看一个POD 卡在了什么地方。*

### 使用卷/申领卷

下面看一个示例的配置文件
```yaml
# pvc-example.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Mi
  storageClassName: local-storage
  selector:
    matchLabels:
      release: "test"
```
- 如果 填写了`matchExpressions` 那么里面的条件也必须被满足， 才会绑定到对应的卷。 
- `matchLabels` 则绑定的 `metadata.labels`  属性。 
- `requests.storage`  表示要申领的卷的大小

**1个PV 只能绑定1个PVC**
```shell
$  kubectl get pvc -o=wide
NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS    AGE     VOLUMEMODE
example-pvc    Bound     example-pv   2Gi        RWO            local-storage   3m53s   Filesystem
example-pvc2   Pending                                          local-storage   23s     Filesystem
```


- example-pvc2 和 example-pvc 的内容一样， 只是换了一个名字。 
- example-pvc 的 CAPACITY 自动变成了 2Gi, 而不是申请的 500Mi, 看起来会寻找一个比申请大的持久卷。

**应用PVC 到 deployment 上面去。**

```yaml
# deploy_nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: pv-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pv-nginx
  template:
    metadata:
      labels:
        app: pv-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data1
          mountPath: /data
      volumes:
      - name: data1
        persistentVolumeClaim:
          claimName: example-pvc
```

关键点在 `volumes[].persistentVolumeClaim` 部分，指定了一个要绑定的名称。

**执行命令的记录**
```shell
$ kubectl apply -f pv-example.yaml
persistentvolume/example-pv created
$ kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS    REASON   AGE
example-pv   2Gi        RWX            Delete           Bound    nginx-test/example-pvc   local-storage            7m7s

$ kubectl apply -f pvc-example.yaml
persistentvolumeclaim/example-pvc created
$ kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS    AGE
example-pvc   Bound    example-pv   2Gi        RWX            local-storage   7m11s

$ kubectl apply -f deploy_nginx.yaml
deployment.apps/nginx-deployment created

$ kubectl get pods
NAME                              READY   STATUS    RESTARTS      AGE
nginx-deployment-7dd456d7-52zt6   1/1     Running   0             3s
nginx-deployment-7dd456d7-czbvb   1/1     Running   0             3s
nginx-deployment-7dd456d7-pmvdf   1/1     Running   0             3s

$ # 修改 /home/vagrant/storage1/data 目录内的文件。
$ kubectl exec --stdin --tty nginx-deployment-7dd456d7-czbvb -- ls /data
1.txt  2.txt
$ kubectl exec --stdin --tty nginx-deployment-7dd456d7-czbvb -- ls /data
1.txt  2.txt
$ kubectl exec --stdin --tty nginx-deployment-7dd456d7-pmvdf -- ls /data
1.txt  2.txt
```

### nfs 网络位置

参考 镜像： https://hub.docker.com/r/erichough/nfs-server/

*TODO 待补充*

### 了解一下 S3 是否可以挂载？ 

*TODO 待补充*

## 其他

- 删除资源的时候， 应该先删除 PVC， 再删除PV。 
- 如果先尝试删除了PV， 则会提示 PV的资源已经删除， 但是命令会卡主。 当你 ctrl+c 强制结束的时候， 再次查看， PV和PVC 都是还在的。
- 当你手动删除PVC的时候， PV 就会自动被删除。 

如果PVC 被挂载到了一个 POD， 在删除的时候就会卡主。
所以删除顺序是
- Deploymenet/Pod
- PVC
- PV


```shell
vagrant@bullseye:~/storage1$ kubectl delete -f pv-example.yaml
persistentvolume "example-pv" deleted

\

^Cvagrant@bullseye:~/storage1$ kubectl delete -f pvc-example2.yaml
persistentvolumeclaim "example-pvc2" deleted
vagrant@bullseye:~/storage1$ kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                    STORAGECLASS    REASON   AGE
example-pv   2Gi        RWO            Delete           Terminating   nginx-test/example-pvc   local-storage            18m
vagrant@bullseye:~/storage1$ kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS    AGE
example-pvc   Bound    example-pv   2Gi        RWO            local-storage   15m
vagrant@bullseye:~/storage1$ kubectl delete -f pvc-example.yaml
persistentvolumeclaim "example-pvc" deleted
vagrant@bullseye:~/storage1$ kubectl delete -f pv-example.yaml
Error from server (NotFound): error when deleting "pv-example.yaml": persistentvolumes "example-pv" not found
vagrant@bullseye:~/storage1$
vagrant@bullseye:~/storage1$ kubectl get pvc
No resources found in nginx-test namespace.
vagrant@bullseye:~/storage1$ kubectl get pv
No resources found
vagrant@bullseye:~/storage1$
```





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/k8s/4.-%E6%8C%81%E4%B9%85%E5%8D%B7%E7%9A%84%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF/  

