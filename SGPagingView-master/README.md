
# SGPagingView


## 目录
* [效果图](#效果图)
* [主要内容的介绍](#主要内容的介绍)
* [SGPagingView集成](#SGPagingView集成)
* [代码介绍](#代码介绍)
* [问题及解决方案](#问题及解决方案)
* [版本介绍](#版本介绍)
* [Concludingremarks](#Concludingremarks)


## 效果图
![](https://github.com/kingsic/SGPagingView/raw/master/Gif/sorgle.gif) 


## 主要内容的介绍
* `多种指示器长度样式`<br>
* `多种指示器滚动样式`<br>
* `标题按钮文字渐显效果`<br>
* `标题按钮文字缩放效果`<br>


## SGPagingView 集成
* 1、CocoaPods 导入 pod 'SGPagingView', '~> 1.2.3'
* 2、下载、拖拽 “SGPagingView” 文件夹到工程中


## 代码介绍
#### SGPagingView 的使用（详细使用, 请参考 Demo）
```
    /// 子控制器及 pageContentView 的创建
    ChildVCOne *oneVC = [[ChildVCOne alloc] init];
    ChildVCTwo *twoVC = [[ChildVCTwo alloc] init];
    ChildVCThree *threeVC = [[ChildVCThree alloc] init];
    ChildVCFour *fourVC = [[ChildVCFour alloc] init];
    // 子控制器数组
    NSArray *childVCArr = @[oneVC, twoVC, threeVC, fourVC];
    CGFloat contentViewHeight = self.view.frame.size.height - 108;
    self.pageContentView = [[SGPageContentView alloc] initWithFrame:CGRectMake(0, 108, self.view.frame.size.width, contentViewHeight) parentVC:self childVCs:childVCArr];
    _pageContentView.delegatePageContentView = self;
    [self.view addSubview:_pageContentView];
    
    /// 子标题及 pageTitleView 的创建
    NSArray *titleArr = @[@"精选", @"电影", @"电视剧", @"综艺"];
    self.pageTitleView = [SGPageTitleView pageTitleViewWithFrame:CGRectMake(0, 64, self.view.frame.size.width, 44) delegate:self titleNames:titleArr];
    [self.view addSubview:_pageTitleView];
```

* 滚动内容视图的代理方法
```
- (void)pageContentView:(SGPageContentView *)pageContentView progress:(CGFloat)progress originalIndex:(NSInteger)originalIndex targetIndex:(NSInteger)targetIndex {
    [self.pageTitleView setPageTitleViewWithProgress:progress originalIndex:originalIndex targetIndex:targetIndex];
}
```

* 滚动标题视图的代理方法
```
- (void)pageTitleView:(SGPageTitleView *)pageTitleView selectedIndex:(NSInteger)selectedIndex {
    [self.pageContentView setPageCententViewCurrentIndex:selectedIndex];
}
```


#### SGPagingView 的介绍
|主要属性、方法|描述|
|----|-----|
|**selectedIndex**|选中标题下标|
|**resetSelectedIndex**|重置标题下标|
|**indicatorScrollStyle**|指示器滚动样式|
|**resetTitleWithIndex:newTitle:**|更改指定下标的标题|
|**initWithFrame:delegate:titleNames:titleFont:**|带有标题字号的初始化方法，与之对应一个类方法|


## 问题及解决方案
#### 1、CocoaPods 安装 SGPagingView 时，遇到的问题及解决方案
* 若在使用 CocoaPods 安装 SGPagingView 时，出现 [!] Unable to find a specification for SGPagingView 提示时，打开终端先输入 pod repo remove master；执行完毕后再输入 pod setup 即可 (可能会等待一段时间)
***

#### 2、关于父子控制器的说明（SGPageContentView 与 SGPageContentScrollView）
>**SGPageContentView 使用的是 UICollectionView 的重用机制管理子视图
内部是先添加子视图控制器到父视图控制器上（[self.parentViewController addChildViewController:childVC]），再添加子视图的 view 到父视图的 view 上的（[cell.contentView addSubview:childVC.view]），这时会存在一个问题：即第一次加载第一个子视图时，第一个子视图的 viewWillAppear 方法不会被调用；原因是，先调用 addChildViewController，子视图控制器与父视图控制器的事件同步，即当父视图控制器的 viewDidAppear 调用时，子视图控制器的 viewDidAppear 方法会调用一次，再调用 addSubView 也不会触发viewWillAppear 和 viewDidAppear；所以第一次加载子视图控制器时 viewWillAppear 不会被调用，再去加载其他子视图控制器不会出现这种问题了。说明：针对这种情况网络数据请求建议在 viewDidLoad 或 viewDidAppear 中作处理

>**SGPageContentScrollView 使用的是 UIScrollView 拖拽结束后的方法加载子视图
内部是先添加子视图的 view 到父视图的 view 上的（[self.scrollView addSubview:childVC.view]），再添加子视图控制器到父视图控制器上（[self.parentViewController addChildViewController:childVC]），这时会存在一个问题：即第一次加载第一个子视图时，第一个子视图的 viewDidAppear 方法会调用二次；原因是，先调用 addSubView 时，viewWillAppear 和 viewDidAppear 会各调用一次，再 addChildViewController 时，子视图控制器与父视图控制器的事件同步，即当父视图控制器的 viewDidAppear 调用时，子视图控制器的 viewDidAppear 方法会再调用一次；所以第一次加载的子视图控制器时 viewDidAppear 方法会被调用两次，再去加载其他子视图控制器不会出现这种问题。说明：针对这种情况网络请求数据建议在 viewWillAppear 或 viewDidLoad 中作处理
***

#### 3、SGPageContentView 内容存在偏移量问题及解决方案（v1.1.5 之后不需要做处理）
* 父控制器中一定要加上 self.automaticallyAdjustsScrollViewInsets = NO; 这句代码；否则 pageContentView 会存在偏移量下移问题
* 子控制中使用纯代码创建 tableView 时，会存在内容区域显示问题
* 纯代码在 viewDidLoad 方法中创建 tableView 时，高度一定要等于 SGPageContentView 的高度或使用 Masonry 进行约束；
* XIB 创建 tableView 时，不会出现这种问题，是因为 XIB 加载完成之后会调用 viewDidLayoutSubviews 这个方法，所以 XIB 中创建 tableView 不会出现约束问题

##### 下面提供三种解决方案（仅供参考）
```
- (void)viewDidLoad {
    [super viewDidLoad];
    /// 解决方案一（tableView 的高度等于 SGPageContentView 的高度）
    self.tableView = [[UITableView alloc] initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, self.view.frame.size.height - 108) style:UITableViewStylePlain];
    _tableView.dataSource = self;
    [self.view addSubview:self.tableView];

    /// 解决方案二（使用 Masonry 进行约束）
    self.tableView = [[UITableView alloc] initWithFrame:CGRectZero style:UITableViewStylePlain];
    _tableView.dataSource = self;
    [self.view addSubview:self.tableView];
    [self.tableView mas_makeConstraints:^(MASConstraintMaker *make) {
         make.edges.equalTo(self.view);
    }];
}
```

```
- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];
    /// 解决方案三（懒加载 tableView，并在 viewDidLayoutSubviews 方法中调用）
    [self.view addSubview:self.tableView];
}
```

```
- (UITableView *)tableView {
    if (!_tableView) {
        _tableView = [[UITableView alloc] initWithFrame:self.view.frame style:UITableViewStylePlain];
        _tableView.dataSource = self;
    }
    return _tableView;
}
```


## 版本介绍

* 2016-10-7 ：初始版本的创建

* 2017-4-13 ：版本升级（根据标题内容自动识别 SGPageTitleView 是静止还是滚动）

* 2017-5-12 ：SGPageContentView 新增是否需要滚动属性

* 2017-6-1  ：v1.1.0 解决标题中既有中文又有英文存在的 Bug

* 2017-6-15 ：v1.1.5 新增新浪微博模块以及代码的优化

* 2017-7-21 ：v1.1.7 新增 SGPageContentScrollView 类（滚动结束之后加载子视图）以及加入 pods 管理

* 2017-8-11 ：v1.2.0 新增指示器滚动样式

* 2017-9-5  ：v1.2.2 新增 SGPageTitleView 带有 titleFont 的创建方法


## Concluding remarks

* 如在使用中, 遇到什么问题或有更好建议者, 请记得 [Issues me](https://github.com/kingsic/SGPagingView/issues) 或 kingsic@126.com 邮箱联系我

