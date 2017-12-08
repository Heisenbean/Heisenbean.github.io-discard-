---
layout: post
title:  "通过视频地址获取视频缩略图和标题
"
date:   2017-06-16 12:35:00 +0800
tags: iOS
---
类似需求:在微博发布一条视频链接,查看的时候并不是一条视频链接的微博,而是一条有视频标题和视频封面图的微博.  

目前可获取的网站有腾讯,优酷,土豆,爱奇艺,哔哩哔哩.

### 腾讯  

#### 视频缩略图

`https://v.qq.com/x/cover/8jnnyshzame1k0h/r0513rpuuq0.html`    
视频链接是有两个变量和其他字符拼接而成.我们称那两个变量为cid和vids    
`https://v.qq.com/x/cover/cid/vids.html`    

> cid:8jnnyshzame1k0h  
> vids:r0513rpuuq0

上面这个视频的缩略图地址为:  
`http://vpic.video.qq.com/21845167/r0513rpuuq0.png`  
这里我称`21845167`这是图片的id  
那图片URL的构成方式就是`http://vpic.video.qq.com/+图片id/+vids.png`  
那么现在所需要做的就是得出图片id.  

取到图片id之前,先写一个方法,从视频链接中获取到vids.

    // first obj is cid, last obj is vids
    - (NSArray<NSString *> *)getTencentVideoCidAndVidsFromLink:(NSString *)link{
        NSMutableArray *array = [NSMutableArray array];
        NSArray *temp = [link stringByDeletingPathExtension].pathComponents;
        [array addObject:temp[temp.count - 2]];
        [array addObject:temp.lastObject];
        return array;
    }



获取到vids之后,通过一些列算法,我们可以得到图片id,拼接之后就是图片URL.

    - (NSInteger)getPictureIdFromLink:(NSString *)link{
        NSInteger pid, e = 1e8;
        NSInteger h = 0;
        
        for (NSInteger g = 4294967296, i = 0; i < link.length; i++) {
            NSInteger j = [link characterAtIndex:i];
            h = 32 * h + h + j;
            h >= g && (h %= g);
        }
        return pid = h % e;
    }

拼接后的图片URL:        

    NSInteger pid = [self getPictureIdFromLink:link];
    NSString *cid = [self getTencentVideoCidAndVidsFromLink:link].lastObject;
    NSString *imageUrl = [NSString stringWithFormat:@"http://vpic.video.qq.com/%ld/%@.png",pid,cid];


#### 视频标题

视频标题需要通过API来获取.

`baseURL:http://h5vv.video.qq.com/getinfo`

示例:  
`http://h5vv.video.qq.com/getinfo?otype=json&cid=8jnnyshzame1k0h&vids=r0513rpuuq0`

|参数名|说明|数据类型|是否必须|示例
 ---|---|---|---|---|---|
|otype|数据格式|字符串|是|json
|cid|封面id|字符串|是|8jnnyshzame1k0h
|vids|视频id|字符串|是|r0513rpuuq0

不过返回的并非是标准的JSON字符串,需要简单处理一下.

 ![](http://oclnty4pg.bkt.clouddn.com/get-pic-title-from-link.jpeg)

把开头的`QZOutputJson=`和末尾的`;`去掉然后再进行格式化.


### 优酷  
[http://v.youku.com/v_show/id_XMjgxOTg4MTY3Ng==.html?spm=a2h0z.8244218.2371631.d6373](http://v.youku.com/v_show/id_XMjgxOTg4MTY3Ng==.html?spm=a2h0z.8244218.2371631.d6373)  
> videoid:XMjgxOTg4MTY3Ng

优酷给开发者提供的有开放API,可以直接调,需要去创建相关的应用获取`client_id`.  
具体的参数和调用方法参考优酷文档:  
[http://cloud.youku.com/docs?id=46](http://cloud.youku.com/docs?id=46)  

示例: 
[https://api.youku.com/videos/show.json?video_id=XMjgxOTg4MTY3Ng==&client_id=yourClient_id](https://api.youku.com/videos/show.json?video_id=XMjgxOTg4MTY3Ng==&client_id=yourClient_id)  

### 土豆  
[http://video.tudou.com/v/XMjgyMDc5NDk0NA==.html?spm=a2h28.8313475.c1.dimg2&t=1&seccateId=10016](http://video.tudou.com/v/XMjgyMDc5NDk0NA==.html?spm=a2h28.8313475.c1.dimg2&t=1&seccateId=10016)  
> videoid:XMjgyMDc5NDk0NA

因为土豆和优酷合并,所以视频源是通用的,也就是说土豆的视频也可以用优酷的API来调,然后获取视频信息.

### 爱奇艺

爱奇艺也为开放中提供了开放API.需要去[爱奇艺开放平台](http://open.iqiyi.com/)创建应用获取`apiKey`.  
API具体使用方法请参考爱奇艺文档:  
[http://open.iqiyi.com/lib/play/api.html#A2](http://open.iqiyi.com/lib/play/api.html#A2)

### 哔哩哔哩  

[http://www.bilibili.com/video/av11297195/](http://www.bilibili.com/video/av11297195/)  

> id:11297195

哔哩哔哩已经关闭了开发者的功能,无法申请`appkey`,但是接口还保留着.这个接口调用有风险,因为是用的别人的`appkey`,如果`appkey`过期或者调用次数过多可能会失效.

|参数名|说明|数据类型|是否必须|示例
|---|---|---|---|---|---|
|type|数据格式|字符串|是|json
|appkey|应用id|字符串|是|yourAppKey
|id|视频id|字符串|是|11297195


示例:  

[http://api.bilibili.com/view?type=json&appkey=yourAppKey&id=11297195](http://api.bilibili.com/view?type=json&appkey=yourAppKey&id=11297195)

### 参考:    
- [腾讯视频加载方案](http://chenkun.life/2017/02/11/web/%E8%85%BE%E8%AE%AF%E8%A7%86%E9%A2%91%E5%8A%A0%E8%BD%BD%E6%96%B9%E6%A1%88/)

iOS端可参考demo:  
[https://github.com/Heisenbean/BlogDemos](https://github.com/Heisenbean/BlogDemos)