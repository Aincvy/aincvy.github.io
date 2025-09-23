# PfSense 软路由的使用


### 简述

- 软路由系统: `pfSense`， 基于`FreeBSD`
- 软路由是放在 `esxi`上的一个虚拟机
- 使用电信拨号上网
- LAN 口1 `192.168.200.0/24` 自用
- LAN 口2 `192.168.100.0/24` 因某些原因分享网络给其他一起合租的租客使用。



## 开始

我现在是电信1000M/100M 的宽带， 之前使用的不到300块钱的 TPLink 路由器在 speedtest.cn 上只能跑700M 左右的下行。 更换了之后能跑到1000M 左右， 感觉还是 OK 的。

### ESXI 

*没有使用虚拟机的读者可以跳过这一部分。*

我目前的情况是这样的

- `pfSense` 作为主路由
  - 其他虚拟机
  - 我的无线路由器
    - 我的 PC 台式机
    - 我的无线设备， 笔记本电脑，平板，手机等
  - 千兆交换机1
    - 其他租客1的路由
    - 其他租客2的路由

*因为不知道博客的主题是否支持流程图就先用这种表达方式了。*

#### 新建 VMkernel 网卡

我本来是使用的别的路由， 现在的 esxi 的管理地址是`192.168.0.23` ， 我更换为pfSense 之后，就没这个网段了， 所以需要处理下 esxi 的 web 管理地址的问题。

1. 新建虚拟交换机
   1. 名字起个有含义的就可以， 比如`MyRoute`
   2. 上行链路， 选择一个要连接路由器的网口即可（我的自用服务器上有4个网口）
2. 新建端口组
   1. 名字示例：  `esxiPort`
   2. `VLAN` 填0 即可
   3. 虚拟交换机 选择我们上一步创建好的。
3. 新建 VMkernel 网卡
   1. 端口组选择上一部创建的
   2. IP上选择 `仅 IPv4` 应该就可以了
   3. IPv4设置 选择静态
      1. 我给自用网络设计的网段是 `192.168.200.1-192.168.200.254` 
      2. 所以我给 VMkernel 的地址是`192.168.200.10` 同一个网段应该就 OK 了
      3. 子网掩码填写`255.255.255.0` 即可
      4. 服务选择 `管理`
4. 现在差不多就可以从新地址上访问了。
   1. 不使用其他路由的情况下， 把网线从电脑上直连服务器上选择的那个网口。
   2. 本机 PC 的地址也设置在 `192.168.200.x` 网段里面
   3. 打开浏览器 访问 `192.168.200.10` 应该就可以访问了。

#### 其他口的网络设置

1. WAN  口
   1. 添加一个虚拟交换机
      1. 比如起名叫 `WAN_PPPOE`
      2. 上行链路选择链接电信光猫使用的网口
   2. 新建一个端口组
      1. 比如起名叫`pppoe1`
      2. 选择 `WAN_PPPOE` 虚拟交换机
      3. `VLAN ID` 选择0 
2. LAN 口
   1. 因为要使用我们自己的 LAN 管理 esxi 所以会使用之前建立好的`MyRoute`虚拟交换机
   2. 新建一个端口组
      1. 比如起名叫 `pfSenseMyRouteLan`
      2. 虚拟交换机选择 `MyRoute` 
3. 其他人使用的 网口
   1. 新建一个虚拟交换机 `OTHERS_ROUTE` ，网口选择别的，用于链接千兆交换机1
   2. 新建一个端口组 `OthersRoute` 使用 `OTHERS_ROUTE ` 作为虚拟交换机





### pfSense

从 https://www.pfsense.org/download/ 官网地址上下载最新的镜像即可。

如果需要安装在全新的物理机上， 下载一个镜像写入的工具， 比如 软碟通，啥的把镜像写入 U盘，然后从 U盘中启动。

如果是安装在虚拟机里面， 则新建一个虚拟机， 分配配置，选择镜像文件。 

我使用的虚拟机配置如下

- 2 x 2 = 4核 的 CPU (*E5-2650v2*)
- 4GB 内存
- 600GB 硬盘（本来想用`squid`做透明代理的，所以给的多）
- 3个网络适配器
  - WAN 口1个  *端口组 pppoe1*
  - LAN 口1个   *端口组 pfSenseMyRouteLan*
  - 其他人的网络口 1个  *端口组 OthersRoute*



