---
layout: post
title:  "iOS中的定时器"
date:   2016-09-15 23:10:20 +0800
tags: iOS
---

## 认知

讲到定时器,需要了解一个概念,叫做RunLoop,从字面意义上理解为`运行循环`.iOS里有两套API可以访问到RunLoop,`NSRunLoop`和`CFRunLoopRef`,一个来自`Foundation`框架,另一个来自`Core Foundation`.

每一个线程都会对应一个RunLoop,但是RunLoop并不是随着线程的创建而生成的,实际上RunLoop也不允许被开发者创建,可以通过调用`[NSRunLoop currentRunLoop];`或者`CFRunLoopGetCurrent();`来获取系统当前线程的RunLoop.需要注意的是主线程的RunLoop会在启动的时候自动开启,其他线程的则需要手动开启.  

那么RunLoop究竟有什么作用呢?  

保证程序不会退出,这是它最大一个用处,因为RunLoop本身就是一个死循环.当没有事件传入的时候,RunLoop会让线程进入休眠,当有事件传入的时候会唤醒线程处理事件.

总结来说RunLoop的作用大概有以下几种:
- 保证程序不退出
- 处理事件(触摸事件/运动事件/远程控制事件)
- 节省资源

> 但RunLoop实际上并不止这些功能,AutoreleasePool,UI更新,网络请求,还有今天要讲到定时器都和RunLoop有关系.

## 应用  

iOS中用来实现定时器的API有很多,NSTimer,GCD,performSelecter,CADisplayLink等.因为NSTimer最为简单,所以先讲下NSTimer的一些用法.

定时器一般用在延迟处理,轮询事件等.比如之前有做到过一个项目,每隔10S给用户发送一条消息.那么用NSTimer编写代码如下:

      NSTimer *timer = [NSTimer timerWithTimeInterval:10.0f target:self selector:@selector(pushMessage) userInfo:nil repeats:YES];
      [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];

      - (void)pushMessage{
        // Do Something
      }


