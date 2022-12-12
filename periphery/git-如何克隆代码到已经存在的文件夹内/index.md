# Git 如何克隆代码到已经存在的文件夹内


这里不能再使用clone 方法了。

1. 打开 git bash(windows 下)
2. 利用 cd 命令 切换到 要克隆到的目录的位置 比如： `mkdir projectA && cd projectA`
3. 输入命令： `git init` 在当前目录下初始化一个仓库
4. 输入命令： `git pull [url]` 从目标仓库中拉取文件到本地
5. 输入命令： `git remote add origin [url]` 添加一个远程分支
6. 输入命令： `git add .` 把当前目录的文件添加到提交列表
7. 输入命令： `git commit` 填写提交消息， 然后输入 `:wq` 结束
8. 输入命令： `git push –set-upstream origin master` 推送代码， 会要求你输入用户名和密码
9. 以后再推送的时候 直接输入 git push 就好了。 如果你使用什么IDE的话， 如果IDE 有GIT 的插件的话，那么现在应该就可以看到GIT 的信息了



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/git-%E5%A6%82%E4%BD%95%E5%85%8B%E9%9A%86%E4%BB%A3%E7%A0%81%E5%88%B0%E5%B7%B2%E7%BB%8F%E5%AD%98%E5%9C%A8%E7%9A%84%E6%96%87%E4%BB%B6%E5%A4%B9%E5%86%85/  

