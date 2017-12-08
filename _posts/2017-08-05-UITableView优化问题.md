---
layout: post
title:  "UITableView优化问题
"
date:   2017-08-05 11:12:30 +0800
tags: iOS
---
关于UITableView的优化问题网上可以查到很多文章,也不知道看哪一篇.这篇文章我会简单扼要的讲下UITableView引起卡顿的原因,解决方案以及怎样预防.

- ## 认知

  首先你要先知道自己的UITableView是否需要优化,比如滑动的时候有明显的停滞感或者卡顿,那肯定是要优化的.  
  如果肉眼无法明显的看出来,那么就需要借助工具: 
  
    1. 可以从Instruments里的`Core Animation`看`Frames Per Second`,也就是我们说的FPS(帧);  
    
    2. 借助[YYFPSLabel](https://github.com/yehot/YYFPSLabel)来查看帧数.帧数稳定在60左右就是比较流畅的了.


- ## 思考
  UITableView在滑动的时候会频繁调用其数据源和代理方法,那我们着手优化的就是以下这两个方法:
  `- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;`
  
  `- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath`

- ## 方案
  ### 造成UITableView滚动时卡顿的原因大致分为两种:  
  
  - 计算Cell高度耗费时间长
  - Cell中的子视图渲染耗费时间长
  
  ### 针对上面两点,我总结出一些优化的方案.

  - Cell的高度优化: 
    - 尽量使用数学四则运算,避免复杂运算
    - 以尽快的速度计算好Cell的高度并返回
    - 缓存行高,避免重复计算
  
  - 渲染优化  
  
    - 减少混合渲染,控件的背景色**尽量不要使用透明色**,设计师的切图也尽量不要有透明色,因为即使图片使用透明色,也会消耗GPU去渲染
    - 减少离屏渲染,比如尽量不要设置控件的cornerRadius和setClipsToBounds.  
    - 能用UIImageView就不用UIButton,iOS 10之后UIImageView切圆角不会引发离屏渲染.
    - `UILabel`在iOS8之后的底层从CALayer换为_UILabelLayer,导致UILabel为中文的时候依然会引起混合渲染.英文和数字不受影响,可能是中文编码引起的原因,本文不做过多深究.
      > 如何查看界面是否引起了离屏渲染或者混合渲染呢?  
      用模拟器打开项目,跳转到需要检测的页面,点击模拟器`Debug`菜单,可以看到`Color Blended Layers`和`Color Off-screen Rendered`选项,目前我们也只需要关心这两个就行.   
      `Color Blended Layers`是把你UI中有混合渲染的地方以红色高亮显示出来  
      `Color Off-screen Rendered`是以黄色高亮把UI中引发离屏渲染的部分显示出来.

      ![](http://oclnty4pg.bkt.clouddn.com/1512720271555.jpg?imageView2/0/w/300/q/100)

  - 性能优化
   	 - 异步UI :  
    把耗时的UI操作放到后台线程,在主线程更新.
    比如对一个图片做模糊效果,模糊算法是比较耗时的.这时就需要把耗时操作放到异步线程,完成之后再到主线程更新UI.
		
		        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
		           GPUImageGaussianBlurFilter * blurFilter = [[GPUImageGaussianBlurFilter alloc] init];
		           blurFilter.blurRadiusInPixels = 20;
		           UIImage *blurImage = [blurFilter imageByFilteringImage:image];
		           dispatch_async(dispatch_get_main_queue(), ^{
		                    cell.icon.image  = blurImage;
		                });
		         }
		         
		         
  	  - 删除子控件:   
    如果你Cell中子控件是在数据的Set方法中创建的,那么记得在创建之前先从父控件删除,不然会重复创建很多.通过Xcode自带的查看项目图层结构的Feature就能看出来.
  
  通过以上几点的优化方案,基本上界面不会再有明显的卡顿.当然我这里也并不是所有的解决方案,UITableView优化的还有很大的空间,比如预加载Cell,缓存View等等.如果以上方案还不能解决你的UITableView滚动时卡顿的问题,那么我建议你参考下这篇文章.   
  
  [保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)