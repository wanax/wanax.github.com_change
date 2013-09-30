---
layout: post
title: "iOS开发之使用CorePlot绘制折线图并添加拖动功能"
date: 2013-06-07 11:06
comments: true
categories: Tec
---
做ios开发快两个月了 从基本的语法开始学起看看这个学学那个的。一开始对delegate controller view各种头大，就老老实实地跟着入门教程一步步的来。所幸之前做了一年多的java开发，熟悉了eclipse的界面和Deja的字体，换到mac下面用xcode也觉得很新鲜。

<!--more-->

感觉做ios开发这一块对内存管理要求还是不低的，因为之前做java，对这一块基本不怎么关心，突然转到ios上的时候在内存管理上还是犯过几次错误的。开始学习的时候看书上说谁创建谁释放，就简单的以为但凡创建出来的对象都要显式地去release，后来果然出过一次诡异的错误，修正之后对内存管理就更加用心地去学习了，这一块有时间后面再讨论，现在先把这几天对coreplot的使用记录一下，以备后用。

公司的需求是对数据解析进行折线图绘制，并且用户可以通过触摸调整数据点通过公式计算使折线图重新绘制。数据与公式都是服务器端获取的并且是打包成了一个json的字符串。这里面还牵扯到了字符串解析成公式的问题。上网搜了下，oc下并没有很好的解决方案，像这样做的话似乎只能自己写方法，让我想起了上学时候学编译原理写过的语法树，满满的都是泪啊~妥妥的能不写就不写。后来公司大牛说有js引擎可以拿过来用，因为公司这边也有做网页端的折线图绘制，在js下公式转换异常简单
```javascript
	var fun=new fun(str);  
	fun(); 
```
	
如上所示，两行代码就把str代表的字符串公式解析并计算一遍了，实在是太无解了，妥妥必须使用，这个东西下篇文章再讲吧。
图像绘制的话google出来ios上比较常用的是CorePlot和PowerPlot，看了下两者的用法，感觉CorePlot会使用更多的代理之类的东西，感觉比较符合苹果的编码思路，就选了它来用了。

首先是下载coreplot库，把.a文件和头文件引入，这里就不赘述了。

先在头文件里定义几个后面要用到的量

``` c
	//转换坐标系字典，与手指触摸点匹配  
    @property (nonatomic,retain) NSMutableDictionary *reverseDic;  
	//js引擎上下文  
	@property (nonatomic) JSGlobalContextRef context;  
	//绘图view  
	@property (nonatomic,retain) CPTGraphHostingView *hostView;    
	//坐标转换方法，实际坐标转化相对坐标  
	- (CGPoint)CoordinateTransformRealToAbstract:(CGPoint)point;  
	//坐标转换方法，相对坐标转化实际坐标  
	- (CGPoint)CoordinateTransformAbstractToReal:(CGPoint)point;  
	//判断手指触摸点是否在折点旁边  
	-(BOOL)isNearByThePoint:(CGPoint)p;  	  
	//js代码运行  
	- (NSString *)runsJS:(NSString *)aJSString; 
```	
因为要对图像上的折点拖动，而我没有找到合适的对点的事件绑定方法，所以先直接对整个view进行手势监听，通过实际坐标与坐标轴坐标相对转换来判定触摸点是否是在折点附近已达到合理拖动的目的。

接下来便开始.m文件的编码工作啦。我先定义一个简单的折线图公式，y=x+n；把一些数据点塞到point数组里，做后面的datasource。接下来便是折线图的绘制，代码注释写得比较清楚，就不一一赘述了。值得注意的是，这里点的绘制也是使用ios里常用的datasource的方法，所以先别急着找那一系列点出现在哪里~

