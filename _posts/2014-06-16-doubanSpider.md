---
layout: post
title:  "手把手教你爬取豆瓣相册"
date:   2014-06-16 11:29:08 +0800
categories: Python
---

Python版本:[2.7](https://www.python.org/downloads/)  
开发工具:[Pycharm Community for Mac](https://www.jetbrains.com/pycharm/download/)  

>我从简单到复杂,从用Python自带库到用到三方库来慢慢讲解怎么写这个爬虫.  
>最好有其他语言基础,有正则基础.  

最终目标:  
 ✅ 动态加载网页爬取图片   
 ✅ 翻页处理  
 ✅ 异常处理

### <a name="markdown-通过本地HTML网页爬取图片"></a>通过本地HTML网页爬取图片

#### 1. 分析静态网页源码
- Pycharm新建项目.  
- 随便找一个豆瓣相册地址`https://www.douban.com/photos/album/102879847/`  
把网页源代码复制一下,在项目路径下vim一个新文件后缀名为`.txt`,`html`都可以的,然后把复制的网页源码粘贴进去,这里我的源文件为`source.html`

- 分析源码可看到相册里图片的url都在`<div class="photolst clearfix">`和`<div id="link-report"`之间,先不要管这两个东西是什么,只要能找到这个规律就行了.  

- 那我们要做的就是通过代码拿到所有的图片url,然后下载到电脑上.

####2. 读取源码,获取图片url

- 通过下面几句代码就能读取到源代码了.

		# -*- coding: utf-8 -*-
		file = open('source.html','r')
		html = file.read()
		file.close()
		print html
	> `	# -*- coding: utf-8 -*-`由于编码问题,要加上这句声明,不然代码中出现中文会报错.
- 下面将要用到正则表达式的一些知识来获取我们想要的内容.上面提到过所有图片的url都在`<div class="photolst clearfix">`和`<div id="link-report"`之间,所以我们通过正则可以这样拿到图片url.    

		# -*- coding: utf-8 -*-
		# 引入正则库
		import re 
		file = open('source.html','r')
		html = file.read()
		file.close()
		# 截取想要的部分
		images = re.findall('"photolst clearfix">(.*?)<div id="link-report',html,re.S)[0]
		print images

	>`.*?`就是正则里的任意匹配,比如`abcdefg`你想要`a`和`g`之间的字符,那么就是`'a(.*?)g'`  
	
	print结果中的其中一条数据:
	
		   <div class="photo_wrap">
		        <a href="https://www.douban.com/photos/photo/2230900788/" class="photolst_photo" title="">
		        <img src="https://img3.doubanio.com/view/photo/thumb/public/p1972176302.jpg" />
		        </a><br/>
		        <div class="pl"></div>
		        <div style="color:#999">
		        </div>
		    </div>
 

	通过打印发现此时拿到并不是纯粹的url,而我们想要的只是`img src`标签后面的url.  
那么还可以继续利用正则匹配拿到,思路和上面一样.

		# -*- coding: utf-8 -*-
		# 引入正则库
		import re 
		
		file = open('source.html','r')
		html = file.read()
		file.close()
		# 截取想要的部分
		images = re.findall('"photolst clearfix">(.*?)<div id="link-report',html,re.S)[0]
		# 截取url
		pic_url = re.findall('img src="(.*?)" /',images)
		print pic_url

####3. 通过图片url下载图片

- 现在已经拿到所有图片的url,并且已经在一个数组里了,那剩下的就是遍历这个数组,把图片一个一个下载下来.
 
		# -*- coding: utf-8 -*-
		# 引入正则库
		import re 
		# 网络请求库
		import requests
		
		file = open('source.html','r')
		html = file.read()
		file.close()
		
		images = re.findall('"photolst clearfix">(.*?)<div id="link-report',html,re.S)[0]
		pic_url = re.findall('img src="(.*?)" /',images)
		
		i = 0
		for url in pic_url:
		    print '正在下载第' + str(i) + '张图片' 
		    pic = requests.get(url)
		    fp = open(str(i) + '.jpg','wb')
		    fp.write(pic.content)
		    fp.close()
		    i += 1
- 但是现在下载的图片都是存放到当前文件夹了,期望的效果是自动创建一个文件夹并把图片下载到里面,也很好实现.

		# -*- coding: utf-8 -*-
		import re
		import os
		import requests
		file = open('source.html','r')
		html = file.read()
		file.close()
		
		images = re.findall('"photolst clearfix">(.*?)<div id="link-report',html,re.S)[0]
		pic_url = re.findall('img src="(.*?)" /',images)
		
		i = 0
		# 如果不存在此文件夹才创建
		if not os.path.exists('pic'):
		    os.makedirs('pic')
		    for url in pic_url:
		        print '正在下载第' + str(i) + '张图片'
		        pic = requests.get(url)
		        fp = open(os.getcwd() + '/pic/' + str(i) + '.jpg', 'wb')
		        # fp = open(str(i) + '.jpg','wb')
		        fp.write(pic.content)
		        fp.close()
		        i += 1  
		else:
		    print('您已下载过了')
> `os.getcwd`是获取当前路径,以及`makedirs`都是`os`库里的方法,要记得引用os库.

至此,从静态html源代码爬取图片地址并下载算是已经做完了,接下来就直接通过网址来爬取图片.
###通过网址动态爬取图片
####1. 解析相url地址,获取HTML源码
- 还以`https://www.douban.com/photos/album/102879847/`这个相册地址为例.通过网址来把HTML源码解析出来很简单,首先引用`urllib2`库,然后打开,读取.

		album_url = 'https://www.douban.com/photos/album/102879847/'
		html = first_html = urllib2.urlopen(album_url).read()
		print html
既然能获取到HTML源码,那剩下的就和上面的[通过本地HTML网页爬取图片](#markdown-通过本地HTML网页爬取图片)一样了.

####2. 翻页处理
- 当在豆瓣相册我翻页的时候留意了地址的变化.
	- 第一页 `https://www.douban.com/photos/album/102879847/`
	- 第二页 `https://www.douban.com/photos/album/102879847/?start=18`
	- 第三页 `https://www.douban.com/photos/album/102879847/?start=36`  
	- 第n页 `https://www.douban.com/photos/album/102879847/?start=18*(n-1)`
	
	不难发现其中的规律,就是在原来的基础上加了`?start=x`,这个`x`为18的倍数,因为每一页至多有18张照片,那么第一页后面应该也是有的`?start=0`,只不过第一页作为缺省值直接隐藏掉了.  
	
	那这就好办了,先把思路理好.我们把有照片的页码相册地址都放到数组里`[album_1,album_2,album_3]`,然后遍历出每一个相册的地址,拿到相册地址,就能接着上面的继续做了.
	
- 分析每一页的源码  
	上面有讲到,照片的url都在源码中的`<div class="photolst clearfix">`和`<div id="link-report"`之间.我们先假设从他们之间截取的字符串的值为`images`,那么如果一个相册只有三页,但是通过地址访问第四页的时候,`images`是什么呢?  
	
	比如我这个相册,`https://www.douban.com/photos/album/102879847/?start=54`,查看源码会发现`images`是:
	
			'
		
		
		</div>
		
		'  
	那么去掉这个字符串中的空格,只需判断下`images`是否等于`</div>`.假设相册url后面那个`?start=x`中的x默认是0,那么从0开始,只要`images`的值不等于`</div>`就把这个的url加到数组中.
	
		# -*- coding: utf-8 -*-
		import re
		import os
		import requests
		import urllib2
		
		album_url = 'https://www.douban.com/photos/album/102879847/'
		html = urllib2.urlopen(album_url).read()
		images = re.findall('"photolst clearfix">(.*?)<div id="link-report',html,re.S)[0]
		
		index = 0
		album_list = []	#初始化数组
		while images != "</div>":   #no photos
			# 数据处理,把有照片的相册地址添加到数组中
		    temp_url = album_url + '?start=' + str(index * 18)
		    temp_html = urllib2.urlopen(temp_url).read()
		    temp_images = re.findall('"photolst clearfix">(.*?)<div id="link-report',temp_html,re.S)[0]
		    #去空格
		    images = temp_images.strip()	
		    album_list.append(temp_url)
		    index += 1
		    
		# 移除最后一个多余元素    
		album_list.pop()
		print album_list	
	
	
- 照片的相册地址都已经添加到数组里了,剩下的工作就是遍历这个数组,来下载所有的照片了.
	
		if not os.path.exists('pic'):
	    j = 0
	    # 遍历相册地址url
	    for url in album_list:
	        html = urllib2.urlopen(url).read()
	        # match pics
	        large = re.findall('"photolst clearfix">(.*?)<div id="link-report', html, re.S)[0]
	        pic_url = re.findall('img src="(.*?)" /', large)
	        j+=1
	        i = 0
	        # create folder
	        if not os.path.exists('pic'):
	            print '##开始下载.共' + str(imagesList.__len__() - 1) + '页##'
	            os.makedirs('pic')
		
	        for url in pic_url:
	        # 之前下载的都是缩略图,如果想下载大图,那么把图片url中的`thumb`替换为`large`就行
	            largeImg_url = url.replace('thumb', 'large')
	            print '下载中...' + str(j) + '-' + str(i)
	            pic = requests.get(largeImg_url)
	            fp = open(os.getcwd() + '/pic/' + str(j) + '-' + str(i) + '.jpg', 'wb')
	            fp.write(pic.content)
	            fp.close()
	            i += 1
	            if i == pic_url.__len__():
	                print '##第' + str(j) + '页下载完毕!!##'
		else:
		    print '已下载过了!'	
- 至此,分页下载的功能也已经实现了.如果人性化一点,可以让用户输入,相册地址来实现下载,只需要把相册的url作为input的入参就行了.  
`album_url = raw_input('请输入豆瓣相册地址:')`

###改换三方库
这里我使用了[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)这个三方库,用法文档上写的很清楚了,这个库就是来替代正则截取HTML源码的工作,我们就不需要去找规律来截取想要的字段了,而是想要哪些内容,通过他的类名就可以获取诸如此类的人性化方法.
  
更换后的代码全貌:

	# -*- coding: utf-8 -*-
	# __author__ = 'Heisenbean'
	import os
	import re
	import requests
	import urllib2
	from bs4 import BeautifulSoup
	from requests.packages.urllib3.exceptions import InsecureRequestWarning
	requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
	
	
	index = 0
	base_url = raw_input('请输入豆瓣相册地址:')
	url = base_url + '?start=0'
	html = urllib2.urlopen(url).read()
	
	#match pics
	imagesList = []
	soup = BeautifulSoup(html,'html.parser',from_encoding='utf-8')
	node = soup.find_all('div', {'class': 'photo_wrap'})
	
	while node:   #no photos
	    imagesList.append(url)
	    index += 1
	    url = base_url + '?start=' + str(index * 18)
	    html = urllib2.urlopen(url).read()
	    soup = BeautifulSoup(html, 'html.parser', from_encoding='utf-8')
	    node = soup.find_all('div', {'class': 'photo_wrap'})
	
	if not os.path.exists('pic'):
	    j = 0
	    for url in imagesList:
	        html = urllib2.urlopen(url).read()
	        # init picsList
	        pic_url = []
	        soup = BeautifulSoup(html, 'html.parser', from_encoding='utf-8')
	        node = soup.find_all('div', {'class': 'photo_wrap'})
	        for imageUrl in node:
	            for l in imageUrl.find_all('img'):
	                pic_url.append(l.get('src'))
	        j+=1
	        i = 0
	        # create folder
	        if not os.path.exists('pic'):
	            print '##开始下载.共' + str(imagesList.__len__() - 1) + '页##'
	            os.makedirs('pic')
	
	        for url in pic_url:
	            largeImg_url = url.replace('thumb', 'large')
	            print '下载中...' + str(j) + '-' + str(i)
	            pic = requests.get(largeImg_url)
	            fp = open(os.getcwd() + '/pic/' + str(j) + '-' + str(i) + '.jpg', 'wb')
	            fp.write(pic.content)
	            fp.close()
	            i += 1
	            if i == pic_url.__len__():
	                print '##第' + str(j) + '页下载完毕!!##'
	else:
	    print '已下载过了!'

这里我就过多来介绍这个库的使用了,整篇文章也没有引入特别深的Python技能,都是一些基础的知识,目的就是能够希望有更多的人可以使用Python,喜欢上Python.

毕竟,**人生苦短,_______**.  
最后,代码也放到[Github](https://github.com/Heisenbean/Spiders)了,如果觉得文章对你有帮助,请给一个**star**.
