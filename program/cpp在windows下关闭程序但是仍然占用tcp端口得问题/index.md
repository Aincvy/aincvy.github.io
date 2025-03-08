# Cpp在windows下关闭程序但是仍然占用tcp端口得问题


笔者碰到得状况是：
- 使用调试启动程序， 监听TCP端口
- 野指针调用了， 或者 断言 失败了， 程序进入断点模式。 
- 手动关闭调试模式，终止程序。
- 调整代码后再次启动程序， 无法监听TCP端口， 程序自动退出

笔者尝试使用下面得方式处理问题，但是失败了：
- 命令 `netstat -ano | findstr &#34;31551&#34;`  给出得结果是端口确实被占用了，  但是给出得PID 在任务管理器里面根本找不到。
- 使用 `taskkill /F /PID $PID`  会提示无法完成该操作。

经过一番仔细探查， 笔者终于找到了原因： 笔者使用 system() 函数打开了 notepad&#43;&#43;.exe ， 此时notepad&#43;&#43;.exe 会成为 当前进程得一个子进程， 如果notepad&#43;&#43;.exe 没有关闭得话，父进程得 TCP端口资源就不会被释放。 

当笔者手动关闭 notepad&#43;&#43;.exe 之后， 父进程自动消失了， TCP端口也自动释放了。 

随后笔者使用 `system(&#34;start notepad&#43;&#43;.exe xxx_file&#34;)` 得方式来启动 notepad&#43;&#43;.exe 就避免了这个问题。


参考阅读： 
- https://serverfault.com/a/273727
- https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/cpp%E5%9C%A8windows%E4%B8%8B%E5%85%B3%E9%97%AD%E7%A8%8B%E5%BA%8F%E4%BD%86%E6%98%AF%E4%BB%8D%E7%84%B6%E5%8D%A0%E7%94%A8tcp%E7%AB%AF%E5%8F%A3%E5%BE%97%E9%97%AE%E9%A2%98/  

