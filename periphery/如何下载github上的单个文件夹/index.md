# 如何下载github上的单个文件夹



使用第三方工具
- https://download-directory.github.io/
- http://kinolien.github.io/gitzip
- https://minhaskamal.github.io/DownGit


或者使用SVN

1. 导航到想要下载的文件夹， 比如 `https://github.com/Aincvy/sight/tree/main/plugins/sight-base`
2. 将 `tree/master`  或者 `tree/main` 替换成 `trunk`
3. 执行命令 `svn checkout https://github.com/Aincvy/sight/trunk/plugins/sight-base`

这样就可以了， 注意，这种方法只能下载github上的文件夹， 其他git服务器上的可能不行。   


参考阅读：  https://stackoverflow.com/a/18194523/11226492



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%A6%82%E4%BD%95%E4%B8%8B%E8%BD%BDgithub%E4%B8%8A%E7%9A%84%E5%8D%95%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9/  

