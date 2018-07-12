---
title: 通过docker部署easy-mock到内网
tags: 
  - easy-mock
  - docker
---

easy-mock是一个mock数据系统，[github](https://github.com/easy-mock/easy-mock)。

需要将此系统部署至内网，供前端使用。内网服务使用了docker，所以需要为此系统配置一个docker。

<!-- more -->

## easy-mock docker

### easy-mock

easy-mock是一个mock数据系统，[github](https://github.com/easy-mock/easy-mock)

需要将此系统部署至内网，供前端使用。内网服务使用了docker，所以需要为此系统配置一个docker。


### [docker](https://www.docker.com/)

#### 安装

macOS

```sh
brew update && brew install docker
```

或者至官网下载安装包。

#### 安装镜像

> 国内安装需要配置代理，否则速度比较差

easy-mock使用了redis(>= v4.0)、MongoDB(>= v3.4)

```sh
docker pull redis
docker pull mongo
```

安装成功之后，编写`Dockerfile`文件及编写`docker-compose.yml`文件。参考文章[docker nodejs](https://ciphertrick.com/2017/10/23/dockerize-nodejs-service-with-mongodb-docker-compose/)


#### 启动docker

`Dockerfile`和`docker-compose.yml`编写完成后，可以执行`docker-compose up --build`启动docker build

