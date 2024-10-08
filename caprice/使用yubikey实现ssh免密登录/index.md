# 使用yubikey实现ssh免密登录


## 简述

免密登录ssh 有以下好处：
- 无需记忆密码
- 节省时间， 因为输入密码需要时间， 如果密码很长， 不小心输错了一位， 就又需要重头再来
- 防盗
  - 有一部分盗号木马的原理就是监控键盘输入，免密登录不从键盘上输入密码， 自然无从盗起

yubikey 是一个硬件密钥，  形状类型于U盘， 接口有 USB-A, USB-C, LIGHTNING 接口的。

使用一个硬件密钥的好处：
- 可以在任意的机器上登录自己的账号， 不怕被盗，  静态密码方式除外。
- 在支持FIDO2的网站上会多一层保护。 

## 主要内容

免密登录的实现方式可以选择SSH密钥对， 或者 OPENPGP 密钥对， 笔者使用的是 SSH密钥对。  
硬件密钥相关的SSH算法是ecdsa-sk 和 ed25519-sk， 要求的最低 openssh版本是 8.2。   
有一点需要注意， Windows 的ssh 组件好像是阉割版本的， 需要从 https://github.com/PowerShell/Win32-OpenSSH 下载。 

注意点： 
- 留在电脑上的私钥不是真正的私钥， 真正的私钥在硬件密钥里面。 
- 如果想免密在服务器上使用sudo 的话， 建议使用  OPENPGP / GPG 做认证， 可参考： https://github.com/drduh/YubiKey-Guide?tab=readme-ov-file

### 密钥对生成以及认证

实现思路和常规思路是一样的， 生成密钥对， 然后把公钥上传到服务器上就可以了。

- 插上 yubikey
- 使用管理员模式 打开一个 powershell窗口， 随后执行命令：  `ssh-keygen -t ed25519-sk -O resident -O verify-required -C &#34;yubikey1&#34;`
  - 如果提示读者“某个XX不存在， 需要覆盖吗 ？”  这个看读者的经验， 如果KEY是新的话， 就直接覆盖即可。 
  - `-O resident` 表示公钥也写入到密钥里面
  - `-O verify-required`  表示使用的时候需要验证 
  - 在生成证书的时候应该会要求设置一个PIN， 输入一个8位的PIN 即可。 
- 复制公钥到服务器上
  - 可以手动复制， 或者使用命令 `ssh-copy-id &#34;[公钥文件路径， 比如id_ed25519_sk.pub]&#34; 用户名@服务器地址`
    - 这个命令执行一次就可以了， 执行多次， 就会复制多次
  - 公钥的内容将会被复制到服务器上的 `~/.ssh/authorized_keys` 文件里面， 如果想要手动复制的话， 也是将公钥文件的内容复制进来，一行一个。 
- 使用 `ssh -v 用户名@服务器地址`  测试一下是否可用。   `-v` 表示开启更多的日志输出。 
  - 笔者在Windows上发现， 使用 `用户名@服务器IP`  不行， 使用 `用户名@服务器域名` 就可以， 暂时没弄懂原因。  服务器域名是笔者自建的本地DNS 做的解析。  *可能和 .ssh 目录里面的 config 文件或者 known_hosts 文件有关系，但是笔者没深入测试*

### 在新机器上使用

插上硬件密钥， 使用管理员模式打开一个powershell 窗口， 执行命令 `ssh-add -K`  就可以把公钥从硬件密钥上加载到本地。  使用 `ssh-keygen -K` 会把密钥保存到本地。   注意， 这两个命令会要求读者输入PIN， 在输入完PIN之后需要摸一下 yubikey 的小圆圈。 *小圆圈上有一个黄色的小灯， 当这个灯在闪烁的时候基本上就表示你需要摸一下它才能执行后续得操作。* 

如果输入了正确PIN之后， ssh-add -K 提示导入失败则可以考虑试试下面的方案： 
- 使用 新版本（2024年9月28日以后）的 Mobaxterm 来执行 `ssh-add -K` 命令。 需要注意的是 Mobaxterm 自己会携带一套openssh工具箱，  导入的公钥只能在 Mobaxterm 窗口里面使用。  如果想在powershell里面使用， 则需要把密钥对复制出去。
- 或者将私钥文件和公钥文件打包上传到网盘或者别的网络存储上， 然后再新机器上下载文件，并解压。  



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/%E4%BD%BF%E7%94%A8yubikey%E5%AE%9E%E7%8E%B0ssh%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95/  

