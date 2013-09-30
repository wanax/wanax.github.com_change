---
layout: post
title: "垂直发布—Httpclient模拟登陆获取页面信息"
date: 2012-02-13 18:24
comments: true
categories: Tec
---
过年回了趟老家，回来后寒假还有好大一截，闲着也是没事就这两天初步了解了下垂直发布的东西，收获一点点吧，感觉离预期还差很远。趁着还有印象写下来，步步为营呗。网上搜索，看到很多模拟登录都是用Httpclient的，另外一个是HttpURLConnection。
HttpURLConnection是java的标准类，HttpClient 是 Apache Jakarta Common 下的子项目，用来提供支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。因为是初学，就图着简单易用的目的选择了Httpclient，感觉还是挺容易上手的，不过这两天遇到的问题也不少。

<!--more-->

既然是模拟正规的登录，那首先要抓包，通过一次正常的登录来获取表单提交的信息，既而自己在后台把相应的参数填好发出。

网上的抓包工具不少，刚开始也下了好几个，像Wireshark，SmartSniff都是好多人推荐的。开始用的Wireshark，确实好多功能，还有过滤什么的，但贪图方便用着用着就转去SmartSniff了。就是准备登录时点一下开始键，然后登录，再停止抓包，挨个找一下这段时间的TCP，根据登录网址，很容易就找到目标信息了。

就像这样
![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/100001114023512_zpsa214f4b2.jpg)

可以看到蓝色字的最后一行就是该网站正常登录的表单提交信息了。顺便一提，上面的蓝色是浏览器发送的请求，下面的粉色则是服务器返回来的数据，一般情况下HTTP/1.1 200就表示登录正常了，这是一个判定标准。

因为开始4.x网上用的人不多，出的时间不久的缘故吧，所以搜到的信息很多时候都是3.x的写法，自己也就照葫芦画瓢了，后来看到说4.x基本上是重写的，跟之前的3.x有很大不同，所以第二天又重新用4.x试了一遍，写法上稍微复杂了点，但方式都差不多感觉。

这里我是用的biketo网站做的小白鼠，平常喜欢骑下单车再估计这网站也不会做得很有技术难度估计能好模拟一点就用这个了，哈哈。

<script src="https://gist.github.com/wanax/6752363.js"></script>
	
首先正常登录时，发送的post是这个


<script src="https://gist.github.com/wanax/6752367.js"></script>

然后就是模拟

<script src="https://gist.github.com/wanax/6752368.js"></script>

这里要注意一下

client.getParams().setParameter(HttpMethodParams.USER_AGENT
的那行代码，因为我们是用的java来发的，不是浏览器，所以默认的user-agent是Jakarta Commons-HttpClient/3.0.1，很多服务器看到这个就直接不鸟你了，不过biketo嘛，没有这层过滤，呵呵。但一般情况下还是要伪装成浏览器的信息，这样才容易得手~
这个样子发出的包便是

<script src="https://gist.github.com/wanax/6752370.js"></script>

对比之下会发现比正常的浏览器似乎少了很多信息，转了一篇文章写得很详细，大家有兴趣自己看下吧http://blog.csdn.net/mxiaochi/article/details/7252582
post是请求，get便是验证下登录有没有成功，找一个只有登录才能访问的页面放进GetMethod 里，发送请求，然后outprint出来，贴到记事本里用浏览器打开就可以验证一下啦
这是同样功能4.x的代码，有兴趣也可以试一下

<script src="https://gist.github.com/wanax/6752373.js"></script>


前面注释的一大段就是模仿3.x中缺少的那些浏览器信息了，可以根据正常登录抓回来的信息自己慢慢匹配。
刚刚说的这些都是些很简单的东西，仔细看看不到一天就可以大概掌握了，下面的就是些自己遇到的问题。
我第二个尝试的是一个房地产的网站，发现它的页面是会自动跳转的，再请求的话则会退回登录页面很是苦恼。

<script src="https://gist.github.com/wanax/6752376.js"></script>


抓包是这样的

<script src="https://gist.github.com/wanax/6752381.js"></script>

可以看到是能正常取回cookie的，但当我尝试get去别的页面的时候都被弹到登录页面了，可是我已经是带上cookie访问的了，不知道这有没有扯上网站安全之类的，这方面没什么经验

<script src="https://gist.github.com/wanax/6752384.js"></script>

或是用session？个人猜测。













