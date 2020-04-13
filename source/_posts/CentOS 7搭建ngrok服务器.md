---
title: CentOS 7搭建ngrok服务器
date: 2017-09-21
categories: 学习
tags: [ngrok, CentOS]
keywords: ngrok,内网穿透,CentOS
description: CentOS 7搭建ngrok服务器,实现内网穿透，通过外网访问本地服务。
---

## 搭建ngrok服务端

搭建服务端可以参照 [这篇文章](https://ubock.com/article/31)，我是用的京东云服务器，按博主步骤服务端搭建都很顺利，这里就不再重述了。

## 域名解析

如果之前已经添加了域名到服务器的解析的话，这一步可以忽略。
因为上一步我设置的是`NGROK_DOMAIN="ngrok.donhac.top"`，所以这里添加了如下两条解析记录。

![](http://owmmis3gj.bkt.clouddn.com/WX20170921-191521@2x.png)

## 生成客户端

生成客户端这里遇到了一些坑。原文是针对windows的，这里记录下mac的客户端生成过程。

在第一步编译好服务端的基础上，开始编译mac客户端：

```shell
cd /usr/local/go/src
GOOS=darwin GOARCH=amd64 ./make.bash
```

这里会报如下错误：

```shell
##### Building Go bootstrap tool.
cmd/dist
ERROR: Cannot find /root/go1.4/bin/go.
Set $GOROOT_BOOTSTRAP to a working Go tree >= Go 1.4.
```

搜了下网上说是Go高版本的编译过程需要Go1.4的二进制来实现引导(bootstrap)，简单来说就是：Go需要Go自身来编译。
所以这里需要先安装Go1.4版本，这里我们在已经安装了1.8版本的基础上继续安装。

```shell
wget https://storage.googleapis.com/golang/go1.4.3.linux-amd64.tar.gz
tar -C /root/ -xvf go1.4.3.linux-amd64.tar.gz
mv /root/go/ /root/go1.4
cd /root/go1.4/src
./all.bash
```

继续重新编译mac客户端：

```shell
cd /usr/local/go/src
GOOS=darwin GOARCH=amd64 ./make.bash
cd /usr/local/src/ngrok
GOOS=darwin GOARCH=amd64 make release-client
```

完了后在`/usr/local/src/ngrok/bin/darwin_amd64/`下可以看到生成了一个ngrok文件，将它拷贝到mac上。

到这里，mac客户端的编译也完成了，后面将应用跑起来跟windows基本是一样的了。
新建ngrok文件夹，将前面生成的ngrok文件放到该文件夹，新建名为`ngrok.cfg`的文件，配置如下：

```
server_addr: "ngrok.donhac.top:8083"
trust_host_root_certs: false
```
执行以下命令启动:

```
./ngrok -config=./ngrok.cfg -subdomain=test 8080
```

启动成功。

![](http://owmmis3gj.bkt.clouddn.com/WX20170921-195901@2x.png)

这里如果不指定`-subdomain=test`的话会生成一个随机前缀的域名。

<font color=#FF0000>注：服务端启动后如果关闭远程连接服务端后台也会被杀掉断开，这里可以设置服务端的服务后台运行：
</font>

```shell
cd /usr/local/src/ngrok
setsid ./bin/ngrokd -tlsKey="assets/server/tls/snakeoil.key" -tlsCrt="assets/server/tls/snakeoil.crt" -domain="ngrok.donhac.top"  -httpAddr=":8081" -httpsAddr=":8082" -tunnelAddr=":8083"
```

## TODO

- 微信本地开发，若服务器80端口被暂用，在不影响80端口服务的基础上利用nginx做反向代理。

