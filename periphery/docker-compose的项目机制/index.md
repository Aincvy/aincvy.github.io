# Docker Compose的项目机制


docker compose 存在一个项目机制， 官方文档参考： https://docs.docker.com/reference/cli/docker/compose/#use--p-to-specify-a-project-name

项目名称的遵从下面的顺序来设置：
- 使用 -p 的命令行参数
- 环境变量： `COMPOSE_PROJECT_NAME`
- 配置文件里面的顶层配置项 `name:` (或者在指定了一系列配置文件时最后一个`name:`)
- 配置文件所在的目录名字（或者第一个配置文件所在的目录的目录名字）
- 如果没有指定项目名称， 那么将会使用当前目录的目录名字作为项目名称。 目录名字必须由这些元素组成： 小写字母，数字，- 或_ 。 同时必须以小写字母或者数字开头， 否则就必须使用其他机制来指定项目名称。 

当完全没有指定过项目名称的时候， 将会使用`docker-compose.yaml` 文件所在目录的目录名作为项目名称， 所以如果存在 A/X, B/X 两个目录的时候， 两个项目名称都将会是 X 。 

这意味着，在 B/X 目录下执行 docker compose up 的时候， 会把 A/X 的容器给顶掉， 反过来也一样。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/docker-compose%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%9C%BA%E5%88%B6/  