上面的代码其实是有问题的,当UIScrollView滑动的时候,定时器其实已经失效了.因为timer的CurrentLoopMode是`NSDefaultRunLoopMode`,当屏幕滑动的时候`NSDefaultRunLoopMode`会切换为`NSEventTrackingRunLoopMode`,所以定时器就不起作用了.解决起来也很简单,添加定时器到CurrentRunLoop的时候,把Mode设置为不会被占用的Mode,即`forMode:NSRunLoopCommonModes`,代码更新如下:

      NSTimer *timer = [NSTimer timerWithTimeInterval:10.0f target:self selector:@selector(pushMessage) userInfo:nil repeats:YES];
      [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

      - (void)pushMessage{
        // Do Something
      }

## 注意

NSTimer使用的注意点再总结一下.   

#### 1.RunLoopMode问题(上面有提到过).  
#### 2.内存泄露问题.
 
如果不重复执行的话,NSTimer在执行完之后就直接销毁了,但是如果是循环执行那么就需要注意了.timer必须调用`invalidate`来销毁,而且必须在同一个线程.  

举个例子,控制器A push到控制器B,在控制器B创建了一个循环执行的timer,当从控制器B返回到控制器A的时候,控制器B是并没有被释放,所以在控制器B的dealloc方法里执行`invalidate`方法也是不可行的.原因是timer在创建的时候target指定的是self,也是控制器B,那么当控制器B想要释放的时候,就需要先把它所有的对象释放掉,当释放到timer的时候,因为timer要释放他的target,而target指定的控制器B本身,所以就造成了循环引用,从而形成内存泄露.
  
解决方法也挺多.
1. `dealloc`方法不走,那么可以在`viewWillDisappear:(BOOL)animated`方法里执行`invalidate`方法.(但总觉得这个方案不怎么友好).  
2. 既然是timer的target强引用了self造成的循环引用,那么如果把target从self缓存其他对象不就OK了.可以自定义一个类来保存target,或者用一个类对象带代替self对象.  
3. 三方库[YYTimer](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYTimer.m)和[MSWeakTimer](https://github.com/mindsnacks/MSWeakTimer),这两个都是基于GCD定时器的又一层封装,相比较来说`MSWeakTimer`接口更丰富一点,毕竟YYTimer只是[YYKit](https://github.com/ibireme/YYKit)中一个工具类而已.  

除此之外,NSTimer的准确度也不是很高,它的精准度和线程的当前的空闲度成反比.在iOS7之后苹果引入了一个新概念叫做`tolerance`.

`@property NSTimeInterval tolerance API_AVAILABLE(macos(10.9), ios(7.0), watchos(2.0), tvos(9.0));`

> Setting a tolerance for a timer allows it to fire later than the scheduled fire date, improving the ability of the system to optimize for increased power savings and responsiveness. The timer may fire at any time between its scheduled fire date and the scheduled fire date plus the tolerance. The timer will not fire before the scheduled fire date. For repeating timers, the next fire date is calculated from the original fire date regardless of tolerance applied at individual fire times, to avoid drift. The default value is zero, which means no additional tolerance is applied. The system reserves the right to apply a small amount of tolerance to certain timers regardless of the value of this property.
> 
 As the user of the timer, you will have the best idea of what an appropriate tolerance for a timer may be. A general rule of thumb, though, is to set the tolerance to at least 10% of the interval, for a repeating timer. Even a small amount of tolerance will have a significant positive impact on the power usage of your application. The system may put a maximum value of the tolerance.
@property NSTimeInterval tolerance API_AVAILABLE(macos(10.9), ios(7.0), watchos(2.0), tvos(9.0));

这个属性是允许NSTimer的误差值,但是也只有差,不会多.也就是说,你用NSTimer做一个3.0s的延迟操作,你设置了`tolerance = 0.1`,那么如果由于线程繁忙的原因这个操作可能会在3.1s后执行,但是绝对不会再2.9s后执行.上面说到的`MSWeakTimer`也有`tolerance`这个属性.


## 延伸

上面有提到[YYTimer](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYTimer.m)和[MSWeakTimer](https://github.com/mindsnacks/MSWeakTimer),他们的内部实现其实都是GCD定时器.我们知道GCD有现成的API(dispatch_after)可以做延迟操作,它有个缺点就是一旦执行之后无法撤销,而且也无法做循环执行.

其实GCD还有一套API,可以弥补上面那两个缺点,我们直接上代码.

      // 声明一个全局的timer 
      @property (strong, nonatomic) dispatch_source_t timer;
      
      ...
    
    - (void)begin {
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
        self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
        
        dispatch_source_set_timer(self.timer, DISPATCH_TIME_NOW, 2.0 * NSEC_PER_SEC, 0.1 * NSEC_PER_SEC);
        
        dispatch_source_set_event_handler(self.timer, ^{
            NSLog(@"1");
        });
        
        dispatch_resume(self.timer);
    }
    
    - (void)end {
        dispatch_source_cancel(self.timer);
    }

使用GCD的好处显而易见,不会引起内存泄露,不用考虑RunLoopMode,又是线程安全的,就是API写起来有点长,不过自己封装一下或者用别人封好的库用也是很方便的.

除了GCD和NSTimer之外其实还有一些做定时器的方法.

- PerformSelector

  `performSelector:withObject:afterDelay`,它是`NSObject`对`NSTimer`的又一层封装,用起来更是简单,但是只能用于延迟操作.  
  
  `[self performSelector:selector(after) withObject:nil afterDelay:2.0];`  

  
- CADisplayLink    

  `CADisplayLink`其实并不是真正意义上的定时器,它一般用于图片的渲染绘制,首先`CADisplayLink`是属于``框架里的东西,在这它的调用频率和屏幕的刷新频率是一样的,也就是60FPS,使用起来也很简单.

		- (void)CADisplayLinkTimer {
			CADisplayLink *displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(display)];
			displayLink.preferredFramesPerSecond = 1.0;
			[displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
			self.displayLink = displayLink;
		}
			      
		- (void)display {
			 NSLog(@"%s",__func__);
		}
			      
		- (void)stop{
		     [self.displayLink invalidate];
		}

这些其实都不是常用的定时器,最常用的还是NSTimer和GCD,如果自己封装的有GCD的轮子或者不介意引用别人的库的话,个人还是建议用GCD定时器,


## 结尾
这篇文章是我在实际项目中以及在网上看了其他人的文章后总结的个人心得,欢迎有更深见解的朋友一起来探讨.


## 参考
[https://blog.ibireme.com/2015/05/18/runloop/](https://blog.ibireme.com/2015/05/18/runloop/)  
[https://www.jianshu.com/p/3614d92d41f1](https://www.jianshu.com/p/3614d92d41f1)  
[http://ifujun.com/ios-timer-pan-dian/](http://ifujun.com/ios-timer-pan-dian/)  
[http://blog.lessfun.com/blog/2016/08/05/reliable-timer-in-ios/](http://blog.lessfun.com/blog/2016/08/05/reliable-timer-in-ios/)