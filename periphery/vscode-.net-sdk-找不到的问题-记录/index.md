# Vscode .Net Sdk 找不到的问题 记录

是windows 下碰到的问题。 

提示错误是：  `vscode The .NET Core SDK cannot be located: A valid dotnet installation could not be found`

vs code 里面已经设置了正确的 dotnetPath  和 sdkPath ， 但是还是提示这个。
使用 cmd.exe 里面输入 `dotnet --version` 是看得到输出信息的。

参考的设置内容是： 
```json
{
    "omnisharp.dotnetPath": "C:\\Program Files\\dotnet",
    "omnisharp.sdkPath": "C:\\Program Files\\dotnet\\sdk\\7.0.102"
}
```

**解决方法。**   
关掉vs code ， 重新打开就好了。   
但是如果使用的是 unity 双击脚本打开的， 则需要重新启动 unity.   
也可以考虑不关闭unity, 手动关闭vs code 然后从开始菜单或者桌面图标打开 vs code。  

如果只安装了 .net core，  在加载项目的时候 可能会提示 .net runtime 找不到。   
这个时候根据提示的地址下载 .net 开发人员工具包就可以了。 

具体错误可以看 vs code 里面的 output 的 OmniSharp Log 一栏 。  
这里的日志应该还会提示 .net 开发人员工具包的版本， 安装提示的版本可能更好一些。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/vscode-.net-sdk-%E6%89%BE%E4%B8%8D%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98-%E8%AE%B0%E5%BD%95/  

