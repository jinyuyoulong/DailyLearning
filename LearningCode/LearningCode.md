## 代码技巧

### **1. 一种懒加载写法**

```   
if (!self.statusBar.superview) {
    [self.view addSubview:self.statusBar];
}

- (UIView *)statusBar {
    if (_statusBar == nil) {
        _statusBar = [[UIView alloc] initWithFrame:CGRectMake(0, 0, SCREEN_WIDTH, STATUSBAR_HEIGHT)];
        _statusBar.backgroundColor = [UIColor blackColor];
    }
    return _statusBar;
}
```

### **2. UITableView的一种删除、添加cell后的刷新方法:**
```
[self.dataArray removeObjectsAtIndexes:indexs];
[self.tableView beginUpdates];
[self.tableView deleteRowsAtIndexPaths:indexPaths withRowAnimation:UITableViewRowAnimationFade];
//        [_tableView reloadSections:[NSIndexSet indexSetWithIndex:indexPath.section]
   withRowAnimation:(UITableViewRowAnimationNone)];
[self.tableView endUpdates];
```

### **3. 字符串遍历**

```
（1） 通过查找的方式来(这方式适合所有格式的子符串，推荐使用)
   NSString *newStr =@"sdfdfs15dfdfdf15fdfdow们啊as阿达阿达打啊";
   NSString *temp =nil;
   for(int i =0; i < [newStr length]; i++)  
   {   
       temp = [newStr substringWithRange:NSMakeRange(i,1)];
       NSLog(@"第%d个字是:%@",i,temp);
   }  
NSRange hotTopicRange = [dynamicComplex.desc rangeOfString:hotTopicComplex.title];

```

### **4. 在解决tableView头部插入view的时候，SectionHeaderView的contentInset问题**

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    CGFloat contentOffsetY = scrollView.contentOffset.y;
    if (contentOffsetY < -bannerHeaderHeight) {
        topicDetailBannerView.top = contentOffsetY;
        topicDetailBannerView.height = -contentOffsetY;
    }
    //解决sectionHeader的问题
    if(contentOffsetY >= 0 ){
        scrollView.contentInset = UIEdgeInsetsZero;
    }else{
        scrollView.contentInset = UIEdgeInsetsMake(MIN(-contentOffsetY, bannerHeaderHeight), 0, 0, 0);
    }
}
```
![](http://oc98nass3.bkt.clouddn.com/2017-07-27-15011211526057.jpg)


### **5. 在计算UITableView中某个view在UITableView的位置，和tableHeaderView的联系**


发现获取`SectionHeaderView`的`y`坐标时，得减去`tableHeaderView`的高度，原因尚且未知。

```
    _tableView.contentInset = UIEdgeInsetsMake(bannerHeaderHeight, 0, 0, 0);

    CGFloat sectionHeaderBottom = sectionHeaderViewRect.origin.y + sectionHeaderViewRect.size.height - self.tableView.tableHeaderView.height;//减去tableHeaderView高度
```

![](http://oc98nass3.bkt.clouddn.com/2017-07-28-15012486582998.jpg)

### 6. **NSMutableArray addObject: -[__NSArrayI addObject:]: unrecognized selector sent to instance**


原因就是对`NSMutableArray`属性的定义没定好，

```
@property (nonatomic, strong) NSMutableArray *dynamicComplexList;/**< 动态列表 */
@property (nonatomic, copy) NSMutableArray *subscribeList;
```
`subscribeList`定义成`copy`的话是`immutable copy`不可变的数组 = = （还不是很知道）

```              
[self.subscribeList addObjectsFromArray:rsp.subscribeList];
```


参考 [iphone - NSMutableArray addObject: -[__NSArrayI addObject:]: unrecognized selector sent to instance - Stack Overflow](https://stackoverflow.com/questions/3220120/nsmutablearray-addobject-nsarrayi-addobject-unrecognized-selector-sent-t)

```
The synthesized setter for @property (copy) sends a copy message to the array, which results in an immutable copy.

You have no choice but the implement the setter yourself here, as detailed in the Objective-C guide.
```


### 7. 计算`SectionHeaderView`位置


```
        UIView *sectionHeaderView = [self.tableView headerViewForSection:0];
        CGRect sectionHeaderViewRect = [self.view convertRect:sectionHeaderView.frame toView:self.view];
        CGRect sectionHeaderViewRect = [self.view convertRect:sectionHeaderView.frame toView:self.view];
```

方式1：

```
        if (self.tableView.contentOffset.y >= self.sectionHeaderStayTop) {
            sectionHeaderBottom = self.naviBar.height + sectionHeaderView.height;
        }else{
            sectionHeaderBottom = self.tableView.contentInset.top + sectionHeaderViewRect.origin.y + sectionHeaderView.height;
        }

```


方式2：

```
        sectionHeaderBottom = self.tableView.contentInset.top + sectionHeaderView.height + ((self.tableView.contentOffset.y >= self.sectionHeaderStayTop)?0:sectionHeaderViewRect.origin.y);
```

参考代码：

```
/**< sectionHeader的悬停位置 */
- (CGFloat)sectionHeaderStayTop{
    return self.tableView.tableHeaderView.height - self.naviBar.height;
}
```

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    CGFloat naviAlpha = 0;
    CGFloat contentOffsetY = scrollView.contentOffset.y;
    if (contentOffsetY < -bannerHeaderHeight) {
        topicDetailBannerView.top = contentOffsetY;
        topicDetailBannerView.height = -contentOffsetY;
    }
    if(contentOffsetY <= (self.naviBarScrollMargin - bannerHeaderHeight)){
        naviAlpha = (contentOffsetY + bannerHeaderHeight) / (self.naviBarScrollMargin);
    }else{
        naviAlpha = 1;
    }
    self.naviBar.backgroundColor = [UIColor colorWithWhite:0 alpha:naviAlpha];
    self.naviBar.titleLabel.textColor = [UIColor colorWithWhite:1 alpha:naviAlpha];

    //解决sectionHeader的问题
    if(contentOffsetY >= self.sectionHeaderStayTop){
        scrollView.contentInset = UIEdgeInsetsMake(self.naviBar.height, 0, 0, 0);
    }else{
        scrollView.contentInset = UIEdgeInsetsMake(MIN(-contentOffsetY, bannerHeaderHeight), 0, 0, 0);
    }
    publishButton.hidden = (contentOffsetY >= tableContentOffset.y);
    tableContentOffset.y = contentOffsetY;
}
```

title滚动范围
```
- (CGFloat)naviBarScrollMargin{
    return (topicDetailBannerView.topicTitleLabelTop - self.naviBar.titleLabel.top);
}
```

## `Xcode快捷键`

1. 交换上下行代码：  `Cmd + Option + [` or `Cmd + Option + ]`


