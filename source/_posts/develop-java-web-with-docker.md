---
title: 在Java开发中使用Docker
date: 2017-11-28 20:40:27
tags:
- Java
- Docker
---

上一篇博客中提到，最近在写一个Java Web应用，因为某些原因换了开发环境，然后折腾了半天。于是就想起来使用Docker来搭建开发环境，妈妈以后就再也不用担心我开发中途换环境了！搭建过程还是比较顺利的，现在来记录一下如何将基于Maven的Java项目部署到Docker。如果不是基于Maven的项目，可参考该博客进行相应的修改。

## 1.安装Docker

这一步不用多说什么，直接进入[官网](https://www.docker.com/)按照提示安装即可。Linux 需要单独安装 Docker Compose 。 Mac OS 和 Windows版的Docker安装包里已经内置了 Docker Compose， 不用单独安装。

## 2.添加并修改配置文件

在应用根目录里添加 `config` 目录，然后复制Tomcat的 `Server.xml` 配置文件到该目录。然后根据自己的需求修改该配置文件。

> 因为本人习惯于将应用部署到Tomcat根目录，所以需要修改Tomcat配置文件。而Docker容器里直接修改配置文件不是很方便，所以新建一个配置文件用于替换容器里的配置文件。如使用默认配置文件即可跳过这步。

## 3. 编写 `Dockerfile` 文件

在应用根目录中添加 `Dockerfile` 文件，并写入一下内容：

```Dockerfile
FROM tomcat:8.0-jre8-alpine

# 删除Tomcat默认根目录，可根据自己需求保留或删除
RUN rm -rf /usr/local/tomcat/webapps/ROOT/

# 替换Tomcat配置文件，可根据自己需求修改或删除
COPY ./config/server.xml /usr/local/tomcat/conf/server.xml

# 挂载应用目录，根据自己需求修改，需与Tomcat配置文件一致
VOLUME /usr/local/tomcat/webapps/forus/

# 暴露8080端口
EXPOSE 8080

# 运行Tomcat，并启用远程调试
CMD ["catalina.sh", "jpda", "run"]
```

在应用根目录下添加 `.dockerignore` 文件。该文件与 `.gitignore` 类似，用于避免将某些文件添加到创建Docker镜像时的上下文。在其中添加除了 `config` 目录之外的其他目录及文件：

```gitignore
src/
.idea/
target/
```

> 可以将 `config` 目录及 `Dockerfile` 添加到另一个目录中来避免 `.dockerignore` 文件

## 4. 编写 `docker-compose.yml` 文件

因为自己的项目用到了Mysql和Redis，需要运行多个服务容器。所以用了Docker Compose 来管理这些服务。

```yaml
version: "2.3"
services:
  mysql:
    image: mysql
    expose:
      - "3306"                      # Mysql 服务端口
    environment:
      - MYSQL_ROOT_PASSWORD=123456  # Mysql root 用户密码

  redis:
    image: redis:alpine
    expose:
      - "6379"

  forus:
    build: .
    links:
      - mysql
      - redis
    environment:
      - spring.profiles.active=test # 激活 Spring 的 Profile
      - JPDA_ADDRESS=0.0.0.0:8000   # 远程调试地址
    volumes:
      - ./target/forus/:/usr/local/tomcat/webapps/forus/
    expose:
      - "8080"
    ports:
      - "8080:8080"   # 应用端口映射
      - "8000:8000"   # 远程调试端口映射
```

## 5. 运行/调试

### 运行

```
1. 在应用根目录下运行 `mvn war:exploded` 命令编译项目。

2. 运行 `docker-compose up` 命令构建镜像并运行相应服务。

3. 在浏览器中打开 `localhost:8080` 即可访问该应用。

4. 在应用根目录下运行 `docker-compose down` 命令可停止相应服务并删除相关容器和镜像。
```

### 调试

```
通过远程调试的方式连接到 `localhost:8000' 可进行调试。
```