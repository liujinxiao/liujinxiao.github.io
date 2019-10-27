---
title: Docker部署NodeJs实践（二、部署images和容器）
date: 2019-10-27 21:14:33
tags: Docker
categories: Docker
---

本NodeJs工程基于node-v8的docker镜像版本。

## 1、Dockerfile文件
首先，在node工程的根目录创建Dockerfile文件，该文件是node工程中对docker的配置文件。

1、创建Dcokerfile文件
<pre><code>vi Dockerfile</code></pre>
2、编写文件内容如下：
<pre><code>#node镜像版本
FROM node:8-alpine
#声明作者
MAINTAINER LIU
#在image中创建文件夹
RUN mkdir -p /home/Service
#将该文件夹作为工作目录
WORKDIR /home/Service

# 将node工程下所有文件拷贝到Image下的文件夹中
COPY . /home/Service

#使用RUN命令执行npm install安装工程依赖库
RUN npm install

#暴露给主机的端口号
EXPOSE 8888
#执行npm start命令，启动Node工程
CMD [ "npm", "start" ]
</code></pre>

## 2、构建image
执行命令`docker build -t node-app:v1 .`&nbsp;需要注意v1后面还有一个`.`<br />
其中 -t node-app:v1 为构建的镜像名称及标签
<pre><code>[root@localhost test1]# docker build -t node-app:v1 .
Sending build context to Docker daemon 4.096 kB
Step 1/8 : FROM node:8-alpine
 ---> dd574b216ad7
Step 2/8 : MAINTAINER LIU
 ---> Using cache
 ---> f3f22f068507
Step 3/8 : RUN mkdir -p /home/Service
 ---> Using cache
 ---> 2222ce103ae1
Step 4/8 : WORKDIR /home/Service
 ---> Using cache
 ---> e60fd914f709
Step 5/8 : COPY . /home/Service
 ---> Using cache
 ---> 58000275f835
Step 6/8 : RUN npm install
 ---> Using cache
 ---> e66dc16c44f4
Step 7/8 : EXPOSE 8888
 ---> Using cache
 ---> 2adff3739104
Step 8/8 : CMD npm start
 ---> Using cache
 ---> 190fba2814a6
Successfully built 190fba2814a6
</code></pre>
查看生成的image: `docker images`命令
<pre><code>[root@localhost test1]# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
node-app                v1                  190fba2814a6        2 months ago        71 MB
</code></pre>
## 3、运行container
> 执行命令 `docker run -d -p 8888:8888 190f` <br />
其中， -d表示在容器后台运行，-p表示端口映射，将本机的8888端口映射到container的8888端口，外网访问本机的8888端口即可访问container。190f为生成的IMAGE的ID,只需要写入对应ID的前几位系统能辨识出对应的image即可。
```
[root@localhost test1]# docker run -d -p 8888:8888 190f
1b335de60ff4ad0f75aa6c4458d4e91cc3839f6e606c5b09ff926bcebd6c9770
```
> 执行命令`docker ps`查看container是否运行
```
[root@localhost test1]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
1b335de60ff4        190f                "npm start"         4 seconds ago       Up 3 seconds        0.0.0.0:8888->8888/tcp   epic_swartz
````
> 通过命令docker logs 1b335 还可查看container的日志
```
[root@localhost test1]# docker logs 1b335
> webtest@1.0.0 start /home/Service
> node server.js
Running on http://localhost:8888
```
>说明服务已经启动。
## 4、进入容器
为了方便查看容器内部文件和调试，可以通过命令进入容器中。容器内部就像一个小型的linux系统一样。命令为`docker exec -it 1b33 /bin/sh`
```
[root@localhost test1]# docker exec -it 1b33  /bin/sh
/home/Service # ls
Dockerfile         node_modules       package-lock.json  package.json       server.js
```
## 5、日志
* docker镜像中node工程会有打印日志功能，因为docker容器一旦挂掉，容器中的文件也会访问不了，所以日志必须要放在docker镜像外的文件路径下。此时，必须要将centos系统中的日志文件目录挂在到docker容器中，在容器启动时开启数据卷，实现日志采集。<br/>
* 在启动容器时，使用命令`docker run -d -p 8888:8888 -v /home/logs:/data/logs 190f`即可。`/home/logs`为centos系统中日志文件目录，`data/logs`为docker容器中node工程写入日志路径。
* 如果docker容器中工程需要写入文件，则在启动时要加上`--privileged=true`才可以。
## 6、打包与解压
> 如果没有私有仓库，则可以通过save和load命令来打包和解压。
save将docker镜像压缩为tar文件，load为将tar文件解压生成镜像。

```
[root@localhost docker]# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
node-app                v1                  190fba2814a6        2 months ago        71 MB
[root@localhost docker]# docker save 190fba -o /home/docker/node-app-1.0.tar
[root@localhost docker]# ll
-rw-------. 1 root root 78526976 8月  26 19:38 node-app-1.0.tar
```
> 其中`/home/docker`文件路径为tar存放目录，必须提前建好，docker不会自动创建。<br />
> 而解压命令为:
```
[root@localhost docker]# docker load < /home/docker/node-app-1.0.tar
Loaded image ID: sha256:190fba2814a66291d06368a8afef499aa6f96f5d6def0b808d1fa5b76d862d53
```
> 此时容器将部署到环境中,可使用`docker images`查看。