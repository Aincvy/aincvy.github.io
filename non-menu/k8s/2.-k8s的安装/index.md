# 2. K8s的安装


可以考虑下面几种方式进行安装
- minikube
  - 主要目的应该是学习或者开发测试。
  - 不应该应用于生产环境    
  - 安装简单
- kubeadm
  - 可用于生产环境
  - 安装配置比较麻烦。
- kuboard-spray 
  - 开源项目： https://github.com/eip-work/kuboard-spray
  - 大部分安装操作都是自动化的， 不过支持的操作系统列表有限。
  - 底层是基于kubeadm，  允许部署到多节点中

## 主要内容

### minikube 

官方文档参考：  https://minikube.sigs.k8s.io/docs/start/

#### 安装
下面是简要内容： 
- 依赖
  - Docker, 或者虚拟机环境
- 安装，  采用下载二进制的方式
  - `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`
  - `sudo install minikube-linux-amd64 /usr/local/bin/minikube`
- 启动
  - `minikube start`
- 手动下载 kubectl  或者使用 `minikube kubectl -- `  来让minikube进行下载。 
- 此时集群应该已经可用， 正常发布应用即可。
- 使用结束可以使用 `minikube stop` 关闭集群。 

#### 其他
minikube 推送镜像的时候好像会简单一些，详情可见： https://minikube.sigs.k8s.io/docs/handbook/pushing/
- 进入docker-env之后， 使用docker build 命令可以直接把镜像推送到集群里面。
- 使用`minikube addons enable registry` 可以开启一个 registry， 然后推送。
  - 使用 `$(minikube ip):5000` 作为 registry 的url 前置部分。 
- 以及其他等等

{{< admonition "info" >}}  
有关 Service 的 Type 部分。   
~~Minikube 不支持 LoadBalancer 的Service 类型，如果指定了此类型， 会自动转成 NodePort~~  
现在好像支持了， 官方文档原话： `To access a LoadBalancer deployment, use the “minikube tunnel” command` 

当使用 NodePort类型的时候， 使用命令 `minikube ip` 可以获取到 minikube的 IP地址。
{{< /admonition >}}

Driver
> minikube relies on docker-machine drivers to manage machines

简单来说， minikube 依赖于一个 docker-machine driver 去管理机器。  拓展阅读： https://minikube.sigs.k8s.io/docs/contrib/drivers/

在使用 `minikube start` 启动集群的时候， 可以指定一个driver, 例如： `minikube start --driver=docker` 

- driver列表： https://minikube.sigs.k8s.io/docs/drivers/
- docker 不是默认的driver, 默认的driver 应该是 virtualbox
- 使用命令 `minikube config set driver docker` 可以设置docker成默认的driver

在一些需求之下， 需要minikube 和kubectl 两个命令混用。

官方文档有明说，简化k8s的生产环境部署的体验并不是minikube的一个目标， 地址为： https://minikube.sigs.k8s.io/docs/contrib/principles/#non-goals

### kubeadm

这是一个生产环境方案。 

- 笔者在国内机器上尝试手动安装kubeadm的时候，体验很差， 各种小问题，网络问题， 最后还是安装失败了。。。
- 但是使用 国外的虚拟机，就可以成功安装。  体验的地址是： https://labs.play-with-k8s.com/   
  - 国内访问有点卡顿
  - 每个会话有4个小时的使用时间。
  - 免费

这里笔者将会整理一下碰到的小问题和如何解决。   

*吐槽：我应该是要学习如何使用，而不是如何安装k8s才对。。。* 

官方文档： https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/   
操作步骤大致如下： 
- 准备3个虚拟机
  - 1个管理节点
  - 2个工作节点
