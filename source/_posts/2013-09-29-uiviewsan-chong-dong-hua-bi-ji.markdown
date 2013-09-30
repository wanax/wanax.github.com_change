---
layout: post
title: "UIView三种动画笔记"
date: 2013-06-09 10:30
comments: true
categories: Tec
---
``` c
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
  
``` c
	[UIView beginAnimations:@"animation" context:nil];  
    [UIView setAnimationDuration:0.8f];  
    [UIView setAnimationCurve:UIViewAnimationCurveEaseOut];  
    [UIView setAnimationTransition:UIViewAnimationTransitionCurlDown forView:delegate.tabBarController.view cache:YES];  
    [UIView commitAnimations];  
  CATransition *animation = [CATransition animation];  
	    animation.duration = 0.8f;  
	 /*动画时间控制      
     UIViewAnimationCurveEaseInOut,         // slow at beginning and end      
     UIViewAnimationCurveEaseIn,            // slow at beginning      
     UIViewAnimationCurveEaseOut,           // slow at end     
     UIViewAnimationCurveLinear      
     */  
    animation.timingFunction = UIViewAnimationCurveEaseInOut;  
    animation.fillMode = kCAFillModeForwards;  
    /*动画类型      
     kCATransitionFade;      
     kCATransitionMoveIn;      
     kCATransitionPush;      
     kCATransitionReveal;      
     */  
    animation.type = kCATransitionMoveIn;  
     /*动画进入方式      
     kCATransitionFromRight;      
     kCATransitionFromLeft; 
     kCATransitionFromTop; 
     kCATransitionFromBottom; 
     */  
     animation.subtype = kCATransitionFromBottom;  
     [[self.view layer] addAnimation:animation forKey:@"animation"];  
     animation=nil; 
```