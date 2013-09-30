---
layout: post
title: "iOS上js引擎的使用"
date: 2013-06-07 11:47
comments: true
categories: Tec
---
 如上篇文章所述ios折线图绘制，公司开发需要要用到一些字符串转公式的需求。自己实在有点惧自己写语法书啥的听公司大牛说可以使用js引擎试试，于是就抱着试试的态度调研了一把，感觉还不错。

这次试用了两个js引擎，一个是javascriptcore，这个比较方便可以直接引入便使用，另一个是google的v8引擎，这个有点麻烦要先编译通过才行，搞了好一阵才派上用场~倒是学到了好多新知识，哈哈。

<!--more-->

先说简单的javascriptcore吧，下载好库引入静态链接库和头文件便可以了。这里有时候会报错说找不到文件。google了一下发下一般是因为xcode本身对外来拷贝文件没有正确引用导致的，解决办法很简单，进入Build Phases 的Compile Source对新添加的头文件进行引用就好了。

![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/20130607111925093_zps428cacd3.png)

接下来便是怎样使用的问题。

先在头文件里引用文件：
``` c
	import "JavaScriptCore/JavaScriptCore.h" 
```
	
然后是构造js的上下文context，这个还蛮重要的东西，具体的原理大家可以自行google，因为我自己理解的也很肤浅，略知皮毛而已。

```	c
	context = JSGlobalContextCreate(NULL);  
```
	
完了以后我把js运行的方法封装成了一个方法，如下：
  
``` c
- (NSString *)runsJS:(NSString *)aJSString {  
    if (!aJSString) {  
        NSLog(@"[JSC] JS String is empty!");  
        return nil;  
	}
    JSStringRef scriptJS = JSStringCreateWithUTF8CString([aJSString UTF8String]);  
    JSValueRef exception = NULL;        
    JSValueRef result = JSEvaluateScript(context, scriptJS, NULL, NULL, 0, &exception);  
    NSString *res = nil;        
    JSStringRef jstrArg = JSValueToStringCopy(context, result, NULL);  
    res = (NSString*)JSStringCopyCFString(kCFAllocatorDefault, jstrArg);       
    JSStringRelease(jstrArg);        
    JSStringRelease(scriptJS);        
    return res;  
}
``` 

    
封装完毕便可以顺利使用啦：

``` c
NSString *str=[self.engine runJS:@"var str='';for(var i=0;i<2;i++){str= str + '\\n' + i + ' :Hello' + ', World!';}"];  
      //NSLog(@"%@",str);   
```
	   
这便是大体的javascriptcore的使用方法了。

接下来是v8引擎。

v8引擎的编译很痛苦，网上一般比较常见的教程是编译成动态库，lib之类的，但苹果貌似只让用.a的静态库，无形中又增加了负担。我网上搜了好多编译的方法，无非都是些svn check下v8的文件，然后安装python和对应python的一个编译工具scons进行编译。看攻略说，google先在又不提倡用scons了，又开发了另外一种东西....真是能折腾。

我遇到的主要问题是好像编译的时候要搭配一个叫breakpad的东西一起，而这玩意儿又死活装不了，就一直卡在那里了。但比较意外的是最后经竟意外发现在v8根目录还有个make的命令，也可以直接编译.....就这样稀里糊涂的把静态库给整出来了，还隐藏得很深，search了一下才发现在哪里。

![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/20130607113710843_zpsd81cc322.png)

这便是粗略的使用方法啦。

>一个简单的示例


```javascript
	   var test=function(){  
		   var val=[];  
		   val["1.0"]=1;  
		   val["1.1"]=2;  
		   val["1.2"]=3;  
		   val["1.3"]=4;  
		   val["2"]=34;  	 
		   var str="var v=this.val; v[\"1.3\"]=v[\"1.0\"]*v[\"1.2\"];";  
		   var fun=new Function(str);  
		   val["0"]=5;  
		   fun();  
		   return val;  
	 }  
```