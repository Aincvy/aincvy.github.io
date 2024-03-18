# Protobuf number overflow 的解决方法



基本信息:

- java 服务端
- 网络框架用的 netty 
- Unity的客户端，用的 tcpClient. 
- 通信协议选择的 protobuf. 
- java端采用的是谷歌提供的解析器， 而客户端则采用的protobuf-net  
- 我在设计中 使用了一个叫 GameMsg 的协议来封装所有协议的内容。



在本来中设计中 使用了一个 `string paramStr = 1; ` 的字段来存放协议的具体内容。 然后发现 在 unity 端中 如果协议的属性带有负数， 则报出一个 `number overflow `的异常。 把 string 改成 bytes 就好了。

如果是 java 和java 这样通信， string 是允许的。 但是 unity 不行。



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/protobuf-net-parse-from-string-%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/  

