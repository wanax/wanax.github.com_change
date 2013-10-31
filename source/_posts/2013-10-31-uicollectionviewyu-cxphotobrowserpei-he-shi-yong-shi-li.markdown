---
layout: post
title: "UICollectionView与CXPhotoBrowser配合使用示例"
date: 2013-10-31 15:56
comments: true
categories: Tec
---
UICollection是在iOS6.0中新加的功能，用于图片的集中展示，大概就是图片墙的意思。基本的使用类似于UITableView，也是设置代理，数据源，对单个的Cell可以自行定制样式。稍有不同的是在UITableView中整体样式的设置只有两种，一种是Group和Plain，而在UICollection中则有较多的选择方案，具体的实现方式需要编码人员对UICollectionViewFlowLayout自行调整。

开发的App中有用到这种功能，结合一个台湾哥儿们Chris Xu的开源插件CXPhotoBrower，可以达到比较好的展示图片的效果。

具体效果图如下：

![image](/images/tec/uicollectionImg.png)

![image](/images/tec/cxphotobrowserImg.png)

现在开始一步步搭建吧。

<!--more-->

###一.备齐原料
UICollection是系统自带的，这个不用关心，CXPhotoBrowser需要自行下载引入。建议使用CocoaPods进行配置，`pod 'CXPhotoBrowser'`即可，没有用过的话那进入下面的页面按指引搭建吧。

>https://github.com/ChrisXu1221/CXPhotoBrowser

###二.UICollection的使用
####1.新建一个UICollectionViewController的VC
####2.初始化配置UICollectionViewFlowLayout，内嵌于UICollectionViewController中对CollectionCell进行样式管理。
```objc
	UICollectionViewFlowLayout *flowLayout = [[UICollectionViewFlowLayout alloc] init];
	/*
	对四方边界与cell之间的距离进行设置，也可以通过代理设置
	*/
    [flowLayout setItemSize:CGSizeMake(163, 309)];
    flowLayout.sectionInset = UIEdgeInsetsMake(10,15,5,15);
    [flowLayout setMinimumLineSpacing:5.0];
    [flowLayout setMinimumInteritemSpacing:5.0];
    [flowLayout setScrollDirection:UICollectionViewScrollDirectionVertical];
    self.collectionView = [[UICollectionView alloc] initWithFrame:CGRectZero collectionViewLayout:flowLayout];
    //使用自定义的CollectionCell，出队列，与UITableView的思路相近
    [self.collectionView registerNib:[UINib nibWithNibName:@"PicCollectionCell" bundle:nil] forCellWithReuseIdentifier:GraphExcCellIdentifier];
    [self.collectionView setBackgroundColor:[Utiles colorWithHexString:@"#5b4d41"]];
    self.collectionView.delegate = self;
    self.collectionView.dataSource = self;
    //使用开源插件进行下拉分页的功能，读者自行谷歌
    [self.collectionView addInfiniteScrollingWithActionHandler:^{
        [self addPics:@"" reset:NO];
    }];
    self.collectionView.indicatorStyle = UIScrollViewIndicatorStyleWhite;
```
####3.CollectionCell的创建
使用nib创建Cell，过程类似于tableCell，不再赘述了。
####4.Delegate与DataSource的设置
```objc
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return [self.images count];
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    FinanPicCollectCell *cell = (FinanPicCollectCell *)[collectionView dequeueReusableCellWithReuseIdentifier:GraphExcCellIdentifier forIndexPath:indexPath];
    id model=[self.images objectAtIndex:indexPath.row];
    //使用SDWebImage进行图片加载
    [cell.imageView setImageWithURL:[NSURL URLWithString:[model objectForKey:@"smallImage"]]
                   placeholderImage:[UIImage imageNamed:@"LOADING.png"]];
    return cell;
}
/*
等同于
[flowLayout setMinimumLineSpacing:5.0];
[flowLayout setMinimumInteritemSpacing:5.0];
但可针对每个Cell单独定制
*/
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section{
    return 10.0;
}
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section{
    return 2.0;
}
```

###至此，一个基本的UICollection便简单的搭建完毕了。下面使用CXPhotoBrowser实现图片的单张浏览功能，支持左右切换与捏合放大缩小。

