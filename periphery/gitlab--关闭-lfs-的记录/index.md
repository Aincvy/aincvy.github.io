# Gitlab  关闭 Lfs 的记录


###  基本内容

在本地客户端 关闭 lfs 之后，  推送到远端提示 这个

remote: GitLab: LFS objects are missing. Ensure LFS is properly set up or try a manual "git lfs push --all".

解决办法就是 使用 gitlab 的网页管理端， 找到 项目， 用户设置， 通用， 可见性，项目功能，权限， Git 大文件存储   ，把这个取消 掉就可以了。


###  为什么要在客户端上关闭 lfs
起因是 使用 lfs 推送的时候 ， 提示  413 错误。   

笔者谷歌之后，别人说修改 `nginx['client_max_body_size']` 可以修复错误。 值可以修改成一个较大的值 ，或者修改为 0。 

但是笔者尝试之后 并没有解决问题。 


### 参考阅读

https://gitlab.com/gitlab-org/omnibus-gitlab/-/issues/4272

https://github.com/git-lfs/git-lfs/issues/3026#issuecomment-696697599


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/gitlab--%E5%85%B3%E9%97%AD-lfs-%E7%9A%84%E8%AE%B0%E5%BD%95/  

