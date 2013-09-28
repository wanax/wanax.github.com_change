---
layout: post
title: "struts跳转控制笔记"
date: 2012-01-04 17:08
comments: true
categories: Tec
---
###当`namespace="/login"`时
===

子路目中：有`login/`   **no action mapped for**

无`login/`   **no result defined**

根目录中：    **no result defined**

**no action mapped for**

<!--more-->

###当`namespace="/"`时
===

子目录中：      **可跳转，无样式，URL重复发送**

**no result defined**

根目录中：    **no result defined**

跳转成功

结果跳转位置为“namespace+result”

===

###struts.xml 匹配action方式：
===
1.当各目录jsp页面发送action时，系统首先在其加上所在目录                                               名，与namespace匹配。

`namespace=“/”`与不写有区别，不写则匹配所有，写等同于`“/test”`之类，`“/”`表示根目录

故当`namespace=“/test” `时，根目录的action应写`test/*.action，所发出的全称即"test/*.action"`，即可找到对应struts，但服务器会记录信息，使URL叠加，不推荐使用。

另：同子目录下发`action="*.action"，会自动加上对应文件名，即"test/*action"`，可与namespace匹配。

故总体来说，推荐使用重定向的方式，不论再哪级目录下，均返回根目录对应配置文件填写action

ex.   二级目录下，可写成`"../../test/test/*action"`重新定向。

在匹配过程中有时候结果返回页面会发现css全部无法加载，此为因为重定向后URL改变，对应不上原来的css加载路径，此时应应变对应尝试各种更改action的发送URL。

result 前加`/` 表返回根目录，再依具体位置写目录

通配符的使用 ex. `action="jump*"    result=/test{1}/test{1}Success.jsp`