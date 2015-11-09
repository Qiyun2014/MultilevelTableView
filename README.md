# UITableView使用，建立二级功能栏


```
* iOS中，使用UITableView一定要记得利用好本身自带的复用机制，减轻ViewController的负荷。
* 同时，引用良好的设计模式，也是必要，常用的是mvc及mvvm。
* UITableView继承UIScrollView，如果有些方法需要实现的时候，可以先去找一下自带的，然后才是自定义。
```


`- (instancetype)initWithFrame:(CGRect)frame style:(UITableViewStyle)style NS_DESIGNATED_INITIALIZER; // must specify style at creation. -initWithFrame: calls this with UITableViewStylePlain`


```
* UITableView的初始化，分为UITableViewStylePlain和UITableViewStyleGrouped两种。
* Plain是默认的表视图显示，常用的一类表格；Grouped是类似组的显示，自带脚标题视图。
* 当设置脚标题视图的高度为0是没有用的，一定要设置一个大于0的常量才能改变它的高度。
```

`typedef NS_ENUM(NSInteger, UITableViewStyle) {
    UITableViewStylePlain,          // regular table view
    UITableViewStyleGrouped         // preferences style table view
};`

##现在，要做出这样的一个效果，点击一级菜单的时候，展现子菜单


```
UITableViewStyleGrouped样式更简便
```
![](/Users/apple/Desktop/EC30C232-DE09-4D4D-84AE-795D450B8603.png
)

##下面是TableView的datasource  方法 cellForRowAtIndexPath
> 
```
//配置cell
```
> 
>  	  - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

>     static  NSString    *cellIdentifier = @"cellIdentifier";

>     JKMultilebelTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];

>     if (!cell) {

>         cell = [[JKMultilebelTableViewCell alloc] initWithStyle:UITableViewCellStyleValue1 reuseIdentifier:cellIdentifier];
>     }

>     NSLog(@"section = %ld row = %ld",indexPath.section,indexPath.row);

>     JKMultilevelModel *model = (JKMultilevelModel *)_subTitleDataArray[MIN(indexPath.section-1, _subTitleDataArray.count)][indexPath.row];


>     [cell fillDataWithMultilevelModel:model];
>     
>     return cell;
>     }


``UITableViewCell的配置就是这样，此处使用的时候，我是只考虑主次两部分的情况。相对来说，cell上的元素不多，可以简单的构建一些子视图进行搭配。使用mvc，可以更好的协调这些元素，进行区分。
``


``主菜单的界面设定完全可以在下面这个代理里去实现：``

``- (nullable UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section;   // custom view for header. will be adjusted to default or specified header height
``


``给主菜单加个手势方法进行响应，并判断展开和收起的状态
``

    UITapGestureRecognizer  *tap = [[UITapGestureRecognizer alloc] initWithTarget:self
                                                                               action:@selector(sectionClick:)];
    [headerView addGestureRecognizer:tap];


``如果有需求，在cell上有类似图上小箭头的图标或者微博那样点赞的功能，我们就需要进一步的做处理，防止cell复用的时候重新加载已有的cell，而覆盖用户响应后的界面效果
``


```//设置每组多少行
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{

    NSString *sectionString = [NSString stringWithFormat:@"%ld",section];

    //展开
    if ([_unfoldSubViewArray containsObject:sectionString] && (section != 0)) {

        UIImageView *aIV = (UIImageView *)[tableView viewWithTag:section + 1000];

        [UIView animateWithDuration:0.5 animations:^{

            aIV.transform = CGAffineTransformMakeRotation(2*M_PI);

        } completion:^(BOOL finished) {

            aIV.image = LOAD_IMAGE(@"grzx-jt-x", @"png");
        }];

        return 3;
    }else{

        UIImageView *aIV = (UIImageView *)[tableView viewWithTag:section + 1000];

        [UIView animateWithDuration:0.5 animations:^{

            aIV.transform = CGAffineTransformMakeRotation(2*M_PI);

        } completion:^(BOOL finished) {

            aIV.image = LOAD_IMAGE(@"grzx-jt-z", @"png");
        }];
        return 0;
    }
}
```

```
1. 简单更换图片的话，并不需要加入动画效果进行选择，所以可以更简单的来做。
 
2. 本次Tableview有自定义的headerView，所以section为1的时候有其他的处理机制。
 
3. 最好的处理cell上子视图的方法就是更改数据源，让cell复用的时候直接根据数据源的数据进行显示，才是最好的处理机制，而不是自己根据tag值来进行强制区分。
```

##以下是我的数据源更换的set方法
```
- (void)setTwoLevelSubTitleArray:(NSMutableArray *)twoLevelSubTitleArray{

    _twoLevelSubTitleArray = twoLevelSubTitleArray;

    [_twoLevelSubTitleArray enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {

        NSMutableArray  *modelArray = [NSMutableArray array];

        [obj enumerateObjectsUsingBlock:^(id  _Nonnull obj2, NSUInteger idx2, BOOL * _Nonnull stop2) {

            JKMultilevelModel *model = [[JKMultilevelModel alloc] init];

            model.titleString = _twoLevelTitleArray[idx][idx2];
            model.subTitleString = obj2;

            [modelArray addObject:model];
        }];

        [_subTitleDataArray addObject:modelArray];
    }];

    //[self reloadData];
}
```



```
核心的代码就是这些了，并没有多么复杂。
同时，写blog也是第一次，希望能帮到需要的人，同时也是增进彼此学习的一个过程。

嗯，就是这样！
```

