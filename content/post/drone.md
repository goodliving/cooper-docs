---
title: "Drone, 基于容器的CI/CD"
date: 2018-06-18T15:26:33+08:00
draft: false
subtitle: "带你认识drone"
bigimg: [{src: "/img/drone.jpg", desc: "drone"}]
tags: ["docker", "cicd", "gitea", "golang"]
---
持续集成(`Continuous Integration`)、持续交付(`Continuous Delivery`)以及持续部署(`Continuous Deployment`)主要实现的是在保证软件质量的情况下保持高速的迭代。关于三者介绍，请参考[what are ci cd and devops for us](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)一文。目前已经有很多的`CI`工具，具体的可以查看[开源中国收集的持续集成工具列表](https://www.oschina.net/search?scope=project&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90)，基于目前的`docker`的使用以及软件设计理念，选择了`drone`，以下是介绍`Drone`的工具安装和使用场景。
<!--more-->
### Why Drone
[Drone](http://docs.drone.io/)不同于其他工具，如`jenkins`，它一开始就按照容器`docker`的开发模式来进行设计使用的，基于`golag`编写，安装方便。同时，与`gitea`、`docker`、`kubernetes`搭配使用，是一种较为快速的实现基于`docker`的持续集成与继续交付的解决方案。`gitea`实现项目代码版本控制，`drone`提供基于`docker`形式的源码编译以及镜像化，通过邮件插件、消息插件等将相关流程信息发送给相关人员，然后，通过`kubernnetes`的`helm`包管理器自动化安装开发、测试环境，当环境安装完毕后，触发`robotframework`的自动化测试。测试流程结束后，将镜像发布服务到生产。  

整个的流程：通过`kubernetes`来分配开发、测试、预生产、生产环境，开发构建开发环境、测试编写自动化用例，当有版本迭代时，开发可以拉取接口自动化用例进行回归测试，通过后将版本镜像交给测试，进行新功能的测试。测试人员更新`kubernetes`中的测试环境版本，编写新功能的自动化用例。测试通过后，在预生产环境中进行版本构建，运行自动化测试、人工测试来验收。验收通过后，将镜像交付给运维来升级生产，同时进行功能验证。

> 当然，我们也可以构建灰度环境等来进行版本发布，以及一些其他 `kubernetes`应用场景

### Quick start
`drone`的安装参考[官方安装示例](http://docs.drone.io/installation/)，以下是我个人安装的`docker-compose.yml`文件
```shell
version: '2'

services:
  drone-server:
    image: drone/drone:0.8.5

    ports:
      - 8080:8000
      - 9000
    volumes:
      - /var/lib/drone:/var/lib/drone/
    restart: always
    depends_on:
      - db
    environment:
      - DRONE_OPEN=true
      - DRONE_HOST=http://192.168.200.157:8080
      - DRONE_GITEA=true
      - DRONE_GITEA_URL=http://192.168.200.157:3000
      - DRONE_SECRET=x25WM5TaTXGK5vQ0kgv
      - DRONE_DATABASE_DRIVER=mysql
      - DRONE_DATABASE_DATASOURCE=root:86LzseBTczbp@tcp(db:3306)/drone?parseTime=true

  drone-agent:
    image: drone/agent:0.8.5

    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_SERVER=drone-server:9000
      - DRONE_SECRET=x25WM5TaTXGK5vQ0kgv

  db:
    image: bitnami/mariadb
    ports:
      - 3306
    restart: always
    volumes:
      - ./mariadb:/bitnami
    environment:
      - MARIADB_ROOT_PASSWORD=86LzseBTczbp
      - MARIADB_DATABASE=drone
```
将官方的默认的`sqlite`数据库改成`mysql`数据库，需要注意的是将`gitea`的`url`改成你安装的`gitea`地址。对于`gitea`的安装，请参考[Gitea, 源码版本控制](http://localhost:1313/post/gitea/)一文。在`docker-compose.yml`文件的目录下，执行以下命令
```shell
[root@nodejs-200 drone]# docker-compose up -d
```
执行完毕后，会在该目录有以下文件
```shell
[root@nodejs-200 drone]# l
total 4
-rw-r--r-- 1 root   root   1011 Jun 11 16:34 docker-compose.yml
drwxrwxrwx 3 tomcat tomcat   20 Jun 11 16:36 mariadb
```
其中`mariadb`是数据库的目录，需要注意的是由于`bitnami/mariadb`镜像有个`bug`，需要在执行`docker-compose up -d`之前，创建好`mariadb`目录，同时将目录权限改成`777`来规避问题。 当然，你也可以使用其他数据库镜像来代替。接下来，我们通过`docker-compose`命令来确认安装成功，出现以下结果表示一切正常
```shell
[root@nodejs-200 drone]# docker-compose ps
        Name                    Command                State                                    Ports
-----------------------------------------------------------------------------------------------------------------------------------
drone_db_1             /app-entrypoint.sh /run.sh   Up             0.0.0.0:32777->3306/tcp
drone_drone-agent_1    /bin/drone-agent             Up (healthy)   3000/tcp
drone_drone-server_1   /bin/drone-server            Up             443/tcp, 80/tcp, 0.0.0.0:8080->8000/tcp, 0.0.0.0:32776->9000/tcp
```
最后，我们打开浏览器，输入安装`drone`的`url`，http://$IP:$port，就会看到drone的登陆界面，如下所示  

![avatar](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fsivoxgognj31d20g6gmj.jpg)
输入`gitea`用户名密码后，会跳转到使用界面，如下所示  

![avatar](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fsivscgi21j31hc090dg8.jpg)
这样，我们就安装好了`drone`，接下里就可以使用`drone`功能了。  

### How to use
`drone`的使用很简单，只要在源码项目根目录下创建一个`.drome.yml`，编写我们的工作流程即可。我们可以先看下官方提供的一个`demo`
```yaml
pipeline:
  backend:
    image: golang
    commands:
      - go get
      - go build
      - go test

  notify:
    image: plugins/slack
    channel: developers
    username: drone
```
`drone`根据`yaml`格式来配置我们的工作流，以上配置表明一个`pipeline`，该流程有两个步骤：`backend`和`notify`，其中`backend`是对`golang`项目源码编译的过程，当`backend`结束，接下来执行`notify`，也就是发送`slack`通知到`developers`频道，使用的用户名为`drone`。插件都是通过`docker`镜像执行，可以在[drone插件市场](http://plugins.drone.io/)来获取有用的镜像，同时，也可以根据需要编写插件，参考[官方提供插件编写实例](http://docs.drone.io/creating-custom-plugins-bash/)。  
接下来，我们通过一个项目来使用`drone`，首先，在`gitea`上创建一个`demo`项目，然后添加一个`gin`项目，如下所示
```shell
[root@nodejs-200 demo]# ls -a
.  ..  Dockerfile  .drone.yml  .git  main.go
[root@nodejs-200 demo]# cat Dockerfile
FROM centos
ADD app /opt/demo
EXPOSE 8080
CMD [ "/opt/demo" ]
[root@nodejs-200 demo]# cat main.go
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
[root@nodejs-200 demo]# cat .drone.yml
workspace:
  base: /go
  path: src/app

pipeline:
  build:
    image: golang
    commands:
      - go get
      - go build
      - echo "hello, world!"

  publish:
    image: plugins/docker
    repo: dhub.juxinli.com/demo
    username: panzhengming
    tags: [ 1.0, latest ]
    registry: dhub.juxinli.com

  wechat:
    image: clem109/drone-wechat
    corpid: corpid  # 改成自己微信的corpid，然后删除注释
    corp_secret: corp_secret # 改成自己微信的corp_secret，然后删除注释
    agent_id: agent_id # 改成自己微信的agent_id，然后删除注释
    title: ${DRONE_REPO_NAME}
    description: "Build Number: ${DRONE_BUILD_NUMBER} 部署成功. Check the results here: ${DRONE_BUILD_LINK} "
    msg_url: ${DRONE_BUILD_LINK}
    btn_txt: btn
    when:
      status: [ success ]
```
> 以上的微信通知需要注册企业微信号，对于微信通知设置，登录微信官网注册企业账号，之后学习官方微信的使用教程，在我的企业下获取corpid，在企业应用下注册一个应用获取AgentId、Secret后，最后在.drone.yml的wechat工作流中填写对应值即可。其他场景使用参考官方使用教程，不在此做介绍。

然后，在提交代码之前，我们需要在`drone`上开启项目的自动构建开关，打开后，我们提交到代码到仓库。你发现`drone`页面上该项目自动根据`.drome.yml`中配置项开始构建
![avatar](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fsixxcnaz5j31gt0et75x.jpg)

> 以上实例是下载默认的镜像，比如每次构建都需要下载对应的库，我们可以先构建好一个编译环境的镜像，以后直接通过该镜像来编译源码，当我们的库变更时，只要再次更新编译镜像即可，其他的则不需要变动。对于拉取公网镜像，可以将阿里云加速器配置到 `docker`的插件中，构建项目需要的镜像。

当以上流程结束后，你会得到一个镜像以及一个微信发送给你的构建结果消息。最后，我们在一台装有`docker`机器运行该镜像，需要注意的是映射端口，之后访问之前暴露出来的接口，出现以下结果表示构建成功。
```shell
[root@node1 ~]# curl http://localhost:8080/ping
{"message":"pong"}
```

### 应用场景
以上简单介绍了`drone`的`golang`的使用，对于其他语言的配置类似，可以根据需要使用相关插件，同时也可以根据诉求编写插件。接下来聊聊，我们可以通过`drone`来完成什么任务。在软件开发过程中，整个工作场景，有沟通`IM`、文档编写`markdown`、源码管理`gitea`、源码编译、持续集成`drone`或者`jenkins`、环境安装、自动化测试、问题单管理系统、上线发布、上线监控等。而这些场景我们通过`drone`来实现，`drone`实现的是粘合剂的作用，快速实现我们的`devops`原型。

> `IM` 工具 `rokcet.chat`、文档编写 `hugo`、源码管理 `gitea`、编译镜像 `docker`、运维机器人 `hubot`、基础环境 `kubernetes`、监控 `Prometheus`