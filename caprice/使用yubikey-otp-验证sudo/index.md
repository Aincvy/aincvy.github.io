# 使用yubikey Otp 验证sudo



## 简述

笔者使用的是 yubico PAM 模块实现的。 

参考网址： https://developers.yubico.com/yubico-pam/

## 主要内容

笔者使用的 Debian 系统， 所以在本文说明一下 Debian 系统如何操作。 

### 安装 Yubico PAM 模块

由于 yubico pam 模块并没有提供debian 的软件安装包， 所以需要去ubuntu 的仓库里面拉取安装包。

ubuntu 的官方安装指令是 
```bash
sudo add-apt-repository ppa:yubico/stable
sudo apt-get update
sudo apt-get install libpam-yubico
```

首先确认一下 Debian系统的版本。 

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux trixie/sid
Release:        n/a
Codename:       trixie
$　cat /etc/debian_version
trixie/sid
```

Codename 是 Debian 系统的版本， 在本示例中是 trixie 。

- 先确定对应的ubuntu版本是什么， 可以参考网址：　[https://askubuntu.com/a/445496/1701779](https://askubuntu.com/a/445496/1701779)
- 由上面的表格可知， trixie对应的是 ubuntu 24.04 LTS  版本
- 打开网址  https://launchpad.net/~yubico/&#43;archive/ubuntu/stable
  - 点一下 `Technical details about this PPA`    会将一个隐藏的区域显示出来。
  - 在 `Choose your ubuntu version` 的地方选择 24.04  
  - 导入公钥：　`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 990BD85D1C4F411ABC0ECE8DFA4EC8F2EE58CE0B`
    - 公钥ID在页面中 `Signing key` 下面的部分， 去掉前面的 `4096R/` 和最后的 `(What is this?)` 之后得到的中间部分就是ID
  - 添加 仓库：  `sudo add-apt-repository -S &#34;deb https://ppa.launchpadcontent.net/yubico/stable/ubuntu noble main&#34;`
    - 如果提示 add-apt-repository command not found， 则使用命令 `apt install add-apt-repository` 来安装
  - 接下面就是简单的 `sudo apt-get update &amp;&amp; sudo apt-get install libpam-yubico` 即可

### 配置 PAM 模块

- 打开网址 https://upgrade.yubico.com/getapikey/  获取到一个 id 和secret 
  - 需要输入邮箱和 一个 yubikey OTP 
- 创建文件 `/etc/yubikey_mappings`
  - 填充内容：  `username:yubikey_token1:yubikey_token2`
    - username 就是 需要执行sudo 的 linux 用户
    - yubikey_token1, yubikey_token2 就是 yubikey OTP 的 token， 如果只有1个yubikey的话， 写一个就可以了。 值是 OTP 的前12位字符串。  *多生成几个OTP，你会发现每个OTP的前12位都是一样的:joy:*
- 修改文件： `/etc/pam.d/sudo`
  - 在第一行添加 `auth sufficient pam_yubico.so id=[client id] key=[secret] authfile=/etc/yubikey_mappings`
  - client id 和 secret 是第一步在 yubico 网站上获取到的。 
- 执行sudo 之后就会看到一个 `YubiKey for username:`  此时键入 yubikey OTP 即可。  
  - 出厂默认的时候是在 slot 1, 同时也可以手动切换到 slot 2 。

## 其他

使用 OPENPGP 好像也可以实现， 但是笔者并没有尝试，下面是一些参考文章： 
- https://github.com/drduh/YubiKey-Guide?tab=readme-ov-file
- https://serverfault.com/a/847794


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/%E4%BD%BF%E7%94%A8yubikey-otp-%E9%AA%8C%E8%AF%81sudo/  