安装过程一般是下一步下一步就好了， 中间可能需要指定 WAN 口，LAN 口。使用 esxi 的读者查看网络适配器的 mac 地址即可辨认， 物理机的读者 我这里的办法只有尝试了。。

这里需要说明一下， 设置好的之后口 是可以变化的。

LAN 口的地址， 我是设置为的 `192.168.200.1`  ，然后设置 DHCP 服务为启用， 地址池可以填写`192.168.200.100-192.168.200.254`

WAN 口，其实可以先忽略， 或者设置成 从 DHCP 获取地址。 

非常需要安装记录的读者 请百度/谷歌搜索一下，这类资料还是很多的。



#### 链接 web 管理页面

此时，我的电脑使用网线链接 pfSense 上指定的 LAN 口，并且地址是 `192.168.200.x`。

我一般会先 使用命令`ping 192.168.200.1` 看看网络通不通。

可以 ping 的到的话，就说明 Luck， 可以使用 web 界面管理了。  :+1:  :+1: :+1:

不能访问的读者， 很可能是连错 网口了。 

如果你连接了正确的网口， 但是 ip 地址没有给你自动设置好。 

你可能需要手动设置，信息填写如下

- 地址: `192.168.200.100`
- 子网掩码: `255.255.255.0`
- 网关: `192.168.200.1`

如何查看当前的 ip 地址呢？

Windows: `ipconfig /all`

Linux: `ifconfig`

#### web 管理界面

成功访问到`192.168.200.1` 之后， 使用 `admin/pfsense` 进行登录，首次登录可能有点慢。

首次登录之后会要求设置一些事情。 

地址按照上面说的填写即可。

时区选择自己所在的区域， 我一般选择 `Asia/Shanghai`

成功设置之后， 看到的界面是 `Dashboard` ，可以从导航栏中的`Status/Dashboard`按钮打开。

#### 电信拨号

- 使用网线链接电信光猫和 `WAN_PPPOE` 对应的网口。
- 在 web 管理界面选择 `Interfaces/WAN` 进入 WAN接口的管理界面
- `IPv4 Configuration Type`  选择 `PPPoE` ,`IPv6` 的配置可以不用管
- `MTU` 填写`1480`
- 向下滑动， 会看到 `PPPoE Configuration` 填写宽带的用户名密码即可。
  - 注意，电信师傅给的用户名密码， 可能需要添加`ad`开头才能使用
  - 原因我暂时不知道。 
- `Dial on demand` 勾选。 这个选项可以使宽带断线重连。
- **Idle timeout**: `172800`  
- **Reserved Networks**: 两个都勾选
- 填写完成之后点击 **save** 保存， 之后会提示 是否`apply changes`  ，选择`Yes`
- 之后可能会卡一会， 因为会进行宽带拨号。
- 这里需要注意下， 如果你刚从别的路由把网线拔下来， 可能需要换个光猫上的网口。
- 点击导航栏`Status/Interfaces`   可以查看几个网络适配器的状态。 此时的 WAN 口有一个**Status** 

  - 值为 up 表示链接成功， 并且会有 ip 和 dns 服务器的信息
  - 值为 down 表示链接失败
- 大概信息如下，就是说明链接成功了。 (`*.*.*.*` 是隐藏信息)

> Status up  
> PPPoE up   
> Uptime 1d 20:23:57  
> IPv4 Address *.*.*.*  
> Subnet mask IPv4 255.255.255.255  
> Gateway IPv4 *.*..   
> IPv6 Link Local fe80::20c:29ff:fef8:3670%em0  
> Gateway IPv6 fe80::ce1a:faff:feed:920  
> DNS servers  
> 116.228.111.118  
> 180.168.255.18  
> MTU 1480  
> In/out packets  
> 52170750/29521762 (63.12 GiB/3.75 GiB)  
>
> In/out packets (pass)  
> 52170750/29521762 (63.12 GiB/3.75 GiB)  
> In/out packets (block)  
> 23698174/3 (406.33 MiB/189 B)  
> In/out errors 0/0  
> Collisions 0   

#### 上网

能成功拨号的话， 就差不多快能上网了。 

此时你可以尝试一下， 如果你能上网，那么 OK， 就没太多事情了。

如果你不能，就检查下`Firewall/Rules/Lan` 的内容，此时一般都是防火墙规则的问题。

