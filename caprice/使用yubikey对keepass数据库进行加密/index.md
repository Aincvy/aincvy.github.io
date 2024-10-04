# 使用yubikey对keepass数据库进行解密


## 简述

使用yubikey 对keepass进行解密的方式有下面几种：
- 使用静态密码
  - 可以设置一个长的静态密码， 或者手输几位，再拼上一个不算很长的静态密码
- 使用 yubikey的挑战-响应机制
- 使用 yubikey的 HOTP 功能
  - 如果 HOTP功能设置给了keepass， 那么这个HOTP 就只能给keepass使用， 因为HOTP是基于次数的， 如果客户端和服务器的次数相差过大， 可能两者都需要进行一个重置操作。

## 主要内容

**本文使用的是挑战-响应(Challenge-Response)机制。** 

### 在电脑上修改数据库的加密方式

下载 https://www.yubico.com/products/yubico-authenticator/  这个工具， 笔者发现这个工具除了能显示 TOTP的验证码之外， 也能对yubikey进行设置。  而 yubikey manager 只能对 yubikey进行设置，  无法查看TOTP。 *yubikey personalization tool 笔者没有尝试过，但是听说很难操作。*

默认情况下 yubikey的 slot 1是 yubico OTP, 那么我们可以把slot 2配置成 挑战-响应机制。在配置的过程中，需要输入一个密钥， 建议直接生成， 然后将密钥复制出来以作备份。   如果有2个yubikey， 则可以将密钥写入第二个yubikey里面。 同时建议将密钥备份到一个安全的地方上，  如果不备份的话， 就无法设置新的yubikey了。 因为这个密钥写入yubikey之后就无法读取出来了，换句话说， 如果不备份的话， 就只有现存的设备存在该密钥了， 丢了一个就会少一个。


keepass的选择则有些困难， 原因如下： 
- 原生的keepass 不支持 挑战-响应模式，需要使用一个第三方插件，而这个第三方插件已经好几年没人维护了。 
- keepassXC支持 挑战-响应模式， 但是自身不支持 webdav读取数据库

在多次斟酌之后， 笔者还是选择了 keepassXC来完成这个操作。  

- 使用keepassXC打开现存数据库文件， 或者新建一个数据库
- `数据库 / 数据库安全 / 质询响应`  上添加质询响应。 *如果原本数据库使用了一个额外的文档进行加密， 则质询响应的添加可能在密钥文件区域内。*
- 添加了质询响应之后， 可以考虑将密码修改成一个不是常见的短密码， 比如4位的。

在添加了挑战响应的验证之后， 每次数据库的打开和保存都需要验证一次 yubikey 。 

### 在安卓手机上使用

笔者使用的是 keepass2android 这个工具：  https://github.com/PhilippC/keepass2android

在使用 yubikey的挑战-响应模式的时候， 还需要安装一个额外的工具ykDroid：  https://github.com/pp3345/ykDroid

在 keepass2android 尝试解锁的时候，先输入密码， 然后点击解锁， 会要求你插入yubikey 或者 NFC 读取一下。 

如果使用纯文本密码的话， keepass2android 可以用指纹解锁， 无需输入密码。 但是使用挑战响应模式的时候， 就算设置了指纹解锁， 在指纹之后仍然需要yubikey。 

注意： yubikey的静态密码在NFC模式下无法使用， 需要将Yubikey插入手机之后才能使用。 

## 参考阅读
- https://keepass.info/help/kb/yubikey.html
- https://support.yubico.com/hc/en-us/articles/360013779759-Using-Your-YubiKey-with-KeePass
- https://www.yubico.com/works-with-yubikey/catalog/keepass/



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/%E4%BD%BF%E7%94%A8yubikey%E5%AF%B9keepass%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9B%E8%A1%8C%E5%8A%A0%E5%AF%86/  

