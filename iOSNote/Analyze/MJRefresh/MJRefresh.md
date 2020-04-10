
## MJRefresh

- 下拉刷新原理
- MJRefreshComponent
- 关联下拉刷新控件到UIScrollView
- 继承结构可以分为三层
 ![](http://images0.cnblogs.com/blog2015/497279/201506/132232456139177.png)

### 下拉刷新原理

> 首先把刷新控件添加到scrollView的头部或者底部, 然后监控到scrollView的滚动进度(底部刷新控件还需要监控scrollView的内容的改变, 每次改变后再次将控件调整到scrollView的底部), 在scrollView的滚动过程中, 根据滚动的偏移量来计算出拖拽的进度, 然后计算出对应头部/底部的状态, 根据不同的状态 来相应的调整不同的UI或者动画

- `contentInset`
	- 下拉的话，contentOffset就会越来越小，如果上滑，contentOffset就会增大
	- 而大部分的下拉刷新控件，都是将自己放在UIScrollView的上方，起始y设置成负数，所以平时不会显示出来，只有下拉的时候才会出现
	- 然后在loading的时候，临时把contentInset增大，相当于把UIScrollView往下挤，于是下拉刷新的控件就会显示出来，然后刷新完成之后，再把contentInset改回原来的值，实现回弹的效果


### MJRefreshComponent

> MJRefreshComponent作为基类，定义了MJRefresh的整体流程，其它子类只是在此流程的基础上，通过覆写基类的方法，实现定制

- 控件四种状态
	- 1, 正常状态, 即未开始和已经结束的状态.
	- 2, 拖拽状态, 这个时候拖拽的进度小于1, 如果继续拖拽直到拖拽进度等于(>)1的时候, 进入下一种状态.
	- 3, 松手即进入刷新的状态, 这个时候松开手才能进入下一个状态, 如果不松开手, 向反方向拖拽, 则拖拽进度会减小, 如果进度<1, 则会进入上一个状态 ...
	- 4, 加载动画状态, 这个时候进入加载状态, 知道收到 结束动画的指定, 才结束刷新动画进入正常状态等待

- 处理事务
	* 声明控件的所有状态。
	* 声明控件的回调函数。
	* 添加监听。
	* 提供刷新，停止刷新接口。
	* 提供子类需要实现的方法。



### 关联下拉刷新控件到UIScrollView

- 这是利用了UIScrollView+MJRefresh里的一个category，为UIScrollView增加了属性header和footer。这里用到了关联对象的技巧（AssociatedObject）

- 把header添加到了UIScrollView的subviews里，并保留了一个引用。但是这个header的frame还没有确定，也没有任何行为

```
- (void)setHeader:(MJRefreshHeader *)header
{
    if (header != self.header) {
        // 删除旧的，添加新的
        [self.header removeFromSuperview];
        [self addSubview:header];

        // 存储新的
        [self willChangeValueForKey:@"header"]; // KVO
        objc_setAssociatedObject(self, &MJRefreshHeaderKey,
                                 header, OBJC_ASSOCIATION_ASSIGN);
        [self didChangeValueForKey:@"header"]; // KVO
    }
}

```
- 由于上面执行了addSubview，接下来就会进入header的生命周期方法willMoveToSuperview，这个方法是在公共的基类MJRefreshComponent里实现的。因为这是基础的行为，所以写在公共的基类里，所有的子类都能共享：

```
- (void)willMoveToSuperview:(UIView *)newSuperview
{
    [super willMoveToSuperview:newSuperview];

    // 如果不是UIScrollView，不做任何事情
    if (newSuperview && ![newSuperview isKindOfClass:[UIScrollView class]]) return;

    // 旧的父控件移除监听
    [self removeObservers];

    if (newSuperview) { // 新的父控件
        // 设置宽度
        self.mj_w = newSuperview.mj_w;
        // 设置位置
        self.mj_x = 0;

        // 记录UIScrollView
        _scrollView = (UIScrollView *)newSuperview;
        // 设置永远支持垂直弹簧效果
        _scrollView.alwaysBounceVertical = YES;
        // 记录UIScrollView最开始的contentInset
        _scrollViewOriginalInset = self.scrollView.contentInset;

        // 添加监听
        [self addObservers];
    }
}

```
- 这里关键是设置了alwaysBounceVertical，这样才能确保UIScrollView可以下拉
置，以及其中每个subview的位置。并且侦听了UIScrollView的contentOffset和contentSize变化

- 下拉会导致contentOffset变化，由于前面已经添加了KVO侦听，所以会执行scrollViewContentOffsetDidChange方法：

```
- (void)scrollViewContentOffsetDidChange:(NSDictionary *)change
{
    [super scrollViewContentOffsetDidChange:change];

    // 在刷新的refreshing状态
    if (self.state == MJRefreshStateRefreshing) {
        // sectionheader停留解决
        return;
    }

    // 跳转到下一个控制器时，contentInset可能会变
    _scrollViewOriginalInset = self.scrollView.contentInset;

    // 当前的contentOffset
    CGFloat offsetY = self.scrollView.mj_offsetY;
    // 头部控件刚好出现的offsetY
    CGFloat happenOffsetY = - self.scrollViewOriginalInset.top;

    // 如果是向上滚动到看不见头部控件，直接返回
    if (offsetY >= happenOffsetY) return;

    // 普通 和 即将刷新 的临界点
    CGFloat normal2pullingOffsetY = happenOffsetY - self.mj_h;
    CGFloat pullingPercent = (happenOffsetY - offsetY) / self.mj_h;
    if (self.scrollView.isDragging) { // 如果正在拖拽
        self.pullingPercent = pullingPercent;
        if (self.state == MJRefreshStateIdle && offsetY < normal2pullingOffsetY) {
            // 转为即将刷新状态
            self.state = MJRefreshStatePulling;
        } else if (self.state == MJRefreshStatePulling && offsetY >= normal2pullingOffsetY) {
            // 转为普通状态
            self.state = MJRefreshStateIdle;
        }
    } else if (self.state == MJRefreshStatePulling) {// 即将刷新 && 手松开
        // 开始刷新
        [self beginRefreshing];
    } else if (pullingPercent < 1) {
        self.pullingPercent = pullingPercent;
    }
}

```

- 这段代码比较长，主要是判断offset变化是否达到了临界值，以及当前的手势，切换header的state状态，然后根据state状态变化，驱动不同的行为：

```
- (void)setState:(MJRefreshState)state
{
    MJRefreshCheckState
   // 	setState方法是第四个扩展点，这里的MJRefreshCheckState是个宏，也调用了父类的setState的方法。下拉的时候临时增大contentInset，令header保留在屏幕上，然后调用callback block；结束之后还原contentInset

    // 根据状态做事情
    if (state == MJRefreshStateIdle) {
        if (oldState != MJRefreshStateRefreshing) return;

        // 保存刷新时间
        [[NSUserDefaults standardUserDefaults] setObject:[NSDate date] forKey:self.lastUpdatedTimeKey];
        [[NSUserDefaults standardUserDefaults] synchronize];

        // 恢复inset和offset
        [UIView animateWithDuration:MJRefreshSlowAnimationDuration animations:^{
            self.scrollView.mj_insetT -= self.mj_h;

            // 自动调整透明度
            if (self.isAutoChangeAlpha) self.alpha = 0.0;
        } completion:^(BOOL finished) {
            self.pullingPercent = 0.0;
        }];
    } else if (state == MJRefreshStateRefreshing) {
        [UIView animateWithDuration:MJRefreshFastAnimationDuration animations:^{
            // 增加滚动区域
            CGFloat top = self.scrollViewOriginalInset.top + self.mj_h;
            self.scrollView.mj_insetT = top;

            // 设置滚动位置
            self.scrollView.mj_offsetY = - top;
        } completion:^(BOOL finished) {
            [self executeRefreshingCallback];
        }];
    }
}

```