# 一次更新jellyfin折腾的记录


## 主要内容

因为笔者在手机上安装  Swiftfin IOS  打开链接服务器之后提示服务器版本过低， 所以笔者就尝试对服务器进行升级操作。 

笔者是使用 apt 添加了jellyfin 的源进行安装的， 所以更新的时候就直接使用命令 `sudo apt install --only-upgrade jellyfin jellyfin-server jellyfin-web`  进行了更新。 

apt 执行成功之后就安装了最新版本的jellyfin， 但是出现了数据库迁移失败的情况。   日志大概如下： 

```
Jellyfin Server has encountered an error and was not able to start.

05/14/2026 22:22:00 +08:00 - Error while starting server
05/14/2026 22:22:00 +08:00 - Migration "20250420193000_MigrateLibraryDbCompatibilityCheck" failed
05/14/2026 22:22:00 +08:00 - Error: "Your database does not meet the required standard. Only upgrades from server version 10.9.11 or above are supported. Please upgrade first to server version 10.10.7 before attempting to upgrade afterwards to 10.11"
05/14/2026 22:21:59 +08:00 - Migrate stage AppInitialisation.
05/14/2026 22:22:00 +08:00 - 20250420193000_MigrateLibraryDbCompatibilityCheck
05/14/2026 22:22:00 +08:00 - Attempt to rollback JellyfinDb.
05/14/2026 22:22:00 +08:00 - Attempt to rollback librarydb.
05/14/2026 22:22:00 +08:00 - Migration 20250420193000_MigrateLibraryDbCompatibilityCheck failed
05/14/2026 22:22:00 +08:00 - Error: Your database does not meet the required standard. Only upgrades from server version 10.9.11 or above are supported. Please upgrade first to server version 10.10.7 before attempting to upgrade afterwards to 10.11
05/14/2026 22:22:00 +08:00 - Perform migration 20250420193000_MigrateLibraryDbCompatibilityCheck
```

然后笔者又折腾了一下 去安装  10.10.7 ， 折腾了一大群， 最后的解决方法是在 10.10.7 上修改配置文件。 

安装特定版本的命令是： `sudo apt install jellyfin=10.10.7+deb11 jellyfin-server=10.10.7+deb11 jellyfin-web=10.10.7+deb11` 

操作流程是：

1. 关闭 jellyfin 
2. 找到 `/etc/jellyfin/system.xml`  将 `IsStartupWizardCompleted` 设置成 `false`
3. 删除 `/etc/jellyfin/migrations.xml`    不方向的话，改名也行
4. 启动 jellyfin,  这时会自动生成 `/etc/jellyfin/migrations.xml`
5. 关闭 jellyfin
6. 修改 `IsStartupWizardCompleted` 为 `true`
7. 启动 jellyfin   

现在应该就升级成功了， 随后 就可以继续更新到最新版本了。 

## 拓展阅读

- https://github.com/jellyfin/jellyfin/issues/12254#issuecomment-2224013816

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%B8%80%E6%AC%A1%E6%9B%B4%E6%96%B0jellyfin%E6%8A%98%E8%85%BE%E7%9A%84%E8%AE%B0%E5%BD%95/  

