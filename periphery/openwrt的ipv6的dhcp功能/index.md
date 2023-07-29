# Openwrt的ipv6的dhcp功能


笔者曾经有一段时间发现，在Windows机器上设置了ipv4的dns服务器之后没有效果。 之前返回的是什么，修改之后返回的还是什么。。 在把ipv6关掉之后修改才会生效。

笔者最近发现是因为openwrt的 ipv6的dhcp服务的原因，把ipv6的dhcp服务关掉之后，一切就正常了，Windows机器也不需要禁用ipv6了。



笔者使用openwrt做旁路由，使用pfSense做主路由。 

ipv4的dhcp服务由pfSense提供，pfSense会把openwrt的地址作为网关和dns服务器分配给客户机。



似乎是因为 windows 和安卓在开了ipv6得时候， 会主动使用ipv6得dns 而忽略ipv4的dns。

在openwrt中找到 `network > interface > LAN edit > DHCP Server > IPV6`
全部停用即可停用 LAN接口的服务， 如果存在多个接口，则可能需要每一个接口都设置一下。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/openwrt%E7%9A%84ipv6%E7%9A%84dhcp%E5%8A%9F%E8%83%BD/  

