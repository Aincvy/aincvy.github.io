# Mattermost中文搜索不生效的问题处理


原因是pgsql 默认不支持 中文分词搜索，所以调整一下数据库就可以了  
官方文档可见：  https://docs.mattermost.com/administration-guide/configure/enabling-chinese-japanese-korean-search.html


- 手动编译数据库插件然后启动太麻烦了，所以笔者直接使用的别人build好的镜像： `abcfy2/zhparser:13-alpine`  
  - 镜像来源可见  https://hub.docker.com/r/abcfy2/zhparser
  - docker compose 里调整一下 image 即可， 别的都不需要变动
- 启动数据库
- 在数据库里执行sql 语句开启一下扩展
- 重启一下mattermost 就可以使用中文搜索了


数据库执行的语句参考
```shell
$ # 在 mattermost 的 docker compose 目录里面执行
$ sudo docker compose exec -it postgres /bin/bash
$ psql -U mmuser -d mattermost

# 下面是sql 语句 部分
```

```sql
-- 初始化插件
CREATE EXTENSION zhparser;
CREATE TEXT SEARCH CONFIGURATION chinese_zh (PARSER = zhparser);
ALTER TEXT SEARCH CONFIGURATION chinese_zh ADD MAPPING FOR n,v,a,i,e,l WITH simple;

-- 重建索引
-- 删除消息内容的英文索引
DROP INDEX idx_posts_message_txt;

-- 删除标签的英文索引  
DROP INDEX idx_posts_hashtags_txt;

-- 为消息内容创建中文全文搜索索引
CREATE INDEX idx_posts_message_txt ON posts USING gin (to_tsvector('chinese_zh'::regconfig, message::text));

-- 为标签创建中文全文搜索索引
CREATE INDEX idx_posts_hashtags_txt ON posts USING gin (to_tsvector('chinese_zh'::regconfig, hashtags::text));

-- 查看索引是否创建成功
\d posts

-- 测试中文搜索功能
SELECT id, message FROM posts WHERE to_tsvector('chinese_zh', message) @@ to_tsquery('chinese_zh', '测试');
```

记得一定要重启一下 mattermost 实例。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/mattermost%E4%B8%AD%E6%96%87%E6%90%9C%E7%B4%A2%E4%B8%8D%E7%94%9F%E6%95%88%E7%9A%84%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86/  

