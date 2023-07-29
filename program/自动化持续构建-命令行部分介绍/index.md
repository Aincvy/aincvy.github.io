# 自动化持续构建 命令行部分介绍


在开发项目的时候可能会使用到持续集成工具， 比如 [jenkins](https://www.jenkins.io/) 之类的， 在使用的时候， 可能会需要写一些shell脚本来辅助完成工作， 本文就介绍一下相关的shell脚本部分。

由于笔者是一个后端程序， 所以就只介绍后端程序的情况了。 



## 详细内容

笔者基本上是使用的JAVA语言， 大多是采用`maven`做包管理器，其他语言大多应该是类似的。

脚本应该具有下面几个功能

- 编译，打包
- 打`zip/tar.gz`包    *可选步骤*
- 发布到目标服务器   *可选步骤*
- 重启程序



### 编译，打包 以及打压缩包。

使用命令 `mvn clean && mvn install` 基本上就可以完成编译打包的命令。  *maven相关的配置文件这里就不描述了。*

使用cpp的话， 如果使用cmake的话， 就使用下面的命令。

```shell
mkdir build && cd build
cmake ..
make
```

在执行了编译命令之后， 在shell 里面可以使用`$?` 变量获取上一个命令的结束状态码。 状态码基本上是`0`表示执行成功，其他是失败，状态码可以用于做失败处理。  *在fish里面使用`$status`这个变量，fish这个shell 很多地方和其他的shell都不一样。。:joy:*

打压缩包的话， 可以考虑下面的命令。

- `zip -q -r server.zip /path/to/server`
- `zip`对应的解压命令是 `unzip`

其他压缩命令这里暂时不介绍了。  如果不想压缩的话，直接使用文件夹应该也是OK的。



### 发布程序到目标服务器

这里有下面几种情况。 

- 发布到本机    
- Linux主机/Windows主机发布到另外一台Linux主机
- Linux主机/Windows主机发布到另外一台Windows主机    *暂不讨论*



#### 发布到本机

这个其实必要性不是很大， 但是也可以做。 

主要思路是下面几个。 

1. 杀掉之前的进程
2. 复制文件
3. 开启新的进程。

**Windows**

在Windows下， 杀掉进程可以使用使用命令`Taskkill /PID [pid] /F` 或者`Taskkill /IM [exeName] /F`

这里的一个可选思路是这样的，在程序启动的时候把当前的pid写入到一个文本文件里面。 在需要杀死旧进程的时候读取文本文件获得pid，然后执行taskkill命令。

下面是一个示例     *笔者并没有实际测试过正确性。*

```shell
set /p pid=<pid.txt
Taskkill /PID %pid% /F
```

接下来就是复制文件， 使用命令 `copy /f` 即可， 如果文件过大也可以考虑使用`xcopy`命令。

启动的话，就很简单了， 使用命令 `java -jar [jarFile]` 即可。  在开发阶段可以加启动参数， 也可以不加把， 笔者觉得。

**Linux**

在Linux下，杀掉进程可以使用使用命令`kill -9 [pid]` 

同样在Linux下也可以采取写入pid到文件的方式， 同时这里也提供另外一种方式。那就是使用`ps` 命令， 这个命令加上参数之后可以获取进程启动附带的参数， 所以使用其他命令组合在一起就可以完成杀死旧进程的目标了。

这里是一个示例： `ps ax | grep -E 'keyword1|keyword2' | grep -v grep | awk '{print $1}' | xargs kill`

将`keyword1`和`keyword2` 替换成自己要启动的jar文件的名称即可。   *多个文件使用 | 符号分割。*

复制文件使用 `cp -f [src] [dst]` 即可。 如果是需要复制文件夹，则使用`cp -fR [src] [dst]` 即可。

启动的话， 没有啥区别。

#### 发布到远程linux服务器

这个操作和发布到Linux本机是类似的， 只不过要把3个部分拆分成3个脚本。

**Linux** 

现在介绍几个Linux工具：

- sshpass   可以代替用户输入ssh链接的密码的命令
- rsync   文件同步工具
- ssh   ssh链接或者执行ssh命令的工具。

下面给一段示例。

```shell
# 使用环境变量的方式给予 sshpass 相关的密码
export SSHPASS="$pass"
# ssh 命令的一些参数
SSH_FLAGS=(-q -o StrictHostKeyChecking=no -p ${s_port})
# 在目标服务器上创建文件夹
$(sshpass -e ssh "${SSH_FLAGS[@]}" ${user}@${s_addr} "mkdir -p ${real_remote_folder}")

# 杀掉目标服务器上的进程
sshpass -e ssh "${SSH_FLAGS[@]}" ${user}@${s_addr} "bash ${remote_target_folder}/control_scripts/killservers.sh"

# 传送文件
FLAGS=(--include="*.jar" --include="lib")
sshpass -e rsync -e "ssh -o StrictHostKeyChecking=no -p ${s_port}" -avz  "${FLAGS[@]}" --exclude="*" ./ ${user}@${s_addr}:${real_remote_folder}
```

下面对上述脚本的变量做一些解释。

- s_port    目标服务器的ssh链接端口
- s_addr   连接目标服务器使用的地址
- user      连接目标服务器使用的用户名
- pass      连接目标服务器使用的密码
- real_remote_folder    目标服务器的内部文件路径， 用于放置传送过去的文件
- remote_target_folder     也是目标服务器上的一个目录

这个脚本是在本地执行的，它大概实现下面3个部分的功能。

1. 杀掉目标服务器上的进程
2. 传输文件
3. 启动目标服务器上的进程。 



**Windows**

Windows下使用这个工具：  https://www.putty.org/

思路和Linux下没有变化，主要是具体实现细节上的差异。 

`plink` 是一个`putty`包里面的工具，下面看一段脚本示例。

```powershell
# 本脚本是纯手写， 没有经过测试

plink -batch -ssh %ssh_host% -l %ssh_user% -pw %ssh_pw% -P %ssh_port% echo hello, world

pscp -q -batch -pw %ssh_pw% -P %ssh_port% c:\local\folder %ssh_user%@%ssh_host%:/path/to/folder/
```

下面对上述脚本的变量做一些解释。

- ssh_port    目标服务器的ssh链接端口
- ssh_host   连接目标服务器使用的地址
- ssh_user   连接目标服务器使用的用户名
- ssh_pw    连接目标服务器使用的密码

更多关于参数的解释， 笔者附在了下面的拓展阅读部分， 有兴趣的读者可以自行查阅。

在执行文件的时候， 可以将最后的`echo hello, world` 替换成`bash [filepath]`  这里的`filepath`是目标服务器上的路径。 



#### Jenkins相关的内容

如果使用jenkins 调用我们自己的启动脚本的话，可能会发现我们的程序被干掉了， 这时可以考虑在脚本里面添加这两行。

```shell
export BUILD_ID=dontKillMe
export JENKINS_NODE_COOKIE=dontKillMe
```



## 结尾

下面是一些吐槽部分。 

笔者感觉上前端更需要持续集成， 至少需要大于后端。  在笔者经历过的游戏公司中，前端每次打包都至少要折腾到半夜12点， 甚至凌晨2点至更晚。    *似乎现在行业上的人都接受了这个现状，也不求改变。 :joy:*

很多问题都是在打包当天发现，然后要求处理的， 这让笔者觉得很不可思议。 

从关心人文的角度上看： 首先人的精力有限，越到很晚的时候越没有精力。 其次， 除非经常熬夜的人， 不然的话那个时候人很可能会感到不舒服。 

从技术角度上看： 有不少情况是编辑器上看是正常的，但是手机上看效果不好（手游）。 但是在开发阶段测试的时候， 很多团队都是仅使用编辑器测试。 然后一大堆手机相关的问题在打包当天爆发。  如果能够提前打包的话，就可以减少一定的手机上相关的问题。 打包是一个比较缓慢和浪费时间的过程， 如果有脚本或者相关的东西辅助的话， 也可以节省一定的时间。

上面是笔者推荐使用自动化构建的理由， 但是笔者没有实际操作过前端的自动化构建， 所以并不是很清楚有没有实施难度。

*等待后面笔者有机会尝试了之后*， 会更新这篇博文。 

### 拓展阅读

http://c.biancheng.net/cpp/view/2739.html

https://www.runoob.com/linux/linux-comm-zip.html

https://www.runoob.com/linux/linux-comm-unzip.html

https://the.earth.li/~sgtatham/putty/0.60/htmldoc/Chapter5.html

https://the.earth.li/~sgtatham/putty/0.70/htmldoc/Chapter7.html#plink


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E8%87%AA%E5%8A%A8%E5%8C%96%E6%8C%81%E7%BB%AD%E6%9E%84%E5%BB%BA-%E5%91%BD%E4%BB%A4%E8%A1%8C%E9%83%A8%E5%88%86%E4%BB%8B%E7%BB%8D/  

