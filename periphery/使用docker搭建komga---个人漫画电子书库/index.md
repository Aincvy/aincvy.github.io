# 使用docker搭建komga   个人漫画，电子书库


## 简述

项目主页：  https://github.com/gotson/komga

这是一个 漫画，电子书得媒体服务器。 
- 支持 CBZ, CBR, PDF and EPUB 文件 
- 提供 OPDS v1, v2 支持
- 提供 web 阅读器， 虽然不太好用。 
- 支持 mihon, koreader 等安卓阅读器
- ios 也有相应得阅读器软件。 

## 主要内容

### 安装

可见官方文档：  https://komga.org/docs/installation/docker

笔者使用得 docker-compose.yaml 为： 
```yaml
version: &#39;3.3&#39;
services:
  komga:
    image: gotson/komga
    container_name: komga
    volumes:
      - ./data/config:/config
      - ./data/content:/data
    ports:
      - 25600:25600
    user: &#34;1000:1000&#34;
    environment:
      - TZ=Asia/Shanghai
    restart: unless-stopped

```

*如果当前用户不是root,则需要先创建./data/config和./data/content目录。*

使用 `docker compose up -d` 命令 即可启动容器。 

将漫画， 电子书文件放入 ./data/content 目录即可。  
参考示例： 哆啦A梦得全部漫画可以放到  `./data/content/哆啦A梦/`  里面， 这样会自动在komga 里面建立一个系列。    
*需要在komga得web界面里面添加Library，路径填写`/data`即可。*

### nginx 反代

参考 nginx 反代配置 ： 

```conf
# Upstream where your authentik server is hosted.
upstream komgaserver {
    server your_local_ip:25600;    # your_local_ip 使用你得 ip
    # Improve performance by keeping some connections alive.
    keepalive 10;
}


server {
    # HTTP server config
    listen 81;
    server_name your_server_name;   # your_server_name 使用你得域名
    # 301 redirect to HTTPS
    return 301 https://$host$request_uri;
}
server {
    # HTTPS server config
    listen 443 ssl http2;
    server_name your_server_name;    # your_server_name 使用你得域名

    # TLS certificates
    ssl_certificate     /root/cert/acme/fullchain.pem;   #  你得 ssl 证书
    ssl_certificate_key /root/cert/acme/your_pem_file;    #  你得 ssl 证书
    add_header Strict-Transport-Security &#34;max-age=63072000&#34; always;

    location / {
        proxy_pass http://komgaserver;
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Proto $scheme;
#        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#       proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade_keepalive;
    }
}
```

其他得一些问题： 
- 非标准https 端口有问题  解决方法：  https://github.com/gotson/komga/issues/1735#issuecomment-2409000639 


## 缺点

几个阅读器， 各有各得问题。。
- mihon 不能进行页级得同步， 只能按本同步进度，除此之外， 感觉体验是最好得。   *https://github.com/mihonapp/mihon/issues/236 提供了一个可以实现页级同步得pr，但是目前并没有合并进去。*
- koreader 能按页同步， 但是需要先把文件下载到本地， 如果使用 opds 进行页面流式传输是没有同步得， opds 得阅读界面和本地文件得阅读界面不一样，很垃圾。  
- web 阅读器不能记住设置， 每次都要重新设置一下翻页模式。
- 电子书支持 epub 和pdf 文件格式， 但是在web阅读器里面， pdf 会被渲染成图片， 即使pdf原来得文件是文字也会。  epub得支持则还不错。



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAkomga---%E4%B8%AA%E4%BA%BA%E6%BC%AB%E7%94%BB%E7%94%B5%E5%AD%90%E4%B9%A6%E5%BA%93/  