###三.CXPhotoBrowser的使用
此插件也是使用数据源代理的思路实现的，我们在载入UICollection的同时也一起对CX进行数据源的设置，另外实现两个代理方法，一个是点击单张图片获取index一遍进行此图片的放大浏览，另一个是在单张浏览功能里左右滑动时获取前后图片的index用于最上面信息栏的展示。
```objc
//pragma mark -
//pragma mark CXPhotoBrowserDelegate
//点击图片弹出详细页面
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
    [self.browser setInitialPageIndex:indexPath.row];
    [self presentViewController:self.browser animated:YES completion:^{
    }];
}
//左右切换视图事件
- (void)photoBrowser:(CXPhotoBrowser *)photoBrowser didChangedToPageAtIndex:(NSUInteger)index{
    [self.imageTitleLabel setText:[NSString stringWithFormat:@"%d/%d",(index+1),[self.images count]]];
}
//pragma mark - CXPhotoBrowserDataSource
- (NSUInteger)numberOfPhotosInPhotoBrowser:(CXPhotoBrowser *)photoBrowser
{
    return [self.photoDataSource count];
}
- (id <CXPhotoProtocol>)photoBrowser:(CXPhotoBrowser *)photoBrowser photoAtIndex:(NSUInteger)index
{
    if (index < self.photoDataSource.count)
        return [self.photoDataSource objectAtIndex:index];
    return nil;
}
//对上方信息栏进行设置
- (CXBrowserNavBarView *)browserNavigationBarViewOfOfPhotoBrowser:(CXPhotoBrowser *)photoBrowser withSize:(CGSize)size
{
    CGRect frame;
    frame.origin = CGPointZero;
    frame.size = size;
    if (!navBarView)
    {
        navBarView = [[CXBrowserNavBarView alloc] initWithFrame:frame];       
        [navBarView setBackgroundColor:[UIColor clearColor]];       
        UIView *bkgView = [[UIView alloc] initWithFrame:CGRectMake( 0, 0, size.width, size.height)];
        [bkgView setBackgroundColor:[UIColor blackColor]];
        bkgView.alpha = 0.2;
        bkgView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
        [navBarView addSubview:bkgView];        
        UIButton *doneButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [doneButton.titleLabel setFont:[UIFont boldSystemFontOfSize:12.0]];
        [doneButton setTitle:NSLocalizedString(@"返回",@"Dismiss button title") forState:UIControlStateNormal];
        [doneButton setFrame:CGRectMake(size.width - 60, 5, 50, 30)];
        [doneButton addTarget:self action:@selector(photoBrowserDidTapDoneButton:) forControlEvents:UIControlEventTouchUpInside];
        [doneButton.layer setMasksToBounds:YES];
        [doneButton.layer setCornerRadius:4.0];
        [doneButton.layer setBorderWidth:1.0];
        CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
        CGColorRef colorref = CGColorCreate(colorSpace,(CGFloat[]){ 1, 1, 1, 1 });
        [doneButton.layer setBorderColor:colorref];
        doneButton.autoresizingMask = UIViewAutoresizingFlexibleLeftMargin;
        [navBarView addSubview:doneButton];        
        self.imageTitleLabel = [[UILabel alloc] init];
        [self.imageTitleLabel setFrame:CGRectMake((size.width - 60)/2,0, 60, 40)];
        //[self.imageTitleLabel setCenter:navBarView.center];
        [self.imageTitleLabel setTextAlignment:NSTextAlignmentCenter];
        [self.imageTitleLabel setFont:[UIFont boldSystemFontOfSize:20.]];
        [self.imageTitleLabel setTextColor:[UIColor whiteColor]];
        [self.imageTitleLabel setBackgroundColor:[UIColor clearColor]];
        [self.imageTitleLabel setText:@""];
        self.imageTitleLabel.autoresizingMask = UIViewAutoresizingFlexibleHeight | UIViewAutoresizingFlexibleRightMargin;
        [self.imageTitleLabel setTag:BROWSER_TITLE_LBL_TAG];
        [navBarView addSubview:self.imageTitleLabel];
    }    
    return navBarView;
}
//pragma mark - PhotBrower Actions
- (void)photoBrowserDidTapDoneButton:(UIButton *)sender
{
    [self dismissViewControllerAnimated:YES completion:nil];
}
```
###至此一个简答的图片墙加详细浏览的功能便做完了，欢迎交流新的想法。












