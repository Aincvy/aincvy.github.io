# Manjaro使用了一段时间的体验


**图片预警！（文件尺寸较大）**

## 安装

笔者使用的是`manjaro-KDE` ， 是从官方网站下载的包。 

安装过程很简单， 基本上就是下一步下一步，如果自己不懂分区的话，就选择自动分区即可。 

因为笔者希望这块硬盘也分配一些空间给windows使用， 所以是手动分区的。 

手动分区也很简单， 基本上就是分成下面的形式。 *下面是笔者使用的方式*

- 一个 `/boot/efi` 大约500M 打上 `boot`的flag.
  
  - 这个是引导分区， 如果没有这个的话， 安装程序会提示你的。 

- 一个`/` 分区， 大约200G， 是系统根目录分区。

- 一个 `/home` 分区， 大约500G， 是家目录的分区。 

## 截图
*右键，在新标签页中打开图片可以查看大图。*
![](/img/periphery/manjaro-kde/1.png)

![](/img/periphery/manjaro-kde/2.png)

![](/img/periphery/manjaro-kde/3.png) 

![](/img/periphery/manjaro-kde/4.png)

![](/img/periphery/manjaro-kde/5.png)

## 碰到的问题与其他

### Unity相关问题

因为笔者的显示器是4K的， 而在高分辨率下， Unity中的字和窗口内容都很小。 

可以使用下面的内容 放大2倍 但是不能放大1.5倍。 这就导致Unity的窗口会变得很大。 
总之效果不是很好。 

```bash
export GDK_SCALE=2
export GDK_DPI_SCALE=0.5
```
此外， `steam-vr`是无法运行在 Linux上面的。  

### 使用manjaro的初衷
笔者刚开始是选择的黑苹果， 但是因为黑苹果更新比较麻烦， 而苹果系统总是弹窗让我更新， 所以想换Linux试一试。   *其实也有喜新厌旧的成分在里面。*

### 终端 
笔者使用的是 `fish`， 使用 https://github.com/oh-my-fish/oh-my-fish 进行管理的。

主题是`bobthefish`, 运行终端是`KConsole`, KDE附带的组件。 

### 浏览器
`Chrome` 和 `Firefox` 都可以使用， 笔者没感觉到和别的平台有太大区别。

### 其他
KDE的可定制程度较高。
- 笔者的顶部栏和底部栏都是自己定制的。 
- 文件管理器的ICON 也是自己找的 

中文输入法虽然有， 但是词库感觉比较怪异， 不是那么方便。  

### KDE-Connect
在手机上安装了这个应用之后就可以和电脑链接了。 

可以在电脑上查看手机的短信，也可以发送短信。 
可以在电脑上查看手机的通知。
还可以互相发送文件，以及发送粘贴板。 
由于笔者没有使用腾讯类的软件， 所以这个工具还算好用。

### 软件相关

#### MarkText
这是一个 markdown 写作工具， 笔者在manjaro 上发现， 这个应用程序在某些情况下会崩溃。 

#### vs-code
安装命令： `yay -S visual-studio-code-bin`
更新使用相同的命令，需要手动更新


#### chrome
安装命令：  `yay -S google-chrome`
更新使用相同的命令，需要手动更新

#### java
安装jdk11的命令:  `sudo pacman -S jre11-openjdk-headless jre11-openjdk jdk11-openjdk openjdk11-doc openjdk11-src`
来源：  https://linuxconfig.org/how-to-install-java-on-manjaro-linux
注意， doc 和 src 包需要单独安装， 否则在开发的时候会发现jdk的类没有源码。

#### 其他
vscode 和 chrome 在系统 更新的时候并不会自动更新， 所以需要手动运行命令。 



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/manjaro%E4%BD%BF%E7%94%A8%E4%BA%86%E4%B8%80%E6%AE%B5%E6%97%B6%E9%97%B4%E7%9A%84%E4%BD%93%E9%AA%8C/  

