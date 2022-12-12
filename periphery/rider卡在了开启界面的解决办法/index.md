# Rider卡在了开启界面的解决办法


在rider打开的时候， 电脑如果意外关闭了， 则可能会造成rider无法启动的情况。 

原因可能是因为项目的缓存坏掉了， 所以就无法启动。 

解决方案是清理掉缓存。

笔者使用的操作方法比较粗暴，直接删除了整个Rider相关的目录。 

1. 关闭Rider
2. 删除文件夹 `c:\users\[username]\AppData\Local\JetBrains\Rider[version]`
3. 删除文件夹 `c:\users\[username]\AppData\Roming\JetBrains\Rider[version]`
4. 开启Rider

**注意， 这种方法下也会删掉rider安装的插件等。 也许有直接清理项目缓存的方法， 但是笔者并没有去找。**



#### 拓展阅读

https://rider-support.jetbrains.com/hc/en-us/articles/207803955-Where-are-the-ReSharper-cache-files-kept-

https://stackoverflow.com/questions/17561826/how-to-clean-project-cache-in-intellij-idea-like-eclipses-clean



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/rider%E5%8D%A1%E5%9C%A8%E4%BA%86%E5%BC%80%E5%90%AF%E7%95%8C%E9%9D%A2%E7%9A%84%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95/  

