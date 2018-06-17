---
title: "静态网站生成器Hugo"
date: 2018-06-17T18:49:53+08:00
draft: false
subtitle: "初次安装使用Hugo"
bigimg: [{src: "/img/hugo.jpg", desc: "Hugo"}]
tags: ["hugo", "golang", "nginx", "github"]
---

Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。本博客就是通过hugo生成，以下是介绍其安装使用过程。

<!--more-->

### 安装使用
1、二进制安装，推荐使用  
到[Hugo Release](https://github.com/gohugoio/hugo/releases)中下载对应的操作系统版本的二进制文件(windows系统的hugo.exe)，之后将文件目录添加到系统的环境变量中，之后就可以在cmd或者其他终端中输入`hugo -h`来查看命令使用详情。  

2、源码安装   
源码编译需要准备Go编译环境以及[Git](https://git-scm.com/)工具，参考博客[golang安装和配置](https://www.jianshu.com/p/b6f34ae55c90)，之后我们在cmd上输入`go version`，当显示版本号时安装成功。之后，在cmd或者其他终端上输入`go get -u -v github.com/spf13/hugo`通过go来下载安装hugo，一切顺利的话，会在`GOPATH`的`bin`目录下生成`hugo.exe`文件。  

> `go get`下载源码由于有些包网站可能被封导致下载失败，可以查看百度解决  

3、生成站点  
hugo生成站点比较简单，直接在cmd上输入`hugo new site mysite`就会在`mysite`目录下生成网站的全部模板文件。

4、站点配置  
在使用网站之前需要进行`theme`选择，在[hugo皮肤列表](http://www.gohugo.org/theme/)中选择一种喜欢的主题，以下是`Hyde`为例
```shell
$ cd themes
$ git clone https://github.com/spf13/hyde.git
```  
  
5、创建文章  
在站点根目录下，运行`hugo new post/new.md`后，会在`content`下生成`post`目录以及`new.md`文件，该文件就是我们需要编辑的，格式是`markdown`，对于`markdown`语法不熟悉的，可以查看[markdown语法介绍](https://www.appinn.com/markdown/)。  
打开编辑`post/new.md`  
```markdown
---
date: "2015-10-25T08:36:54-07:00"
title: "new" 
draft: true
---
###下面就是要输入的文章内容，上面是自动生成的
### Hello Hugo
this is a test post, you can see me!
 ```  

6、运行网站  
在你的站点根目录执行`Hugo`命令进行调试：  
```shell
$ hugo server --theme=hyde --buildDrafts
```
浏览器里打开：`http://localhost:1313`，就会看到刚才输入的内容。  

7、发布文章  
当我们写好文章后，就通过使用`hugo`来生成网站静态文件，会在根目录生成`public`目录，该目录下就是整个网站的文件，之后将配置一个`nginx`就可以通过其`IP`来访问你的博客。  

当你按照安装上面的操作成功后，你会发现你看不到你写的博客，这是正常的，你需要设置文件的`draft: false`或者删除该字段后即可。之后，重新按照上面操作，生成最新的静态文件后，将文件更新到`nginx`，最后你就会看到你刚才写到的内容。   

> 对于进阶使用，仔细查看对比下载的`theme`目录，会有惊喜

### 说明  
本博客是根据[hugo中文文档教程](http://www.gohugo.org/)安装完成，对于高级使用参考[jimmysong.io博客的hugo教程](https://jimmysong.io/posts/hugo-universal-theme-guide/)。

