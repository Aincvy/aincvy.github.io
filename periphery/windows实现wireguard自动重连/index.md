# Windows实现wireguard自动重连


笔者家里得公网IP是非固定得， 所以使用DDNS 来实现域名得动态解析， 笔者使用wireguard来和其他电脑组成内网。  

这种操作下有一个致命得问题， 那就是DDNS变化之后， wireguard 会断网， 同时它不会自动重连。。 

所以这时候就需要有一个Windows 脚本来监控链接， 如果网络断开了得话， 就自动重连一下。 

```powershell
# PingMonitor.ps1
# Requires -Version 3.0
# 目标 IP 与要重启的服务名
$target      = '192.168.1.1'
$svcToRestart= 'WireGuardTunnel$home-wg'
# 上次重启时间初始化为一个很久以前的时间
$lastRestart = Get-Date '1970-01-01'
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
                Write-Output "$($now): 无法 ping 通 $target，已重启服务 $svcToRestart"
            } catch {
                Write-Output "$($now): 重启服务 $svcToRestart 失败：$_"
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
  - 更具体名字可以这样找到： 按下 Windows 键， 输入 `服务`  打开服务列表， 然后找到类似 `WireGuardTunnel$home-wg` 这种得名字， 就是要重启得服务名

把这个脚本保存一下， 比如保存到 `D:\Documents\PingMonitor.ps1`
 
现在可以打开一个powershell 窗口， 执行这个脚本了。    

手动执行脚本得方法固然可以用， 但是每次开机都需要操作一遍， 万一不小心关了窗口也是问题， 所以可以考虑把这个脚本做成一个服务。

开源项目  https://github.com/winsw/winsw  可以将任意程序， 脚本封装成 Windows 服务。  从项目 Releases 里面找到 Pre-Release 3.x 得版本， 下载一下。 然后将文件重命名成 `winsw.exe`


```xml
<service>
  <id>WireGuardHomeTunnelMonitor</id>
  <name>WireGuard Home Tunnel Monitor</name>
  <description>监控 192.168.1.1，可用性检查失败则重启 WireGuardTunnel$home-wg</description>
  <executable>C:\Program Files\PowerShell\7\pwsh.exe</executable>
  <arguments>-NoProfile -ExecutionPolicy Bypass -File "D:\Documents\PingMonitor.ps1"</arguments>
  <startmode>Automatic</startmode>
  <stoptimeout>15sec</stoptimeout>
  <logpath>D:\Documents\logs</logpath>
  <logmode>rotate</logmode>
</service>
```

将上面得脚本保存一下， 比如到`D:\WireGuardHomeTunnelMonitor.xml` ， 
- 然后执行命令 `winsw install D:\WireGuardHomeTunnelMonitor.xml`  来安装服务
- 然后执行命令 `winsw start D:\WireGuardHomeTunnelMonitor.xml` 来启动服务 

如果没安装过 powershell 7， 则可以从 https://github.com/PowerShell/PowerShell 这里下载。

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/windows%E5%AE%9E%E7%8E%B0wireguard%E8%87%AA%E5%8A%A8%E9%87%8D%E8%BF%9E/  

