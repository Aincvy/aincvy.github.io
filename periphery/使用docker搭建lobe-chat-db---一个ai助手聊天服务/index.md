# 使用docker搭建lobe Chat - 本地的openai客户端


## 简述

项目主页：  https://github.com/lobehub/lobe-chat

这是一个使用API 调用多家 LLM服务商的本地聊天客户端。 
- 支持： OpenAI / Claude 3 / Gemini / Ollama / Azure / DeepSeek
- 支持文件上传，向量化，引用文件内容
- 网页爬虫插件， 时间助手插件， 股票走势插件等

lobe-chat 有2个版本， 一个是单用户版本， 使用一个固定code访问。 另外一个是 database版本， 允许多用户， 使用用户名密码注册登录。  虽然允许多用户， 但是好像无法限制用量。

单用户版本的部署非常简单， 而database版本的部署就有点困难了， 本文的内容是 database版本的部署。 

## 主要内容

官方文档可见： https://lobehub.com/zh/docs/self-hosting/server-database/docker-compose

本着不重复造轮子的理念， 本文将会描述碰到的一些问题， 以及如何解决。 

### logto 没有SSL证书的问题

着重说一下 logto， 这个应用要么使用localhost访问， 要么必须使用https访问， 如果lobe部署到非本机的其他机器，比如（家里的NAS服务器上）， 这就变成了一个很大的问题。 

如果有公网签发的SSL证书，可以直接使用， 但是如果没有的话， 可以考虑下面的方案： 
- 使用 ssh 的端口转发来模拟localhost 访问。
  - `ssh -L 3001:localhost:3001 -L 3002:localhost:3002 user@server`
  - 成功链接到服务器之后， 在本地访问 localhost:3001 / localhost:3002(这个是管理界面)  就可以访问logto 了
  - 笔者不确定这种方式是否需要每次使用lobe-chat的时候都需要端口转发。
- 使用自签名证书
  - 使用 https://github.com/FiloSottile/mkcert  制作自签名证书
  - 使用 https://github.com/pi-hole/pi-hole  做本地的DNS解析， 或者使用别的本地解析服务  
  - 在pi-hole上添加了解析之后， 使用 mkcert 创建证书。  
    - 示例命令： 
    - mkcert auth.logto.mylab
  - 使用 nginx 反代 logto 的服务
    - 将证书复制 nginx 能访问到的目录里面， 使用下面的nginx 配置来配置反代
  - 修改 docker-compose.yaml 文件
    - 修改 `lobe` 的环境变量： `- 'LOGTO_ISSUER=https://auth.logto.mylab/oidc'`
    - 将 mkcert 生成的 rootCA.pem 复制到 可访问的地方， 然后映射到容器里面
    - 设置node.js 的环境变量， 使CA证书生效，  node.js 有自己的信任机制， 它不信任linux系统的证书列表。 
  - 注意： 这里没有代理 管理页面的端口3002， 所以如果需要访问管理页面的话， 还是需要使用上面的那种端口转发的形式。 

**nginx配置参考**

```
    server {
        listen 80;
        server_name auth.logto.mylab;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl;
        server_name auth.logto.mylab; 
        ssl_certificate /root/cert/auth.logto.mylab.pem; 
        ssl_certificate_key /root/cert/auth.logto.mylab-key.pem; 
        ssl_session_cache builtin:1000 shared:SSL:10m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
        ssl_prefer_server_ciphers on;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_pass http://NAS的IP:3001; 
            proxy_read_timeout 90;
            proxy_redirect http://NAS的IP:3001 https://auth.logto.mylab; 
        }
    }

```

**docker-compose修改的示例**

```yaml
  lobe:
    image: lobehub/lobe-chat-database
    container_name: lobe-database
    # 其他的选项省略了
    volumes:
      - './root_certs/rootCA.pem:/usr/local/share/ca-certificates/mkcertRootCA.crt'
    environment:
      - 'LOGTO_ISSUER=https://auth.logto.mylab/oidc'
      - 'S3_ENDPOINT=http://NAS的IP:${MINIO_PORT}'
      - 'S3_PUBLIC_DOMAIN=http://NAS的IP:${MINIO_PORT}'
      - 'NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/mkcertRootCA.crt'
      # 其他的省略了
```

配置到 Nginx上的是 `auth.logto.mylab.pem` 和 `auth.logto.mylab-key.pem` ，配置到 lobe-chat里面的是`rootCA.pem` ，总共是3个不同的文件， rootCA.pem 是根证书。

### 使用代理访问openai等LLM提供商

在 .env 里面添加 
```
PROXY_URL=http://地址:端口
```

即可


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAlobe-chat-db---%E4%B8%80%E4%B8%AAai%E5%8A%A9%E6%89%8B%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1/  

