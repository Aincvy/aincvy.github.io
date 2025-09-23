# 让docker Compose 挂载一个smb远程目录


## 主要内容

下面看一个简短的示例，这个示例说明了如何挂载一个smb目录到docker 容器里面。 

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - UPLOAD_SMB:/usr/src/app/upload

volumes:
  UPLOAD_SMB:
    #driver: local
    driver_opts:
      type: cifs
      device: "//192.168.200.23/音频/immich"
      o: "username=yohello,password=******,vers=3.0"
```

详细解释： 
- device 写 smb 的服务地址以及目录位置。 如果使用主机名可能不行， 使用ip地址最保险
- o 是参数选项，  username 是用户名， password 是密码，  vers 是版本 


基本上这样就可以挂载这个目录到 docker 容器里面了。  

在挂载之前需要安装 `cifs-utils` , 如果没安装的话， 可以使用命令`sudo apt-get install cifs-utils` 进行安装。 

## 其他

需要注意一个问题是如果挂载失败了， 卷是不会随着`docker compose down` 而被删除的， 需要手动找到这个卷， 然后删除， 这样才会在 执行 `docker compose up -d` 的时候，使用新的参数进行重新创建。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E8%AE%A9docker-compose.yaml%E6%8C%82%E8%BD%BD%E4%B8%80%E4%B8%AAsmb%E8%BF%9C%E7%A8%8B%E7%9B%AE%E5%BD%95/  

