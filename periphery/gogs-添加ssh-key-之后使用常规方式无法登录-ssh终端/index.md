# Gogs 添加ssh Key 之后 无法登录到 系统Ssh终端


##### 解决办法 
让ssh客户端强制使用密码登录就可以了。

`ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no example.com`


##### 前置情况描述
笔者的 gogs 是直接安装在机器上的， 没有使用 docker 等容器服务。 所以 gogs 会使用系统的 sshd 服务。   
当客户机的ssh key 添加到了 gogs 的管理面板之后， 客户端就无法使用密钥登录到系统的 ssh 服务。 *不提供上面的参数的时候， 会自动使用密钥方式登录。*   
每次登录的时候都被 gogs 拦截， 然后会提示你 gogs 不提供 ssh shell。  

此时就可以使用上面提示的， 使用密码进行登录。   

##### 备选方法
*仅在 mac 下测试过。*    
在客户机的 hosts 文件里面添加两个域名解析到服务器上，  或者添加到自己的私有 DNS 服务器上。
比如 `gogs.m1.link` 和`m1.link`  。   
在客户机上生成两个 ssh 密钥对 ， 一个用于 gogs， 一个用于登录机器。

添加新建 或者修改 文件 `~/.ssh/config`  
内容如下：
```plain
Host gogs.m1.link
  AddKeysToAgent yes
  UseKeychain yes
  IdentitiesOnly yes
  IdentityFile ~/.ssh/id_ed25519

Host m1.link
    AddKeysToAgent yes
    UseKeychain yes
    IdentitiesOnly yes
    IdentityFile ~/.ssh/ssh_login_ed25519


Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  User git
  IdentitiesOnly yes
  IdentityFile ~/.ssh/id_ed25519
```

就是使用这种方式的时候， 如果 git 仓库的地址是 ssh:// 的形式， 一定要使用gogs.m1.link 这个 域名， 或者会无法操作仓库。 

### 参考阅读

https://unix.stackexchange.com/a/15141

---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/gogs-%E6%B7%BB%E5%8A%A0ssh-key-%E4%B9%8B%E5%90%8E%E4%BD%BF%E7%94%A8%E5%B8%B8%E8%A7%84%E6%96%B9%E5%BC%8F%E6%97%A0%E6%B3%95%E7%99%BB%E5%BD%95-ssh%E7%BB%88%E7%AB%AF/  