``` c
- (void)viewDidLoad  {
	    [super viewDidLoad];  
	    UIButton *bt=[[UIButton alloc] init];  
	    [self.view addSubview:bt];  
	    points=[[NSMutableArray alloc] init];  
	    reverseDic=[[NSMutableDictionary alloc] init];  
	    NSUInteger i;  
	    for(i=0;i<NUM;i++){  
	        id x=[NSNumber numberWithFloat:0+i];  
	        id y=[NSNumber numberWithFloat:3*i];  
	        [reverseDic setObject:y forKey:[NSString stringWithFormat:@"%.0f",[x floatValue]]];  
	        [points addObject:[NSMutableDictionary dictionaryWithObjectsAndKeys:x,@"x",y,@"y",nil]];  
	    }  
	    // Do any additional setup after loading the view.  
	    //初始化图形视图  
	    graph=[[CPTXYGraph alloc] initWithFrame:CGRectZero];  
	    CPTTheme *theme=[CPTTheme themeNamed:kCPTDarkGradientTheme];  
	    [graph applyTheme:theme];  
	    self . view =[[ CPTGraphHostingView alloc ] initWithFrame :[ UIScreen mainScreen ]. bounds ];  
	    hostView=(CPTGraphHostingView *)self.view;    
	    [hostView setHostedGraph : graph ];      
	    NSLog(@"orgin:%.2f,%.2f",hostView.frame.origin.x,hostView.frame.origin.y);  
	    NSLog(@"size:%.2f,%.2f",hostView.frame.size.height,hostView.frame.size.width);      
	    // CPGraph 四边不留白  
	    graph . paddingLeft = 0.0f ;  
	    graph . paddingRight = 0.0f ;  
	    graph . paddingTop = 0.0f ;  
	    graph . paddingBottom = GRAPAHBOTTOMPAD ;  
	    // 绘图区 4 边留白  
	    graph . plotAreaFrame . paddingLeft = 0.0 ;  
	    graph . plotAreaFrame . paddingTop = 0.0 ;  
	    graph . plotAreaFrame . paddingRight = 0.0 ;  
	    graph . plotAreaFrame . paddingBottom = 00.0 ;      
	    //绘制图形空间  
	    CPTXYPlotSpace *plotSpace=(CPTXYPlotSpace *)graph.defaultPlotSpace;  
	    //plotSpace.allowsUserInteraction=YES;    
	    //设置x，y坐标范围  
	    plotSpace.xRange=[CPTPlotRange plotRangeWithLocation:  
	                      CPTDecimalFromCGFloat(XRANGEBEGIN)                                                  length:CPTDecimalFromCGFloat(XRANGELENGTH)];  
	    plotSpace.yRange=[CPTPlotRange plotRangeWithLocation:  
	                      CPTDecimalFromCGFloat(YRANGEBEGIN)  length:CPTDecimalFromCGFloat(YRANGELENGTH)];    
	    //绘制坐标系  
	    CPTXYAxisSet *axisSet=(CPTXYAxisSet *)graph.axisSet;  
	    CPTXYAxis *x=axisSet.xAxis;     
	    CPTMutableLineStyle *lineStyle = [CPTMutableLineStyle lineStyle];  
	    lineStyle.miterLimit = 1.0f;  
	    lineStyle.lineWidth = 2.0;  
	    lineStyle.lineColor = [CPTColor colorWithComponentRed:255/255.0 green:211/255.0 blue:155/255.0 alpha:1.0];  
	    x.majorIntervalLength=CPTDecimalFromFloat(XINTERVALLENGTH);  
 x.orthogonalCoordinateDecimal=CPTDecimalFromFloat(XORTHOGONALCOORDINATE);  
	    x.minorTicksPerInterval=XTICKSPERINTERVAL;  
	    x.minorTickLineStyle = lineStyle;     
	    CPTXYAxis *y=axisSet.yAxis;  
	    y.majorIntervalLength=CPTDecimalFromFloat(YINTERVALLENGTH); y.orthogonalCoordinateDecimal=CPTDecimalFromFloat(YORTHOGONALCOORDINATE);  
	    y.minorTicksPerInterval=YTICKSPERINTERVAL;  
	    y.minorTickLineStyle = lineStyle;   
	    y. labelingPolicy = CPTAxisLabelingPolicyNone ;     
	    //修改折线图线段样式  
	    CPTScatterPlot *boundLinePlot=[[[CPTScatterPlot alloc] init] autorelease];  
	    //CPTMutableLineStyle *lineStyle=[CPTMutableLineStyle lineStyle];  
	    lineStyle.miterLimit=2.0f;  
	    lineStyle.lineWidth=2.0f;  
	    lineStyle.lineColor=[CPTColor whiteColor];  
	    boundLinePlot.dataLineStyle=lineStyle;  
	    boundLinePlot.identifier=@"Blue Plot";  
	    boundLinePlot.dataSource=self;//需实现委托        
	    // Do a red-blue gradient: 渐变色区域  
	    //  
	    /*CPTColor * blueColor        = [CPTColor colorWithComponentRed:0.3 green:0.3 blue:1.0 alpha:0.8]; 
	    CPTColor * redColor         = [CPTColor colorWithComponentRed:1.0 green:0.3 blue:0.3 alpha:0.8]; 
	    CPTGradient * areaGradient1 = [CPTGradient gradientWithBeginningColor:blueColor 
	                                                              endingColor:redColor]; 
	    areaGradient1.angle = -120.0f; 
	    CPTFill * areaGradientFill  = [CPTFill fillWithGradient:areaGradient1]; 
	    boundLinePlot.areaFill      = areaGradientFill; 
	    boundLinePlot.areaBaseValue = [[NSDecimalNumber numberWithFloat:0.5] decimalValue]; // 渐变色的起点位置 
	    */  	      
	    // Add plot symbols: 表示数值的符号的形状  
	    //  
	    CPTMutableLineStyle * symbolLineStyle = [CPTMutableLineStyle lineStyle];  
	    symbolLineStyle.lineColor = [CPTColor colorWithComponentRed:102/255.0 green:204/255.0 blue:255/255.0 alpha:1.0];  
	    symbolLineStyle.lineWidth = 2.0;  	      
	    CPTPlotSymbol * plotSymbol = [CPTPlotSymbol ellipsePlotSymbol];  
	    //plotSymbol.fill          = [CPTFill fillWithColor: [CPTColor colorWithComponentRed:102/255.0 green:204/255.0 blue:255/255.0 alpha:1.0]];  
	    plotSymbol.lineStyle     = symbolLineStyle;  
	    plotSymbol.size          = CGSizeMake(1.8, 1.8);  
	    boundLinePlot.plotSymbol = plotSymbol;	      
	    // Animate in the new plot: 淡入动画  
	    boundLinePlot.opacity = 0.0f;  	      
	    CABasicAnimation *fadeInAnimation = [CABasicAnimation animationWithKeyPath:@"opacity"];  
	    fadeInAnimation.duration            = 3.0f;  
	    fadeInAnimation.removedOnCompletion = NO;  
	    fadeInAnimation.fillMode            = kCAFillModeForwards;  
	    fadeInAnimation.toValue             = [NSNumber numberWithFloat:1.0];  
	    [boundLinePlot addAnimation:fadeInAnimation forKey:@"shadowOffset"];        
	    [graph addPlot:boundLinePlot];  	      
	    //手势添加  
	    UIPanGestureRecognizer *panGr=[[UIPanGestureRecognizer alloc]initWithTarget:self action:@selector(viewPan:)];  
	    [hostView addGestureRecognizer:panGr];  
	    [panGr release];       
	    //NSString *str=[self.engine runJS:@"var str='';for(var i=0;i<2;i++){str= str + '\\n' + i + ' :Hello' + ', World!';}"];  
	    //NSLog(@"%@",str);  	  
	}
```
	
