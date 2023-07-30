# 软路由上的DNS使用记录


## 简述

- 软路由系统： `pfSense`
- 使用的 DNS 服务： `DNS Resolver`
- 可以考虑的 DNS 服务： `BIND DNS`

笔者曾经成功的设置好了 `BIND DNS` ,但是最近这次的安装，怎么都无法成功设置， 所以就尝试了下自带的`DNS Resolver` ，非常简单，一次搞定。 

## 详细内容

### DNS Resolver

`pfSense` 自带这个包，而且这个是默认启用的服务，不需要安装新的 package。

点击导航栏的`Services / DNS Resolver / General Settings` 即可打开常规设置的面板。

向下拉取， 找到  `Host Overrides` 的字样， 点击下面的 `+ Add` 按钮。 

比如我现在想添加 `owncloud.home.link		192.168.200.33` 这样的一条记录。


- Host: 		owncloud
- Domain: 	home.link
- IP Address:   192.168.200.33
- Description:  owncloud 
- 如果你有另外一个域名也指向 192.168.200.33  比如 blog.home.link
- 那么可以在 Additional Names for this Host 里面直接追加
- Host name: 	blog
- Domain: 	home.link
- Description: blog


然后点击 **Save** 按钮即可保存。 

返回到 General Settings 窗口之后可能会提示你需要 应用变化， 此时点击 Apply 应用下即可。

----

现在在客户机里面尝试命令 `ping owncloud.home.link` 可以看到有没有成功解析。

使用`nslookup owncloud.home.link` 也是可以的。

:white_check_mark: 如果成功了， 那么你把想要解析的记录都添加进去即可。

:x: 如果没有成功。。  则需要调试下。

使用命令 `nslookup owncloud.home.link 192.168.200.1`  查看是否能成功解析。 

此处的 `192.168.200.1` 是 pfSense 的LAN 地址。也是 DNS 服务所在的地址。

:x: 不能成功解析的话， 就查看上一步的操作，看看有没有问题，有没有写错。

:white_check_mark: 能成功解析的话， 说明客户机的 DNS 服务器没有使用 `192.168.200.1` 。

在客户机里面使用下面的命令查看当前使用的 DNS 服务器。

- Windows :  `ipconfig /all`   
  - 在结果里面查看 DNS 服务器对应的 IP 地址即可。
- Mac/Linux:  `cat /etc/resolv.conf`
  - **nameserver** 字样后面跟着的是当前使用的 DNS 服务器

此时，结果值应该没有`192.168.200.1` 或者除了 `192.168.200.1` 之外还存在其他的 DNS 服务器。

*注： 如果存在多个 DNS 服务器，则是随机使用其中一个，而不是按序使用*

客户机如果是手动指定的 DNS 服务器， 则设定为 `192.168.200.1`即可。 

如果是依赖软路由的 **DHCP服务**，则设置下即可。

打开菜单栏 `Services / DHCP Server / LAN`

下拉找到 **Servers** 标题， 里面有部分是 DNS Servers。 留空或者设置第一条为`192.168.200.1`，其他都留空 应该可以解决问题了。

注意： 客户机需要重新请求 DHCP 才能生效， 简单来说就是把网线拔掉5秒再插上去应该就可以了。



### BIND DNS

这个工具，我使用过好几次， 因为我重装了 pfSense 好几次。。 前几次折腾了挺久之后可以用了，本次无论如何折腾都不行， 所以就放弃了。 （本次的包是通过配置还原安装的， 不知道有没有关联。。 ）  

所以本部分内容可能是无价值的， 你可以选择跳过不读。 

我个人对这个工具的感觉是：  极其难用

好了， 吐槽结束， 让我们开始设置把。 

简单来说， 大概需要做下面的步骤。

1. 添加一个 `View`
2. 添加一个或者几个 `Zone`
3. 关闭其他的 DNS 服务（DNS Resolver/DNS Forwarder 等)
4. 启用 BIND DNS

点击 `Services / BIND DNS Server` 可以打开 BIND 的设置面板。

#### 添加 View

点击 Views Tab, 点击  Add 按钮

- View Name: LanView
- Description: Default View Of LAN
- Recursion: Yes
- match-clients: any
- allow-recursion: any 或者 localnets
- Custom Options: 


点击 Save 保存即可。

#### 添加 Zone

一个 Zone 差不多对应一个 Domain ,就是一个域名。 

比如我的域名是 `owncloud.home.link && blog.home.link` ，我建立的 ZONE 就是`home.link` `owncloud && blog` 则是在 zone 里面填写的。

- Domain Zone Configuration
  - Disable This Zone:	unchecked ( 不勾选 )
  - Zone Name:	home.link
  - Description: zone of homelink
  - Zone Type: Master
  - View: LanView

- Master Zone Configuration: 
  - TTL: 128
  - Name Server: 192.168.200.1     (也许不对，但是我感觉是这样配置的。。)
  - Base Domain IP: 192.168.200.1
  - allow-query: any
- Zone Domain records:
  - Record: owncloud
  - Type: A
  - Priority: 
  - Alias or IP address: 192.168.200.33


点击 Save 保存即可。

#### 关闭其他 DNS 服务

在 `Services` 菜单栏里面找找， 带 **DNS** 字样的可能都是， 点击进去之后， 第一个选项 `Enable` 确认是未勾选状态即可。

#### 启动 BIND DNS

- Daemon Settings
  - Enable BIND: checked  (勾选)
  - IP Version: IPv4
  - Listen on: LAN && loopback  ( 按住 CTRL 可以选俩）


点击 Save 即可。

过一会 服务应该就启动了，使用 `nslookup www.baidu.com 192.168.200.1` 可以先试试 BIND DNS 有没有成功启动， 然后使用`nslookup` 命令测试自己填写的域名即可。  



## 扩展阅读

nslookup 命令 OK， 但是 ping 不通是怎么回事？[https://plantegg.github.io/2019/01/09/%E5%B0%B1%E6%98%AF%E8%A6%81%E4%BD%A0%E6%87%82ping--nslookup-OK-but-ping-fail/](https://plantegg.github.io/2019/01/09/就是要你懂ping--nslookup-OK-but-ping-fail/)


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E8%BD%AF%E8%B7%AF%E7%94%B1%E4%B8%8A%E7%9A%84dns%E4%BD%BF%E7%94%A8%E8%AE%B0%E5%BD%95/  

