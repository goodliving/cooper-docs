---
title: "Docker, 主流的环境虚拟化工具"
date: 2018-06-18T11:39:17+08:00
draft: false
subtitle: "安装Docker，体验高效的开发测试流程" 
bigimg: [{src: "/img/docker.jpg", desc: "Docker"}]
tags: ["linux", "docker", "centos"]
---
`Docker`是一个开源的应用容器引擎，一种新的虚拟化工具，开发者将他们的应用以及依赖包安装到一个可移植的容器中，然后发布到任何流行的Linux机器上，简化了环境的安装，对于测试来说，实现的是开发、测试以及生产环境尽可能的保持一致，从而减少一些难以发现`bug`。
<!--more-->
### Why docker?
在介绍`docker`之前，我们先说说传统的项目开发，程序员写好业务代码，通过测试，当要发布版本时，我们需要准备一个`linux`系统，安装好对应的运行环境，如`java`的`jdk`，之后要配置好数据库以及系统加固等。当项目业务流量大了，需要横向扩容时，我们会再次进行之前的操作，准备一个新的`linux`环境，还可能要配置下负载均衡器以及其他等等。这是单体应用的情况，繁琐且易出错。  

而`Docker`提供的正是对环境进行封装的作用，将应用的运行环境打包成一个镜像，当你要发布你的服务，只要将装有项目源码的`docker`的镜像下载到一台装有`docker`的系统中，然后运行该镜像，就可以对外提供我们的服务给用户，同时也可以进行横向扩容，只要在运行一个业务容器即可。  

### Quick start
对于在`Linux`系统安装`docker`有两种方式

- 二进制安装
- 源码编译安装

