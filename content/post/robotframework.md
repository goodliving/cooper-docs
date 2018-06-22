---
title: "接口自动化测试工具"
date: 2018-06-20T15:10:35+08:00
draft: false
subtitle: "记录一次robotframe工具容器化的使用过程"
bigimg: [{src: "/img/robotframework.jpg", desc: "robotframework"}]
tags: ["tester", "interface", "gitea", "git"]
---

[Robot Framework](http://robotframework.org)是一个基于`Python`可扩展地关键字驱动的测试自动化框架，对于测试人员来说，不需要编写脚本或者插件，同时也有`GUI`界面，相对来说，用例编写比较容易。最主要的是网上关于其工具的使用的教程比较多，对于一些常见问题都有十分详细的解决方案。它一个跨平台的工具，可以在`linux`中使用，而这正是`docker`所需要的环境。
<!--more-->

### Quick start
`robotframework`的安装比较方便，首先在本机上安装`python`，可以从[python官网](https://www.python.org/)下载对应的版本，在安装好`python`后(在安装过程中选择`add path`，这样我们就不需要手动添加环境变量)，打开一个`cmd`或者`powershell`，输入`pip --version`，显示版本则表示一切安装正常，接下来，我们通过`pip`来安装`robotframework`，在`cmd`下执行以下命令
```shell
pip install robotframework
```
等待安装结束后，接下来，我们安装`robotframework-ride`图形库，由于其是基于`wxpython`开发的，所以我们还需要安装`wxpython`环境，从[wxpython下载页面](http://sourceforge.net/projects/wxpython/files/wxPython/2.8.12.1/)中选择对应的操作系统以及`python`版本下载安装，最后，在`cmd`中输入`pip install robotframework-ride`，执行完毕后，会在`python`的安装路径`Script`目录下生成一个`ride.py`，由于之前我们已经将其添加到环境变量中，所以我们可以在任何目录下输入`ride.py`来打开`robotframework`的图形化界面。
![avatar](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fshps3z40gj31hc0rn0v6.jpg)

> 对于`python`的软件安装环境，推荐安装 `virtualenv` 库，之后使用 `virtualenv` 来隔离软件安装的环境，之后通过 `pip freeze` 来生成对应的软件需要的 `requirements.txt` ，不會安裝其他不必要的第三库。具体的参考 [virtualenv使用介绍](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432712108300322c61f256c74803b43bfd65c6f8d0d0000)

关于`robotframework`的使用教程参考[robotframework初次使用](https://www.cnblogs.com/dreamyu/p/6856841.html)。

### robotframework的容器化
现在有这样一个场景，我们在`robotframework`写好用例，将其放在`gitea`上进行用例版本迭代管理，对于不了解`gitea`的，参考之前的文章[Gitea, 源码版本控制](http://localhost:1313/post/gitea)。当版本有个迭代，我们需要下载用例仓库，切换到对应的分支，然后执行用例进行全回归测试，等待结束后查看结果。  

这样操作了10次之后，我们可能会写一个自动执行对用版本用例的脚本。当然，这样也可以，但是没有集成到我们现有版本测试流程中来，正确的姿势应该是当我们通过持续集成工具完成软件版本环境的安装后，通过一个`webhook`执行我们的用例，最后执行完毕后，发邮件或者发微信等方式将执行情况发送给我们。

> 上面的操作前提需要我们将环境安装集成到持续集成中，如果没有自动化安装环境，那么上面所有的自动执行就跟手动执行区别不大。

对于上面的场景有个很容易实现的解决方案，就是将`robotframework`执行用例的环境容器化，然后通过`Drone`持续集成工具，完成环境自动安装、用例自动执行以及发送通知等，以及后续的生产环境自动更新服务。以下描述个人的一些做法，对于`robotframework`的容器化是在用例的项目添加一个`Dockerfile`文件，目录结构如下所示
```shell
[root@node1 Demo]# ll
total 12
-rw-r--r-- 1 root root 736 Jun 20 18:00 01 login.txt
drwxr-xr-x 2 root root  27 Jun 20 18:00 Dashboard
-rw-r--r-- 1 root root   0 Jun 20 18:00 Dockerfile
-rw-r--r-- 1 root root 131 Jun 20 18:00 __init__.robot
-rw-r--r-- 1 root root   0 Jun 20 18:01 requirements.txt
-rw-r--r-- 1 root root 127 Jun 20 18:00 var.py
```
其中，`Dockerfile`是构建`robotframework`镜像文件，`requirements.txt`是用例所需要的库列表，`Dockerfile`文件内容如下
```shell
FROM python
RUN mkdir /opt/demo -p
WORKDIR /opt/demo
ADD . /opt/demo
RUN pip install -r requirements.txt
CMD ["robot", "/opt/demo"]
```
主要过程是拉起一个`python`环境的镜像，将用例文件拷贝到镜像中，安装`requirements.txt`对应的库，最后执行我们的用例。那么就发现一个问题，不能执行特定版本的用例，解决的方式是通过`Drone`构建过程中设置版本来解决。

> 以上只是简单介绍了一个场景描述，对于用例执行如何统计，以及结果展示还需要进一步的修改。

### 深度思考
关于接口自动化工具市面有很多种，而对于自动化工具与软件开发过程如何深度结合没有详细的描述。在此，希望通过容器`docker`和`kubernetes`结合来构思一个新的自动化方案，提高软件开发过程中的自动化程度。目前，对于自动化测试过程中的问题：

- 接口文档不全
- 环境无法自动安装
- 如何写自动化用例
- 持续集成与持续交付

### 自动化工具
以下统计一些常用的自动化工具

- [robotframework](http://robotframework.org/)
- [httprunner](http://cn.httprunner.org/)
