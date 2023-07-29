# 家里的网络图示


## 图示

![Home Network](/img/periphery/home_network.png)



## 其他说明

**我家里就一台二手服务器,所有的虚拟机都在这台服务器里。**

### pfSense 的作用

1. pppoe 拨号上网
2. DHCP 服务
   1. 推送的网关为 `192.168.200.2` 即 `openwrt`的地址， 而非`pfSense`自身的地址。
   2. 推送的 dns 服务器为`192.168.200.33` 即`debian-work`的地址，而非`pfSense`的地址。
3. squid 透明缓存静态资源 |  估计图片，视频比较多把。。  但是我目前只能缓存 http 的。httpds 的则会被跳过。
4. DNS 服务器
   1. 主要是自己家里用的域名解析
   2. 然后再请求外部的 DNS 服务器



### openwrt 的作用

1. 旁路由
2. 科学上网



### debian-work 的作用

1. 主要使用 `docker` 放一些服务 (`mysql/redis/xxx...`)
2. `nextcloud` 类似`owncloud` 。 用于外网可访问的私人存储服务，我主要存储些流程图，思维导图，代码 之类的。
3. `pi-hole` 是一个按照域名阻止广告的服务，  但是好像没啥用。 这个还能做域名黑名单，防止其他人浏览，  但是我一个人住， 这个对我来说也没啥用。
4. `git 服务`， `gogs`是我在外网使用的服务， 里面主要存储一些小的代码库。 `gitlab`则是我在家里的使用的服务， 一般用于存储`unity`的项目， 里面的资源文件比较多。



### openmediavalut 的作用

1. 充当 NAS 。  家里的主要存储服务
2. 主要存储游戏，电影，系统镜像，软件镜像等的内容
3. 开启 苹果的文件协议，支持 TimeMachine 进行备份。
4. 开启 SMB 协议， 支持这个协议的电脑，手机 都可以访问里面的文件。 （小米投影仪也支持）
5. PLEX ，安装了之后可以在外网看家里的电影， 但是清晰度似乎不是很好。



### win10下载机 && win7 游戏机

1. 没啥好说的， 下载就是 百度云，迅雷等。
2. 显卡直通之后就可以使用 虚拟机玩游戏了， 我目前用模拟器玩玩手游稍多一点。我使用 `anydesk` 远程， 但是感觉效果不是很理想。



### DNS 的链路

1. 使用`DHCP` 获取地址的设备，默认 DNS 服务器都是 `debian-work`
2. `debian-work`里的服务是`pi-hole` 在提供， `pi-hole`的上游 DNS 服务器为`openwrt`
3. `openwrt`则是有两个 dns 服务，一个是`dnsmasq`,另外一个是`clash` 。 `dnsmasq` 的请求被设置成转发给`clash`  。
4. `clash`的上游服务器有 N 个， 其中有一个是`pfSense` ，这样的话， 就可以提供本地域名的解析了。



## 参考阅读

[软路由上的DNS使用记录]({{< ref "软路由上的DNS使用记录.md" >}})

[esxi显卡直通的操作]({{< ref "esxi显卡直通的操作" >}})

[软路由的使用]({{< ref "软路由的使用" >}})





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%AE%B6%E9%87%8C%E7%9A%84%E7%BD%91%E7%BB%9C%E5%9B%BE%E7%A4%BA/  

