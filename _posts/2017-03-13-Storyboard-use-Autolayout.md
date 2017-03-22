---
layout: post
title:  "在Storyboard中使用Autolayout
"
date:   2017-03-13 21:15:08 +0800
categories: iOS
---

### 前言:
Autolayout出了这么多年,普及率已经很高了,但是Autolayout大概也会分两种.就是通过代码来布局还是Storyboard来布局?Autolayout刚出那会我是前者,而且使用[UIView-Autolayout](https://github.com/smileyborg/UIView-AutoLayout)这个库来布局.  

但是后来我发现这样布局,并不能直观的看到自己写的约束是什么样子,只能运行一下来看,代码量也比较多.后来通过研究学习了在Storyboard中使用AutoLayout,再配合`User Defined Runtime Attributes`真的是太爽了.个人觉得提高了开发效率.  

有人说项目使用了Git管理,再使用Storyboard的话,其他同事只要点进去Storyboard文件,这个Storyboard就会被标记已修改,其实如果使用好Git的话,都在自己分支下工作,是不会出现这种情况的.如果真的出现了,那选中Storyboard右键点击选择`Source Control`,然后Discard一下就行了.

还有就是在较大型项目中可能会影响编译运行效率,因为Storyboard有一个转化为xml文件的过程,这个也只是理论上的猜测,我也没有实际去测试.

但不管怎样,我个人觉得用Storyboard比较爽,这篇文章也不是讨论是Storyboard布局好还是代码布局好的,只是教大家如何在Storyboard中使用Autolayout.

没用过这种方法布局,但接收的项目中用Storyboard很多,并且使用了Autolayout的同学,可能一看到这么多线会一脸懵逼.WTF?这都什么东西?  
![](http://oclnty4pg.bkt.clouddn.com/wtf.png)

那么下面,我将带领大家来看看Storyboard+Autolayout到底是个什么东西.
### LET'S BEGIN!
目标是做一个市面上很常见的App布局,最后的成品大概长这个样子.  
![](http://oclnty4pg.bkt.clouddn.com/1.gif)

布局上基本是在Storyboard通过拖线(Autolayout)完成.  

1. 给`TableView`添加约束  
首先在`Main.storyboard`里`View Controller`中,在它里面添加一个`TableView`.`TableView`的约束在`View`中居上下左右为0.  
选中刚添加的`TableView`,然后添加新约束.  
![](http://oclnty4pg.bkt.clouddn.com/shot4.gif)

	![](http://oclnty4pg.bkt.clouddn.com/shot7.png)  
	解释下上图中的五个按钮A,B,C,D,E.  
	
	- A: 可以更新所选中控件的frame,比如刚添加过约束,控件的frame如果和约束不一样的话,在storyboard中会出现黄色虚线,按下此按钮可以更新frame,同等于快捷键**option键** + **command键** + **=键**
	
	- B: 不常用,在所选控件中插入(替换掉所选控件)StackView.
	
	- C: 控制控件的排列位置,比如水平方向居中还是竖直方向居中.或者选中两个控件,可以设定他们上下左右对齐.
	
	- D: 可以添加控件自身约束,比如宽高的约束,它在父控件中的约束等.
	
	- E: 选中控件后可以更新或者清除它的约束,添加缺失的约束.也可批量操作所有的控件.
2. 添加UITableViewCell,并设置其可重用标识符(Identifier).  
	![](http://oclnty4pg.bkt.clouddn.com/shot11.png)  
	
3. 在UITableViewCell里添加UILabel作为标题控件并添加约束.
	UILabel比较特殊,添加两个约束就可以了,一般添加居上和居左就可以了,因为没有添加居右的约束,所以就要控制Label的宽度.  
	iOS6之后UILabel增加了一个`preferredMaxLayoutWidth`属性,这个属性就可以控制Label的宽度,但是要想这个属性起作用,`UILabel`的`numberOfLines`属性必须≥1.所以如果没有行高限制的需求,直接把`numberOfLines`设为0就行.Storyboard里可以设置Label的`preferredMaxLayoutWidth`属性,但是写死肯定是不行,不同尺寸屏幕适配会有问题,所以需要使用代码动态来设置这个值.但如果这个Label的宽度如果本身就是设计的很短那就另当别论了.  
至于怎么动态设置Label的`preferredMaxLayoutWidth`等到自定义UITableViewCell再讲.    
	![](http://oclnty4pg.bkt.clouddn.com/shot9.png?imageView3/w/750)    
	![](http://oclnty4pg.bkt.clouddn.com/shot5.png?imageView3/w/750)
4. 在TableViewCell继续添加UILabel作为内容控件并添加约束.  
	这个Label可以让他和之前添加的标题Label产生关系,就是让它和标题Label左对齐,这样做的好处是标题居左的约束如果改变,这个Label也会跟着改变.然后再给它添加一个居标题Label的约束.当然这个Label也要设置`preferredMaxLayoutWidth`属性.
	> 按**shit键**可以选中多个控件  
	
	![](http://oclnty4pg.bkt.clouddn.com/shot8.gif?imageView3/w/750)
5. 设置TableView的代理和数据源为视图控制器(View Controller).这个操作等同于在代码里实现.    
	![](http://oclnty4pg.bkt.clouddn.com/shot10.png?imageView3/w/750)

		self.tableView.delegate = self;
	  	self.tableView.dataSource = self;
	  	
	然后把TableView拖线到ViewController中.    
	
	`@property (weak, nonatomic) IBOutlet UITableView *tableView;`
	
6. 实现TableView的数据源和代理.

		#import "ViewController.h"
		@interface ViewController () <UICollectionViewDataSource,UICollectionViewDelegateFlowLayout>
		@property (strong,nonatomic) NSMutableArray *datas;
		@property (weak, nonatomic) IBOutlet UITableView *tableView;
		
		@end
		
		@implementation ViewController
		
		- (NSMutableArray *)datas{
		    if (!_datas) {
		        _datas = [NSMutableArray array];
		    }
		    return _datas;
		}
		
		
		- (void)viewDidLoad {
		    [super viewDidLoad];
		    [self loadData];
		
		    // Do any additional setup after loading the view, typically from a nib.
		}
		
		- (void)loadData{
		    NSString *path = [[NSBundle mainBundle] pathForResource:@"Datas.plist" ofType:nil];
		    self.datas = [NSMutableArray arrayWithContentsOfFile:path];
		}
		
		#pragma mark - UITableViewDataSource
		- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
		    return self.datas.count;
		}
		
		- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
		    static NSString *identifier = @"cell";
		    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier forIndexPath:indexPath];
		    return cell;
		}
		
		- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
		    return 44;	
		}
	目前的问题有三个:
	
	- 尚未自定义Cell
	- 数据没有赋值到Cell里的控件.
	- Cell高度没有动态返回

 

7. 自定义<a name="UITableViewCell">UITableViewCell</a>,新建一个基于UITableViewCell类,我这里取名为`MyTableViewCell`.然后返回到`Main.`里,把的UITableViewCell的的类继承改为刚新建的类.  
![](http://oclnty4pg.bkt.clouddn.com/shot12.png)  
接着把`Main.storyboard`中UITableViewCell中的控件都拖到自定义的Cell里.

		@interface MyTableViewCell : UITableViewCell
		@property (weak, nonatomic) IBOutlet UILabel *myTitle;
		@property (weak, nonatomic) IBOutlet UILabel *myContent;
		@end
		
	进入`MyTableViewCell.m`文件中设置Label的preferredMaxLayoutWidth.

		- (void)awakeFromNib {
		    [super awakeFromNib];
		    _myTitle.preferredMaxLayoutWidth = [UIScreen mainScreen].bounds.size.width - 20;
		    _myContent.preferredMaxLayoutWidth = [UIScreen mainScreen].bounds.size.width - 20;
		}
	
		
8. 给Cell里的控件赋值,第六步我已经写出了加载数据的方法,是通过加载plist文件里的假数据.
自己可以新建一个plist文件.
数据结构大概是一个数组里元素为字典,字典里有两个key,分为别`title`和`content`.

		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
		<plist version="1.0">
		<array>
			<dict>
				<key>title</key>
				<string></string>
				<key>content</key>
				<string></string>
			</dict>
		</array>
		</plist> 		
		
	![](http://oclnty4pg.bkt.clouddn.com/shot13.png)  
	在自定义Cell的`.h`文件中声明一个字典类型的属性,因为我们数据数组里放的是字典.

		@interface MyTableViewCell : UITableViewCell
		@property (weak, nonatomic) IBOutlet UILabel *myTitle;
		@property (weak, nonatomic) IBOutlet UILabel *myContent;
		@property (strong,nonatomic) NSDictionary *data;
		@end


	在自定义Cell的`.m`文件中实现`data`属性的set方法并给控件赋值.

		- (void)setData:(NSDictionary *)data{
			_data = data;
		    _myTitle.text = data[@"title"];
		    _myContent.text = data[@"content"];
		}		
		
9. 计算Cell的高度
	在自定义Cell的`.h`文件中声明一个返回Cell行高的方法.     
	
		@interface MyTableViewCell : UITableViewCell
		@property (weak, nonatomic) IBOutlet UILabel *myTitle;
		@property (weak, nonatomic) IBOutlet UILabel *myContent;
		@property (strong,nonatomic) NSDictionary *data;
		- (CGFloat)rowHeight:(NSDictionary *)data;
		@end
	在`.m`文件中实现这个方法.  
	
		- (CGFloat)rowHeight:(NSDictionary *)data{
		  self.data = data;
	  	  [self layoutIfNeeded];
	  	  return CGRectGetMaxY(self.myContent.frame);
		}

	`self.data = data;`这个方法给`data`赋值又会重新走一遍`data`的set方法保证是当前Cell传进来的数据,以防计算错误.  
`[self layoutIfNeeded];`给Cell的控件赋值之后需要强制更新下布局,让所有控件的frame为准确的值,确保最后我们返回的值为内容Label的最大Y值.
	> 一般返回的都是Cell中最靠下的控件的最大的Y值,就是Cell的高度了.所以所谓的计算Cell高度,其实就是返回最后一个控件的最大Y值而已.

10. 最后在控制器(ViewController.m)中修改下TableView的数据源和代理方法.

		- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
		    static NSString *identifier = @"cell";
		    MyTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier forIndexPath:indexPath];
		    cell.data = self.datas[indexPath.row];
		    return cell;
		}
		
		- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
		    MyTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"];
		    return  [cell rowHeight:self.datas[indexPath.row]];
		}

最后运行一下,应该会看到的是大概下面这种界面.  
![](http://oclnty4pg.bkt.clouddn.com/shot14.png?imageView2/2/w/600/q/95)  
## 重点来了!
下面的图片布局,其实也可以用九宫格算法来写,但是既然是在Storyboard中使用Autolayout,那就一路走到底.
那么我采用的是用UICollectionView,不用自定义Layout.如果单独把UICollectionView拿出来使用,那么是挺简单的,但是如果UITableViewCell中,就稍微有点麻烦了.

如果按照以往的思路,UICollectionView是在UITableViewCell中的,那么UICollectionView的数据源和代理应该是谁? 如果是UITableViewCell的话那么会出现问题, TableView滚动的时候,UICollectionView的数据源和代理方法也会触发,就会导致UICollectionView布局出现问题.那么还是把UICollectionView的数据源和代理设置为视图控制器(ViewController).既然理清了这个思路,下面就好办了.  

1. 首先把UICollectionView拖入到之前的[UITableViewCell](#UITableViewCell)中,然后添加约束为,居*上下左*一定距离,高度约束写死一个值.因为UICollectionView不像UILabel那样设置好约束会随着内容的增多而换行增高,所以只能把UICollectionView的高度先写死,然后动态计算高度.
	添加完约束之后,选中UICollectionView,然后点击`尺寸检查器`,往下拉可以看到添加的约束.
	
	![](http://oclnty4pg.bkt.clouddn.com/shot15.png)  
	
2. 在UICollectionViewCell中添加UIImageView,然后设置UIImageView的约束为居上下左右为0.

3. 然后选中UICollectionView,在`尺寸检查器`中找到高度的约束,双击之后可以看到Storyboard中高度的约束线会被选中,最左边也可以看到选中,这个约束像控件一样可以拖到代码文件中.顺便把添加的UICollectionView和UICollectionViewFlowLayout也拖入到自定义的UITableViewCell中.

		@interface MyTableViewCell : UITableViewCell
		@property (weak, nonatomic) IBOutlet UILabel *myTitle;
		@property (weak, nonatomic) IBOutlet UILabel *myContent;
		@property (strong,nonatomic) NSDictionary *data;
		@property (weak, nonatomic) IBOutlet NSLayoutConstraint *collectionViewHeightConst;
		@property (weak, nonatomic) IBOutlet UICollectionViewFlowLayout *layout;
		@end
		
	![](http://oclnty4pg.bkt.clouddn.com/shot16.gif)  
4. 在之前的plist文件中再添加图片字段,为了不引用第三方库就直接在本地设置图片.先把图片下载到本地拖到项目的`Assets.xcassets`中.
	> 此处安利下我的[图片爬虫](https://github.com/Heisenbean/Spiders/tree/master/doubanPic_spider),用于爬取豆瓣相册,挺方便的.
	
5. 还需在新建一个UICollectionView类,用与接收控制器传来的UICollectionViewCell的index.
	
		#import <UIKit/UIKit.h>
		@interface PhotoCollectionView : UICollectionView
		
		@property (nonatomic, strong) NSIndexPath *indexPath;
		
		@end
		@interface MyTableViewCell : UITableViewCell
		@property (weak, nonatomic) IBOutlet UILabel *myTitle;
		@property (weak, nonatomic) IBOutlet UILabel *myContent;
		@property (weak, nonatomic) IBOutlet PhotoCollectionView *collectionView;
		@property (strong,nonatomic) NSDictionary *data;
		@property (weak, nonatomic) IBOutlet NSLayoutConstraint *collectionViewHeightConst;
		@property (weak, nonatomic) IBOutlet UICollectionViewFlowLayout *layout;
		- (CGFloat)rowHeight:(NSDictionary *)data;
		- (void)setCollectionViewDataSourceDelegate:(id<UICollectionViewDataSource, UICollectionViewDelegate>)dataSourceDelegate indexPath:(NSIndexPath *)indexPath;
		
		@end

	
	`- (void)setCollectionViewDataSourceDelegate:(id<UICollectionViewDataSource, UICollectionViewDelegate>)dataSourceDelegate indexPath:(NSIndexPath *)indexPath;`
	此方法就是设置设置控制器为UICollectionView的代理和数据源.

6. 计算UICollectionView的layout和高度约束以及TableViewCell的高度.

		#import "MyTableViewCell.h"
		#import "MyCollectionViewCell.h"
		#define photoMargin 5
		@implementation PhotoCollectionView
		
		@end
		@implementation MyTableViewCell
		
		- (void)awakeFromNib {
		    [super awakeFromNib];
		    _myTitle.preferredMaxLayoutWidth = [UIScreen mainScreen].bounds.size.width - 20;
		    _myContent.preferredMaxLayoutWidth = [UIScreen mainScreen].bounds.size.width - 20;
		    CGFloat size = ([UIScreen mainScreen].bounds.size.width - 20 - photoMargin * 2) / 3;
		    self.layout.itemSize = CGSizeMake(size,size);
		    self.layout.minimumLineSpacing = photoMargin;
		    self.layout.minimumInteritemSpacing = photoMargin;
		}
		
		- (void)setData:(NSDictionary *)data{
		    _data = data;
		    _myTitle.text = data[@"title"];
		    _myContent.text = data[@"content"];
		}
		
		- (void)layoutSubviews{
		    [super layoutSubviews];
		    [self setCollectionViewSize:self.data];
		}
		
		- (void)setCollectionViewSize:(NSDictionary *)data{
		    CGFloat singlePhotoSize = ([UIScreen mainScreen].bounds.size.width - 20 - photoMargin * 2) / 3;
		
		    NSArray *images = data[@"images"];
		    if (images.count == 0) {
		            self.collectionViewHeightConst.constant = 0;
		    }else{
		        if (images.count <= 3) {    // 1 line
		            self.collectionViewHeightConst.constant = singlePhotoSize;
		        }else if(images.count > 3 && images.count <= 6){ // 2 line
		            self.collectionViewHeightConst.constant = singlePhotoSize * 2 + photoMargin;
		        }else{  // 3 line
		            self.collectionViewHeightConst.constant = singlePhotoSize * 3 + photoMargin * 2;
		        }
		    }
		}

	没有图片的时候, collectionViewHeightConst就是0,1-3张图片就是一行的高度,4-6张就是2行的高度加上一个图片间距,大于6张就是3行的高度加上两个图片间距.
	
	因为多了UICollectionView,所以高度就要重新计算,也很简单.如果有图片的话,那返回的最大Y值就是UICollectionView的,如果没有图片那就返回内容label的.
		
		- (CGFloat)rowHeight:(NSDictionary *)data{
	    	self.data = data;
	    	[self layoutIfNeeded];
	    	NSArray *images  = data[@"images"];
	    	if (images.count == 0) {
	      	  return CGRectGetMaxY(self.myContent.frame);
	    	}else{
	      	  return CGRectGetMaxY(self.collectionView.frame);
	   	 	}
		}
	
7. 先在自定义的TableViewCell中实现设置数据源和代理的方法.
	
		- (void)setCollectionViewDataSourceDelegate:(id<UICollectionViewDataSource, UICollectionViewDelegate>)dataSourceDelegate indexPath:(NSIndexPath *)indexPath{
		    self.collectionView.dataSource = dataSourceDelegate;
		    self.collectionView.delegate = dataSourceDelegate;
		    self.collectionView.indexPath = indexPath;
		    [self.collectionView reloadData];
		}
	然后再控制器调用这个方法.
	
		-(void)tableView:(UITableView *)tableView willDisplayCell:(MyTableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath{
			// 该方法中本是UITableViewCell,为了方便可以改为自定义的,比如我的是MyTableViewCell
	        [cell setCollectionViewDataSourceDelegate:self indexPath:indexPath];    
		}
		
	最后在控制器中实现UICollectionView的代理和数据源.
	
		#pragma mark - UICollectionViewDataSource
		- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
		    
		    NSDictionary *dic = self.datas[[(PhotoCollectionView *)collectionView indexPath].row];
		    NSArray *images = dic[@"images"];
		    return images.count;
		}
		
		- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
		    NSDictionary *dic = self.datas[[(PhotoCollectionView *)collectionView indexPath].row];
		
		    MyCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"cell" forIndexPath:indexPath];
		    NSArray *images = dic[@"images"];
		    cell.imageName = images[indexPath.row];
		    return cell;
		}

	至此,就做到了把UICollectionView的数据源和代理从UITableViewCell传递出来,让控制器来控制了.

#### 运行项目,应该就是文章最前面的[效果](#)了.

最后分享一下Autolayout的一些小tips.

1. 快速给选中的控件添加当前frame的约束.  
		![](http://oclnty4pg.bkt.clouddn.com/shot17.gif)  
2. 快速给选中的控件添加宽高约束和上左下右的约束.  
	> 按着**control键**在不超出控件范围的随意一个斜方向拖线,(左上右上左下右下都可以),松了之后会发现可以设置宽高约束,然后按着**shift键**可以同时设置多个值,按下**回车键**确定.如果只想设置高度约束,那么竖直方向拖线就行,单独设置宽度约束的话就水平方向拖线.  
	
	> 和上面那个操作基本一样,就是拖线超出所选控件本身.比如拖到了它父控件上,就可以设置上左下右的约束了,同理拖线的方向也决定可设置约束的类型.
	
	![](http://oclnty4pg.bkt.clouddn.com/shot18.gif)
3. 快速给两个控件添加相关约束,比如让Label2和Label1左对齐,垂直方向添加一定约束.  
	> 也可以设置两个控件等宽等高,水平或垂直居中等约束.如果添加完之后frame不对,按**option键** + **command键** + **=键**更新frame.  
	
	![](http://oclnty4pg.bkt.clouddn.com/shot19.gif)	
	
源码:	[https://github.com/Heisenbean/BlogDemos](https://github.com/Heisenbean/BlogDemos)  

其中UITableViewCell中添加UICollectionView参考了下面这篇文章:  
[https://ashfurrow.com/blog/putting-a-uicollectionview-in-a-uitableviewcell/](https://ashfurrow.com/blog/putting-a-uicollectionview-in-a-uitableviewcell/)	