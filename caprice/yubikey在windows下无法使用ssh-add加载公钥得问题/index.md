# Yubikey在windows下无法使用ssh Add加载公钥得问题


## 简述

使用 yubikey 访问ssh得时候，需要在本地得 ssh-agent 里面加载 yubikey 密钥对得公钥得一部分。  

可以考虑手动复制 id_ed25519_sk, id_ed25519_sk.pub 文件到目标机器， 然后使用 ssh-add 进行加载。  但是这种方法终归是麻烦了一些。 

另外一种方法就是直接从 yubikey 里面加载到 ssh-agent。 

## 主要内容

Windows 需要更新一下 ssh 组件得版本， 从 https://github.com/PowerShell/Win32-OpenSSH 下载 最新得版本安装即可。 

使用 **管理员权限** 打开一个 cmd/powershell ， 随后执行命令 `ssh-add -K -S internal`  输入正确得PIN 即可加载 公钥到 ssh-agent。 

如果使用了普通 cmd/powershell， 则很可能会看到下面提示中得一种
- Cannot download keys without provider
- Unable to load resident keys: invalid format

PIN 输入错误，则会出现另外一个错误提示。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/yubikey%E5%9C%A8windows%E4%B8%8B%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8ssh-add%E5%8A%A0%E8%BD%BD%E5%85%AC%E9%92%A5%E5%BE%97%E9%97%AE%E9%A2%98/  

