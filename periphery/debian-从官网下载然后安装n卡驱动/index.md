# Debian 从官网下载，然后安装N卡驱动


## 安装流程
- 从 https://www.nvidia.com/en-us/drivers/unix/linux-amd64-display-archive/ 上下载驱动， 注意不要下载 beta 版本的驱动。
- 下载之后， 使用 sftp, scp,ftp 等工具上传到服务器。   *任选其一即可*
- 安装 build-essential， libglvnd   
  - 比如 `sudo apt install build-essential libglvnd`
- 关闭桌面环境，  笔者是xfce,  使用命令： `sudo systemctl stop lightdm`
  - 如果是 GNOME 的话， 可能需要使用命令 `sudo systemctl stop gdm3`
  - 如果不能确定自己的桌面环境是哪个服务的话， 可以使用命令`systemctl | grep running`  寻找一个包含`dm` 的服务
- 执行命令： `sudo ./驱动文件`    比如： `sudo ./NVIDIA-Linux-x86_64-525.116.04.run`
  - 32位兼容性， 似乎很多人说 x-org 不要安装。
- 安装成功之后， 使用 `nvidia-smi`  命令查看信息
- `nvidia-smi -L`  查看GPU型号



## 拓展阅读
-  https://blog.csdn.net/lien0906/article/details/54312166
-  https://unix.stackexchange.com/questions/569514/how-to-disable-and-enable-gui-of-debian-10



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/debian-%E4%BB%8E%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E7%84%B6%E5%90%8E%E5%AE%89%E8%A3%85n%E5%8D%A1%E9%A9%B1%E5%8A%A8/  

