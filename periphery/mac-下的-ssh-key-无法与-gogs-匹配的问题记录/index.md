# Mac 下的 Ssh Key 无法与 Gogs 匹配的问题记录


### Mac 下的 Ssh Key 无法与 Gogs 匹配的问题记录

在生成了 ssh 密钥，并且添加到了 gogs 里面之后， 笔者发现还是无法使用密钥进行 push。

进行了一番调试之后， 笔者发现问题应该出现在之前残留的配置上。 

在文件 `~/.ssh/config` 中， 有下面一段内容。

```text
Host *
    ControlMaster auto
    ControlPath ~/.ssh/%h-%p-%r
    ControlPersist yes
```

在笔者把这段内容删除了之后， 就好了。 

这段内容的作用应该是持久化 ssh 链接， 就是在终端进行登录的时候， 无需重复输入密码。 *好像不太好用，时间太过久远，记不得了。*



### 同一台虚拟机上有两个 git 服务器的时候，怎么应用 ssh key

把当前客户机的 ssh-key 分别添加到目标服务器之后， 链接不同的端口应该就可以了。

如果想对每个服务器使用不同的密钥， 可以参考下面的配置文件。 

```text
Host git.debian.mylab
  AddKeysToAgent yes
  UseKeychain yes
  IdentitiesOnly yes
  IdentityFile ~/.ssh/id_ed25519

Host debian.mylab
    AddKeysToAgent yes
    UseKeychain yes
    IdentitiesOnly yes
    IdentityFile ~/.ssh/ssh_login_ed25519
```

这里的两个域名 `debian.mylab`,`git.debian.mylab` 都指向了同一个虚拟机。

### 参考阅读

- [关于在docker中的gitlab添加ssh-key后无法推送的问题]({{< ref "关于在docker中的gitlab,添加ssh key后无法推送的问题" >}})


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/mac-%E4%B8%8B%E7%9A%84-ssh-key-%E6%97%A0%E6%B3%95%E4%B8%8E-gogs-%E5%8C%B9%E9%85%8D%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

