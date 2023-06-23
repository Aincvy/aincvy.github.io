#  Code Server + Jdk17 的docker 镜像


## 基本内容

Dockerfile 示例： 

```Dockerfile
FROM codercom/code-server:latest

# from https://adoptium.net/zh-CN/temurin/releases/
# version 17.0.7_7
ADD OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz /opt/

# from https://maven.apache.org/download.cgi
# version 3.9.2
ADD apache-maven-3.9.2-bin.tar.gz /opt/


ENV JAVA_HOME /opt/jdk-17.0.7+7
ENV MAVEN_HOME /opt/apache-maven-3.9.2
ENV PATH="${PATH}:${JAVA_HOME}/bin:${MAVEN_HOME}/bin"

EXPOSE 8080/tcp
USER 1000

ENV USER=coder
WORKDIR /home/coder
ENTRYPOINT ["/usr/bin/entrypoint.sh", "--bind-addr", "0.0.0.0:8080", "."]
```

先从 https://adoptium.net/zh-CN/temurin/releases/   下载jdk文件   
然后从 https://maven.apache.org/download.cgi  下载 maven 文件。   
把下载好的文件和Dockerfile 放到同一个目录。之后执行命令：  `sudo docker build -t code-server-java:v0.2 .`  
成功之后就可以使用标签 `code-server-java:v0.2` 创建容器或者使用到docker compose 里面了。 

把容器的 8080端口暴露出去就可以从主机访问code server 了。  
当前版本的code server 并没有安装任何一个插件， 需要登录进去之后自行安装插件。  *根据网络环境不同， 安装插件的耗时也不一样。*

成功安装了maven for java 插件之后， 需要设置一下下面的选项， 否则会自动下载一个 maven 3.6.5
- Maven Executable Path: /opt/apache-maven-3.9.2/bin/mvn

`JAVA_HOME` 和 `MAVEN_HOME` 这两个环境变量可以在终端中查看和使用。 但是 code server 的终端并没有应用 `PATH` 变量。   
执行的时候需要这样： `${JAVA_HOME}/bin/java --version` ， 稍微有点麻烦， 不过可以写一个脚本来自动处理。   
使用 `docker exec -it [container_id] /bin/bash` 的方式获取的shell, 是应用了`PATH` 变量的。 

笔者把自己打包的镜像上传到了 [docker hub](https://hub.docker.com/r/aincvy/code-server-jdk17)， 有需要的读者可以直接使用。

### 发布镜像到 docker hub

- `sudo docker login` 登录账户
- `sudo docker images`  查看 image id
- `sudo docker tag imageId username/repo:tag`   
- `sudo docker push username/repo:tag`    推送到 docker hub 

*私有的 Registry 的流程应该也类似。*


### 镜像更新
在创建了容器之后， 你可能会安装 vscode 的java相关的插件， 之后又下载了很多依赖包。  
此时， 你可能会想， 如果基于此再做一个镜像的话， 新的容器就不用再下载这么多依赖了。   
那么， 你可以使用这个命令：  `sudo docker commit container_id imagename` *https://stackoverflow.com/a/52006170/11226492*


### 使用 docker compose 运行java 程序
```yaml
version: '3.1'

services:
    myApp:
      image: 'eclipse-temurin:17-jre-alpine'
      restart: always
      container_name: myApp
      working_dir: /home
      environment:
        TZ: Asia/Shanghai
      volumes:
        - './data:/home'
      command: java -jar my-app.jar
```
目录结构如下： 
- docker-compose.yml
- data
  - my-app.jar
  - xxx
  - xxx

这样不需要安装java 到本地就可以在容器里面运行java程序了。 


## 其他


拓展阅读
- [How to push image to docker hub repo](https://stackoverflow.com/a/55965206/11226492)

---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/-code-server-+-jdk17-%E7%9A%84docker-%E9%95%9C%E5%83%8F/  

