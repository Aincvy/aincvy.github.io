# Windows实现wireguard自动重连


笔者家里得公网IP是非固定得， 所以使用DDNS 来实现域名得动态解析， 笔者使用wireguard来和其他电脑组成内网。  

这种操作下有一个致命得问题， 那就是DDNS变化之后， wireguard 会断网， 同时它不会自动重连。。 

所以这时候就需要有一个Windows 脚本来监控链接， 如果连不上得话， 就自动重连一下就可以了。 

```powershell
# PingMonitor.ps1
# Requires -Version 3.0
# 目标 IP 与要重启的服务名
$target      = &#39;192.168.1.1&#39;
$svcToRestart= &#39;WireGuardTunnel$home-wg&#39;
# 上次重启时间初始化为一个很久以前的时间
$lastRestart = Get-Date &#39;1970-01-01&#39;
$minInterval = [TimeSpan]::FromSeconds(30)

while ($true) {
    # 测试连通性
    $ok = Test-Connection -ComputerName $target -Count 1 -Quiet -TimeoutSeconds 5
    if (-not $ok) {
        $now = Get-Date
        if (($now - $lastRestart) -ge $minInterval) {
            # 重启服务
            try {
                Restart-Service -Name $svcToRestart -Force -ErrorAction Stop
                Write-Output &#34;$($now): 无法 ping 通 $target，已重启服务 $svcToRestart&#34;
            } catch {
                Write-Output &#34;$($now): 重启服务 $svcToRestart 失败：$_&#34;
            }
            $lastRestart = $now
        }
    }
    # 每 5 秒检测一次，可根据需要调整
    Start-Sleep -Seconds 5
}
```

解释说明：
- $target 是 家里得任意服务得IP地址， 只要 wireguard连接后是可以ping通得就可以， 也可以用wiregurad 服务器得地址
- $svcToRestart  是要重启得服务， 对于wireguard来说，重启服务就会产生重连操作。  它得服务名格式是： `WireGuardTunnel$[隧道名字]`
  - 更具体名字可以 按下 Windows 键， 输入 `服务`  打开服务列表， 然后找到类似 `WireGuardTunnel$home_server` 这种得名字， 就是要重启得服务名

把这个脚本保存一下， 比如到 `D:\Documents\PingMonitor.ps1`
 
现在可以打开一个powershell 窗口， 执行这个脚本了。    

上面得方法固然可以用， 但是每次开机都需要操作一遍， 万一不小心关了窗口也是问题， 所以可以考虑把这个脚本做成 服务

开源项目  https://github.com/winsw/winsw  可以将任意程序， 脚本封装成 Windows 服务。  从项目 Releases 里面找到 Pre-Release 3.x 得版本， 下载一下。 然后将文件重命名成 `winsw.exe`


```xml
&lt;service&gt;
  &lt;id&gt;WireGuardHomeTunnelMonitor&lt;/id&gt;
  &lt;name&gt;WireGuard Home Tunnel Monitor&lt;/name&gt;
  &lt;description&gt;监控 192.168.1.1，可用性检查失败则重启 WireGuardTunnel$home-wg&lt;/description&gt;
  &lt;executable&gt;C:\Program Files\PowerShell\7\pwsh.exe&lt;/executable&gt;
  &lt;arguments&gt;-NoProfile -ExecutionPolicy Bypass -File &#34;D:\Documents\PingMonitor.ps1&#34;&lt;/arguments&gt;
  &lt;startmode&gt;Automatic&lt;/startmode&gt;
  &lt;stoptimeout&gt;15sec&lt;/stoptimeout&gt;
  &lt;logpath&gt;D:\Documents\logs&lt;/logpath&gt;
  &lt;logmode&gt;rotate&lt;/logmode&gt;
&lt;/service&gt;
```

将上面得脚本保存一下， 比如到`D:\WireGuardHomeTunnelMonitor.xml` ， 
- 然后执行命令 `winsw install D:\WireGuardHomeTunnelMonitor.xml`  来安装服务
- 然后执行命令 `winsw start D:\WireGuardHomeTunnelMonitor.xml` 来启动服务 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/windows%E5%AE%9E%E7%8E%B0wireguard%E8%87%AA%E5%8A%A8%E9%87%8D%E8%BF%9E/  

