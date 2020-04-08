---
layout: post
title:  "iOS性能优化(一)"
date:   2018-1-26 17:02:39 +0800
tags: iOS
---

# 前言
之前有写过一篇关于[UITableView优化问题](https://heisenbean.me/2017/08/Optimize-Of-UITableView/)的文章,现在回头再去看,其实只是蜻蜓点水,写了一点皮毛而已,这次再次看Y神的[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)以及其他人的博客,又有不少收获,来给大家分享一下我现在对iOS界面优化的一些拙见.后面应该还会继续分享一些其他方面的优化,比如说内存,电量等.
# 界面卡顿分析
首先看一张图,屏幕成像的过程.  

![](http://oclnty4pg.bkt.clouddn.com/FlJi1AFme0tZZMxPiXLoU2z4YE5m)

从深层次来看,界面之所以卡顿,是因为CPU和GPU在一个垂直同步信号(VSync)的时间区间内没有完成他们各自的任务,可能是CPU创建对象和计算显示内容耗时过长,也可能是GPU渲染时间过长,两者的耗时加起来超过了VSync的时间区间,那么这一帧就会被丢弃,这就是我们所说的掉帧.

既然知道了掉帧的原因,那么我们就可以对症下药.我们无法增加VSync信号的时间区间,因为显示器是以固定的频率刷新的,这个频率就是VSync信号产生的频率,所以无法改变.(为了方便理解,上面的VSync的时间区间就是VSync的频率).那么我们可以缩短CPU和GPU处理任务的时常,也就是我们常说的CPU和GPU的优化.

## CPU优化
### 对象创建
对象的创建([alloc init]/new等)会分配内存空间,调整属性,或者文件的读取等会消耗很多CPU资源.那么我们可以用毕竟轻量级的对象替代一些重量级的,比如用CALayer替代UIView.如果不是UIKit的子类对象,创建的时候还可以放到后台线程去创建.  

还有一些对象创建的时候开销比较大但又没有可替代的对象,那么可以采用懒加载,缓存或者单例的方式来优化.   

> Creating a date formatter is not a cheap operation. If you are likely to use a formatter frequently, it is typically more efficient to cache a single instance than to create and dispose of multiple instances. One approach is to use a static variable
     



比如NSDateFormatter和NSCalendar这两个就是创建时比较耗内存的.如果采用缓存的方式来优化NSDateFormatter的话要注意下,NSDateFormatter在iOS 7之前并不是线程安全的,需要从当前线程来获取缓存中的数据.

>On iOS 7 and later NSDateFormatter is thread safe.
In macOS 10.9 and later NSDateFormatter is thread safe so long as you are using the modern behavior in a 64-bit app.
On earlier versions of the operating system, or when using the legacy formatter behavior or running in 32-bit in macOS, NSDateFormatter is not thread safe, and you therefore must not mutate a date formatter simultaneously from multiple threads.

    static const NSString *kDateFormatter = @"DateFormatter";
    + (NSDateFormatter *)currentDateFormatter{
        NSMutableDictionary *currentThreadDic = [[NSThread currentThread] threadDictionary] ;
        NSDateFormatter *dateFormatter = [currentThreadDic objectForKey: kDateFormatter] ;
        if (!dateFormatter){
            dateFormatter = [[NSDateFormatter alloc] init] ;
            [dateFormatter setLocale: [[NSLocale alloc] initWithLocaleIdentifier: @"zh_CN"]] ;
            [dateFormatter setDateFormat: @"YYYY-MM-dd HH:mm:ss"] ;
            [currentThreadDic setObject: dateFormatter forKey: kDateFormatter] ;
        }
        return dateFormatter ;
    }

### <span id = "jump">视图对象调整</span>
设置UIView的一些和显示相关的属性例如frame/bounds/transform等也是很耗资源的,因为当调整这些属性的时候UIView和CALayer之间会产生很多的方法调用和通知.所以在优化性能的时候应该尽量减少这些属性的调整.


### 文本渲染
iOS中能够显示的文本的比如UILabel,UITextView,其底层都是通过Core Text在主线程排版绘制的,所以当有大量的文本的时候也是非常耗CPU资源的.解决方案只有一个,就是自定义文本控件,用TextKit或者Core Text对文本异步绘制,当然也有很多封装好的异步文本排版和渲染的库,比如[YYText](https://github.com/ibireme/YYText),[DTCoreText](https://github.com/Cocoanetics/DTCoreText)等.

### 文本计算
当界面中包含大量的文本的时候,文本的宽高计算也会占用很大一部分资源.一般都可以用UILabel提供的方法`[NSAttributedString boundingRectWithSize:options:context:]`来计算,最好放到后台线程来执行以避免线程阻塞.  

如果界面中文本视图比较少而且不需要计算宽度,那么也建议用Autolayout来实现,不用计算文本宽高,添加Leading和Top的约束,设置UILabel的`preferredMaxLayoutWidth`(期望的最大宽度)属性,最后返回UILabel的MaxY就行.这里需要注意的是`preferredMaxLayoutWidth`属性只在UILabel多行的情况下有效,而这个属性的值我一般设置为屏幕的宽度减去UILabel左右间距.

### 布局计算
在上面的[视图对象调整](#jump)也说过,调整UIView的一些可现实属性是比较耗资源的,而不论什么布局方式最后都是对UIView的frame/bounds等属性进行调整.如果能计算一次,并且把结算结果缓存下来这样就节省不少开销,这个也就是我们所用的高度缓存.

### 图片的解码
图片的解码其实我们能继续优化的地方没多少,现在的网络库都已经做过了优化.但是优化的原理需要稍微了解一下.当用CGImage或者UIImag初始化一个图片对象的时候,图片数据不会立刻解码,当把图片对象赋值给UIImageView的image属性或者CALayer的contents属性的时候,并且在CALayer没有提交给GPU之前,CGImage的数据才会得到解码,而且解码是在主线程进行的,并且不可避免.常见的做法在后台线程先把图片绘制到CGBitmapContext中,然后从Bitmap直接创建图片.

### 图像的绘制
图像的绘制大多是在Core Graphics这个系统库中的一些方法,这些方法都是线程安全的,所以CG开头的一些图像绘制的方法可以放到后台线去进行,最后再在主线程更新UI.


## GPU优化

### 纹理渲染
图片,文本,栅格化的内容,最终都会由内存提交到GPU显存,绑定为GPU Texture,然后由GPU来渲染.那么不管是从内存提交到缓存,还是GPU渲染Texture,这些都要消耗不少GPU资源.当Tableview中有很多图片,并在快速滑动的时候,GPU消耗更大.另外在图片过大,大到超过GPU的最大处理尺寸时,会给CPU带来额外的开销,大尺寸图片需要由CPU进行预处理才能交付给GPU.  
所以纹理渲染的优化可以从两个方面入手,即减少图片数量和质量.

### 混合渲染
当有多个视图重叠的时候,重叠部分会被GPU优先混合到一起.如果视图结构过于复杂,混合的过程也会给GPU带来不小的开销.所以应当适量减少视图的层级结构,尽量不要有重叠的部分,并且在不透明的视图属性里标明`opaque`属性以避免额外的透明值通道合成.

需要注意一下,iOS8以后当UILabel如果是中日韩文的话也会引起混合渲染,英文和数字不受影响.可能是汉字编码问题导致的混合渲染,通过查看图层结构可以发现汉字周围又多渲染了一层边框,所以调用下layer的`masksToBounds`方法即可解决这个问题.

### 离屏渲染
设置CALayer的圆角,阴影,边框,遮罩通常会引起离屏渲染,而离屏渲染通常发生在GPU中.观察下如果CPU此时比较空闲的话可以通过CALayer的`shouldRasterize`属性,把原本离屏渲染的操作转嫁到CPU中,当然这不是最优方案.可以把需要显示的图片在后台线程绘制为图片,不要使用CALayer的圆角,阴影,遮罩等属性.


     - (UIImage*)imageAddCornerWithRadius:(CGFloat)radius andSize:(CGSize)size{
        CGRect rect = {CGPointZero,size};
        UIGraphicsBeginImageContextWithOptions(size, NO, [UIScreen mainScreen].scale);
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        UIBezierPath * path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:UIRectCornerAllCorners cornerRadii:CGSizeMake(radius, radius)];
        CGContextAddPath(ctx,path.CGPath);
        CGContextClip(ctx);
        [self drawInRect:rect];
        CGContextDrawPath(ctx, kCGPathFillStroke);
        UIImage * newImage = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        return newImage;
    }

参考:
[性能优化之 NSDateFormatter](https://www.jianshu.com/p/82c1104aea6c)  
[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

