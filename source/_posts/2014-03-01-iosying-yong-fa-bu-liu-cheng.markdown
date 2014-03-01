---
layout: post
title: "iOS应用发布流程"
date: 2014-03-01 17:20
comments: true
categories: Tec
---

##iOS应用发布流程

###概述

整个发布流程有四个关键词，*Certificates*，*Identifiers*，*Devices*与*Provisioning Profiles*.

它们的相互关系如下图所示，

![image](/images/tec/iOSPub/pic_1.png)

简单说来便是用户身份验证书（Certificates），App应用身份ID（Identifiers），可用于开发的Devices三者合一生成一份Provisioning Profile，并将此profile下载到本机双击安装后就自然可以在xcode中找到了(注意左上角的“GooGuu”在配置发布时应将Target与Project的设置调未相同选项，以避免发布失败)。

<!--more-->

![image](/images/tec/iOSPub/pic_3.png)



###步骤分解

####一.Certificates证书申请

付费申请到开发者账号后首先要做的便是使用它来生成一个开发或者发布的证书，这个证书是App发布的起点，有了它便表明你是名已经被苹果公司认证过的开发人员并有权力发布自己开发的App到AppStore上去。

[iOS开发者申请发布证书](http://my.oschina.net/joanfen/blog/133624)

证书申请的步骤这位姐姐讲解地很详细了，不需要再一一赘述，看到第J步--*下载证书，双击安装*便可以停一下，先来看Identifier。

####二.Identifiers-App ID的申请

Certificates是表明你开发者身份的东西，花了近100的大洋肯定不可能只满足用来开发一款App，而是会进行多方面的尝试，这时候就需要通过App ID来对我们开发出的各种应用进行唯一的标示。

可以简单的理解为Certificates的苹果公司给你的身份证，而App ID是你给自己开发的App的一个身份证。

![image](/images/tec/iOSPub/pic_4.png)

####三.Devices的用处

开发的App可以在Xcode自带的模拟器下运行调试，想要进行真机测试的话则需要用到Devices的配置选项了。

当你将外置设备连入苹果电脑，进入Xcode的Window->Organizer->Devices中便会发现自己的设备名称：

![image](/images/tec/iOSPub/pic_5.png)

现在的Xcode已经集成了添加Device的功能，点击连接后的设备名称，你会在右边的窗口中发现诸如“添加该设备到开发机”之类的选项。

![image](/images/tec/iOSPub/pic_6.png)

之后到Device里简单的验证一下便可以了，你会发现自己的设备序列号已经添加到了该列表里面。

这个Devices列表按照第二张图的样子看应该是会在最后跟AppID与Certificate打包在一起来生成Profile文件，这样开发者身份信息，App的身份信息，可用测试的设备信息都可以随时的调用。

####四.Provisioning Profiles的生成与使用

还是继续从第K步--*生成证书对应的provision File* 看刚刚那位姐姐的讲解吧。

[iOS开发者申请发布证书](http://my.oschina.net/joanfen/blog/133624)

###发布

这样一步步下来，一份完成的Profile便拿在了手上，分为Development和Distribution两种，分别用于开发与发布的使用。

还不清楚的话可以看下这位哥儿们的讲解，更加的专业与细致。

[关于Certificate、Provisioning Profile、App ID的介绍及其之间的关系](http://www.cnblogs.com/cywin888/p/3263027.html)

接下来还剩下两步工作，一个是本地二进制文件的打包验证，一个是苹果网站的新App申请或者旧App版本更新的申请。

####1.苹果网站上的相关工作

我们去到[App Product](https://itunesconnect.apple.com/WebObjects/iTunesConnect.woa)这里，在网站上对App进行配置。

大致可以理解为通知苹果公司我将会有一个App通过Xcode进行上传，你要注意接收啊之类云云。

如果之前已经有旧的版本了那很简单，进入“Manage Your Apps”的页面，点击你要更新的App进入一个新的页面，里面有个Button叫“Add Version”点击进入，有一个流程页面，唯一需要注意的是*填写的新的版本号需要跟本地项目的Target->General的下图位置处的版本号填写相同*，不然在最后打包完毕验证时会发生错误导致无法提交。

![image](/images/tec/iOSPub/pic_10.png)

![image](/images/tec/iOSPub/pic_9.png)

如果是第一次发布App则稍微有点麻烦，具体的还是参见刚刚那位姐姐的文章吧...

[iOS 发布应用程序到App Store](http://my.oschina.net/joanfen/blog/133642)

####2.本地Xcode的相关工作

将发布设备调节到下图的位置，前提是没有外置的设备连在苹果机上，其实连上也无所谓，只是文字显示不一样。

![image](/images/tec/iOSPub/pic_7.png)

然后点击Product->Archive来静待二进制文件的生成吧。

生成后会自动跳转到Window->Organizer->Archives，然后分别对打包好的App进行验证与最后的提交，就是这样了~

![image](/images/tec/iOSPub/pic_8.png)