###这样一个初步的折现图便绘制完毕啦。数据源委托和数据标签的实现如下：

``` c
	// 添加数据标签  
  -( CPTLayer *)dataLabelForPlot:( CPTPlot *)plot recordIndex:( NSUInteger )index  {  
	    // 定义一个白色的 TextStyle  
	    static CPTMutableTextStyle *whiteText = nil ;  
	    if ( !whiteText ) {  
	        whiteText = [[ CPTMutableTextStyle alloc ] init ];  
	        whiteText.color=[CPTColor colorWithComponentRed:152/255.0 green:251/255.0 blue:152/255.0 alpha:1.0];  
	    }  
	    // 定义一个 TextLayer  
	    CPTTextLayer *newLayer = nil ;  
	    NSString * identifier=( NSString *)[plot identifier];  
	    if ([identifier isEqualToString : @"Blue Plot" ]) {        
	        newLayer=[[[CPTTextLayer alloc] initWithText:[[[NSString alloc] initWithFormat:@"%0.1f",[[[points objectAtIndex:index] objectForKey:@"y"] doubleValue]] autorelease] style:whiteText] autorelease];  
	    }  
	    return newLayer;  
	}   
	//散点数据源委托实现  
	-(NSUInteger)numberOfRecordsForPlot:(CPTPlot *)plot{  
	    return [points count];  
	}    
	-(NSNumber *)numberForPlot:(CPTPlot *)plot field:(NSUInteger)fieldEnum recordIndex:(NSUInteger) index{  
	    NSString *key=(fieldEnum==CPTScatterPlotFieldX?@"x":@"y");  
	    NSNumber *num=[[points objectAtIndex:index] valueForKey:key];  
	   //NSLog(@"key:%@,num:%@",key,num);  
	    return  num;  
	} 
```	
	
###可以在代码里看到我实现了对页面的手势监听，以便实现折线图拖动的效果，具体方法如下：

