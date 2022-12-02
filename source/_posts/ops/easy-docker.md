---
title: docker 简易入门
categories:
  - ops
img: ../../coverImages/docker.jpg
tags:
  - docker
date: 2020-12-17 23:00:00
---

![齐天大圣孙悟空](/images/20180125141147_61638d31e51d85f2a5d62e04191fe55f_1.jpg)

> 拔一根毫毛，吹出猴万个。他叫孙悟空，也叫 docker。

## docker 无所不能？

1. 安装 wordpress

```shell
docker run --name wordpress -p 8080:80 -d wordpress
```

2. 安装 Nextcloud 网盘

```shell
docker run -d -p 8080:80 nextcloud
```

wordpress 安装中，我们只看到安装界面，数据库需要选择本地或者远程。Nextcloud 网盘安装中，我们安装完成就可以直接使用，因为 Nextcloud 内部默认携带了 SQLite 数据库，所以安装之后，就能够直接使用。

## docker 基本概念

**镜像（Image）**：Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

**容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

**仓库（eg:dockerHub）**：一个 Docker Registry 中可以包含多个 仓库（Repository）；每个仓库可以包含多个 标签（Tag）；每个标签对应一个镜像。

## docker 常用命令

显示可用容器：

```shell
docker images
```

删除指定容器：

```shell
docker rmi <镜像ID>
```

查看容器：

```shell
docker ps
```

拉取并运行容器：

```shell
docker run hello-world
```

进入容器内部，并运行 bash：

```shell
docker exec -it <id/container_name>  /bin/bash
```

## Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
有时候一个镜像需要很多命令，这时候可以写到 Dockerfile 文件中，一次性执行完毕。
我们尝试将[欧姆龙血压计](https://github.com/Yaob1990/OMRON_Blood_Pressure_Analyse)解析工具源码编译成一个 docker 镜像

1. 在项目根目录新建 Dockerfile 文件，并写上以下内容。
   ```shell
   FROM node:14.15.1-slim
   RUN yarn global add http-server
   COPY dist public
   EXPOSE 4000
   CMD ["http-server","-p","4000"]
   ```
2. 编译项目：`npm run build`
3. 打包项目：

   ```shell
   docker build -t yekongbuye/omron .
   ```

4. 推送到 docker hub
   ```shell
   docker login
   docker push yekongbuye/omron
   ```

## docker-compose

主要是对容器进行编排。一次性编排，运行多个容器。

1. 创建 docker-compose.yml 文件

   ```yml
   version: '3.3'
   services:
     db:
       image: mysql:5.7
       volumes:
         - db_data:/var/lib/mysql
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: somewordpress
         MYSQL_DATABASE: wordpress
         MYSQL_USER: wordpress
         MYSQL_PASSWORD: wordpress

     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       ports:
         - '8000:80'
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: wordpress
         WORDPRESS_DB_PASSWORD: wordpress
         WORDPRESS_DB_NAME: wordpress
   volumes:
     db_data: {}
   ```

## 分布式部署

- swarm
- kubernetes（k8s）
