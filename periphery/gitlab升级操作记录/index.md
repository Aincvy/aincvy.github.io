# Gitlab升级操作记录


笔者使用docker的方式安装的gitlab, 升级操作十分的简单。



这里有两篇官方的文档可以阅读一下。 

- https://docs.gitlab.com/ee/raketasks/backup_restore.html

- https://docs.gitlab.com/omnibus/docker/README.html
- https://docs.gitlab.com/ee/policy/maintenance.html#upgrade-recommendations

第一个链接的内容是关于备份和恢复的， 第二个则是关于 docker 镜像的详细介绍，第三个链接则是升级的建议。



**建议读者在进行操作之前，先进行一次备份。**



笔者的操作方式是 先升级 minor,然后升级patch . 然后升级 major 。

在这个链接[Docker hub: gitlab CE](https://hub.docker.com/r/gitlab/gitlab-ce/tags/?page=1&ordering=last_updated)寻找指定版本的镜像即可。 

gitlab的版本号格式是： `(Major).(Minor).(Patch)`

举个🌰🌰， gitlab的版本号是 `12.10.6`

- `12` 表示主版本 *大版本*
- `10` 表示次要的版本号 *小版本*
- `6` 表示🍮      *补丁*

*更详细的信息可以阅读升级建议。 链接地址在上方。*



理论上来说， 先做好备份， 然后按照笔者的流程进行升级， 应该问题不大。

*大版本应该是不能跨版本升级的，最好1个1个的来升级大版本。*



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/gitlab%E5%8D%87%E7%BA%A7%E6%93%8D%E4%BD%9C%E8%AE%B0%E5%BD%95/  