```c
-(void)viewPan:(UIPanGestureRecognizer *)tapGr  {  
	    CGPoint now=[tapGr locationInView:self.view];  
    //手势变化并且接近折点旁边  
    if([tapGr state]==UIGestureRecognizerStateChanged&&[self isNearByThePoint:now]){  
        CGPoint coordinate=[self CoordinateTransformRealToAbstract:now];  
        [points removeAllObjects];  
        [reverseDic removeAllObjects];  
        NSUInteger i;  
        for(i=0;i<NUM;i++){  
            id x=[NSNumber numberWithFloat:0+i];  
            id y=[NSNumber numberWithFloat:3*i+coordinate.y];  
            [reverseDic setObject:y forKey:[NSString stringWithFormat:@"%.0f",[x floatValue]]];  
            [points addObject:[NSMutableDictionary dictionaryWithObjectsAndKeys:x,@"x",y,@"y",nil]];  
        }  
        [graph reloadData];  
    }    
	}  
	//判断手指触摸点是否在折点旁边  
	-(BOOL)isNearByThePoint:(CGPoint)p{  
    //从手指触摸点的实际坐标得到抽象坐标  
    CGPoint abstractCoordinate=[self CoordinateTransformRealToAbstract:p];  
    //获取临近坐标点  
    int acX=(int)(abstractCoordinate.x+0.5);  
    //判断临近坐标点是否存在折点，存在则取出  
    float acY=[[reverseDic objectForKey:[NSString stringWithFormat:@"%d",acX]] floatValue];  
    //构造临近坐标折点，并转化为实际屏幕坐标点  
    CGPoint temp=[self CoordinateTransformAbstractToReal:CGPointMake([[NSNumber numberWithInt:acX] floatValue], acY)];  
    //计算临近坐标点与手指触摸点的距离  
    double distance=sqrt(pow((p.x-temp.x),2)+pow((p.y-temp.y),2));  
    //NSLog(@"%f",distance);  
    return distance>25?NO:YES;  
	}  
	//空间坐标转换:实际坐标转化自定义坐标  
	-(CGPoint)CoordinateTransformRealToAbstract:(CGPoint)point{  
    float viewWidth=hostView.frame.size.width;  
    float viewHeight=hostView.frame.size.height;   
    float coordinateX=(XRANGELENGTH*point.x)/viewWidth+XRANGEBEGIN;  
    float coordinateY=YRANGELENGTH-((YRANGELENGTH*point.y)/(viewHeight-GRAPAHBOTTOMPAD))+YRANGEBEGIN;  
    return CGPointMake(coordinateX,coordinateY);  
	}  
	//空间坐标转换:自定义坐标转化实际坐标  
	-(CGPoint)CoordinateTransformAbstractToReal:(CGPoint)point{  
    float viewWidth=hostView.frame.size.width;  
    float viewHeight=hostView.frame.size.height;  
    float coordinateX=(point.x-XRANGEBEGIN)*viewWidth/XRANGELENGTH;  
    float coordinateY=(-1)*(point.y-YRANGEBEGIN-YRANGELENGTH)*(viewHeight-GRAPAHBOTTOMPAD)/YRANGELENGTH;  
    return CGPointMake(coordinateX,coordinateY);  
	}  
	- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer  
	{  
	    return YES;  
	}
```
	
下面是一些常量的定义

```c
	#define NUM 10  
    //绘图空间与底部距离  
	#define GRAPAHBOTTOMPAD 80.0f  
	//x轴起点  
	#define XRANGEBEGIN 0.0  
	//x轴在屏幕可视范围内的范围  
	#define XRANGELENGTH 5.0  
	//y轴起点  
	#define YRANGEBEGIN 0.0  
	//y轴在屏幕可视范围内的范围  
	#define YRANGELENGTH 100.0  
	//x轴屏幕范围内大坐标间距  
	#define XINTERVALLENGTH 1.0  
	//x轴坐标的原点（y轴将在此与x轴相交）  
	#define XORTHOGONALCOORDINATE 10.0  
	//x轴每两个大坐标间小坐标个数  
	#define XTICKSPERINTERVAL 2  
	#define YINTERVALLENGTH 1.0  
	#define YORTHOGONALCOORDINATE 1.0  
	#define YTICKSPERINTERVAL 0 
```
	
好了，这便是iphone上怎样绘制个折线图并实现拖动效果，其实是个很简陋的例子，离真正能用还有很长的一段路要走，但先贴在这里留个记号，也许你会看到中间有些js引擎的影子，这个留在下一章再讲吧。

![image](http://i1001.photobucket.com/albums/af134/mxiaochi/20130607110518093_zps1865da13.png)