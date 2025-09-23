# 5. Config Map 的相关信息



## 主要内容

### 基本使用

**创建 config map**
- 方法1：从文件夹中创建一个 config map:  `kubectl create configmap game-config --from-file=./config`
- 方法2： 从文件中创建一个 config map, 和上面一样，使用 `--from-file` 参数
- 方法3：使用  yaml 配置文件创建一个 config map 


**创建一个config-map,随后在pod中引用该config-map的某个键**

```yaml
# 用 llama2 生成， 需要自行测试
# ConfigMap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  foo.bar: "value"
  age: "15"
---
# Pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "echo \$FOO; echo $(cat /config/data/age)"]
    environment:
      - name: FOO
        valueFrom:
          configMapKeyRef:
            name: my-configmap
            key: foo.bar
  volumes:
  - emptyDir
  - config
```

`valueFrom.configMapKeyRef`  就是从configmap中读取配置项


**查看config map里面的内容**  
使用命令 `kubectl describe configmaps game-config`  即可。 


### 命令执行记录 

`deploy_nginx.yaml` 的文件内容

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: config-map-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: config-map-nginx
  template:
    metadata:
      labels:
        app: config-map-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          # 提供包含要添加到容器中的文件的 ConfigMap 的名称
          name: game-config
```


```bash
$ # 这些是笔者自己尝试的真实记录。
$ ls config/
item.yaml
$ kubectl create configmap game-config --from-file=./config
configmap/game-config created
$ kubectl describe configmaps game-config
Name:         game-config
Namespace:    nginx-test
Labels:       <none>
Annotations:  <none>

Data
====
item.yaml:
----
# item.yaml

items:
  - id: 1
    name: 'gold'
    maxCount: 10000000
    initCount: 0
  - id: 2
    name: 'diamond'
    maxCount: 10000
    initCount: 0




BinaryData
====

Events:  <none>
$ kubectl apply -f deploy_nginx.yaml
deployment.apps/nginx-deployment created
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS        AGE
nginx-deployment-687bc4475-pf2dj   1/1     Running   0               4s
$ kubectl exec nginx-deployment-687bc4475-pf2dj -- cat /etc/config/item.yaml
# item.yaml

items:
  - id: 1
    name: 'gold'
    maxCount: 10000000
    initCount: 0
  - id: 2
    name: 'diamond'
    maxCount: 10000
    initCount: 0

$ # 可以看到 文件内容已经被映射到了 `/etc/config/item.yaml` 路径下 
$ cp -f config_version/item-v1.yaml config/item.yaml
$ kubectl create configmap game-config --from-file=./config -o yaml --dry-run=client |  kubectl apply -f -
Warning: resource configmaps/game-config is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/game-config configured
$ kubectl describe configmaps game-config
Name:         game-config
Namespace:    nginx-test
Labels:       <none>
Annotations:  <none>

Data
====
item.yaml:
----
# item.yaml

items:
  - id: 1
    name: 'gold'
    maxCount: 10000000
    initCount: 0
  - id: 2
    name: 'diamond'
    maxCount: 10000
    initCount: 0
  - id: 3
    name: 'apple'
    maxCount: 500000
    initCount: 10



BinaryData
====

Events:  <none>
$ kubectl exec nginx-deployment-687bc4475-pf2dj -- cat /etc/config/item.yaml
# item.yaml

items:
  - id: 1
    name: 'gold'
    maxCount: 10000000
    initCount: 0
  - id: 2
    name: 'diamond'
    maxCount: 10000
    initCount: 0
  - id: 3
    name: 'apple'
    maxCount: 500000
    initCount: 10
$ # 可以看到 id=3 已经在文件中变化了， 剩下的就是看应用程序怎么读取文件了 
$ # 此外就是 文件可能有1分钟的 延迟变化。 
$ # 扩增一下 pod， 看看效果
$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           12m
$ kubectl scale deployment/nginx-deployment --replicas=2
deployment.apps/nginx-deployment scaled
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS        AGE
nginx-deployment-687bc4475-lc9pm   1/1     Running   0               6s
nginx-deployment-687bc4475-pf2dj   1/1     Running   0               13m
$ kubectl exec nginx-deployment-687bc4475-lc9pm -- cat /etc/config/item.yaml
# item.yaml

items:
  - id: 1
    name: 'gold'
    maxCount: 10000000
    initCount: 0
  - id: 2
    name: 'diamond'
    maxCount: 10000
    initCount: 0
  - id: 3
    name: 'apple'
    maxCount: 500000
    initCount: 10
$ # 可以看到， 新建立的pod 也是应用了最新的 config map
```



## 其他

*更新 config map， 但是不删除原来的那个configMap*

```shell
kubectl create configmap foo --from-file foo.properties -o yaml --dry-run=client | kubectl apply -f -
```
*来源： https://stackoverflow.com/a/38216458/11226492*


感觉这个的应用场景不是特别多。  
- 数据库的配置文件，  比如 "my.ini" ， "my.cnf" 或者 "redis.conf" 
- 数据库的连接信息， 地址， 用户名密码什么的

简单来说， 就是第三方工具的配置文件， 和一些基本不会更新的，但是需要在pod里面读取的文件。


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/k8s/5.-config-map-%E7%9A%84%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF/  

