---
layout: post
title: "使用Octopress搭建个人博客"
date: 2013-11-14 16:45
comments: true
categories: Tec
---

![image](/images/tec/octopress-header.png)

十月份用octopress搭建了一个个人博客，现在把大概的流程介绍一下，常用的几个命令也在这里记下来。

[Octopress](http://http://octopress.org/)是个利用[Jekyll](https://github.com/mojombo/jekyll)引擎开发的博客系统。它将生成的静态页面push到我们自己的github上，可以很好的在github page上展示。号称程序员博客逼格利器

>A blogging framework for hackers.

<!--more-->

###首先是搭建发布环境

Octopress需要Ruby环境，前几天自己电脑上装的Ubantu10.0自带的Ruby1.8.7试过了不行，起码要更新到Ruby1.9.1吧，官方的建议是1.9.3.

安装Ruby有两个方法，一个是直接去官网下载Ruby包，解压缩，安装。另一个是使用RVM(Ruby Version Manager)来负责安装和管理Ruby的环境。

在Mac电脑上我直接下载安装了Rvm

```
curl -L https://get.rvm.io | bash -s stable --ruby
```

```
rvm install 1.9.3
rvm use 1.9.3
rvm rubygems latest
```

参见[Installing Ruby With RVM](http://octopress.org/docs/setup/rvm/)

这样便简单搞定，但Ubantu上rvm死活搞不定，网上搜了下使用第一种方法直接安装算完了，也不麻烦。

###然后便是安装Octopress

先把Octopress从网上down下来

[Octopress](https://github.com/imathis/octopress)

电脑上有git的话简单一点

```
git clone git://github.com/imathis/octopress.git octopress
```

解压进入项目目录安装相关依赖

```
cd octopress
gem install bundler
rbenv rehash
bundle install
```
参见[Octopress Setup](http://octopress.org/docs/setup/)

这几步是把发布Octopress的环境搞定，还没有正式开始搭建Octopress，我在Mac上发布的博客，在Ubantu从git上check下来，也要这么来一遍才能正式使用Octopress。

最后安装默认的Octopress 主题。

```
rake install
``` 

###接下来是配置Octopress

主要是对`_config.yml`进行修改，其它的话自行研究吧。

具体配置参见[Configuring Octopress](http://octopress.org/docs/configuring/)写得很详细了。

搞定配置后可以先大体的看下样式

```
rake generate//通过引擎生成静态网页
rake preview//本地预览
```
[localhost:4000](localhost:4000)进行本地预览

###觉得差不多了就可以把Octopress发到github上了

首先需要在GitHub上创建一个仓库，并将仓库名称按照：username.github.com的方式命名。待发布完毕可以直接使用`http://username.github.com`来访问博客。同时可以把博客的源码放到source分支下，并把生成的内容提交到master分支。这样随便换到哪里的电脑上都可以check下来接着写了。

第一步是对我们新建的目录进行建立与初始化，会提示输入你的git的账号和密码，一次确认完毕以后你每次提交都会跑到你确认的git账号下的github.com/username/username.github.com下了。

本地生成一下静态Html发上去就妥了

```
rake generate//生成
rake deploy//发布
```

###最后说下写博客的常用命令

```
rake new_post["title"]
```

这是新建一篇文章，存放在`source/_posts`目录下面，使用的是markdown语法，具体规则不麻烦，大家可以自己谷歌下。Mac下用Mou进行编辑很方便，Linux的话线上编辑会好一点，chrome有不少插件，其中一个叫Ma‘de，好像是作者用以表达对母亲的爱…

```
 rake generate
 git add .
 git commit -am "Some comment here." 
 git push origin source
 rake deploy
```

使用上面几条命令生成与发布博客，中间的三条git命令是把博客的源码放到source的分支上，以便在不同的电脑上编辑博客。

参见[Blogging Basics](http://octopress.org/docs/blogging/)



























