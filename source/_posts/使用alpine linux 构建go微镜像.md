---
title: 使用alpine linux 构建go微镜像
date: 2017-08-29
categories: 学习
tags: [go,docker]
keywords: go,微镜像,docker
description: 使用alpine linux构建docker镜像，给镜像瘦身。
---

## 配置环境和Dockerfile

首先构建一个golang build 的环境。
Dockerfile文件:

```
FROM docker.io/golang:alpine
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.4/main" > /etc/apk/repositories
RUN apk add --update curl bash && \
    rm -rf /var/cache/apk/*
```

## 构建镜像

`docker build -t donh/go-build:1.0 .`

## 启动镜像

启动镜像，并把/data/go 目录映射到 /data/go目录，其中–rm 表示退出之后删除镜像。
执行命令后会进入容器命令行，可先编辑好main.go文件再拷贝到挂载目录。

```
docker run -it -v /Users/donh/work/gayhub/go-build/data/go:/data/go --rm donh/go-build:1.0 /bin/bash
#cd /data/go
#go build main.go
```

<font color=#FF0000>注：CGO_ENABLED=0 GOOS=linux go build main.go 交叉编译替换go build main.go，
生成的可执行文件在后面可用scratch基础镜像构建运行，镜像大小更小</font>

编译后获得main的可执行文件。

## 以alphine为基础构建应用镜像

新建第二个Dockerfile

```
FROM alpine
RUN mkdir -p /data/go
COPY main /data/go
EXPOSE 8080
ENTRYPOINT ["/data/go/main"]
```

构建镜像，启动容器

```
docker build -t donh/go-http:1.0 .
docker run -d -p 9013:8080 --name go-http donh/go-http:1.0
```

启动成功，访问 localhost:9013/hello

[示例代码](https://github.com/donhac/go-http)

## TODO

- 建立自动化启动脚本
- 两个Dockerfile文件合并成单个进行构建


