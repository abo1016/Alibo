---
title: docker 命令
tags: [docker]
id: '395'
categories:
    - docker
date: 2019-02-20 17:27:46
---

*   查看正在运行的容器

`docker ps`

*   查看所有容器列出所有container

`docker-ps -a`

*   停止/运行实例

`docker run/stop --help` `docker run/stop container`

*   查看container详情

`docker inspect [container]`

*   删除某个container

`docker rm [container]` ``docker rm `docker ps -a -q` 删除所有容器，-q表示只返回容器的ID``

*   查看某个container的运行日志

```
docker logs [container]

docker logs -f [container] 类似tailf
```

*   查看容器内部的进程信息

`docker top [container]`

*   在容器中运行后台任务，只对正在运行的容器有效

```
docker exec -d [container] [cmd]

docker exec -d edison touch /home/haha
```