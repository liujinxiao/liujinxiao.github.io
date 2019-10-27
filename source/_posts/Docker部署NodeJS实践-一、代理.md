---
title: Docker部署NodeJS实践(一、代理)
date: 2019-10-27 21:11:16
tags: Docker
categories: Docker
---

本文在centos7系统中，采用Docker容器部署Nodejs工程。 Doceker版本1.31。
## 代理
1、在centos系统中安装docker后，有一些服务器是没有连接外网权限的，可以测试是否能ping通。如果ping不通，则需要配置代理。本系统具体的实际代理地址及端口号，输入env即可显示。

2、方法

> 1、在`/etc/systemd/system`目录下创建一个的`docker.service.d`文件夹

```
mkdir -p /etc/systemd/system/docker.service.d

```
> 2、在`docker.service.d`文件夹下创建http-proxy.conf文件，并添加HTTP_PROXY变量，其中proxy-url和proxy-port分别改成实际情况的代理地址和端口：

```
Environment="HTTP_PROXY=http://proxy-addr:proxy-port/"
"HTTPS_PROXY=https://proxy-addr"
```

>3、如果有不需要使用代理来访问的Docker registries，那么还需要制定NO_PROXY环境变量：

```
Environment="HTTP_PROXY=http://proxy-addr:proxy-port/"
"NO_PROXY=localhost,127.0.0.0/8"
```

>4、更新配置：
<pre><code>daemon-reload </code></pre>

>5、重启docker服务
<pre><code>restart docker</code></pre>