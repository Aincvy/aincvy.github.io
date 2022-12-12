# Google Analyis 不能在keepIt主题中使用的问题


KeepIt 是一个优美的Hugo 主题， 笔者目前使用的正是这个主题。 

https://github.com/Fastbyte01/KeepIt



笔者在使用这个主题的时候发现， 部分情况下的谷歌分析是使用不了的。

经过一番搜索之后， 笔者发现，这个主题似乎只支持老版的谷歌分析的API。 即`UA-` 开头的跟踪ID，而`GA-` 开头的则不支持。 



知道了问题， 解决起来就简单了， 在谷歌分析的管理后台中创建一个老版本的应用就可以了。

当然， 也可以考虑修改模板文件， 使其兼容`GA-` 版本的API。 不过笔者觉得那样做有一点麻烦，所以就没有那样做。



**拓展阅读**

https://stackoverflow.com/questions/64993368/my-google-analytics-setup-for-hugo-is-not-working





---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/google-analyis-%E4%B8%8D%E8%83%BD%E5%9C%A8keepit%E4%B8%BB%E9%A2%98%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%9A%84%E9%97%AE%E9%A2%98/  

