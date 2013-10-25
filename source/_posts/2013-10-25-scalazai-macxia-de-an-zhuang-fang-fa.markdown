---
layout: post
title: "Scala在Mac下的安装方法"
date: 2013-10-25 14:36
comments: true
categories: Tec
---
###第一种方法

1.访问scala的官网[这里](http://www.scala-lang.org/)下载最新版的scala。

2.解压缩文件包，可将其移动至`/usr/local/share`下

``` 
mv /download/scalapath /usr/local/share
```

3.修改环境变量，在mac下使用`sudo su`进入管理员权限，修改配置文件profile，

```
vim /etc/profile
```

在文件的末尾加入

```
export PATH="$PATH:/usr/local/share/scala/bin"
```

`:wq!`保存退出，重启终端，完成scala的配置安装。

###第二种方法

如果本机有安装Ruby的话则安装更加简单，可以借助Homebrew进行安装。
首先安装Homebrew

>ruby -e “$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)”


再进行scala的安装`brew install scala`

大功告成~
