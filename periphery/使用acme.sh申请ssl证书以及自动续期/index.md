# 使用acme.sh申请ssl证书，以及自动续期


acme.sh 有一个docker 镜像，直接拉取镜像，然后部署即可。 

下面看一下 acme.sh 的 docker-compose 配置文件

```yaml
services:
  acme-sh:
    image: neilpang/acme.sh
    container_name: acme.sh
    volumes:
      - ./out:/acme.sh
      - /home/debian/docker-compose/nginx/data/mkcerts/acme:/nginx-data
      - /var/run/docker.sock:/var/run/docker.sock
    #network_mode: host
    command: daemon
    stdin_open: true
    tty: true
    restart: always
    environment:
      - Ali_Key=<YOUR_ALIYUN_KEY>
      - Ali_Secret=<YOUR_ALIYUN_SECRET>
      - DEPLOY_DOCKER_CONTAINER_LABEL=sh.acme.autoload.domain=aincvy.com
      - DEPLOY_DOCKER_CONTAINER_RELOAD_CMD=nginx -s reload
```

- 笔者是使用 阿里云DNS API 去认证域名的
  - Ali_Key  填写阿里云申请到的KEY
  - Ali_Secret  填写阿里云申请到的 secret
- 支持的全部DNS API 列表可以在这里找到：  https://github.com/acmesh-official/acme.sh/wiki/dnsapi

下面是 nginx 的 docker-compose 配置文件
```yaml
version: '3.1'

services:
    web:
      image: 'nginx:1.25'
      restart: always
      container_name: nginx
      environment:
        TZ: Asia/Shanghai
      ports:
        - '80:80'
        - '443:443'
      volumes:
        - './data/etc:/etc/nginx'
        - './data/html:/usr/share/nginx/html'
        - './data/mkcerts:/root/cert'
        - './data/root-certs:/root/.local/share/mkcert'
      labels:
        - sh.acme.autoload.domain=aincvy.com
```

nginx 的 labels 和 acme.sh 中的 `DEPLOY_DOCKER_CONTAINER_LABEL` 标签的值保持了一致。 

还需要添加 docker hook 才能实现 证书renew之后自动重载证书。 命令为： `docker exec acme.sh --deploy -d aincvy.com --deploy-hook docker` 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%BF%E7%94%A8acme.sh%E7%94%B3%E8%AF%B7ssl%E8%AF%81%E4%B9%A6%E4%BB%A5%E5%8F%8A%E8%87%AA%E5%8A%A8%E7%BB%AD%E6%9C%9F/  

