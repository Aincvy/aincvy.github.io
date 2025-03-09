# Mattermost安卓手机消息推送折腾


## 简述

Mattermost 得消息通知在ios平台上可以很即时得收到， 但是安卓手机上无论如何也无法收到通知， 折腾了一番之后可以在适当得延迟之后获取到通知了。 

## 主要内容

Mattermost 得消息通知是基于 谷歌 FCM得， 所以消息无法到达得原因需要从两个部分来看， 一个是消息有发送到 FCM吗，另外一个是FCM有发送到手机吗。

**第一个问题，消息有发送到 FCM吗？**   
Mattermost 得 System Console 界面里面有一段是 Push Notification Server 得设置， 默认设置得是 mattermost 提供得推送服务器，这个一般都是可以访问到得。 

**第二个问题，FCM有发送到手机吗？**   

笔者使用得是三星S24国行版手机。 

经过一些探查， 笔者发现确实是手机没有收到推送消息。  

帖子 https://www.v2ex.com/t/981806  介绍了一个工具， 这个工具可以查看哪些应用支持 FCM， 以及 FCM的当前连接状态。

当具备以下条件时便可收到 消息推送
- 手机需要先安装 google play service
- FCM需要显示为成功连接
- 需要把Mattermost得电量限制设置为不受限制，或者优化。 
- 梯子工具将选项`允许其他应用绕过VPN` **关闭**
- 开启梯子工具

上面得帖子上有说， 三星国行系统会检测能不能连上谷歌，如果无法连接，则会自动断开FCM得连接。   需要注意得是 FCM 本身是没被墙得。 

*把FCM折腾好了之后， 所有基于FCM进行消息推送得软件得通知都可以收得到了。*


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/mattermost%E5%AE%89%E5%8D%93%E6%89%8B%E6%9C%BA%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%8A%98%E8%85%BE/  