查看你是否有 类似这样的标记

:heavy_check_mark:(绿色的) `IPv4*  LAN net  *  *  *  *  none  deault allow xxx`

如果没有的话，自己新加一个。

- Action: Pass
- Interface: Lan
- Protocol: Any
- Source: any
- Destination: any
- Description: allow lan access internet

保存后应用 应该就可以上网了。 （如果前面没有 block/reject 的规则的话。。）

我设置`OtherRoute`的规则在后面指出。



#### 端口转发

**内网访问映射出去的端口**

先启用内网可以访问映射的端口，不然只有外网能访问的话就很不方便。

点击`System/Advanced/Firewall & NAT`

向下滑动， 找到`Network Address Translation` 标题。

勾选 `Enable NAT Reflection for 1:1 NAT` 和`Enable automatic outbound NAT for Reflection`  然后 保存就可以了。



**添加要映射的端口**

点击`Firewall/NAT/Port Forward`

点击`add` 添加一条新的规则。

- `Destination port range` 填写端口范围， 单端口就写相同值
- `Redirect target IP` 填写内网的 ip,比如 `192.168.200.33`
- `Redirect target port` 填写端口
- `Description` 写一个有意义的描述
- 点击保存， 然后应用。
- 应该就可以访问了。



## 本次操作我碰到的其他问题

### TPLink 路由器怎么做AP 模式

1. 设置路由地址为手动， 然后设置为`192.168.200.99 ` （没被使用的任意地址即可）
2. 把pfSense 的 Lan 口 链接到路由器的随意 Lan 口（*比如 Lan 口1*）
3. 我的电脑链接 路由器的其他 Lan 口
4. 路由器的 `Wan`口空着。
5. 此时电脑和链接的无线设备应该都可以上网了 



### 其他租客的防火墙规则我是怎么设置的？

默认情况下， `192.168.100.x` 是可以访问到`192.168.200.x` 的， 就是说其他租客有可能会访问到我自己的虚拟机，自己的设备， 这是我不希望看到的。 

所以我的防火墙规则是这样的。

- 点击`Firewall/Rules/OthersRoute` 
- 点击 add 添加第一条规则
  - Action: Pass
  - Protocol: Any
  - Source: OthersRoute net
  - Destination: Network, 192.168.100.1 / 24
  - Description: allow others access their other devices
  - 点击 Save 保存
- 点击 add 添加第二条规则
  - Action: Block
  - Protocol: Any
  - Source: OthersRoute net
  - Destination: Network, 192.168.0.0 / 16
  - Description: deny others access mylab
  - 点击保存
- 点击 add 添加第三条规则
  - Action: Pass
  - Protocol: Any
  - Source: OthersRoute net
  - Destination: any
  - Description: allow others access internet
  - 点击保存
- 这三条规则让他们不能访问除了`192.168.100.x` 的其他局域网内容。 从而避免他们访问到我的虚拟机。 第三条规则是允许他们访问互联网
- 规则是按照顺序来判断的，符合哪个规则就使用哪个 Action 。



### 我是如何给其他人限速的呢？

先添加限速器， 然后使用就可以了。

- 点击 `Firewall/Traffic Shaper/Limiters` 
- 点击 add  新建下载速度的限速器
  - Enable: 勾选
  - Name: DownloadLimiter
  - Bandwidth: 300 Mbit/s
  - 点击 Save 保存
- 点击 add 新建上传速度的限速器
  - Enable: 勾选
  - Name: UploadLimiter
  - Bandwidth: 30 Mbit/s
  - 点击 Save 保存
- 如果需要应用， 则应用一下。 
- 点击`Firewall/Rules/OthersRoute`  找到防火墙规则
- 找到第三条规则（*其他人可以访问互联网的那条*）
- 点击 笔形状的按钮 进行编辑操作。
- 滑动到最下面， 点击 `Display Advanced` 按钮
- 滑动到下面，找到 `In / Out pipe` 第一个下拉框选择`UploadLimiter` ,第二个下拉框选择`DownloadLimiter` 。
- 保存然后应用就可以了。 
- PS: 速度根据自己的需求调整即可。 



*谢谢阅读  :smiley::smiley::smiley::smiley:*







---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E8%BD%AF%E8%B7%AF%E7%94%B1%E7%9A%84%E4%BD%BF%E7%94%A8/  

