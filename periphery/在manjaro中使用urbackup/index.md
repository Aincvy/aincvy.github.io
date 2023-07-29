# 在manjaro中使用urbackup


本文介绍的是安装客户端。 

笔者最近从黑苹果切换到了 manjaro-KDE ，所以使用的是linux的桌面版。 笔者urback的server是使用 openmediavault带的插件进行安装的。

urbackup官方给的 客户端安装 方式是这个 [UrBackup - Download UrBackup for Windows, GNU/Linux or FreeBSD](https://www.urbackup.org/download.html#client_arch)

因为笔者的server版本比较老旧， 所以客户端的版本也需要老旧一点。 否则可能无法正常工作。 

笔者的server是 1.4.11版本的， 所以笔者找了一个 1.4.11版本的客户端。 地址是： [AUR (en) - urbackup-client](https://aur.archlinux.org/packages/urbackup-client)

在安装使用之后， 笔者的客户端总是会自动崩溃。 在使用命令`urbackupclientbackend -v debug` 之后， 笔者发现是在加密的部分出错了。

在一番折腾之后， 笔者发现自己编译urbackup的客户端可以解决问题。

先从官网下载源码。 

`https://hndl.urbackup.org/Client/2.5.20/urbackup-client-2.5.20.tar.gz` 这个地址可以下载2.5.20版本的源码， 把所有的2.5.20 替换成1.4.11 就可以下载1.4.11版本的源码。

下载了源码之后， 进行解压。

使用命令`/configure --prefix='/usr' --sbindir='/usr/local/sbin/' --localstatedir='/var' --sysconfdir='/etc' --enable-headless --enable-embedded-cryptopp` 进行配置。

之后使用  `make`命令进行编译， 可以使用 `make -j 4` 加速编译。

成功编译之后可以使用 `sudo make install` 命令进行安装。

在应用程序安装好了之后可以使用命令 `sudo cp urbackupclientbackend-debian.service /usr/lib/systemd/system/urbackupclientbackend.service` 安装service 文件。 

安装service文件之后， 可以使用命令 `sudo systemctl daemon-reload` 重载系统的service 。

之后可以使用命令 `sudo systemctl enable urbackupclientbackend` 使urbackup开机启动。

urbackup的默认配置文件在 `/etc/default/urbackupclient` 修改文件的RESTORE选项为server-confirms 即可在server 的web界面进行文件的恢复操作。 

`RESTORE="server-confirms"`

成功安装之后， 需要使用命令 `sudo urbackupclientctl add-backupdir [path]`  来添加需要备份的目录


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%9C%A8manjaro%E4%B8%AD%E4%BD%BF%E7%94%A8urbackup/  

