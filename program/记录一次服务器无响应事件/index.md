# 记录一次游戏服务器无响应的事件


服务器的类型是 游戏服务器， 使用TCP连接来提供服务。

服务器的现象是：
- 使用客户端进行发包的时候： socket 能建立链接， 也能发送消息过去， 但是收不到响应， 同时没有超时处理。 
- 查看了一下 jstack：  没有线程被卡住不动。
- 查看了一下 堆内存：  老生代占用率99.2%  新生代40%  看起来有点问题， 但是不大。 

重启之后 服务恢复了正常, 通过对比发现是 [OrderedThreadPoolExecutor](https://javadoc.io/static/org.apache.mina/mina-core/2.0.7/org/apache/mina/filter/executor/OrderedThreadPoolExecutor.html) 所应对的线程池消失不见了。 
- 参考源码：  [OrderedThreadPoolExecutor.java](https://github.com/eclipse-archived/neoscada/blob/master/external/org.apache.mina.core/src/org/apache/mina/filter/executor/OrderedThreadPoolExecutor.java)
- 这个是mina 调度io event 需要的线程池。
- 笔者猜测：当这个线程池消失了， 我们自己的Handler 代码就不会被调用， 所以就无法处理客户端传来的消息。 
- 至于这个线程池为什么会挂掉， 笔者倒没有查出原因。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%97%A0%E5%93%8D%E5%BA%94%E4%BA%8B%E4%BB%B6/  

