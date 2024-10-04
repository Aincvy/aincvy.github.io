# 使用yubikey 签名git提交记录


实现的思路是使用GnuPG 进行签名， 所以需要先生成密钥对， 然后将私钥转移到yubikey里面。  

先说明一下 GPG(GunPG) 和 OpenPGP 的关联：  
&gt; 1. OpenPGP 是一个标准: OpenPGP 是一个开放的加密标准,定义了加密、签名、密钥管理等功能的规范。
&gt; 2. GPG 是 OpenPGP 的实现: GPG 是 OpenPGP 标准的一个具体实现,它遵循 OpenPGP 的规范。
&gt; 3. 兼容性: 由于 GPG 遵循 OpenPGP 标准,它可以与其他遵循该标准的软件进行互操作。


从网址 https://gpg4win.org/get-gpg4win.html  上下载软件， 有钱的话 可以捐赠一下， 没钱可以选择 捐赠0元。 

参考文章 https://chenhe.me/post/yubikey-starting-gpg  创建主密钥， 子密钥， 上传公钥到Github等。 

*笔者不建议给全部历史提交进行签名。* 

其他参考阅读： 
- https://github.com/drduh/YubiKey-Guide?tab=readme-ov-file



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/%E4%BD%BF%E7%94%A8yubikey-%E7%AD%BE%E5%90%8Dgit%E6%8F%90%E4%BA%A4%E8%AE%B0%E5%BD%95/  

