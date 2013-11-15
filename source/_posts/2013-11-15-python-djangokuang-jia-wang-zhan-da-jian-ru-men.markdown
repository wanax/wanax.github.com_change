---
layout: post
title: "Python Django框架网站搭建入门"
date: 2013-11-15 16:22
comments: true
categories: Tec
---
最近学习了下python，传说中的pyhon的框架比关键字还多果然名不虚传。想用pyhon搭个网站练练手，上网一搜果然一大把。看了下推荐各有利弊，但Django的文档比较全面，虽然说有点封闭，但也没要求多高，感觉拿来上手正合适不过了。

[Django 中文文档](http://django-chinese-docs.readthedocs.org/en/latest/index.html)

这篇文章主要是个学习笔记的作用，就是根据上面的网站一步步搞起的。

Django是一个MVC框架，相比于Java的MVC SSH简单的不是一点半点，想之前搭SSH还通了个宵，现在看看Django的MVC简单到只有三个python文件，跟着教程一步步搭起来回过头才想起来这特么也是MVC的框架啊，自不待言。

![image](/images/tec/python-django.png)

上图是一个对Django MVC的简单示意，很容易可以看出，`view.py`是主管视图部分的，`model.py`主管模型层，遗憾没有个`control.py`，哈哈，`url.py`控制跳转，是为controller。

现在便一步步开始吧。

<!--more-->

##一.安装

上周（November 6, 2013）Django才刚更新到1.6，[Django 1.6 release notes](https://docs.djangoproject.com/en/dev/releases/1.6/)这里是一些新特性的介绍。

因为中文文档还是介绍的1.5的安装过程，所以先按着1.5的版本来吧。需要注意的是Django适用`python 2.6.5 `到 `2.7` 的所有 Python 版本，还具有 `3.2` 和` 3.3`版本的实验性支持，因为python3相对于2.x来说有了一些语法与库的更新，所以在版本选择上还是需要注意一下的，就比如在2.x中有关http请求的lib放在`httplib`中，而在python3中则在`urllib`中，倒是有个开源第三方库叫`httplib2`，看一些教程都推荐在python3中使用这个第三方库，是因为有诸如缓存之类的特性。

Django的安装不麻烦，[DownLoad](https://www.djangoproject.com/download/)去这里下载压缩包，解压完进入文件目录执行

```
sudo python setup.py install
```
就可以了。

可以终端进入python shell验证一下

```
>>> import django
>>> print(django.get_version())
1.5
```

##二.项目启动

对于一个Django项目有两个基本的概念

1. project，这是对整个具体的项目的称号，我们一个网站便是一个project
2. app，相对于一个project可以细分很多app，也就是功能块的意思，我们可以建立很多app，并可以在/project/settings.py进行设置，也就是号称的即插即用。

开启终端，定位到一个合适的目录，键入第一行命令，生成我们的第一个Django项目

```
django-admin.py startproject mysite
```
`mysite`的名字便对应着project，这个看个人意志了。

妥当了后便是一个初步的网站架构

>mysite/
>>    manage.py

>>    mysite/

>>>    __init__.py

>>>    settings.py

>>>    urls.py

>>>    wsgi.py

这是一个基本的目录层次

`manage.py`是个命令行工具，可让你以各种方式与该 Django 项目进行交互，`settings.py`是对整个网站的一些基本配置，`urls.py`便是最外层的控制跳转，这里我们可以直接定位到具体的处理方法，但一般还是建议定位到相关的app，然后再让app里的`url.py`进行具体的跳转控制。`wagi.py`是一个对公共网关接口的配置，比如说放到新浪SAE的时候我们需要对这个文件进行编写配置。

初步介绍就是这样，shell定位到第一层的`mysite`上，输入

```
python manage.py runserver
```


去`http://127.0.0.1:8000/`进行访问，便可以看到初步的效果了。

##三.APP激活

项目启动成功后我们可以进入app的建立与关联。

```
python manage.py startapp polls
```

使用上面的命令在根目录建立一个app。

>polls/
>>    __init__.py

>>   models.py

>>    tests.py

>>    views.py


初始的app一般是这样的目录结构，一般还要自己新建一个`url.py`以便接过在根目录下`url.py`跳转控制的大旗~

`model.py`便是模型层中相关的具体模型了，`view.py`顾名思义，在通过`url.py`控制跳转到这个文件后对请求做具体处理，一般传参为`request`而返回一个`HttpResponse`。

进入根目录下面的`settings.py`对app进行插入

```
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    # 'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'polls',
)
```

使用

```
python manage.py syncdb
```
告诉项目对app进行同步安装。至此，一个初步的app便配置成功了。

##四.APP中编写第一个视图

让我们编写第一个视图。打开文件 polls/views.py 并在其中输入以下 Python 代码

```py
from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world. You're at the poll index.")
```

在 Django 中这可能是最简单的视图了。为了调用这个视图，我们需要将它映射到一个 URL – 为此我们需要配置一个URLconf 。

在 polls 目录下创建一个名为 urls.py 的 URLconf 文档。 你的应用目录现在看起来像这样

>polls/
>>    __init__.py

>>    admin.py

>>    models.py

>>    tests.py

>>    urls.py

>>    views.py
    
在 polls/urls.py 文件中输入以下代码：

```py
from django.conf.urls import patterns, url
from polls import views
urlpatterns = patterns('',
    url(r'^$', views.index, name='index')
)
```

这是一个简单的正则匹配，大概意思便是进入根url的话则直接去执行`view.py`里的`index`方法，便是我们刚刚定义过的那个函数。

下一步是将 polls.urls 模块指向 root URLconf 。在 mysite/urls.py 中插入一个 include() 方法，最后的样子如下所示

```py
from django.conf.urls import patterns, include, url
from django.contrib import admin
admin.autodiscover()
urlpatterns = patterns('',
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', include(admin.site.urls)),
)
```

现在你在 URLconf 中配置了 index 视图。通过浏览器访问 http://localhost:8000/polls/ ，如同你在 index 视图中定义的一样，你将看到 “Hello, world. You’re at the poll index.” 文字。

更多的一些视图编写方法请参见[这里](http://django-chinese-docs.readthedocs.org/en/latest/intro/tutorial03.html)

这里只是简单的介绍了下的客户请求服务器控制跳转处理请求并返回http对象的过程，并没有介绍如何搭配数据库模型层的运用，想进行细致的学习请移步[Django 中文文档](http://django-chinese-docs.readthedocs.org/en/latest/index.html)


















