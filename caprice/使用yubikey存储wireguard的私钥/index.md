# 在Windows上 使用yubikey存储wireguard的私钥


## 简述

流程大概是将私钥存储在yubikey里面， 然后wireguard 发起连接的时候从yubikey里读取私钥到文件， 然后设置私钥到wg接口上，然后删除私钥文件。

*wg设置私钥的时候只能设置一个文件路径，不能直接使用字符串进行设置，笔者没有找到其他方法来设置私钥。虽然这样安全性会低一点，但是总体来说应该还好。*

想要实现这个目的涉及到的操作有：
- 修改系统注册表
- wireguard windows 
- ykman， 这个可以从yubikey的官网上下载 

## 主要内容

- 将正确的私钥写入 yubikey 
  - 将私钥写入文件 ./tmp.txt中，需要使用ANSI编码。
  - 使用命令 `ykman piv objects import 0x5fc106 ./tmp.txt`  来写入密钥到yubikey里面
  - 删除 ./tmp.txt 文件
- 准备一份可以正常链接到服务器的wireguard配置文件
- 准备下方的 bat 文件， 可以命名成 ./set_private_key.bat
- 使用命令 `wg genkey`  生成一个占位私钥
- 修改wireguard配置文件 使用占位私钥替换掉正确的私钥， 添加新的选项 `PostUp = [set_private_key.bat的绝对路径]`

bat文件示例和解释： 

```bat
@echo off

set "wireguard_exe=C:\Program Files\WireGuard\wg.exe"

@REM 文件名是随机生成的128个字符，这样用于避免文件冲突
set "private_key_file=%tmp%\HdBNzfQpu035yaa11OLu4VNibm8UeFYZBt9A15oDXwxgAfVKc71D4Bl1Dc1GP1nPYJF9nrGYfF4upWn2FSTfwT8GLCKTslA9gKOdTammvDPPWjxwKEW7xvBrHdKjV0fH.txt"

@REM 删除文件，以防存在
del /q %private_key_file%

@REM  写入私钥到临时文件
ykman piv objects export 0x5fc106 "%private_key_file%"
@REM 设置私钥
"%wireguard_exe%" set _name private-key "%private_key_file%"

@REM 使用结束，删除文件
del /q "%private_key_file%"
if errorlevel 1 (
    echo 删除文件失败
)
```

注意， _name 需要替换成自己的wireguard接口名字。

现在需要修改注册表： 
- 使用管理员模式打开一个cmd.exe 窗口
- 执行命令：  `reg add HKLM\Software\WireGuard /v DangerousScriptExecution /t REG_DWORD /d 1 /f`
- 如果不添加的话， PostUp脚本会不允许执行

使用这种方式的时候不会要求输入yubikey的PIN， 只要插入yubikey就可以正常连接到服务器。 

## 拓展阅读

- https://developers.yubico.com/yubico-piv-tool/Actions/read_write_objects.html
- https://kmg.group/posts/2021-10-05-store-wireguard-private-key-on-yubikey/


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/caprice/%E4%BD%BF%E7%94%A8yubikey%E5%AD%98%E5%82%A8wireguard%E7%9A%84%E7%A7%81%E9%92%A5/  

