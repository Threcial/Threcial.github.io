---
title: "Docker基础"
date: 2026-06-15 15:10:25
permalink: "docker/docker基础/"
updated: 2026-06-15 15:10:25
categories:
  - "docker"
tags:
  - "docker"
  - "虚拟化"
description: "Docker 把应用和运行环境打包成镜像，再把镜像运行成容器 镜像是静态模板，容器是运行中的进程 Docker 与虚拟机 对比项容器虚拟机内核共享宿主机内核每台虚拟机有完整系统内核启动速度秒级或更快通"
cover: /img/covers/cover-001.jpg
top_img: false
---

Docker 把应用和运行环境打包成镜像，再把镜像运行成容器

镜像是静态模板，容器是运行中的进程

## Docker 与虚拟机

| 对比项 | 容器 | 虚拟机 |
| --- | --- | --- |
| 内核 | 共享宿主机内核 | 每台虚拟机有完整系统内核 |
| 启动速度 | 秒级或更快 | 通常分钟级 |
| 资源占用 | 较低 | 较高 |
| 隔离层次 | 操作系统进程级隔离 | 硬件虚拟化隔离 |
| 适用场景 | 应用交付、环境一致性、弹性部署 | 强隔离、多系统运行 |

Docker 的核心收益是环境一致、迁移方便、启动快、资源利用率高。与其说是“小型虚拟机”，不如说是“带隔离环境的进程”

## 安装 Docker

CentOS 7 安装示例：

```text
yum remove -y docker*
yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/dockerce/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
```

## 镜像操作

```text
docker images
docker image ls
docker image ls -q

docker pull redis:6.0
docker pull nginx

docker rmi nginx:latest
docker image rm redis:6.0

docker search redis
```

查看镜像分层：

```text
docker history nginx
docker history nginx --format "table {.ID}	{.CreatedBy}" --no-trunc
```

## 容器运行

```text
docker run nginx:latest
docker run -d -p 1192:80 nginx:latest
docker run -d -p 1721:80 --name nginx01 nginx:latest
docker run -it nginx:latest bash
```

参数：

| 参数 | 作用 |
| --- | --- |
| `-d` | 后台运行 |
| `-p 宿主机端口:容器端口` | 端口映射 |
| `-P` | 随机映射镜像声明端口 |
| `--name` | 指定容器名 |
| `-it` | 交互终端 |
| `--restart=always` | 容器异常退出或 Docker 重启后自动拉起 |

容器启动后必须有前台进程存在。如果容器内没有前台进程那么容器就退出

## 查看、启停、删除容器

```text
docker ps
docker ps -a
docker ps -aq

docker start nginx01
docker stop nginx01
docker restart nginx01
docker kill nginx01
docker rm nginx01
docker rm -f nginx01
docker rm -f $(docker ps -qa)
```

已经启动的容器配置自启：

```text
docker update --restart=always 容器ID
```

取消自启：

```text
docker update --restart=no 容器ID
```

## 进入容器与执行命令

```text
docker exec -it nginx01 bash
docker exec nginx01 nginx -t
```

`docker exec` 是进入已运行容器的常用方式。`docker run -it image bash` 是新建一个容器并进入，退出后容器也退出

## 宿主机与容器复制文件

```text
docker cp nginx01:/usr/share/nginx/html/index.html /root/

docker cp /root/test.html nginx01:/usr/share/nginx/html/
```

`docker cp` 用于临时拷贝

长期同步代码、配置或数据时，使用 volume 或 bind mount

## 容器日志

```text
docker logs nginx01

docker inspect nginx01 | less
```

## commit、save、load

把容器提交为镜像：

```text
docker commit -m "ngx-test" -a "xwx" nginx01 ngx-01:latest
```

保存镜像为 tar：

```text
docker save ngx-01:latest -o ngx-01.tar
```

导入镜像：

```text
docker load -i ngx-01.tar
```

## 数据卷

数据卷用于把数据从容器生命周期中分离出来，删除容器时不会自动删除命名数据卷

### bind mount

```text
docker run -d -P -v /html:/usr/share/nginx/html nginx:latest
```

宿主机目录直接映射到容器目录。注意：宿主机目录内容会覆盖容器目标目录

### named volume

```text
docker volume create web001
docker run -d -p 3333:80 --name nginx20 -v web001:/usr/share/nginx/html nginx
```

查看：

```text
docker volume ls
docker volume inspect web001
```

删除：

```text
docker volume rm web001
docker volume prune
```

**`docker volume prune` 会删除未被容器使用的数据卷，执行前必须确认没有误删风险**

## Docker 网络

安装 Docker 后会自动创建 `docker0` 网桥。默认 bridge 网络中，容器通过 veth pair 接入 docker0，再通过 NAT 访问外部

| 网络模式 | 说明 | 参数 |
| --- | --- | --- |
| bridge | 默认模式，每个容器有独立 IP | `--network bridge` |
| host | 使用宿主机网络命名空间 | `--network host` |
| none | 不配置网络 | `--network none` |
| container | 与另一个容器共享网络 | `--network container:容器名` |

查看网络：

```text
docker network ls
docker network inspect bridge
```

创建用户自定义网络：

```text
docker network create webnet
docker run -d --name nginx01 --network webnet nginx
docker run -d --name nginx02 --network webnet nginx
```

在用户自定义 bridge 网络里，容器可以直接用容器名互相解析

## 部署常见服务

### Redis

```text
docker run -d --name redis   -p 6379:6379   -v redis_data:/data   redis:5.0.7 redis-server --appendonly yes
```

### Nginx

```text
docker run -d --name nginx   -p 80:80   -v /html:/usr/share/nginx/html   nginx:latest
```
