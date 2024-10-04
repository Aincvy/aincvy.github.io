# 在linux里面安装homebrew


homebrew 是一个包管理器， 可以用这个工具安装很多软件包，开发库等。  这个工具在macos 上是比较出名的。

注意，在linux上， 应该考虑把 homebrew 安装到目录`/home/linuxbrew/.linuxbrew` 里面， 如果不想安装到此目录， 可以先建立一个软连接， 然后再执行安装脚本。  

下面是安装流程： 
- 建立软连接  *可选*
  - `sudo ln -s [目录] /home/linuxbrew/.linuxbrew` 
  - 如果使用软连接的话， 还需要注意文件夹以及文件权限的问题。  *homebrew不能使用root权限进行安装*
- 安装： `ALL_PROXY=[代理服务器地址] /bin/bash -c &#34;$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)&#34;`
  - 如果没有代理服务器， 或者不需要的话， 可以执行： `/bin/bash -c &#34;$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)&#34;`
- 成功安装之后， 执行命令：  `(echo; echo &#39;eval &#34;$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)&#34;&#39;) &gt;&gt; ~/.bashrc` 
- 打开一个新的终端， 输入命令`brew --version` ， 如果能看到版本号打印就说明安装成功了。


有一点需要注意的是， 如果系统的 glibc 版本比较低， 使用 brew 安装软件的时候， 会自动下载一个新版本的glibc 。 如果使用brew 安装一些 cpp开发库的话， 就需要考虑同时使用 brew 安装 gcc/g&#43;&#43; , 然后使用 brew 安装的 gcc来编译项目。  否则 可能会碰到大量的链接错误。  笔者同时尝试了clang, 但是没有成功。 此外，如果使用新版本的glibc 代替系统的，常规程序（ls,mv,cat等） 都会报段错误。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%9C%A8linux%E9%87%8C%E9%9D%A2%E5%AE%89%E8%A3%85homebrew/  