以下，介绍在`centos`系统中安装`docker`，对于准备`linux`环境，请参考之前的文章[Linux操作系统](http://localhost:1313/post/linux/)。
如果你的系统可以连上互联网，那么在`centos`安装软件可以通过其自带的`yum`命令来快速安装，需要准备的是增加一个`docker`的[repo](https://www.cnblogs.com/nineep/p/6795692.html)。在命令行输入
```shell
[root@master1 yum.repos.d]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
[root@master1 yum.repos.d]# yum makecache
[root@master1 yum.repos.d]# yum install docker-ce
```
当以上执行都一切顺利，那么`docker`顺利被安装到我们的机器上了。接下来，通过几个简单实例来感受`docker`，首先，我们让`docker`启动运行
```shel
[root@master1 yum.repos.d]# systemctl enable docker
[root@master1 yum.repos.d]# systemctl start docker
```
> systemctl enable docker是将docker启动添加到系统启动项中

之后，我们运行一个`nginx`服务来感受以下`docker`的作用
```shell
[root@master1 yum.repos.d]# docker run -d --name test -p 80:80 nginx
```
该命令表示通过运行一个`nginx`的镜像，设定其运行起来的容器叫`test`，同时将主机的`80`端口映射到容器的`80`端口，也就是我们熟知的`nginx`的端口。

> 镜像是指封装我们的环境，容器是指当该镜像运行起来了，我们称之为容器，镜像是静态，容器是动态的

执行完以上命令，没有报错，你会看到以下类似的输入
```shell
[root@master1 yum.repos.d]# docker run -d --name test -p 80:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
f2aa67a397c4: Already exists
1cd0975d4f45: Pull complete
72fd2d3be09a: Pull complete
Digest: sha256:3e2ffcf0edca2a4e9b24ca442d227baea7b7f0e33ad654ef1eb806fbd9bedcf0
Status: Downloaded newer image for nginx:latest
949e806561c88146ddeba83345be6d54c3c29d0005d3ca7b693ea88558da810c
```
具体过程是`docker`首先会从我们本地查看我们要运行的镜像是否存在，如果没有，则默认是从官方的镜像仓库地址[dockerhub](https://hub.docker.com/)中下载镜像到本地，等待下载完成后，则根据我们运行的指令运行。我们可以通过`docker ps`来查看执行后的结果
```shell
[root@master1 yum.repos.d]# docker ps
CONTAINER ID        IMAGE     COMMAND                CREATED             STATUS              PORTS                NAMES
949e806561c8        nginx     nginx -g 'daemon ..."   2 minutes ago       Up 2 minutes        0.0.0.0:80->80/tcp   test
```
以上表示有个标号为`949e806561c8`容器，镜像是`nginx`,容器名称叫我们刚才命名的`test`，也就是刚才运行的命令结果。最后，我们可以通过在命令行运行`curl http://localhost`来查看`nginx`的默认页面，如下
```shell
[root@master1 yum.repos.d]# curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
当出现如上命令则表示我们成功通过`docker`来运行一个`web`服务器`nginx`，不只是在我们当前的机器上可以运行，从任何装有`docker`而且能访问[docker镜像中心](https://www.cnblogs.com/dfengwei/p/7150455.html)都可以做到。

>Docker的口号：Build, Ship, and Run Any App, Anywhere 

### Dockerfile自制镜像
上面我们通过运行一个公共镜像`nginx`来感受`docker`，接下来，我们通过`Dockerfile`来展示如何将我们应用容器话的过程。  
现在假设我们有个`golang`的项目，源码如下
```shell
[root@master1 golang]# l
总用量 4
-rw-r--r-- 1 root root   0 6月  19 17:49 Dockerfile
-rw-r--r-- 1 root root 221 6月  19 17:48 example.go
```
其中，`example.go`是一个简单的`web`应用，提供一个`ping`接口，代码如下
```shell
[root@master1 golang]# cat example.go
package main

import "github.com/gin-gonic/gin"

func main() {
        r := gin.Default()
        r.GET("/ping", func(c *gin.Context) {
                c.JSON(200, gin.H{
                        "message": "pong",
                })
        })
        r.Run() // listen and serve on 0.0.0.0:8080
}
```
当我们访问`http://0.0.0.0:8080/ping`，该应用就返回一个`pong`的`json`数据。而`Dockerfile`文件是我们需要将该应用容器化的配置文件，如下所示
```shell
FROM golang
RUN mkdir /go/src/demo -p
ADD . /go/src/demo
WORKDIR /go/src/demo
RUN go get -u -v github.com/gin-gonic/gin
RUN go build -o demo
EXPOSE 8080
CMD ["/go/src/demo/demo"]
```
以上表示运行一个`golang`容器，设置工作目录`/go/src/demo`，之后编译源码生成一个`demo`可执行文件，最后设置该镜像的启动命令为刚才编译的`demo`，也就是提供`ping`的接口程序。

> 对于`golang`不熟悉的，请查看`golang`介绍。对于其他语言项目，如`java`，是类似。在这里以`golang`为例，是比较简单方便展示

之后，通过`docker build`命令来生成应用镜像，如下
```shell
[root@node1 golang]# docker build -t demo .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM golang
 ---> 52057de6c8d0
Step 2/8 : RUN mkdir /go/src/demo -p
 ---> Using cache
 ---> 542825b49177
Step 3/8 : ADD . /go/src/demo
 ---> dc00592518c2
Step 4/8 : WORKDIR /go/src/demo
Removing intermediate container 89f074098fbd
 ---> 86612b9ec869
Step 5/8 : RUN go get -u -v github.com/gin-gonic/gin
 ---> Running in f740a6905610
github.com/gin-gonic/gin (download)
github.com/gin-contrib/sse (download)
github.com/golang/protobuf (download)
github.com/ugorji/go (download)
Fetching https://gopkg.in/go-playground/validator.v8?go-get=1
Parsing meta tags from https://gopkg.in/go-playground/validator.v8?go-get=1 (status code 200)
get "gopkg.in/go-playground/validator.v8": found meta tag get.metaImport{Prefix:"gopkg.in/go-playground/validator.v8", VCS:"git", RepoRoot:"https://gopkg.in/go-playground/validator.v8"} at https://gopkg.in/go-playground/validator.v8?go-get=1
gopkg.in/go-playground/validator.v8 (download)
Fetching https://gopkg.in/yaml.v2?go-get=1
Parsing meta tags from https://gopkg.in/yaml.v2?go-get=1 (status code 200)
get "gopkg.in/yaml.v2": found meta tag get.metaImport{Prefix:"gopkg.in/yaml.v2", VCS:"git", RepoRoot:"https://gopkg.in/yaml.v2"} at https://gopkg.in/yaml.v2?go-get=1
gopkg.in/yaml.v2 (download)
github.com/mattn/go-isatty (download)
github.com/gin-gonic/gin/json
github.com/golang/protobuf/proto
gopkg.in/go-playground/validator.v8
gopkg.in/yaml.v2
github.com/gin-contrib/sse
github.com/ugorji/go/codec
github.com/mattn/go-isatty
github.com/gin-gonic/gin/binding
github.com/gin-gonic/gin/render
github.com/gin-gonic/gin
Removing intermediate container f740a6905610
 ---> 8d7c287d32a3
Step 6/8 : RUN go build -o demo
 ---> Running in 0a02af773a2c
Removing intermediate container 0a02af773a2c
 ---> 485d299c7bf3
Step 7/8 : EXPOSE 8080
 ---> Running in d1a345a625fc
Removing intermediate container d1a345a625fc
 ---> 18b4a8899173
Step 8/8 : CMD ["/go/src/demo/demo"]
 ---> Running in ae9bbda5981f
Removing intermediate container ae9bbda5981f
 ---> 08adb93c8c7a
Successfully built 08adb93c8c7a
Successfully tagged demo:latest
```
以上代表我们的应用镜像生成成功，最后，我们来运行`docker run -d --name demo -p 8080:8080 demo`来确认该应用镜像能提供服务
```shell
[root@node1 golang]# curl http://localhost:8080/ping | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    18  100    18    0     0   2071      0 --:--:-- --:--:-- --:--:--  2250
{
    "message": "pong"
}
```
以上表示应用的容器化能正常提供我们预期的功能。

### How to use docker?
参考[大神系列之TDD开发容器化的Python微服务](https://blog.qikqiak.com/post/tdd-develop-python-microservice-app/)  

> Use Docker, More hair, have fun !!!
