# 关于在docker中的gitlab,添加ssh key后无法推送的问题


在使用unity的时候， 经常需要推送一些大的资源文件到项目里面， 使用http的方式，就会出现一些错误， 被拒绝。 我在谷歌转了几圈之后发现，大多人都推荐使用`ssh`的方式推送，这样也不需要用户名和密码。

我按照常规方式配置好`ssh key`之后，竟然无法推送。应该是提示了 “ssh: connect to host port 22: Connection refused”。

经过调试，我发现 **我的gitlab 是装在docker里面的**。 

22端口是我的debian 虚拟机的sshd 服务在使用， 而不是gitlab 容器在使用。

所以我在git客户端上指定一下gitlab的端口 就好了。

参考下面的命令：

`git remote add origin ssh://user@host:1234/srv/git/example`

- 1234 是端口号， 修改成你使用的即可。 
- user 是gitlab的用户名
- host 是gitlab所在的主机
- 在gitlab的项目主界面， 把`HTTP`切换成`SSH` 就可以得到SSH的地址， 然后添加下端口应该就可以了。



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%85%B3%E4%BA%8E%E5%9C%A8docker%E4%B8%AD%E7%9A%84gitlab%E6%B7%BB%E5%8A%A0ssh-key%E5%90%8E%E6%97%A0%E6%B3%95%E6%8E%A8%E9%80%81%E7%9A%84%E9%97%AE%E9%A2%98/  

