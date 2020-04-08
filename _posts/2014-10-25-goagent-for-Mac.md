---
layout: post
title:  "Mac下利用goagent代理教程"
date:   2014-10-25 00:34:08 +0800
tags: Mac
---

FQ=fanqiang

由于度娘吞了我之前上传的工具,所以我把这个教程中用到的插件和文件都整合到一起了.

[请戳](https://pan.baidu.com/s/1bpMer7L),密码:jyc1

作为程序猿,翻墙上谷歌和其他一些网站是家常便饭,但是有一些刚入门的猿不会翻墙,这里我就教大家在Mac下如何使用goagent跨越GFW.

首先感谢goagent的两位作者[Phus Lu](https://github.com/phuslu)和[hewigovens](https://github.com/hewigovens)为广大网友带来了这么好的福利,如果对goagent不了解的可以去wiki搜索一下.

goagent在github上也有很详细的介绍以及goagent的部署方法,但是在Mac下的方法不是很详细,而且这些天github不翻墙的情况下,访问非常不稳定,所以这里我就借花献佛给大家讲一下Mac下goagent在chrome上的部署方法.

### 那么废话少说,直接上教程.

#### 1.Google Chrome

谷歌自己出的浏览器,我个人是很喜欢的,上面有丰富的插件,一键同步书签,开发者模式等等,唯一的缺点就是太吃内存,因为这里讲的goagent代理就是基于chrome的,其他浏览器Firefox应该也有相关的教程,等chrome下的goagent代理做好之后可以谷歌查阅一下Firefox下的配置方法.  

#### 2.appid

- appid谷歌应用的ID,就是用这个ID获取流量的-->[https://appengine.google.com/](https://appengine.google.com/)  

- 关于申请appid的方法大家可以移步至[goagent](https://github.com/goagent/goagent)的github主页,上面有图文并茂的申请方法.不要嫌麻烦,一步一步来.

- 如果没有挂代理或者翻墙的话申请appid的网站可能都上不去,因为申请appid那个网站也是谷歌的,所以建议修改自己的hosts文件后登陆申请appid.

> 注意:
 
> 1.appid申请是比较蛋疼的,因为你电脑没挂代理的情况下,理论上是申请不了appid的,所以这里的建议是找一些免费的vpn或者翻墙软件,再申请appid.

>2.申请appid的时候,appid的名称尽量设的复杂一些,这样唯一性就比较高,有时候检测是可用的,但实际上也不可用.

#### 3.goagent源文件

- 这里除了需要goagent的源文件意外,还需要一个GoagentMac.app文件中的一个info.plist文件.

- goagent的源文件,在它的github上也有下载,也是最新版本,但是我自己用github上的最新版本,不能向服务器上传数据,所以这里,还是用我之前的老版本. 为了方便,已经传[度盘](https://pan.baidu.com/s/1mgA5VuS),密码:qx2a  
- goagent的部署步骤:  

	**本地端(local)的部署:**  
	
	- 进入到goagent的local目录,打开proxy.ini文件,要用文本编辑器打开,然后修改appid为自己之前申请的那个appid,把yourAppID换成你之前申请的.修改完之后保存退出,密码可以不用写.  
![](http://oclnty4pg.bkt.clouddn.com/agent-1.png)  
![](http://oclnty4pg.bkt.clouddn.com/080217502128332.png)

	**服务器端(server)部署:**  
	
	- 下载好后的goagent放到桌面或者其他地方,打开之后进入到server目录,然后打开终端,在终端输入python,然后敲下空格,然后把uploader.zip拖入到终端里,最后回车.如下图   
![](http://oclnty4pg.bkt.clouddn.com/agent-2.gif)

	- 上传数据的时候会让你填appid,邮箱和密码,填写密码的时候要小心一点,因为你输入密码的时候是不显示输入了多少个的,所以要小心写,不要写错了.
	- 最后如果看到英文的上传成功的提示,说明服务器配置完成.
	- 设置代理启动路径,这时就用到了GoagentMac.app,为了方便,也传到了[度盘](https://pan.baidu.com/s/1i3mVydv),密码:9pxf  
	- 把下载好的GoagentMac.app解压缩到桌面后拖到"应用程序"里  
	- 右键点击,打开"显示包内容",在"Contents"文件里找到info.plist,修改proxy.py的路径,其实就是GoAgentPath的路径.
	- proxy.py存放在/goagent/local中,如果是按我之前的步骤做的,那么proxy.py的路径就是/Users/Mac/Desktop/goagent/local/proxy.py,把红框中的地址改成自己proxy.py的路径  
	- 要查看自己的proxy.py路径,可以直接把这个文件拖到终端中查看,为了桌面简洁,也建议把goagent文件夹拖到"应用程序"中,之前把GoagentMac.app拖到"应用程序"里也是为了桌面简洁,全是个人喜好,如果把goagent文件夹拖到"应用程序"中的话,那info.plist中的proxy.py路径也就是下图红框中的内容就不用改了.  
	![](http://oclnty4pg.bkt.clouddn.com/agent3-1.png)
 

#### 4.SwitchSharp插件

一款chrome下的插件,配合goagent一起使用的.

- 为了方便,插件的安装包我也传到了[度盘](https://pan.baidu.com/s/1bn70x7h),密码:3hmd

- SwitchSharp安装到chrome的方法

- 打开chrome后

- 点击右上角的三条杠标志的按钮

- 点击"工具"

- 点击"扩展程序"

- 把下载好的插件拖到"扩展程序"的页面

#### 5.配置SwitchSharp

> 为了方便起见,SwitchSharp的配置信息,我也都备份好放在了[度盘](https://pan.baidu.com/s/1kT7Qdc7),密码:5wev,大家只需要下载备份的配置文件(SwitchyOptions.bak),导入到SwitchSharp里就可以了.

##### 导入的步骤:

- 进入SwitchSharp选项界面,安装好SwitchSharp后chrome的右上角应该多了一个小地球的标志,点它,然后选择选项.

- 点击"从文件恢复"

- 选择下载好的配置文件(SwitchyOptions.bak)

- 保存配置  

	![](http://oclnty4pg.bkt.clouddn.com/agent-3.png)



#### 6.打开proxy.py,开启代理
- 在/goagent/local目录中找到proxy.py文件,右键点击选择其他方式打开,选择使用终端打开,并且把"始终以此方式打开"的勾给选上,打开proxy.py之后就算开启了代理,状态入下图.建议这个时候把proxy.py文件制作一个替身放到桌面,这样以后再开代理直接双击桌面上的proxy.py的替身就行了.  

　　![](http://oclnty4pg.bkt.clouddn.com/agent-4.png)
　

- 打开chrome,点击右上角的小地球,选择"自动切换模式",这样SwitchSharp就会自动帮你切换代理,比如说你上一些国内不需要代理的网站,SwitchSharp就不会开启代理,因为如果用代理上国内的网站是有些卡的,如果你要上一些需要翻墙的网站,他就会帮你自动挂代理.不过有些时候它也不是100%智能,有些需要翻墙的网站它没有帮你挂代理,就需要你自己手动开启代理,比如说你上了一个xxx.com的网站,页面打不开,这个时候你就需要手动开启代理:点击"小地球",然后点击"新建规则"下面的xxx.com这个网址,选择连接方式为"goagent".

- 到此为止,代理就做好了.

 
	![](http://oclnty4pg.bkt.clouddn.com/agent-5.png)



#### 结尾:
这篇教程的目的就是帮那些需要翻墙上网,但又不想花钱买VPN的网友做个一个科普,还有喜欢自己钻研东西的网友也可以做下参考,自己做代理翻墙上网的感觉肯定和花钱买VPN上网的感觉不一样.如果教程有什么不对的地方或者纰漏可以留言给我,我有时间会修改.

最后:祝大家上网愉快!

:)