# 旧服务器换新碰到的问题记录


*本篇更像是一篇日记，而非博客，当成一个故事来看即可。*

背景简介： 
- 旧服务器是洋垃圾， 运行的 esxi 6.5 系统
- 从笔者买到现在已经跑了5年了， 基本上没出什么问题。 
- 本次更换是因为需要使用SD生成图片，所以买了一个3060， 插上去之后， 不知道是电源坏了，还是主板CPU坏了， 无法开机， 所以就换新的了。
- 还有一个原因是笔者有一个闲置的主板， 正好拿来使用。 


CPU更换成了 amd 5600 ， 感觉比以前丝滑多了， 而且省电多了， 这个U是65W的。 

## 小问题记录

### 主板的BIOS 不能识别5600 

解决方法：  更新主板的BIOS即可。 

- 新的问题是这个主板需要插上CPU才支持更新BIOS，而笔者手上又没有一个CPU。 
- 想去看看电脑店能不能帮忙更新， 询问了一下报价。小店200， 大店299。 大店说是直接烧录的BIOS
- 笔者感觉都挺贵的，所以去闲鱼收一个支持的CPU： AMD 2600。  228 包邮， 更新了主板的BIOS 之后， 190包邮出掉了。 所以总共花费是50块左右的样子。    


###  系统转移

ESXI6.5 好像是因为不支持 AMD的CPU，表现为系统安装之后无法启动， 所以换了debian 11 。    

在debian11 成功安装之后， 现在的需求是将esxi 里面的虚拟机都转移到debian 11上。 

最大的麻烦点在于， 硬盘是esxi的文件系统， 要么运行一个 esxi的虚拟机，然后将文件复制到当前的磁盘， 要么使用一个工具来加载 esxi的文件系统。 

- `vmfs-fuse` 工具可以读取 vmfs 文件系统里面的文件， 然后可以将文件复制到挂载的其他硬盘之中。 
- 使用 vmplayer  可以成功运行esxi虚拟机。   
  - esxi里面的虚拟机好像也可以使用vmplayer进行运行。
  - 运行  esxi 的时候， 系统选择 `other/other 64bit`


笔者后来选择了 vbox 作为debian11上的虚拟机工具， 而没有选择vmplayer 。    *现在感觉选择错了*

下面是一些vbox的缺点
- vbox 的开机启动虚拟机， 设置起来麻烦，而且效果不好。 
- 网络有的会很慢。  将网卡调整成 PCNet-Fast III adapter ， 则有可能解决。

vbox 提供了硬盘转换工具，可以将 vmdk 格式的磁盘转换成 vdi 格式的，  还提供了很多的命令行工具。  

### 其他知识增加

- 可以不在当前机器安装jre, 从而使用 docker 的jre镜像运行java程序。 
- 重启transmission：  `/etc/init.d/transmission-daemon reload`
- 使用 rsync 可以传输文件， 文件夹。 rsync 支持断点续传， 1T以上的文件夹可以考虑取消哈希检测， 因为很慢。
- 查看某个端口是否被某个程序占用：  `sudo ss -lptn 'sport = :80'`    
  - https://unix.stackexchange.com/a/106562/575254
- 将当前目录里面的内容移动到父文件夹里面：   `find . -maxdepth 1 -exec mv {} .. \;`
  - https://superuser.com/a/88205/1159248
- 对shell 命令的参数进行解析和解释， 但是这个应该没有chatgpt好用：   https://explainshell.com/explain?cmd=find%20.%20-maxdepth%201%20-exec%20mv%20%7B%7D%20..%20%5C%3B
- 软碟通无法刻录笔者下载的debian镜像， 使用的 rufus-4.1 进行刻录的。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E6%97%A7%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%8D%A2%E6%96%B0%E7%A2%B0%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