- 在管理节点上进行一些安装
  - 安装docker 
    - 官方文档：  https://docs.docker.com/engine/install/
    - 清华大学镜像源： https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/
  - ~~修改配置文件： `/etc/containerd/config.toml`~~
    - 去掉`disabled_plugins = ["cri"]` 中的 `"cri"` 
    - 重启containerd   `systemctl restart containerd`  
    - 如果不这么做的话， 在初始化kubeadm的时候， 可能会提示： ` CRI v1 runtime API is not implemented for endpoint \"unix:///var/run/containerd/containerd.sock\"`
  - containerd的配置文件处理
    - 参考：  https://github.com/kubernetes/kubeadm/issues/2851#issuecomment-1535770518
    - 解释： 即使在kubeadm init命令中使用了国内的镜像， 但是这个 sandbox_image 还是需要额外处理。
  - 安装 kubeadm kubectl kubelet 
    - 官方文档： https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
    - 清华大学源：  https://mirrors.tuna.tsinghua.edu.cn/help/kubernetes/
    - 具体安装命令如下，下面的代码已经使用了清华大学的源： 
    ```shell 
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
    # 可以连接的话， 就不需要 下面两行
    export http_proxy=YOUR_PROXY_
    export https_proxy=YOUR_PROXY_
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    apt update
    apt-get install -y kubelet kubeadm kubectl
    apt-mark hold kubelet kubeadm kubectl
    # 不解除代理的话， 容易出各种问题
    unset http_proxy
    unset https_proxy
    ```
  - kubeadm init
    - 这个很容易挂。。 
    - `kubeadm init -v=5 --kubernetes-version=v1.27.3 --apiserver-advertise-address=192.168.56.10 --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16`
    - `--kubernetes-version=v1.27.3`  指定版本的， 可以去掉
    - `192.168.56.10`  这个地址要对work节点可见
    - `--pod-network-cidr=10.244.0.0/16`   指定pod网络的IP地址范围
    - 当安装成功的时候， 会提示一个 kubeadm join 的命令。 如果没有看到这个命令， 基本上是失败了。 
    - 安装成功之后，如果是root用户， 则执行命令 `export KUBECONFIG=/etc/kubernetes/admin.conf`。 否则根据提示复制配置文件
    - 安装失败的话， 使用`journalctl -xeu kubelet` 查看日志，寻找蛛丝马迹。
  - kubeadm init  失败后的重来
    ```bash
    $ systemctl stop kubelet.service
    $ kubeadm reset
    y #手动输入y 然后按下回车
    $ # 重置成功， 可以再次 kubeadm init
    ```
  - 安装成功之后， 需要应用网络插件
    - 这里安装一下 flannel, 命令为： `kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`
    - 项目地址为：  https://github.com/flannel-io/flannel
    - k8s 网络插件列表： https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy
- 工作节点的安装
  - 安装docker
  - containerd的配置文件处理
  - 使用 kubeadm init 提示的 kubeadm join 命令加入集群。 
- 最后确认
  - 在管理节点上执行命令 `kubectl get nodes`  查看是否是3个节点。 


#### 其他问题记录 

使用清华大学镜像提示的命令之后在执行 apt update 之后， 提示：“W: GPG error: https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05 ”   
解决办法：  删除key,  删除list 使用谷歌提示的方式来操作。  
```shell
$ rm -f /usr/share/keyrings/kubernetes-archive-keyring.gpg
$ rm -f /etc/apt/sources.list.d/kubernetes.list
$ apt update
$ curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ apt update
```

注意： 只是使用了谷歌给出的方法， 实际上还是使用的 清华大学的源。 

- 查看 6443 端口有无开启   `ss -nltp | grep 6443`
- 查看k8s 的镜像列表    ` crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock images`
- 手动抓取  registry.k8s.io/pause:3.6    `ctr -n k8s.io images pull registry.k8s.io/pause:3.6`
- `The connection to the server localhost:8080 was refused`      
  - 没有 找到 admin.conf 文件， 如果使用root 的话， 每次ssh 连接之后都需要 执行命令 `export KUBECONFIG=/etc/kubernetes/admin.conf`  
  - 可以考虑 添加到 `.bashrc` 文件， 以自动执行。 

备选的镜像加速地址：  https://github.com/DaoCloud/public-image-mirror  
下面是官方给出的命令
> kubeadm config images pull --image-repository k8s-gcr.m.daocloud.io

想要使用的话， 可以使用 `k8s-gcr.m.daocloud.io` 替换 `kubeadm init ....` 中的 `registry.aliyuncs.com/google_containers` 部分。 

除了k8s, 还能加速docker镜像，详情查看项目的说明文档。   
注： 笔者并没有尝试过这个。 

### kuboard-spray 

这个安装和使用非常简单。 

官方的自我介绍
> 使用图形化的界面离线安装、维护、升级高可用的 K8S 集群  
> https://kuboard-spray.cn


安装方法： 
- 官方文档： https://kuboard-spray.cn/guide/install-k8s.html#%E5%AE%89%E8%A3%85-kuboard-spray
- 请参考官方文档安装即可， 都是一些表单，根据提示填写即可。 
- 其他需要注意的地方
  - 资源包列表： https://kuboard-spray.cn/support/
  - 导入一个离线资源包之后， 在支持的操作系统里面选取一个喜欢的，然后使用那个系统建立N个虚拟机。 
  - 在本地建立虚拟机可以考虑使用 Vagrant, 参见文章： [vagrant-虚拟机模板工具](../../periphery/vagrant-虚拟机模板工具/)
  - 如果是使用了云主机的话， 则创建同一个内网的新机器即可。


优点： 
- 几乎算是全·自动化， 
- 完全解决了国内网络不好的问题
- 在管理面板上还提供了shell 界面。  当然这个可能存在一些安全隐患。
- 支持备份和恢复 etcd 


{{< admonition "warning" >}}  
理论上来说，  kuboard-spray  应该是可以用于生产环境的， 但是笔者并没有进行过测试。 

{{< /admonition >}}


## 其他

按需选择自己合适的安装方式即可。

*如果使用的是阿里云，腾讯云等云服务器， 使用kubeadm安装应该不会遇到网络问题。*



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/k8s/2.-k8s%E7%9A%84%E5%AE%89%E8%A3%85/  

