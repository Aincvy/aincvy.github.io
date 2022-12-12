# Mysql 时区的问题


在链接字符串尾部添加参数 `serverTimezone=Asia/Shanghai`  即可让 mysql 的链接使用上海的时区。 



### 其他说明

我刚开始尝试的是 `serverTimezone=GMT+8`  但是不知道为啥，没有生效。  目前考虑可能是 `+`的问题, 使用`GMT%2b8`   可能有效（*笔者没有尝试*）。  这是 **URLEncode** 之后的代码。



如果在服务器上使用 `yum/apt` 等包管理器进行安装， mysql应该会使用系统的时区， 这个时候得客户端一般是不需要指定时区的。  

我们这次是使用 `docker` 安装的， 默认可能是 `GMT+0` 的时区。

---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/mysql-%E6%97%B6%E5%8C%BA%E7%9A%84%E9%97%AE%E9%A2%98/  

