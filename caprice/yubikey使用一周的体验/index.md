# Yubikey使用一周的体验


Yubikey 是一个物理密钥， 形状类似于U盘，接口有 USB-A, USB-C, LIGHTNING 接口的， 同时允许使用NFC无线连接手机。

应用场景有： 
- [使用yubikey-Otp验证sudo]({{< ref "使用yubikey Otp 验证sudo" >}})
- [使用yubikey签名git提交记录]({{< ref "使用yubikey 签名git提交记录" >}})
- [使用yubikey存储wireguard的私钥]({{< ref "使用yubikey存储wireguard的私钥" >}})
- [使用yubikey实现ssh免密登录]({{< ref "使用yubikey实现ssh免密登录" >}})
- [使用yubikey对keepass数据库进行加密]({{< ref "使用yubikey对keepass数据库进行加密" >}})

需要注意的点： 
- 手机上NFC链接的时候 无法输入静态密码， 因为yubikey是模拟键盘输入的， 而NFC无法做到模拟键盘。
- 槽位只有2个（短按和长按） 所以只能从下面4个功能中选择2个： 静态密码， OTP（出厂自带），挑战-响应， HTOP 
  - 挑战-响应 不需要手动键入， 但是仍然需要占用槽位。  
  - OTP是出厂自带， 会占用1个槽位， 所以能配置的只有1个。 简单来说2个槽位不够用。 
- 最大支持 64个 TOTP ， 所以可能需要考虑配合一个密码管理工具来使用。 
- 价格不低， 笔者淘宝上买了2个 5NFC， 花了1000块。 
  - 一般都是建议买2个， 一个主要使用，一个是备用的。 
- 国内软件基本上都不支持， 但是国外软件支持的很多。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/yubikey%E4%BD%BF%E7%94%A8%E4%B8%80%E5%91%A8%E7%9A%84%E4%BD%93%E9%AA%8C/  

