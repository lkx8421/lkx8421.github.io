---
title: docker使用
date: 2025-02-05 17:20:11
tags: docker
categories: docker
---


## 安装

```shell
# 查看是否安装
docker -v

# 安装
sudo apt-get update
sudo apt-get install docker.io

# 查看是否安装成功
docker -v
```
<!--more-->

## 基本命令

```shell
# 查看镜像
docker images

# 拉取镜像
docker pull image_name

# 运行容器
docker run -d -p 8080:8080 --name container_name image_name

# 查看容器
docker ps -a

# 启动容器
docker start container_id

# 附加容器
docker attach container_id

# 进入容器
docker exec -it container_id /bin/bash 

# 停止容器
docker stop container_id

# 删除容器
docker rm container_id

# 删除镜像
docker rmi image_id

# 查看容器日志
docker logs container_id

# 查看容器信息
docker inspect container_id

# 进入容器
docker exec -it container_id /bin/bash

# 查看docker版本
docker version

# 查看docker信息
docker info

# 查看docker帮助
docker --help

# 容器导入
docker import image_path image_name

# 容器导出
docker export container_id > image_path

```

