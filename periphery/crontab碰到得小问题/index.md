# Crontab碰到得小问题


笔者碰到得问题有两个：

- 程序好像没执行
- 重定向输出到了文件，但是却没有看到文件。



笔者得有问题得时候，定时任务大概是这样得：

```cron
# move film on 10AM at every saturday
0 10 * * 6  /Users/alias/Projects/self-use-scripts/trello/shMoveFilm.sh >/Users/alias/temp/log/moveFilm.log 2>&1

# move blog
35 2 * * 1  /Users/alias/Projects/self-use-scripts/trello/shMoveBlog.sh >/Users/alias/temp/log/moveBlog.log 2>&1

# clean blog
0 2 * * 3  /Users/alias/Projects/self-use-scripts/trello/shRun.sh clean blog >>/Users/alias/temp/log/clean.log 2>&1
```



修改之后，能成功运行得脚本 大概是这样得：

```cron
SHELL=/usr/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# move film on 10AM at every saturday
0 2 * * 6  (/usr/bin/sh /home/yohello/soft/self-use-scripts/trello/shMoveFilm.sh 2>&1) >/home/yohello/log/trello/moveFilm.log

# move blog
35 2 * * 0  (/usr/bin/sh /home/yohello/soft/self-use-scripts/trello/shMoveBlog.sh 2>&1) >/home/yohello/log/trello/moveBlog.log

# clean blog
0 2 * * 3  (/usr/bin/sh /home/yohello/soft/self-use-scripts/trello/shRun.sh clean blog 2>&1) >>/home/yohello/log/trello/clean.log
```



前面两行指定了 SHELL程序和环境变量，如果读者得脚本能成功运行，这两行有没有其实不重要， 如果读者得脚本在运行得时候提示`xxx command not found` 那可能就需要添加`PATH`了。

*笔者得执行命令添加了 /usr/bin/sh 作为开头， SHELL 指定得程序可能不会使用把。。 :joy:*

#### 如果不添加`2>&1` ，程序得输出是可以重定向到文件得， 但是加了之后就不行了

当出现这个情况得时候，就需要使用括号把命令写到一起去， 然后把整个输出重定向到一个文件。 

即 前面小括号里面得内容看做一个整体， 然后把这个整体得输出重定向到文件。  在整体得内部把错误输出重定向到输出中。 





### 参考阅读

[cron job not running with adding “2>&1”](https://stackoverflow.com/a/28992897/11226492)



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/crontab%E7%A2%B0%E5%88%B0%E5%BE%97%E5%B0%8F%E9%97%AE%E9%A2%98/  

