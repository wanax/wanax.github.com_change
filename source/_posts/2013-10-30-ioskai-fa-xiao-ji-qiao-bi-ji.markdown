---
layout: post
title: "iOS开发小技巧笔记"
date: 2013-10-30 16:03
comments: true
categories: Tec
---

###字体粗度的设置
```objc
button.titlelabel.font = [uifont boldSystemFontOfSize:xx];
```

###UINavigation中间隔button的设置
```objc
	UIBarButtonItem *fontListBt = [[UIBarButtonItem alloc]
                                    initWithTitle:@"Font"
                                    style:UIBarButtonItemStyleBordered
                                    target:self
                                    action:@selector(rightBtClicked:)];
    UIBarButtonItem *fontSizeBt = [[UIBarButtonItem alloc]
                                     initWithTitle:@"Size"
                                     style:UIBarButtonItemStyleBordered
                                     target:self
                                     action:@selector(popoverFontSize:)];    
    UIBarButtonItem *flexibleSpace = [[UIBarButtonItem alloc] 
			    initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace
			    target:nil
			    action:nil];
    self.toolBar=[[UIToolbar alloc] initWithFrame:CGRectMake(0,0,SCREEN_HEIGHT-320,44)];
    [self.toolBar setItems:[NSArray arrayWithObjects:flexibleSpace,fontSizeBt,fontListBt, nil]];
    [self.view addSubview:self.toolBar];
```

<!--more-->

###UILabel的圆角设置
```objc
    label.layer.cornerRadius=3.0;
    label.layer.borderWidth=1.0;
```
###UILabel自动换行
```objc
    label.lineBreakMode=NSLineBreakByCharWrapping;
    label.numberOfLines=2;
```
###UILabel行间距设置的取巧解决方案

####将原本要放入的UILabel替换成同大小的UIWebView
####大致思路是使用webView里内置的JS引擎解析插入的JS代码，以达到调整行间距的目的。
####缺点为若嵌入在Cell中，对Cell的点击事件这一区域无法正常响应，现在的解决思路是对此webView添加独立的事件监听，达到同等目的
```objc
	cell.webView.backgroundColor = [UIColor clearColor];
    cell.webView.opaque = NO;
    cell.webView.dataDetectorTypes = UIDataDetectorTypeNone;\
    [(UIScrollView *)[[cell.webView subviews] objectAtIndex:0] setBounces:NO];//禁止webView的滚动
    //16px/18px:前者为行间距，后者为字体大小
    NSString *webviewText = @"<style>body{margin:0;color:#967c6c;background-color:transparent;font:16px/18px Custom-Font-Name}</style>";
    NSString *htmlString = [webviewText stringByAppendingFormat:@"%@", str];
    [cell.webView loadHTMLString:htmlString baseURL:nil];
```
###URL路径的UTF-8转码
####在一些通过url路径显示图片的页面中，会出现部分图片无法正常显示的问题。有一种可能是抓包分析可以看出这些无法正常显示的图片的url中有中文字体的嵌入，这便涉及到了URL转码的问题，可根据具体的字体编码设置进行转码，大致code如下
```objc
NSURL *url = [NSURL URLWithString:[strUrl stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding]];
```









