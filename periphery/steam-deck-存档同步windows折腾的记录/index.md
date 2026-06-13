# Steam Deck 存档同步windows折腾的记录


现在的效果是 steamdeck 游戏结束后可以自动上传备份到 webdav 服务器， 然后windows 使用 webdav(owncloud) 客户端自动下载文件。  
windows 随后可以打开 ludusavi 手动 restore 文件。

steamdeck 还有两个按钮可以手动 backup / restore 存档。 

这个主要想解决的问题是 steam cloud 因为网络不稳定 影响存档同步， 比如 艾尔登法环的存档要 53M  传来传去又走不了局域网的时候 就很影响体验。

## 简要内容

使用的软件有： 
- https://github.com/mtkennerly/ludusavi
- https://github.com/GedasFX/decky-ludusavi
- https://github.com/SteamDeckHomebrew/decky-loader
- https://nextcloud.com/
  - 或者 owncloud / sftp / smb 等

使用的硬件有： 
- usb c转a口 + 键盘
  - 或者使用无线蓝牙键盘

steamdeck 可以开启 sshd 服务， 也可以使用windows 进行一定的操作。 

## 主要流程 

**steam deck 操作**
1. 按住电源键进入桌面模式
2. 在应用程序菜单里面找到  KConsole  打开终端
3. 按一下  手柄的 X 键 打开 虚拟键盘   或者外接键盘 或者蓝牙键盘
4. 使用 `passwd` 命令设置一下密码
5. 输入 `sudo systemctl start sshd`  开启 ssh 服务
6. 输入 `ip a`  查看自己的LAN IP ，一般是 192.168 开头的

**windows主机操作，或者linux主机**

现在steam deck 已经开启了 sshd 服务， 可以使用 ssh 连上去了。   
选择你喜欢的ssh 客户端工具 然后使用 `ssh deck@192.168.xx.xx` 链接   
密码就是前面设置的密码。 

现在需要安装软件到steamdeck， 如果网络不好， 可以先设置代理。

1. 设置代理， 如果网络好 可以跳过。   
   1. `export http_proxy=http://代理ip:代理端口`  
   2. `export https_proxy=http://代理ip:代理端口`
2. 安装 ludusavi： `flatpak install flathub com.github.mtkennerly.ludusavi`
3. 安装  decky-loader：  `curl -L https://github.com/SteamDeckHomebrew/decky-installer/releases/latest/download/install_release.sh | sh`
   1.  如果不行的话， 就使用 `curl -L https://github.com/SteamDeckHomebrew/decky-installer/releases/latest/download/install_release.sh | sudo -E sh`


**切换到steam deck 进行操作**

4. 配置 ludusavi（这一步在steamdeck 的桌面模式下执行，不要在ssh客户端里面做）： `flatpak run com.github.mtkennerly.ludusavi`
   1. 一定要使用这个命令， 否则会出现配置不同步的问题
   2. 这里找到 "Other" 界面
   3. 然后找到 "Cloud" 
   4. 设置一下  rclone 的路径（如果默认没设置的话） 设置为: `/app/bin/rclone`
   5. 然后选择云端的目的地， 笔者使用的是 webdav 
   6. 勾选 自动同步选项， 好像是 "sync auto...." 之类的选项
5. 测试一下 ludusavi 
   1. 打开 第一个 "BACKUP MODE"  然后点击 "Preview" 勾选要备份的游戏，然后点击"Back Up"
   2. 如果一切正确的话， 就会自动上传云端， 可以去云端看看文件有没有。 
6. 在桌面找到 "Ruturn to Game Mode"  返回游戏模式
7. 按一下 右下角的 "..." 按钮， 找到 一个类似🔌的icon 点进去。
   1. 点击store 按钮 打开商店
   2. 搜索 ludusavi  然后安装 decky-ludusavi 
   3. 安装成功之后返回 decky-loader 的主页， 能看到插件 
   4. 点击插件后进入插件的设置页面， 打开 自动同步， 可能是 auto xxx的选项
   5. 至此 就可以 关闭steamdeck 游戏后自动备份存档到云端了。 
   6. 可能 decky-ludusavi 还需要启动一下游戏后才能正确设置， 这个好像是单游戏设置的。 

⚠️ 如果云端文件和本地的有冲突， 在backup 的时候 会**不上传**到云端

**现在可以在windows操作存档恢复了**

- 安装 https://github.com/mtkennerly/ludusavi 软件， 从 release 里面下载解压即可。 
- 然后打开软件， 设置一样的 云存档 或者单纯的仅备份+恢复， 用其他软件进行同步， 笔者是使用的owncloud 客户端自动把 nextcloud 的目录同步到本地， 然后 ludusavi 直接使用的。 
- 在 windows 上把 "Back up to" 和 "Restore from" 的目录都设置到 owncloud 的路径。  同时在owncloud 里面设置这个路径始终保存在设备上。
- 找到"Other" 页签 拉到最下面的 "Redirects"  添加一个 "Bidirectional" (双向) 的目录转换。  
  - 笔者写的是 `C:\Users\aincv` 到 `/home/deck/.local/share/Steam/steamapps/compatdata/1245620/pfx/drive_c/users/steamuser` 
    - 就是 "C:\Users\\[用户名]"
  - 然后勾选 "reverse sequence of redirects when restoring" 

至此， 笔者已经可以从 steamdeck 同步存档到windows 了，  再加上手动同步的部分，  也能实现从 windows 同步存档到 steamdeck 

理论上来说， 是能双向自动同步的， 但是后续内容， 笔者还在研究。 

*PS其他游戏可能需要添加新的Redirects*

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/steam-deck-%E5%AD%98%E6%A1%A3%E5%90%8C%E6%AD%A5windows%E6%8A%98%E8%85%BE%E7%9A%84%E8%AE%B0%E5%BD%95/  

