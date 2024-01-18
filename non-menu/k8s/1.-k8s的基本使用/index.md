# 1. K8s的基本使用


*建议读者先随意浏览一下本文， 等待安装完k8s之后，再仔细阅读本文。或者使用在线的k8s playground*   [查看k8s的安装教程](./2.%20k8s的安装)


k8s的工作流程大致如下
- 将应用打包成镜像
- 部署应用
  - 创建一个pod/deployment
  - 或者对已经存在的 deployment 执行镜像版本更新的命令
- 使用 service 暴露应用

## 主要内容

### 镜像打包

使用docker 的方式进行镜像打包即可。 

- 方案一：写 Dockerfile， 然后 build
- 方案二：使用 docker compose 进行 build 

如果是一个java项目，并且使用了spring-boot框架的话，则可以考虑使用命令 `./mvnw spring-boot:build-image` 进行打包镜像.

打包好的项目需要推送到一个 docker Registry中， 以供k8s拉取。 

如果想要发布到 docker hub 中的话， 可以参考 [发布镜像到-docker-hub](../../../periphery/-code-server-&#43;-jdk17-的docker-镜像/#发布镜像到-docker-hub)

不过公司项目的话， 应该还是要搭建一个私有的 Registry。

**关于私有的 Registry**  
*待补充*

&lt;!-- https://tonybai.com/2016/11/16/how-to-pull-images-from-private-registry-on-kubernetes-cluster/  --&gt;

### 部署应用

先简单的介绍一下 k8s中的几个概念。 
- Pod,  可以理解为运行中的容器。
- Deployment   可以理解为管理Pod的一个方式。

笔者将直接跳过 Pod,从 Deployment 开始说明 

*推荐使用 yaml配置文件的方式进行配置。*

下面来看一个 nginx 的 Deployment 的配置文件。 

```yaml
# file: deploy-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
*如果能看得懂简单的英语的话， 很多字段的意义就一目了然了。*

笔者来说一下几个重点。
- `kind: Deployment`   指定类型为 Deployment
- `metadata`   当前 Deployment 的 元数据， 可以理解为一些Deployment的基本信息。
  - `labels` 是标签部分， 这个比较重要
    - 内容是 K:V
- `spec`
  - `replicas`    **部署几个pod ?**
  - `selector.matchLabels`   要匹配的pod的label， 应该与 `template.metadata.labels`中的值一致
  - `template`  pod 的模板
    - `metadata`   部署之后的pod的 元数据， 要指定至少1个用于查询的label
    - `spec.containers`    容器相关的信息
      - `image`    **要发布的pod(容器)采用的镜像是哪个？**
      - `ports`    暴露出去的端口


使用命令 `kubectl apply -f ./deploy-nginx.yaml`   即可发布一个 Deployment 。

发布成功之后， 使用命令 `kubectl get pods` 可以查看已经存在的pod的信息。   
如果pod的STATUS是Running的话， 表示已经成功运行了。 如果是其他情况， 可以使用命令 `kubectl describe pods $POD_NAME` 查看pod的状态， 注意 $POD_NAME 要替换成
你具体的pod name 。

```shell
$ kubectl get deploy              # 查看 deployment 的信息
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
netchecker-server   1/1     1            1           27d
nginx-deployment    2/2     2            2           27d
```


### 暴露应用以供外部访问

先来介绍一下下面的K8S得概念
- Service,  可以简单的理解为一个反向代理把， 把内部的服务暴露给外部。 
- Node      工作节点，可以理解为容器的宿主机

创建一个Service 可以使用 `expose` 命令， 也可以写一个yaml 配置文件。

下面来看一个Service的配置文件。 
```yaml
# file: service-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  # type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    nodePort: 30018
```

笔者来说一下几个重点。
- `kind: Service`   指示这是一个 Service类型的资源
- `spec`
  - `type`      类型  ClusterIP/NodePort/LoadBalancer/ExternalName  [更多解释](https://kubernetes.io/zh-cn/docs/tutorials/kubernetes-basics/expose/expose-intro/)
  - `selector`      填写 pod的 label 或者 Deployment中 `spec.template.metadata.labels` 里面的信息。
  - `ports`    端口
    - `port`     监听的端口
    - `targetPort`   目标端口， 容器的端口， 如果没写则与 port一致。 
    - `nodePort`     type: NodePort 时有效， 内容为Node的端口。 

使用命令 `kubectl apply -f ./service-nginx.yaml`   即可发布一个 service。 

成功发布之后， 访问NodeIp:30018  即可访问服务。    

这里还有几个要点
- 如果安装的是minikube的话,则需要访问 `$(minikube ip):30018`  
- 创建了Service之后， 即可在其他Pod 内部使用名称来访问service
  - service-name
  - service-name.namespace
  - service-name.namespace.svc.cluster.local
  - 上面的内容都可以访问到Service，但是端口号还是要附加在名字后面的。

```shell
$ kubectl get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes           ClusterIP   10.233.0.1       &lt;none&gt;        443/TCP          27d
netchecker-service   NodePort    10.233.239.105   &lt;none&gt;        8081:31081/TCP   27d
nginx-service        NodePort    10.233.168.254   &lt;none&gt;        80:30018/TCP     27d
```

EXTERNAL-IP 只有 LoadBalancer 类型才会有， 这个一般是由云服务商提供。 

expose 在有些时候更快捷一些， 内容可以参见：  http://docs.kubernetes.org.cn/475.html

### 删除资源（Deployment, Service）

- kubectl delete -f ./service-nginx.yaml
- kubectl delete -f ./deploy-nginx.yaml


## 其他

### 缩写

pod（po），service（svc），replication controller（rc），deployment（deploy），replica set（rs）

比如：  
- `kubectl get services`   =&gt;  `kubectl get svc`


### 其他

如有错误之处， 欢迎指出。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/k8s/1.-k8s%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/  

