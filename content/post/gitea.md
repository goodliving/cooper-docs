---
title: "Gitea, 源码版本控制"
date: 2018-06-18T15:25:10+08:00
draft: false
subtitle: "初次安装gitea"
bigimg: [{src: "/img/gitea.jpg", desc: "Gitea"}]
tags: ["linux", "docker", "git", "golang", "cicd"]
---
`Gitea`是一个自己托管的`Git`服务程序，它与`Github`、`Bitbucket`、`Gitlab`等比较类似，管理我们的源码管理。`Gitea`的首要目标是创建一个极易安装，运行非常快速，安装和使用体验良好的自建 `Git` 服务。对于测试人员来说，通过对`git`进行学习，更好的理解软件开发的过程中的源码管理，同时通过与一些持续集成工具，如`jenkins`、`drone`等，实现源码的编译的安装以及容器的`CI`，提高软件的交付质量。
<!--more-->
### Why Gitea
[gitea](https://docs.gitea.io/zh-cn/)安装十分方便，类似的[gitlab](https://docs.gitlab.com/)安装时需要安装的模块较多，对于一些故障排除需要一些相关知识。而`gitea`的安装只要安装一个二进制文件即可，十分方便。最重要的是，`gitlab`需要的资源较多，`gitea`则可以安装到一个[树莓派](https://baike.baidu.com/item/%E6%A0%91%E8%8E%93%E6%B4%BE/80427?fr=aladdin)机器上。

### Quick start
对于`gitea`的安装参考官网手册，有两种方式，源码编译以及二进制安装，同时作者已经提供了`docker`镜像。以下，我们通过[docker-compose](https://blog.csdn.net/chinrui/article/details/79155688)来快速安装，对于`docker`还不了解的话，可以阅读之前的文章[Docker, 主流的环境虚拟化工具](http://localhost:1313/post/docker_install/)一文。我们只要准备一个`docker-compose.yml`文件就可以安装好`gitea`，`docker-compose.yml`文件内容如下
```shell
[root@nodejs-200 gogs]# cat docker-compose.yml
version: '2'
services:
  web:
    image: gitea/gitea:1.3.2
    volumes:
      - ./data:/data
    ports:
      - "3000:3000"
      - "10022:22"
    depends_on:
      - db
    restart: always
  db:
    image: mariadb:10
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=123456
    volumes:
      - ./db/:/var/lib/mysql
```
以上配置用到两个镜像，一个是`gitea`的官方镜像，另一个是`mariadb`数据库镜像，之后配置镜像的端口映射以及数据库启动参数。之后，在该文件的目录运行`docker-compose up -d`，等待命令执行完毕，我们通过`docker-compose ps`来查看启动结果，出现下面结果表示执行成功
```shell
[root@nodejs-200 gogs]# docker-compose ps
   Name                 Command               State                       Ports
---------------------------------------------------------------------------------------------------
gogs_db_1    docker-entrypoint.sh mysqld      Up      3306/tcp
gogs_web_1   /usr/bin/entrypoint /bin/s ...   Up      0.0.0.0:10022->22/tcp, 0.0.0.0:3000->3000/tcp
```

> 对于`docker-compose`的安装，直接通过`pip`来安装，运行`pip install docker-compose`后，没有报错，那么`docker-compose`就被安装到我们机器上了。

### Set up
现在`gitea`已经安装到我们机器上了，打开浏览器，输入刚才机器的`ip`以及`3000`端口，如`http://192.168.137.2:3000`，然后，安装下面配置
![avatar](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fshhadbko2j30rn0c4aaw.jpg)
![avatar](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fshhb5b4iuj30ol0n4aba.jpg)
最后，点击`立即安装`，等待后台安装结束后，之后会跳转到`gitea`的登陆界面
![avatar](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fshhdfd0cbj30ys0dgmxb.jpg)
`gitea`没有默认登陆密码，第一个注册的用户就是管理员权限用户。当我们注册完用户后，我们可以创建一个项目来感受`gitea`的使用，对于熟悉`github`的用户，`gitea`十分相似。  

### git使用技巧
从命令行创建一个新的仓库
```shell
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin http://192.168.200.157:3000/panzhengming/test.git
git push -u origin master
```
从命令行推送已经创建的仓库
```shell
git remote add origin http://192.168.200.157:3000/panzhengming/test.git
git push -u origin master
```

> 将其中的 http://192.168.200.157:3000/panzhengming/test.git 地址换成你个人的仓库地址

### Todo

- `git`的经常使用的命令
- [Drone](http://docs.drone.io/zh/)的安装，基于`gitea`的持续集成

