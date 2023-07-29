# 3. 应用的更新以及容器扩缩以及命名空间


现在，让我们回到k8s的使用上来。

## 主要内容

### 命名空间

```shell

$ kubectl create namespace nginx-test       # 创建 命名空间
$ kubectl get ns                            # 获取所有命名空间
$ kubectl config set-context --current --namespace=nginx-test   # 切换命名空间
$ kubectl delete ns nginx-test              # 删除

```


### 应用的更新

```shell
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

- deployment/nginx-deployment    
  - `deployment/`  表示是一个Deployment 资源
  - `nginx-deployment`     是 Deployment资源的名字
- nginx=nginx:1.16.1
  - 格式为：  `容器名称=新的镜像版本`


执行成功之后， k8s会自动滚动更新pod。 
滚动更新的大致流程如下：
1. 先终止N个旧版本的pod
2. 然后启动N个新版本的pod
3. 等N个新版本的Pod启动完毕之后，继续第一步，直到旧的pod全部被替换完毕。 


### 应用的版本回滚 

```shell
$ # 查看可以回退的历史版本， 以及标注更新信息
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1
$ kubectl rollout history deploy/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
4         <none>
5         <none>
6         <none>
$ kubectl annotate deployments.apps/nginx-deployment kubernetes.io/change-cause="version to 1.17.1"  --overwrite=true
$ kubectl rollout history deploy/nginx-deployment 
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
4         <none>
5         <none>
6         version to 1.17.1
```

使用 `--record=true`  可以自动附加使用的命令为 CHANGE-CAUSE， 只是目前这个参数被标记为废弃了。 

```shell
$ kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
$ kubectl annotate deployments.apps/nginx-deployment kubernetes.io/change-cause="rollback to 1.16.1"  --overwrite=true
$ kubectl rollout history deploy/nginx-deployment 
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
4         <none>
6         version to 1.17.1
7         rollback to 1.16.1
$ kubectl rollout undo deployment/nginx-deployment --to-revision=4
deployment.apps/nginx-deployment rolled back
```

简单来说 
- rollout history  查看历史
- rollout undo    回退到上一个版本
- rollout undo --to-revision=4  回退到指定的 REVISION

下面几个应该都是同义词， 喜欢哪个用哪个即可。
- deploy
- deployment
- deployments.apps

### 应用的扩缩

```shell
$ kubectl scale deployment/nginx-deployment --replicas=10
```

- `--replicas=目标pod个数`



### GUI 程序

笔者试用了一下 Kuboard， 发现这个确实很方便， 不需要记各种命令，鼠标点点就可以回滚，更新镜像，修改pod数量。 还可以直接看到变更历史等。


![kuboard1](/img/non-menu/k8s/kuboard1.png)
![kuboard2](/img/non-menu/k8s/kuboard2.png)



官方的web-ui方案：  https://github.com/kubernetes/dashboard



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/k8s/3.-%E5%BA%94%E7%94%A8%E7%9A%84%E6%9B%B4%E6%96%B0%E4%BB%A5%E5%8F%8A%E5%AE%B9%E5%99%A8%E6%89%A9%E7%BC%A9%E4%BB%A5%E5%8F%8A%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/  

