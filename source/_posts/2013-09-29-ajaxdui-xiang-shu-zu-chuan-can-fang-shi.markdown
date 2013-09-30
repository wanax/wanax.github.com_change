---
layout: post
title: "ajax对象数组传参方式"
date: 2012-10-11 15:53
comments: true
categories: Tec
---
项目中一个页面需要的功能是动态添加信息列表，传至后台添加数据库。用的方式是ajax+struts2，因为struts本身的特性，在ajax的data里直接写数组名java后台是无法正常接收到的，最简单的理解便是它无法智能的一一匹配数组里的每一个值，所以我们在前台开始发action请求前便要组装好容易接收的数据格式。

最简单的方法是直接拼接字符串，后台split出来，但有时候用的不是简单的字符串数组而是对象数组，这时候就需要一些其它方法了。

<!--more-->

![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/1349941967_9520_zpsbbaf2803.png)

如上图所示，每次提交是权限一个对象，然后关于资源对象是动态添加与删除的，很明显，这就需要一个对象数组来装这些东西

前台页面的动态添加是这样的：
<script src="https://gist.github.com/wanax/6752390.js"></script>  
		
然后就是对对象数组的拼装与发送了
		
<script src="https://gist.github.com/wanax/6752394.js"></script>