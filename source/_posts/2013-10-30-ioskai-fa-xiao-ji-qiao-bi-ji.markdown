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

###获取文本的长宽尺寸
```objc
CGSize constraint = CGSizeMake(CELL_CONTENT_WIDTH - (CELL_CONTENT_MARGIN * 2), 20000.0f);  
NSDictionary * attributes = [NSDictionary dictionaryWithObject:[UIFont systemFontOfSize:FONT_SIZE] forKey:NSFontAttributeName];
NSAttributedString *attributedText =[[NSAttributedString alloc]
									     initWithString:text
									     attributes:attributes];									     
CGRect rect = [attributedText boundingRectWithSize:constraint
								options:NSStringDrawingUsesLineFragmentOrigin
								context:nil];
```

###NSNumberFormatter的一些使用方法

[学习一](http://objc.toodarkpark.net/Foundation/Classes/NSNumberFormatter.html)
[学习二](https://developer.apple.com/library/ios/documentation/cocoa/Conceptual/DataFormatting/Articles/dfNumberFormatting10_4.html)

现在主要是对setPositiveFormat的使用总结，比较常用的有下面的场景

```objc
if([obj[@"unit"] isEqualToString:@"1000.0"]){
	    [formatter setPositiveFormat:@"$#,###0.00"];//大整数三位一个逗号+小数取舍
	    //还可以添加诸如￥的字符，若要添加0需‘0’包裹
	}else if([obj[@"unit"] isEqualToString:@"%"]){
	    [formatter setPositiveFormat:@"0.00%;0.00%;-0.00%"];//百分数小数取舍
	}else if([obj[@"unit"] isEqualToString:@"1.0"]){
	    [formatter setPositiveFormat:@"##0.0"];//一般浮点的小数取舍
}
NSMutableDictionary *dic=[[[NSMutableDictionary alloc] init] autorelease];
for(id valueData in obj[@"array"]){
	   [dic setValue:[formatter stringFromNumber:valueData[@"v"]] forKey:valueData[@"y"]];
}
```

```objc
NSNumberFormatter *formatter=[[[NSNumberFormatter alloc] init] autorelease];[formatter setPositiveFormat:@"00"];//年份补足的用法，表示至少两位整数。不足左添0
NSMutableArray *temp=[[[NSMutableArray alloc] init] autorelease];
for(int i=[rangeDic[@"begin"] integerValue];i<=[rangeDic[@"end"] integerValue];i++){
	    [temp addObject:[formatter stringFromNumber:[NSNumber numberWithFloat:(6.0+i)]]];
}
```
###ios7下Label自适应

```objc
    NSString * tstring =@"UILabel与breakmode与之前设置的完全ewqewqewqewqewqewqewqewqewqew一致sizeWithFont:font constrainedToSize:size与breakmode与之前设置的完全ewqewqewqewqewqewqewqewqewqew一致sizeWithFont:font constrainedToSize:size与breakmode与之前设置的完全ewqewqewqewqewqewqewqewqewqew一致sizeWithFont:font constrainedToSize:size";
    //高度估计文本大概要显示几行，宽度根据需求自己定义。 MAXFLOAT 可以算出具体要多高
    CGSize size =CGSizeMake(300,2000);
    // label可设置的最大高度和宽度
    //    CGSize size = CGSizeMake(300.f, MAXFLOAT);
    //    获取当前文本的属性
    NSDictionary * tdic = @{NSFontAttributeName:[UIFont fontWithName:@"Heiti SC" size:12.0], NSForegroundColorAttributeName:[UIColor blueColor]};
    NSLog(@"%@",tdic);
    //ios7方法，获取文本需要的size，限制宽度
    CGSize  actualsize =[tstring boundingRectWithSize:size options:NSStringDrawingUsesLineFragmentOrigin  attributes:tdic context:nil].size;
    NSLog(@"%@",NSStringFromCGSize(actualsize));
    // ios7之前使用方法获取文本需要的size，7.0已弃用下面的方法。此方法要求font，与breakmode与之前设置的完全一致
    //    CGSize actualsize = [tstring sizeWithFont:font constrainedToSize:size lineBreakMode:NSLineBreakByCharWrapping];
    //   更新UILabel的frame
    UILabel * testlable = [[UILabel alloc]initWithFrame:CGRectMake(10,20,200,20)];
    testlable.numberOfLines =0;
    testlable.font = [UIFont systemFontOfSize:12];
    testlable.lineBreakMode = NSLineBreakByWordWrapping ;
    testlable.text = tstring ;
    [testlable setBackgroundColor:[UIColor redColor]];
    testlable.frame =CGRectMake(10,20, actualsize.width, actualsize.height);
    [self.view addSubview:testlable];
```

















