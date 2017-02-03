---
layout: post
title:  "UIView-Autolayout的简单使用
"
date:   2014-11-01 11:29:08 +0800
categories: iOS
---

iPhone6和iPhone6 plus问世之后,iOS开发的屏幕适配就又稍微麻烦了一些,但是如果能熟练使用Autolayout的话,屏幕适配也不是什么难事.  

众所周知的Autolayout可以在storyboard里通过添加约束来实现,这样是比较简单的.如果用代码的话,是比较麻烦的,包括用[VLF](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html)语言开发效率也不是很高.  

那么这里给大家推荐一个非常轻量级的三方框架,准确来说也不算是一个框架,它是给UIView添加一个分类,使用起来非常简单.

这里先贴出github地址:[https://github.com/jrturton/UIView-Autolayout](https://github.com/smileyborg/UIView-AutoLayout)

> 后来作者放弃了这个库,新开发了一个叫[Purelayout](https://github.com/PureLayout/PureLayout)的库,用法基本一致

平常的开发我个人是个偏向于用代码,所以自动布局的时候也会用代码,但是用代码还是用sb(storyboard)要取决于需求.因为近期的项目中用到了代码自动布局,而有些朋友也在问关于代码自动布局的问题,我就把我自己掌握的技术点分享给大家.

首选我们知道用autolayout布局的话是四个约束才能固定控件的位置,相当于给控件设置了frame一样控件才能显示,当然如果让一个控件水平方向和垂直方向居中的话也是可以显示的,前提是要给这个控件size.

**注意**:  
如果要给控件设置约束的话,必要的前提条件是要给这个控件找个爹,就是要把它添加到一个父控件中.

sample code:  

	#import "ViewController.h"
	#import "UIView+AutoLayout.h"
	@interface ViewController ()
	@property (nonatomic,weak) UIView *grayView;
	@property (nonatomic,weak) UIView *redView;
	@property (nonatomic,weak) UIView *purpleView;
	 
	 
	@end
	 
	@implementation ViewController
	 
	- (void)viewDidLoad {
	    [super viewDidLoad];
	     
	    UIView *redView = [[UIView alloc]init];
	    redView.backgroundColor = [UIColor redColor];
	    self.redView = redView;
	    redView.frame = CGRectMake(50, 50, 300, 300);
	    [self.view addSubview:redView];
	     
	    UIView *grayView = [[UIView alloc]init];
	    grayView.backgroundColor = [UIColor lightGrayColor];
	    [redView addSubview:grayView];
	    self.grayView = grayView;
	     
	    UIView *purpleView = [[UIView alloc]init];
	    purpleView.backgroundColor = [UIColor purpleColor];
	    [redView addSubview:purpleView];
	    self.purpleView = purpleView;
	     
	 
	    [self demo3];
	}
	 
	- (void)demo{
	    [_grayView autoSetDimensionsToSize:CGSizeMake(200, 200)];
	 
	//    [_grayView autoAlignAxisToSuperviewAxis:ALAxisHorizontal];
	    [_grayView autoAlignAxisToSuperviewAxis:ALAxisVertical];
	    [_grayView autoAlignAxisToSuperviewAxis:ALAxisBaseline];    // 让控件的maxY等于它父类的maxY
	}
	 
	// 设置控件与父控件的约束
	- (void)demo2{
	    // 在父控件中的约束
	    [_grayView autoSetDimensionsToSize:CGSizeMake(200, 200)];
	     
	    // 给控件添加在父控件中上下左右的约束
	    [_grayView autoPinEdgesToSuperviewEdgesWithInsets:UIEdgeInsetsMake(50, 50, 50, 50)];
	     
	    // 也可添加不包括某个方向的约束,比如底部,此时这是的底部约束就不会起作用了
	//    [_demoView autoPinEdgesToSuperviewEdgesWithInsets:UIEdgeInsetsMake(10, 10, 100, 10) excludingEdge:ALEdgeBottom];
	     
	    // 单独给控件设置底部的约束
	    [_grayView autoPinEdgeToSuperviewEdge:ALEdgeBottom withInset:10];
	}
	 
	- (void)demo3{
	    [_grayView autoSetDimensionsToSize:CGSizeMake(50, 50)];
	    // 灰色view的左边和红色view的左边设置20的约束
	    [_grayView autoPinEdge:ALEdgeLeft toEdge:ALEdgeLeft ofView:_redView withOffset:20];
	    // 灰色view的顶部和红色view的底部设置20的约束
	    [_grayView autoPinEdge:ALEdgeTop toEdge:ALEdgeTop ofView:_redView withOffset:20];
	     
	    [_purpleView autoSetDimensionsToSize:CGSizeMake(50, 50)];
	    // 紫色view的左边和灰色view的右边设置20的约束
	    [_purpleView autoPinEdge:ALEdgeLeft toEdge:ALEdgeRight ofView:_grayView withOffset:20];
	    [_purpleView autoPinEdge:ALEdgeTop toEdge:ALEdgeTop ofView:_redView withOffset:20];
	 
	}
	@end
	
	
###方法介绍:

1. `autoSetDimensionsToSize`,这个方法是给控件添加size约束,相当于sb里的Pin中的宽度高度选项.  
![](http://oclnty4pg.bkt.clouddn.com/150003247613990.png)

	同等于`[_grayView autoSetDimensionsToSize:CGSizeMake(46, 30)];`  

2. autoAlignAxisToSuperviewAxis,这个方法是约束控件在它父控件中的方向,比如是垂直居中还是水平居中.相当于sb里的Align中让控件水平或者垂直居中.  
![](http://oclnty4pg.bkt.clouddn.com/150013551837692.png)   
同等于  

		[_grayView autoAlignAxisToSuperviewAxis:ALAxisVertical];
		[_grayView autoAlignAxisToSuperviewAxis:ALAxisHorizontal];


3. autoPinEdge,这个方法相当于sb中Pin里面给添加控件距其他控件的距离,可以是控件的父控件,也可以是和他相邻的其他控件.  
![](http://oclnty4pg.bkt.clouddn.com/150024528087218.png) 
   
	同等于  

		[_grayView autoPinEdgesToSuperviewEdgesWithInsets:UIEdgeInsetsMake(72, 22, 28.5, 245)];
		
	> 顺序为上左下右


	当然也可以单独设置约束:  
		    
		// 单独给控件设置底部的约束
		[_grayView autoPinEdgeToSuperviewEdge:ALEdgeBottom withInset:10];

	也可以设置两个子控件之间的约束:  
	
		 [_grayView autoSetDimensionsToSize:CGSizeMake(50, 50)];
	    // 灰色view的左边和红色view的左边设置20的约束
	    [_grayView autoPinEdge:ALEdgeLeft toEdge:ALEdgeLeft ofView:_redView withOffset:20];
	    // 灰色view的顶部和红色view的底部设置20的约束
	    [_grayView autoPinEdge:ALEdgeTop toEdge:ALEdgeTop ofView:_redView withOffset:20];
	     
	    [_purpleView autoSetDimensionsToSize:CGSizeMake(50, 50)];
	    // 紫色view的左边和灰色view的右边设置20的约束
	    [_purpleView autoPinEdge:ALEdgeLeft toEdge:ALEdgeRight ofView:_grayView withOffset:20];
	    [_purpleView autoPinEdge:ALEdgeTop toEdge:ALEdgeTop ofView:_redView withOffset:20];
	    
	效果如下:  

	![](http://oclnty4pg.bkt.clouddn.com/161147377671051.jpg)


	
	