# 在debian11 中 安装xrdp


## 正文

笔者的是 xfce 桌面环境

执行下面的命令 即可进行安装

```shell
sudo apt install xrdp
sudo adduser xrdp ssl-cert
echo xfce4-session >~/.xsession
sudo apt install dbus-x11
sudo service xrdp restart
```

然后修改配置文件 `/etc/xrdp/startwm.sh`， 只需要在合适的位置添加两行就可以了， 看代码块里面的注释。

```shell
        test -z "${LC_PAPER+x}" || export LC_PAPER
        test -z "${LC_TELEPHONE+x}" || export LC_TELEPHONE
        test -z "${LC_TIME+x}" || export LC_TIME
        test -z "${LOCPATH+x}" || export LOCPATH
fi

# 添加下面两行内容
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR

if test -r /etc/profile; then
        . /etc/profile
fi

test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession
```

修改之后 使用命令 `sudo service xrdp restart` 重启服务


还是不能连接的话， 查看 日志文件： `/var/log/xrdp.log`

*支持ms-rdp协议的客户端应该都能连接。*

## 拓展阅读
‌
- [https://www.cnblogs.com/dwj192/p/15685314.html](https://www.cnblogs.com/dwj192/p/15685314.html "smartCard-inline")
- [https://learn.microsoft.com/zh-cn/azure/virtual-machines/linux/use-remote-desktop?tabs=azure-cli#install-and-configure-a-remote-desktop-server](https://learn.microsoft.com/zh-cn/azure/virtual-machines/linux/use-remote-desktop?tabs=azure-cli#install-and-configure-a-remote-desktop-server "smartCard-inline")



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%9C%A8debian11-%E4%B8%AD-%E5%AE%89%E8%A3%85xrdp/  

