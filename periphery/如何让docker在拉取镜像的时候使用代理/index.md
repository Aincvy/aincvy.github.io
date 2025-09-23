# 如何让docker在拉取镜像的时候使用代理


## 主要内容

在某些情况下， 我们需要设置代理才能从 docker hub上拉取镜像， 本文描述了如何去做。  

本文假设读者的docker 是使用 systemctl 进行控制的， 在这种情况下， 设置一下 systemctl 的配置文件就可以实现代理的目的了。 

在目录 `/usr/lib/systemd/system/docker.service.d` 添加一个 `任意名称.conf` 文件， 比如`http.conf`， 并添加如下内容： 
```conf
[Service]
Environment="HTTP_PROXY=http://hostname_or_ip:port"
Environment="HTTPS_PROXY=http://hostname_or_ip:port"
```
注：
- 将 `hostname_or_ip` 替换成读者的代理服务器，  将 `port` 替换成读者的代理服务器端口。
- 这个文件可能需要root权限， 所以需要使用 `sudo` 来添加。 

后续步骤：
```bash
sudo systemctl daemon-reload               # 来使配置文件生效。 
sudo systemctl show --property Environment docker   # 查看配置文件是否生效了
sudo systemctl restart docker    # 重启docker 服务
```

在笔者的版本中， docker 是拆分成了 docker 和 containerd,  直接重启 `docker` 的时候， 不会重启容器， 也许读者的不一样， 需要自己注意一下。 

这样配置之后， docker pull 的时候就会走代理去拉取镜像了。 

## 其他

### 仍然无法拉取镜像的问题

笔者配置了代理之后， 碰到了仍然无法拉取镜像的问题， 一番排查之后， 笔者发现是因为代理工具自身的代理规则没有使docker 相关的域名走代理， 还是直连。  之后笔者将代理工具的模式从规则修改成全局之后 就可以了。 

### 让容器中的程序使用 HTTP 代理

基本上来说，这个没有什么万能的办法。  如果容器的程序会读取环境变量， 那么就设置一下 `HTTP_PROXY` 和 `HTTPS_PROXY` 或者 设置一下`ALL_PROXY` 的环境变量即可。  可以设置在 `docker-compose.yaml` 文件里面。 

如果容器的程序不读这个环境变量， 就需要去查看容器的说明， 去寻找设置 proxy 的方法。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%A6%82%E4%BD%95%E8%AE%A9docker%E5%9C%A8%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F%E7%9A%84%E6%97%B6%E5%80%99%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%90%86/  

