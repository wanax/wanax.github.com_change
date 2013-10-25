---
layout: post
title: "UIView三种动画笔记"
date: 2013-06-09 10:30
comments: true
categories: Tec
---
``` objc
[UIView animateWithDuration:0.6f  
        delay:0  
        options:UIViewAnimationOptionCurveEaseOut  
	    animations:^{  
	       self.view.alpha = 0;  
	    }  
	    completion:^(BOOL finished) {  
	       [self.view removeFromSuperview];  
}];  
```
  <!--more-->
  
``` objc
	[UIView beginAnimations:@"animation" context:nil];  
    [UIView setAnimationDuration:0.8f];  
    [UIView setAnimationCurve:UIViewAnimationCurveEaseOut];  
    [UIView setAnimationTransition:UIViewAnimationTransitionCurlDown forView:delegate.tabBarController.view cache:YES];  
    [UIView commitAnimations];
```

```objc  
	CATransition *animation = [CATransition animation];  
	animation.duration = 0.8f;  
	animation.timingFunction = UIViewAnimationCurveEaseInOut;  
	animation.fillMode = kCAFillModeForwards;  
	animation.type = kCATransitionMoveIn;   
	animation.subtype = kCATransitionFromBottom;  
	[[self.view layer] addAnimation:animation forKey:@"animation"];  
	animation=nil; 
```