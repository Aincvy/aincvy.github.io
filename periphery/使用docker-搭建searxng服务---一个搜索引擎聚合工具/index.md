# 使用docker 搭建searxng服务 - 一个搜索引擎聚合工具


- github: https://github.com/searxng/searxng
- 容器的github地址： https://github.com/searxng/searxng-docker
- 一些不完全的文档：  https://docs.searxng.org/admin/index.html

## 主要内容

### 安装

使用下面的步骤来安装， *翻译自 searxng-docker 的仓库*： 
- 安装docker
- 从github上把仓库 克隆一下
- ```shell
  cd somewhere
  git clone https://github.com/searxng/searxng-docker.git
  cd searxng-docker
<!-- - ```  -->
- 修改 `.env`文件， 将SEARXNG_HOSTNAME 修改成主机名， 如果是自用的话或者是使用HTTP的话， 就不需要设置 LETSENCRYPT_EMAIL
- 使用命令 `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml` 生成 密钥
- 使用`chmod -R 777 searxng` 修改文件夹的权限；  这一步是可选的， 如果启动容器之后发现没权限， 再执行这一步也可以。
- 修改 `searxng/settings.yml` 配置文件，  全部配置项可见： https://github.com/searxng/searxng/blob/master/searx/settings.yml
- 当读者需要使用代理服务器的时候， 可以按照如下方式设置： 
- ```yaml
  redis:
    url: redis://redis:6379/0

  # 添加下面的部分
  outgoing:
    request_timeout: 6
    proxies:
      all://:
        - http://hostname_or_ip:port

<!-- ``` -->
- 如果不需要使用 LetsEncrypt  服务的话， 就把 `docker-compose.yaml` 文件里面的 caddy 部分都删除了。 
- 修改 docker compose 配置文件， 将 `- "127.0.0.1:8080:8080"`  修改成 `- "8080:8080"`， 除非读者是在本机进行服务搭建的。
- 使用 `docker compose up` 运行容器，随后看看有没有问题。 如果有Timeout 的报错， 有时候也是正常的。  但是如果提示 `Permission Denied`  之类的， 则是启动失败了。 
- 配置 nginx 反代
```conf

    server {
        listen 80;
        listen [::]:80;
        server_name searxng.mylab;

        rewrite ^(.*)$ https://${server_name}$1 permanent;
    }

    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        # listen 80;
        # listen [::]:80;

        server_name searxng.mylab;

        ssl_certificate /root/cert/searxng.mylab.pem;
        ssl_certificate_key /root/cert/searxng.mylab-key.pem;

        # include /etc/nginx/options-ssl-nginx.conf;

        add_header          X-Xss-Protection            "1; mode=block" always;
        add_header          X-Content-Type-Options      "nosniff" always;
        add_header          Strict-Transport-Security   "max-age=15552000; preload" always;
        add_header          X-Frame-Options             "SAMEORIGIN" always;
        add_header          'Referrer-Policy'           'origin-when-cross-origin';
        add_header          X-XSS-Protection            "1; mode=block" always;

        proxy_hide_header   X-Powered-By;
        proxy_buffering     off;

        # access_log  /var/log/nginx/example.com.access.log;
        # error_log   /var/log/nginx/example.com.error.log  warn;

        location / {
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   Host $host;
            proxy_pass         http://192.168.200.101:18090/;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection "upgrade";
        }
    }
```
- nginx 配置文件说明： 
  - 80 部分的配置 只是单纯的将 http 请求变成 https请求而已
  - server_name 后面可以跟域名， 或者IP地址
  - ssl 证书的话， 笔者是使用了一个叫做 mkcert 的工具生成的 本地证书。   如果不使用ssl的话，就在第二段里面监听80端口，将443端口，ssl证书部分的配置都注释即可。
  - proxy_pass 代理的是searxng的端口， 如果是在本地上使用，则填写 `http://127.0.0.1:8080/` 或者填写局域网的地址。 
- 打开浏览器， 访问 nginx 代理后的地址， 比如笔者访问的是 `http://searxng.mylab`
- 使用 curl 访问 searxng的地址或者是 nginx的代理地址，可能会得到 `Too Many Requests` 的返回， 但是不影响浏览器访问。 
- 如果浏览器使用 没什么问题，则可以将容器关掉，然后使用 `docker compose up -d` 来让容器在后台运行。


### 使用体验

感觉速度比直接使用其他引擎会慢很多， 质量上暂时看不出太多区别。 

使用搜索文件功能的时候可以获取到文件磁链， 但是好像只能搜索英文， 中文给的结果都很奇怪。 



## 其他

笔者翻了一些issue, 笔者发现 作者对帮助别人解决和排查问题不是很上心， 所以建议读者采取能用就用，不能用就换的心态来体验这个工具。   

当然，从作者的角度上来说，他完全没有义务来帮助我们解决问题。   但是从我们自身的角度来说， 我们也没必要浪费这个时间。

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%BF%E7%94%A8docker-%E6%90%AD%E5%BB%BAsearxng%E6%9C%8D%E5%8A%A1---%E4%B8%80%E4%B8%AA%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E%E8%81%9A%E5%90%88%E5%B7%A5%E5%85%B7/  

