# 黑苹果的几次安装记录


从我的安装经验来看， 黑苹果的安装非常简单，只是有点耗时而已。

教程地址：  https://dortania.github.io/OpenCore-Install-Guide/prerequisites.html

下面是使用教程的几个前提：

- 你有一定的英文阅读水平。
- 你具有耐心， 可以使用搜索引擎去尝试解决问题。
- 你使用 Intel 的CPU 和 AMD的显卡， 或者你将要更换。 （*非必需 :joy:*）
- 有一个大于8G的空闲U盘。
- 当前电脑可以运行一个系统（Windows/Linux 都可以)
- 在选购硬件的时候，笔者建议选购一个不带无线网卡的主板， 然后自购一个免驱网卡（大多数都带蓝牙）。 而这个网卡再Windows下基本上也都有驱动，可以正常使用。 如果一个电脑有两个物理蓝牙设备，就有点麻烦。  



笔者跟随这个教程，成功在公司电脑， 个人家里电脑上安装了 Mac 系统的 Catalina 版本。

关于AMD的CPU，有一些功能会在成功安装后无法使用。

*以下引用上面教程原文*

>Unfortunately many features in macOS are outright unsupported with AMD and many others being partially broken. These include:
>
>- Virtual Machines relying on AppleHV
>  - This includes VMWare, Parallels, Docker, Android Studios, etc
>  - VirtualBox is the sole exception as they have their own hypervisor
>  - VMware 10 and Parallels 13.1.0 do support their own hypervisor, however using such outdated VM software poses a large security threat
>- Adobe Support
>  - Most of Adobe's suite relies on Intel's Memfast instruction set, resulting in crashes with AMD CPUs
>  - You can disable functionality like RAW support to avoid the crashing: [Adobe Fixes](https://gist.github.com/naveenkrdy/26760ac5135deed6d0bb8902f6ceb6bd)
>- 32-Bit support
>  - For those still relying on 32-Bit software in Mojave and below, note that the Vanilla patches do not support 32-bit instructions
>  - A work-around is to install a [custom kernel](https://amd-osx.com/download/kernel.html), however you lose iMessage support
>- Stability issues on many apps
>  - Audio-based apps are the most prone to issues, ie. Logic Pro
>  - DaVinci Resolve has been known to have sporadic issues as well

N卡的话， 则是因为在mac上没有驱动， 只能使用老旧的 webDriver，所以不推荐。 

AMD的GPU， 笔者尝试的 Rx580, 5700XT 都是免驱的，装好系统，插上显卡就可以使用了。



耐心看几遍上面的教程， 选好硬件， 很容易就可以安装了。

笔者家里的电脑安装的是 Mac/Windows双系统， Windows用于打游戏， Mac用于编码，写作等。

这样选择的原因是想工作/娱乐分离， 在公司安装黑苹果是因为想使用一套软件工具。

还有另外一个原因是， 我之前公司电脑是Windows的。 平时我都不关机，有一日我早上到公司的时候 发现系统的注册表坏了， 就莫名其妙的坏了， 我不知道发生了什么。。   我只好拿同事的注册表复制过来，试试。 折腾了一段时间，勉强能用了， 但是还有不少问题。。  一般来说， Mac就不会有这个问题。

 但 Mac有些别的问题。。  

- Mac在系统升级之后，部分软件可能就无法使用了， 就需要更新软件，而部分软件授权是跟着版本来的。  （**不过因为Mac系统升级是需要手动确认的， 不更新就没什么问题。**） 
- Mac上的软件正版大多都很贵。。  



#### 还有一个TimeMachine相关的内容

我之前使用的淘宝购买的 Macbook pro 15.6寸的。  因为预算问题，购买的128G硬盘空间。 

后来使用过程中发现 内存有点不够，所以购买了一个1T的 M.2 SSD 和一个硬盘盒。这样当做外接硬盘使用，我给硬盘分了两个分区， 一个叫 `mac_extend` 给mac使用， 还有一部分想给Windows使用（*虽然一次也没使用过:joy:*)

在设置TimeMachine的时候， 手动把备份`mac_extend`的屏蔽项 去掉， 让TimeMachine备份 mac_extend 分区。

在对家里电脑安装黑苹果的时候，我把外接的硬盘拆掉，放进主板里给黑苹果使用。  因为部分配置的问题，我在安装过程中，把全盘格掉了。

后来折腾了一下， 成功进入了mac系统的安装界面。 我选择了 从TimeMachine中恢复系统的选项， 等恢复结束之后， 我发现我的 mac_extend 分区没了。。 

经过一番折腾， 我发现TimeMachine里面的文件还是存在的， 按照下面的方法就可以恢复。

1. 利用磁盘工具 建立一个叫做 `mac_extend`的分区。 （和之前的分区保持相同的名字）
2. 划分的空间要比备份的大， 但是不必完全一样。
3. 文件系统我是用的 APFS。
4. 进入TimeMachine ， 找到 `XXX的iMac` 这个目录，会发现里面有一个叫做`mac_extend`的目录， 进入之后会发现 文件还在。
5. 我是进入这个目录之后， 使用CMD+A全选， 然后点击下面的恢复。 
6. 等待若干时间之后就成功恢复了全部的文件。  （**部分文件可能需要输入密码才能恢复**）



*使用TimeMachine恢复系统， 会有部分软件丢失配置， 部分变成未激活状态。*

以上。


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%9A%84%E5%87%A0%E6%AC%A1%E5%AE%89%E8%A3%85%E8%AE%B0%E5%BD%95/  

