# iOS核心动画高级技巧 -知识点记录

#### [**只是阅读摘要记录,便于后期快速查阅**](https://github.com/DevDragonLi/Dev-Repo/tree/master/iOSNote),部分加入索引,方便查找跳转定位重温重点.

- [**性能调优**](#performance)
- [**高效绘制**](#Efficient_rendering)
- [**图像IO**](#ImageIO)
- [**图层性能**](#ThelayerPerformance)


## 1. 图层树
				
#### 为什么iOS要基于UIView和CALayer提供两个平行的层级关系呢？
- **为什么不用一个简单的层级来处理所有事情呢**？
	- 原因在于要做职责分离，这样也能避免很多重复代码。在iOS和Mac OS两个平台上，事件和用户交互有很多地方的不同，基于多点触控的用户界面和基于鼠标键盘有着本质的区别，这就是为什么iOS有UIKit和UIView，但是Mac OS有AppKit和NSView的原因。他们功能上很相似，但是在实现上有着显著的区别


- 绘图，布局和动画，相比之下就是类似Mac笔记本和桌面系列一样应用于iPhone和iPad触屏的概念。把这种功能的逻辑分开并应用到独立的Core Animation框架，苹果就能够在iOS和Mac OS之间共享代码，使得对苹果自己的OS开发团队和第三方开发者去开发两个平台的应用更加便捷

#### UIView没有暴露出来的CALayer的功能
- CALayer的好处:
	- 你能在使用所有CALayer底层特性的同时，也可以使用UIView的高级API（比如自动排版，布局和事件处理
- 阴影，圆角，带颜色的边框
- 3D变换
- 非矩形范围
- 透明遮罩
- 多级非线性动画

## 2.寄宿图

#### contents属性(类型:`id`,但是必须是(CGImage/NSImage)才有内容,事实上是CGImageRef:Core Foundation类型)

```
UIImage *imageContent =[UIImage imageNamed:@"back-LFL"];
// Xcode 会自动红点,点击后加入(__bridge id _Nullable)
Layer.contents = (__bridge id _Nullable)(imageContent.CGImage);

```
#### contentGravity:(CALayer与contentMode对应的属性叫做contentsGravity,type: NSString)

```
决定内容在图层的边界中怎么对齐的常量值

kCAGravityCenter
kCAGravityTop
kCAGravityBottom
kCAGravityLeft
kCAGravityRight
kCAGravityTopLeft
kCAGravityTopRight
kCAGravityBottomLeft
kCAGravityBottomRight
kCAGravityResize
kCAGravityResizeAspect
kCAGravityResizeAspectFill

eg: CA_EXTERN NSString * const kCAGravityResizeAspectFill

```
#### contentsScale:定义了寄宿图的像素尺寸和视图大小的比例,默认1.0

```
//set the contentsScale to match image
layerView.layer.contentsScale = image.scale;
// 或者设置设备:
layer.contentsScale = [UIScreen mainScreen].scale;

```
#### maskToBounds:决定是否显示超出边界的内容

#### contentsRect:允许我们在图层边框里显示寄宿图的一个子域(使用单位坐标)

- **点** — 在iOS和Mac OS中最常见的坐标体系。点就像是虚拟的像素，也被称作逻辑像素。在标准设备上，一个点就是一个像素，但是在Retina设备上，一个点等于2*2个像素。iOS用点作为屏幕的坐标测算体系就是为了在Retina设备和普通设备上能有一致的视觉效果。

- **像素** — 物理像素坐标并不会用来屏幕布局，但是仍然与图片有相对关系。UIImage是一个屏幕分辨率解决方案，所以指定点来度量大小。但是一些底层的图片表示如CGImage就会使用像素，所以你要清楚在Retina设备和普通设备上，他们表现出来了不同的大小。

- **单位** — 对于与图片大小或是图层边界相关的显示，单位坐标是一个方便的度量方式， 当大小改变的时候，也不需要再次调整。单位坐标在OpenGL这种纹理坐标系统中用得很多，Core Animation中也用到了单位坐标。

```
默认的contentsRect是{0, 0, 1, 1}，这意味着整个寄宿图默认都是可见的，如果我们指定一个小一点的矩形，图片就会被裁剪.

典型地，图片拼合后可以打包整合到一张大图上一次性载入。相比多次载入不同的图片，这样做能够带来很多方面的好处：内存使用，载入时间，渲染性能等等+

```
#### contentsCenter:其实是一个CGRect,改变contentsCenter的值并不会影响到寄宿图的显示，除非这个图层的大小改变了，你才看得到效果。
- 可视化编程中对应:stretching  

#### Custom Drawing
- -drawRect: 方法没有默认的实现，因为对UIView来说，寄宿图并不是必须的，它不在意那到底是单调的颜色还是有一个图片的实例。如果UIView检测到-drawRect: 方法被调用了，它就会为视图分配一个寄宿图，这个寄宿图的像素尺寸等于视图大小乘以 contentsScale的值


- 如果你不需要寄宿图，那就不要创建这个方法了，这会造成CPU资源和内存的浪费，这也是为什么苹果建议：如果没有自定义绘制的任务就不要在子类中写一个空的-drawRect:方法

- -drawRect:方法里面的代码利用Core Graphics去绘制一个寄宿图，然后内容就会被缓存起来直到它需要被更新（通常是因为开发者调用了-setNeedsDisplay方法，尽管影响到表现效果的属性值被更改时，一些视图类型会被自动重绘，如bounds属性）。虽然-drawRect:方法是一个UIView方法，事实上都是底层的CALayer安排了重绘工作和保存了因此产生的图片。

- CALayer有一个可选的delegate属性，实现了CALayerDelegate协议(均为可选)，当CALayer需要一个内容特定的信息时，就会从协议中请求.
	- 我们在blueLayer上显式地调用了-display。不同于UIView，当图层显示在屏幕上时，CALayer不会自动重绘它的内容。它把重绘的决定权交给了开发者。
	- 尽管我们没有用masksToBounds属性，绘制的那个圆仍然沿边界被裁剪了。这是因为当你使用CALayerDelegate绘制寄宿图的时候，并没有对超出边界外的内容提供绘制支持。 

```
当需要被重绘时，CALayer会请求它的代理给他一个寄宿图来显示。它通过调用下面这个方法做到的:
-(void)displayLayer:(CALayerCALayer *)layer;

趁着这个机会，如果代理想直接设置contents属性的话，它就可以这么做，不然没有别的方法可以调用了。如果代理不实现-displayLayer:方法，CALayer就会转而尝试调用下面这个方法：
-(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;

```

## 3.图层几何学

#### 布局:UIView有三个比较重要的布局属性：frame，bounds和center，CALayer对应地叫做frame，bounds和position

-  对于视图或者图层来说，frame并不是一个非常清晰的属性，它其实是一个虚拟属性，是根据bounds，position和transform计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值

#### 锚点:anchorPoint位于图层的中点，所以图层的将会以这个点为中心放置

- 图层左上角是{0, 0}，右下角是{1, 1}，因此默认坐标是{0.5, 0.5}。

```
// 旋转
self.imageView.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 

```
#### ###坐标系

- UIView严格的二维坐标系不同，CALayer存在于一个三维空间当中(zPosition和anchorPointZ)
- egCode:`    self.greenView.layer.zPosition = 1.0f;`

#### Hit Testing
- containsPoint : `if ([self.layerView.layer containsPoint:point])`
- hitTest	:接受一个CGPoint类型参数,返回本身

```
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get touch position
    CGPoint point = [[touches anyObject] locationInView:self.view];
    //get touched layer
    CALayer *layer = [self.layerView.layer hitTest:point];
    //get layer using hitTest
    if (layer == self.blueLayer) {
  }  
}

```
#### 自动布局
- (void)layoutSublayersOfLayer:(CALayer *)layer;
-  当图层的bounds发生改变，或者图层的-setNeedsLayout方法被调用的时候，上面这个函数将会被执行,这使得你可以手动地重新摆放或者重新调整子图层的大小，但是不能像UIView的autoresizingMask和constraints属性做到自适应屏幕旋转

##4. 视觉效果

- 圆角:`self.layerView2.layer.cornerRadius = 20.0f;`
- 边框:`self.layerView2.layer.borderWidth = 5.0f;`
- 阴影:可以达到图层深度暗示的效果。也能够用来强调正在显示的图层和优先级（比如说一个在其他视图之前的弹出框）
	- shadowColor属性控制着阴影的颜色，和borderColor和backgroundColor一样，它的类型也是CGColorRef。阴影默认是黑色
	- shadowOffset属性控制着阴影的方向和距离。它是一个CGSize的值，宽度控制这阴影横向的位移，高度控制着纵向的位移。shadowOffset的默认值是 {0, -3}，意即阴影相对于Y轴有3个点的向上位移(注意:Mac和OS相反的,可以具体代码测试)
	- shadowPath:CGPathRef类型

	```
	// 产生一个泛红的虚幻边
	blueLayer.shadowOpacity = 0.9;
    blueLayer.shadowColor = [UIColor redColor].CGColor;
    blueLayer.shadowRadius = 10; // 属性控制着阴影的模糊度
    blueLayer.shadowOffset = CGSizeMake(0, -7);
    
    //:CGPathRef
    CGMutablePathRef squarePath = CGPathCreateMutable();
    CGPathAddRect(squarePath, NULL,blueLayer.bounds);
    blueLayer.shadowPath = squarePath;
    CGPathRelease(squarePath);
    
	```
	 	
#### 图层蒙板

- 蒙板可以通过代码甚至是动画实时生成

```
 //create mask layer
  CALayer *maskLayer = [CALayer layer];
  maskLayer.frame = self.layerView.bounds;
  UIImage *maskImage = [UIImage imageNamed:@"back-LFL.png"];
  maskLayer.contents = (__bridge id)maskImage.CGImage;

  //apply mask to image layer￼
  self.imageView.layer.mask = maskLayer;
```
- 拉伸过滤
	- kCAFilterLinear :默认的过滤器(缩小和放大)
	- kCAFilterNearest
	- kCAFilterTrilinear 

- 组透明 `opacity` (UIView等同的属性Alpha)
	- 给一个图层设置了opacity属性，那它的子图层都会受此影响 (透明度的混合叠加)
	- **设置CALayer的一个叫做shouldRasterize属性（见清单4.7）来实现组透明的效果，如果它被设置为YES**

	```
  button.layer.shouldRasterize = YES;
  button.layer.rasterizationScale = [UIScreen mainScreen].scale;

	```

## 5.变换

## 6.专用图层

## 7.隐式动画:并没有指定任何动画的类型

#### 事物和完成块
- Core Animation基于一个假设，说屏幕上的任何东西都可以（或者可能）做动画。动画并不需要你在Core Animation中手动打开，相反需要明确地关闭，否则他会一直存在
- 改变CALayer的一个可做动画的属性，它并不能立刻在屏幕上体现出来。相反，它是从先前的值平滑过渡到新的值。这一切都是默认的行为
- 实际上动画执行的时间取决于当前事务的设置，动画类型取决于图层行为

- CATransaction:+begin和+commit分别来入栈或者出栈

	- **Core Animation在每个run loop周期中自动开始一次新的事务**（run loop是iOS负责收集用户输入，处理定时器或者网络事件并且重新绘制屏幕的东西），**即使你不显式的用[CATransaction begin]开始一次事务，任何在一次run loop循环中属性的改变都会被集中起来**，然后做一次0.25秒的动画

	```
	// : Begin a new transaction for the current thread
	
	// : CATransaction的+begin和+commit方法在+animateWithDuration:animations:内部自动调用，这样block中所有属性的改变都会被事务所包含
    [CATransaction begin];
    
    [CATransaction setAnimationDuration:0.5];
    
    self.customlayer.backgroundColor =[UIColor redColor].CGColor;
    
    [CATransaction commit];
    
    [CATransaction setCompletionBlock:^{
    	// completionBlock    旋转90的动画
    	//rotate the layer 90 degrees
        CGAffineTransform transform = self.customlayer.affineTransform;
        transform = CGAffineTransformRotate(transform, M_PI_2);
	self.customlayer.affineTransform = transform;
    }];
    
	```
	
	- 在iOS4中，苹果对UIView添加了一种基于block的动画方法：+animateWithDuration:animations:实质上它们都是在做同样的事情

#### 图层行为
##### Core Animation通常对CALayer的所有属性（可动画的属性）做动画，但是UIView把它关联的图层的这个特性关闭了
- CALayer自动应用的动画称作行为，当CALayer的属性被修改时候，它会调用`-actionForKey:方法`

	- 图层首先检测它是否有委托，并且是否实现CALayerDelegate协议指定的-actionForLayer:forKey方法。如果有，直接调用并返回结果。
	- 如果没有委托，或者委托没有实现-actionForLayer:forKey方法，图层接着检查包含属性名称对应行为映射的actions字典。
	- 如果actions字典没有包含对应的属性，那么图层接着在它的style字典接着搜索属性名。
最后，如果在style里面也找不到对应的行为，那么图层将会直接调用定义了每个属性的标准行为的-`defaultActionForKey:`方法。 
	
- 完整的搜索结束之后，-actionForKey:要么返回空（这种情况下将不会有动画发生），要么是CAAction协议对应的对象，最后CALayer拿这个结果去对先前和当前的值做动画


- **UIKit是如何禁用隐式动画的?**
	- 每个UIView对它关联的图层都扮演了一个委托，并且提供了-actionForLayer:forKey的实现方法。当不在一个动画块的实现中，UIView对所有图层行为返回nil，但是在动画block范围之内，它就返回了一个非空值
  - 当属性在动画块之外发生改变，UIView直接通过返回nil来禁用隐式动画。但如果在动画块范围之内，根据动画具体类型返回相应的属性，在这个例子就是CABasicAnimation
 -  CATransacition有个方法叫做+setDisableActions:，可以用来对所有属性打开或者关闭隐式动画		

 - 推进过渡:
 
 ```
  //add a custom action
    CATransition *transition = [CATransition animation];
    transition.type = kCATransitionPush;
    transition.subtype = kCATransitionFromLeft;
    self.colorLayer.actions = @{@"backgroundColor": transition};
 ```

- 总结:
	- UIView关联的图层禁用了隐式动画，对这种图层做动画的唯一办法就是使用UIView的动画函数（而不是依赖CATransaction），或者继承UIView，并覆盖-actionForLayer:forKey:方法，或者直接创建一个显式动画
	- 对于单独存在的图层，我们可以通过实现图层的-actionForLayer:forKey:委托方法，或者提供一个actions字典来控制隐式动画

	
	```
		LFLog(@"Inside: %@", [self.layerView actionForLayer:self.layerView.layer forKey:@"backgroundColor"]);

	```
#### 呈现与模型:CALayer的属性行为其实很不正常，因为改变一个图层的属性并没有立刻生效，而是通过一段时间渐变更新

- 在iOS中，屏幕每秒钟重绘60次。如果动画时长比60分之一秒要长，Core Animation就需要在设置一次新值和新值生效之间，对屏幕上的图层进行重新组织


## 8.显式动画
- 当更新属性的时候，我们需要设置一个新的事务，并且禁用图层行为。否则动画会发生两次，一个是因为显式的CABasicAnimation，另一次是因为隐式动画
- 对CAAnimation而言，使用委托模式而不是一个完成块会带来一个问题，就是当你有多个动画的时候，无法在在回调方法中区分

```
- (void)changeColor
{
    //create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    animation.delegate = self;
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
}

- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag
{
    //set the backgroundColor property to match animation toValue
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    self.colorLayer.backgroundColor = (__bridge CGColorRef)anim.toValue;
    [CATransaction commit];
}

```
#### 关键帧动画: CAKeyframeAnimation

- 动画会在开始的时候突然跳转到第一帧的值，然后在动画结束的时候突然恢复到原始的值。所以为了动画的平滑特性，我们需要开始和结束的关键帧来匹配当前属性的值

```
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.duration = 2.0;
    animation.values = @[
                         (__bridge id)[UIColor blueColor].CGColor,
                         (__bridge id)[UIColor redColor].CGColor,
                         (__bridge id)[UIColor greenColor].CGColor,
                         (__bridge id)[UIColor blueColor].CGColor ];
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
```


## <a name="performance"></a>12.性能调优

#### CPU VS GPU

- 关于绘图和动画有两种处理的方式：CPU（中央处理器）和GPU（图形处理器）。但是由于历史原因，我们可以说CPU所做的工作都在软件层面，而GPU在硬件层面

- 对于图像处理，通常用硬件会更快，因为GPU使用图像对高度并行浮点运算做了优化,我们想尽可能把屏幕渲染的工作交给硬件去处理。问题在于GPU并没有无限制处理性能 

#### 动画的舞台

- 动画和屏幕上组合的图层实际上被一个单独的进程管理(iOS6之后的版本中叫做BackBoard)，而不是你的应用程序。这个进程就是所谓的渲染服务


- **当运行一段动画时候，这个过程会被四个分离的阶段被打破-应用程序之内**
	- **布局** - 这是准备你的视图/图层的层级关系，以及设置图层属性（位置，背景色，边框等等）的阶段
	- **显示** - 这是图层的寄宿图片被绘制的阶段。绘制有可能涉及你的-drawRect:和-drawLayer:inContext:方法的调用路径
	- **准备** - 这是Core Animation准备发送动画数据到渲染服务的阶段。这同时也是Core Animation将要执行一些别的事务例如解码动画过程中将要显示的图片的时间点
	- **提交** - 这是最后的阶段，Core Animation打包所有图层和动画属性，然后通过IPC（内部处理通信）发送到渲染服务进行显示

- 一旦打包的图层和动画到达渲染服务进程，他们会被反序列化来形成另一个叫做渲染树的图层树,使用这个树状结构，渲染服务对动画的每一帧做出如下工作：
	- 对所有的图层属性计算中间值，设置OpenGL几何形状（纹理化的三角形）来执行渲染
	- 在屏幕上渲染可见的三角形
- **总结**:共有六个阶段,最后两个阶段在动画过程中不停地重复。前五个阶段都在软件层面处理（通过CPU），只有最后一个被GPU执行。而且，你真正只能控制前两个阶段：布局和显示。Core Animation框架在内部处理剩下的事务，你也控制不了它.

#### GPU相关的操作
- 它用来采集图片和形状（三角形），运行变换，应用纹理和混合然后把它们输送到屏幕上
- 宽泛的说，大多数CALayer的属性都是用GPU来绘制
- **会降低（基于GPU）图层绘制**
	- **太多的几何结构** - 这发生在需要太多的三角板来做变换，以应对处理器的栅格化的时候。现代iOS设备的图形芯片可以处理几百万个三角板，所以在Core Animation中几何结构并不是GPU的瓶颈所在。但由于图层在显示之前通过IPC发送到渲染服务器的时候（图层实际上是由很多小物体组成的特别重量级的对象），太多的图层就会引起CPU的瓶颈。这就限制了一次展示的图层个数
	- **重绘** - 主要由重叠的半透明图层引起。GPU的填充比率（用颜色填充像素的比率）是有限的，所以需要避免重绘（每一帧用相同的像素填充多次）的发生。在现代iOS设备上，GPU都会应对重绘；即使是iPhone 3GS都可以处理高达2.5的重绘比率，并任然保持60帧率的渲染（这意味着你可以绘制一个半的整屏的冗余信息，而不影响性能），并且新设备可以处理更多。
	- **离屏绘制** - 这发生在当不能直接在屏幕上绘制，并且必须绘制到离屏图片的上下文中的时候。离屏绘制发生在基于CPU或者是GPU的渲染，或者是为离屏图片分配额外内存，以及切换绘制上下文，这些都会降低GPU性能。对于特定图层效果的使用，比如圆角，图层遮罩，阴影或者是图层光栅化都会强制Core Animation提前渲染图层的离屏绘制。但这不意味着你需要避免使用这些效果，只是要明白这会带来性能的负面影响。
	- **过大的图片** - 如果视图绘制超出GPU支持的2048x2048或者4096x4096尺寸的纹理，就必须要用CPU在图层每次显示之前对图片预处理，同样也会降低性能

#### CPU相关的操作	
- 延迟动画的开始时间
	- **布局计算** - 如果你的视图层级过于复杂，当视图呈现或者修改的时候，计算图层帧率就会消耗一部分时间。特别是使用iOS6的自动布局机制尤为明显，它应该是比老版的自动调整逻辑加强了CPU的工作。
	- **视图懒加载** - iOS只会当视图控制器的视图显示到屏幕上时才会加载它。这对内存使用和程序启动时间很有好处，但是当呈现到屏幕上之前，按下按钮导致的许多工作都会不能被及时响应。比如控制器从数据库中获取数据，或者视图从一个nib文件中加载，或者涉及IO的图片显示，都会比CPU正常操作慢得多。
	- **Core Graphics绘制** - 如果对视图实现了-drawRect:方法，或者CALayerDelegate的-drawLayer:inContext:方法，那么在绘制任何东西之前都会产生一个巨大的性能开销。为了支持对图层内容的任意绘制，Core Animation必须创建一个内存中等大小的寄宿图片。然后一旦绘制结束之后，必须把图片数据通过IPC传到渲染服务器。在此基础上，Core Graphics绘制就会变得十分缓慢，所以在一个对性能十分挑剔的场景下这样做十分不好。
	- **解压图片**- PNG或者JPEG压缩之后的图片文件会比同质量的位图小得多。但是在图片绘制到屏幕上之前，必须把它扩展成完整的未解压的尺寸（通常等同于图片宽 x 长 x 4个字节）。为了节省内存，iOS通常直到真正绘制的时候才去解码图片。根据你加载图片的方式，第一次对图层内容赋值的时候（直接或者间接使用UIImageView）或者把它绘制到Core Graphics中，都需要对它解压，这样的话，对于一个较大的图片，都会占用一定的时间。 

- 当图层被成功打包，发送到渲染服务器之后，CPU仍然要做如下工作：为了显示屏幕上的图层，Core Animation必须对渲染树种的每个可见图层通过OpenGL循环转换成纹理三角板。由于GPU并不知晓Core Animation图层的任何结构，所以必须要由CPU做这些事情。这里CPU涉及的工作和图层个数成正比，所以如果在你的层级关系中有太多的图层，就会导致CPU没一帧的渲染，即使这些事情不是你的应用程序可控的

#### IO相关操作
- 上下文中的IO（输入/输出）指的是例如闪存或者网络接口的硬件访问。一些动画可能需要从山村（甚至是远程URL）来加载。一个典型的例子就是两个视图控制器之间的过渡效果，这就需要从一个nib文件或者是它的内容中懒加载，或者一个旋转的图片，可能在内存中尺寸太大，需要动态滚动来加载
-   IO比内存访问更慢，所以如果动画涉及到IO,就是一个大问题。总的来说，这就需要使用聪敏但尴尬的技术，也就是多线程，缓存和投机加载

#### 测量而非猜测
-  模拟器运行在你的Mac上，然而Mac上的CPU往往比iOS设备要快。相反，Mac上的GPU和iOS设备的完全不一样，模拟器不得已要在软件层面（CPU）模拟设备的GPU，这意味着GPU相关的操作在模拟器上运行的更慢，尤其是使用CAEAGLLayer来写一些OpenGL的代码时候。

- 为了做到动画的平滑，你需要以60FPS（帧每秒）的速度运行，以同步屏幕刷新速率。通过基于NSTimer或者CADisplayLink的动画你可以降低到30FPS，而且效果还不错，但是没办法通过Core Animation做到这点。如果不保持60FPS的速率，就可能随机丢帧

### Instruments:Profile选项来打开Instruments（在这之前，记住要把目标设置成iOS设备，而不是模拟器）
- 时间分析器 - 用来测量被方法/函数打断的CPU使用情况。
	- 通过**线程分离** - 这可以通过执行的线程进行分组。如果代码被多线程分离的话，那么就可以判断到底是哪个线程造成了问题。
	- **隐藏系统库** - 可以隐藏所有苹果的框架代码，来帮助我们寻找哪一段代码造成了性能瓶颈。由于我们不能优化框架方法，所以这对定位到我们能实际修复的代码很有用。
	- **只显示Obj-C代码** - 隐藏除了Objective-C之外的所有代码。大多数内部的Core Animation代码都是用C或者C++函数，所以这对我们集中精力到我们代码中显式调用的方法就很有用


- **Core Animation** - 用来调试各种Core Animation性能问题。
	- **Color Blended Layers** - 这个选项基于渲染程度对屏幕中的混合区域进行绿到红的高亮（也就是多个半透明图层的叠加）。由于重绘的原因，混合对GPU性能会有影响，同时也是滑动或者动画帧率下降的罪魁祸首之一。
	- **ColorHitsGreenandMissesRed**- 当使用shouldRasterizep属性的时候，耗时的图层绘制会被缓存，然后当做一个简单的扁平图片呈现。当缓存再生的时候这个选项就用红色对栅格化图层进行了高亮。如果缓存频繁再生的话，就意味着栅格化可能会有负面的性能影响了
	- **Color Copied Images** - 有时候寄宿图片的生成意味着Core Animation被强制生成一些图片，然后发送到渲染服务器，而不是简单的指向原始指针。这个选项把这些图片渲染成蓝色。复制图片对内存和CPU使用来说都是一项非常昂贵的操作，所以应该尽可能的避免。
	- **Color Immediately**- 通常Core Animation Instruments以每毫秒10次的频率更新图层调试颜色。对某些效果来说，这显然太慢了。这个选项就可以用来设置每帧都更新（可能会影响到渲染性能，而且会导致帧率测量不准，所以不要一直都设置它）。
	- **Color Misaligned Images** - 这里会高亮那些被缩放或者拉伸以及没有正确对齐到像素边界的图片（也就是非整型坐标）。这些中的大多数通常都会导致图片的不正常缩放，如果把一张大图当缩略图显示，或者不正确地模糊图像，那么这个选项将会帮你识别出问题所在。
	-**Color Offscreen-Rendered Yellow** - 这里会把那些需要离屏渲染的图层高亮成黄色。这些图层很可能需要用shadowPath或者shouldRasterize来优化。
	- **Color OpenGL Fast Path Blue** - 这个选项会对任何直接使用OpenGL绘制的图层进行高亮。如果仅仅使用UIKit或者Core Animation的API，那么不会有任何效果。如果使用GLKView或者CAEAGLLayer，那如果不显示蓝色块的话就意味着你正在强制CPU渲染额外的纹理，而不是绘制到屏幕。
	- **Flash Updated Regions** - 这个选项会对重绘的内容高亮成黄色（也就是任何在软件层面使用Core Graphics绘制的图层）。这种绘图的速度很慢。如果频繁发生这种情况的话，这意味着有一个隐藏的bug或者说通过增加缓存或者使用替代方案会有提升性能的空间。

- OpenGL ES驱动 - 用来调试GPU性能问题。这个工具在编写Open GL代码的时候很有用，但有时也用来处理Core Animation的工作
	- Renderer Utilization - 如果这个值超过了~50%，就意味着你的动画可能对帧率有所限制，很可能因为离屏渲染或者是重绘导致的过度混合。
	- Tiler Utilization - 如果这个值超过了~50%，就意味着你的动画可能限制于几何结构方面，也就是在屏幕上有太多的图层占用了。 

## <a name="Efficient_rendering"></a>13.高效绘图
- 软件绘图:在Core Animation的上下文中指代软件绘图（意即：不由GPU协助的绘图),**软件绘图的代价昂贵，除非绝对必要，你应该避免重绘你的视图**

#### 矢量图形:CAShapeLayer替代Core Graphics，性能就会得到提高
- 任意多边形（不仅仅是一个矩形）
- 斜线或曲线
- 文本
- 渐变

```
- (void)awakeFromNib
{
    //create a mutable path
    self.path = [[UIBezierPath alloc] init];

    //configure the layer
    CAShapeLayer *shapeLayer = (CAShapeLayer *)self.layer;
    shapeLayer.strokeColor = [UIColor redColor].CGColor;
    shapeLayer.fillColor = [UIColor clearColor].CGColor;
    shapeLayer.lineJoin = kCALineJoinRound;
    shapeLayer.lineCap = kCALineCapRound;
    shapeLayer.lineWidth = 5;
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get the starting point
    CGPoint point = [[touches anyObject] locationInView:self];

    //move the path drawing cursor to the starting point
    [self.path moveToPoint:point];
}

- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get the current point
    CGPoint point = [[touches anyObject] locationInView:self];

    //add a new line segment to our path
    [self.path addLineToPoint:point];

    //update the layer with a copy of the path
    ((CAShapeLayer *)self.layer).path = self.path.CGPath;
}

```
#### 脏矩形:Mac OS和iOS设备将会把屏幕区分为需要重绘的区域和不需要重绘的区域。那些需要重绘的部分被称作『脏区域』
-  当你检测到指定视图或图层的指定部分需要被重绘，你直接调用-setNeedsDisplayInRect:来标记它，然后将影响到的矩形作为参数传入。这样就会在一次视图刷新时调用视图的-drawRect:（或图层代理的-drawLayer:inContext:方法）。
-   传入-drawLayer:inContext:的CGContext参数会自动被裁切以适应对应的矩形。为了确定矩形的尺寸大小，你可以用CGContextGetClipBoundingBox()方法来从上下文获得大小。调用-drawRect()会更简单，因为CGRect会作为参数直接传入
- **我们也可以用CGRectIntersectsRect()来避免重绘任何旧的线刷以不至于覆盖已更新过的区域**。这样做会显著地提高绘制效率

#### 异步绘制:CATiledLayer和drawsAsynchronously属性
- **CATiledLayer**特性：在多个线程中为每个小块同时调用-drawLayer:inContext:方法。这就避免了阻塞用户交互而且能够利用多核心新片来更快地绘制。只有一个小块的CATiledLayer是实现异步更新图片视图的简单方法,缺点如下
	- CATiledLayer的队列和缓存算法没有暴露出来，所以我们只能祈祷它能匹配我们的需求
	- CATiledLayer需要我们每次重绘图片到CGContext中，即使它已经解压缩，而且和我们单元格尺寸一样（因此可以直接用作图层内容，而不需要重绘）。

- **drawsAsynchronously**:对传入-drawLayer:inContext:的CGContext进行改动，允许CGContext延缓绘制命令的执行以至于不阻塞用户交互。它自己的-drawLayer:inContext:方法只会在主线程调用，但是CGContext并不等待每个绘制命令的结束。相反地，它会将命令加入队列，当方法返回时，在后台线程逐个执行真正的绘制


## <a name="ImageIO"></a>14.图像IO

#### 加载和潜伏
-  绘图实际消耗的时间通常并不是影响性能的因素。图片消耗很大一部分内存.且不太可能把需要显示的图片都保留在内存中，所以需要在应用运行的时候周期性地加载和卸载图片
-  图片文件加载的速度被CPU和IO（输入/输出）同时影响。iOS设备中的闪存已经比传统硬盘快很多了，但仍然比RAM慢很多，这就需要很小心地管理加载，来避免延迟
-  只要有可能，试着在程序生命周期不易察觉的时候来加载图片，例如启动，或者在屏幕切换的过程中。按下按钮和按钮响应事件之间最大的延迟大概是200ms，这比动画每一帧切换的16ms小得多。你可以在程序首次启动的时候加载图片，但是如果20秒内无法启动程序的话，iOS检测计时器就会终止你的应用（而且如果启动大于2，3秒的话用户就会抱怨了）。

- 子线程中加载图片。这并不能够降低实际的加载时间（可能情况会更糟，因为系统可能要消耗CPU时间来处理加载的图片数据）
- 延迟解压:图片文件被加载就必须要进行解码,解码需要消耗非常长的时间。解码后的图片将同样使用相当大的内存
	- 加载的CPU时间:对于PNG图片来说，加载会比JPEG更长(Xcode会把PNG图片进行解码优化之后引入工程)
	- ImageIO框架:参考知名图片处理框架

	```
NSURL *imageURL = [NSURL fileURLWithPath:self.imagePaths[index]];
NSDictionary *options = @{(__bridge id)kCGImageSourceShouldCache: @YES}; 
CGImageSourceRef source = CGImageSourceCreateWithURL((__bridge CFURLRef)imageURL, NULL);
CGImageRef imageRef = CGImageSourceCreateImageAtIndex(source, 0,(__bridge CFDictionaryRef)options);
UIImage *image = [UIImage imageWithCGImage:imageRef]; 
CGImageRelease(imageRef);
CFRelease(source);
	```

- 两种方式可以为强制解压提前渲染图片
	- 将图片的一个像素绘制成一个像素大小的CGContext。这样仍然会解压整张图片，但是绘制本身并没有消耗任何时间。这样的好处在于加载的图片并不会在特定的设备上为绘制做优化，所以可以在任何时间点绘制出来。同样iOS也就可以丢弃解压后的图片来节省内存了。
	- 将整张图片绘制到CGContext中，丢弃原始的图片，并且用一个从上下文内容中新的图片来代替。这样比绘制单一像素那样需要更加复杂的计算，但是因此产生的图片将会为绘制做优化，而且由于原始压缩图片被抛弃了，iOS就不能够随时丢弃任何解压后的图片来节省内存了

#### 缓存
- 缓存其实很简单：就是存储昂贵计算后的结果（或者是从闪存或者网络加载的文件）在内存中，以便后续使用，这样访问起来很快。问题在于缓存本质上是一个权衡过程 - 为了提升性能而消耗了内存，但是由于内存是一个非常宝贵的资源，所以不能把所有东西都做缓存。+
- +imageNamed:方法:**它在内存中自动缓存了解压后的图片，即使你自己没有保留对它的任何引用**
- 自定义缓存
	- 选择一个合适的缓存键 - 缓存键用来做图片的唯一标识。如果实时创建图片，通常不太好生成一个字符串来区分别的图片。在我们的图片传送带例子中就很简单，我们可以用图片的文件名或者表格索引。
	- 提前缓存 - 如果生成和加载数据的代价很大，你可能想当第一次需要用到的时候再去加载和缓存。提前加载的逻辑是应用内在就有的，但是在我们的例子中，这也非常好实现，因为对于一个给定的位置和滚动方向，我们就可以精确地判断出哪一张图片将会出现。
	- 缓存失效 - 如果图片文件发生了变化，怎样才能通知到缓存更新呢？这是个非常困难的问题（就像菲尔 卡尔顿提到的），但是幸运的是当从程序资源加载静态图片的时候并不需要考虑这些。对用户提供的图片来说（可能会被修改或者覆盖），一个比较好的方式就是当图片缓存的时候打上一个时间戳以便当文件更新的时候作比较。
	- 缓存回收 - 当内存不够的时候，如何判断哪些缓存需要清空呢？这就需要到你写一个合适的算法了。幸运的是，对缓存回收的问题，苹果提供了一个叫做NSCache通用的解决方案

- NSCache:在系统低内存的时候自动丢弃存储的对象
	- -setCountLimit:方法设置缓存大小，
	- setObject:forKey:cost:来对每个存储的对象指定消耗的值

#### 文件格式
-  **PNG图片使用的无损压缩算法可以比使用JPEG的图片做到更快地解压，但是由于闪存访问的原因，这些加载的时间并没有什么区别**
	- 相对于不友好的PNG图片，相同像素的JPEG图片总是比PNG加载更快
	- 苹果在iOS系统中对PNG和JPEG都做了一些优化
- **混合图片**:对于包含透明的图片来说，最好是使用压缩透明通道的PNG图片和压缩RGB部分的JPEG图片混合起来加载

	```
	//:从PNG遮罩和JPEG创建的混合图片
	- (void)viewDidLoad
{
    [super viewDidLoad];
    //load color image
    UIImage *image = [UIImage imageNamed:@"Snowman.jpg"];
    //load mask image
    UIImage *mask = [UIImage imageNamed:@"SnowmanMask.png"];
    //convert mask to correct format
    CGColorSpaceRef graySpace = CGColorSpaceCreateDeviceGray();
    CGImageRef maskRef = CGImageCreateCopyWithColorSpace(mask.CGImage, graySpace);
    CGColorSpaceRelease(graySpace);
    //combine images
    CGImageRef resultRef = CGImageCreateWithMask(image.CGImage, maskRef);
    UIImage *result = [UIImage imageWithCGImage:resultRef];
    CGImageRelease(resultRef);
    CGImageRelease(maskRef);
    //display result
    self.imageView.image = result;
}

	```
## <a name="ThelayerPerformance"></a>15.图层性能

#### 隐式绘制
- **文本**:CATextLayer和UILabel都是直接将文本绘制在图层的寄宿图中。事实上这两种方式用了完全不同的渲染方式：在iOS 6及之前，UILabel用WebKit的HTML渲染引擎来绘制文本，而CATextLayer用的是Core Text.后者渲染更迅速，所以在所有需要绘制大量文本的情形下都优先使用它吧。但是这两种方法都用了软件的方式绘制，因此他们实际上要比硬件加速合成方式要慢
	- 尽可能地避免改变那些包含文本的视图的frame，因为这样做的话文本就需要重绘
	- 但是这个图层经常改动，你就应该把文本放在一个子图层中

- **光栅化**:
	- 启用shouldRasterize属性会将图层绘制到一个屏幕之外的图像。然后这个图像将会被缓存起来并绘制到实际图层的contents和子图层。如果有很多的子图层或者有复杂的效果应用，这样做就会比重绘所有事务的所有帧划得来得多。但是光栅化原始图像需要时间，而且还会消耗额外的内存
	- **要避免作用在内容不断变动的图层上** 

#### 离屏渲染:当图层属性的混合体被指定为在未预合成之前不能直接在屏幕中绘制时

- 图层的以下属性将会触发屏幕外绘制()
	- 圆角（cornerRadius和maskToBounds独立作用的时候都不会有太大的性能问题，但是当他俩结合在一起，就触发了屏幕外渲染）

	- 图层蒙板

	- 阴影

- CAShapeLayer:画一个圆角矩形

```
CAShapeLayer *blueLayer = [CAShapeLayer layer];
    blueLayer.frame = CGRectMake(50, 50, 100, 100);
    blueLayer.fillColor = [UIColor blueColor].CGColor;
    blueLayer.path = [UIBezierPath bezierPathWithRoundedRect:
    CGRectMake(0, 0, 100, 100) cornerRadius:20].CGPath;
    ￼
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
```

- 使用可伸缩图片的优势在于它可以绘制成任意边框效果而不需要额外的性能消耗。举个例子，可伸缩图片甚至还可以显示出矩形阴影的效果。

```
blueLayer.contentsScale = [UIScreen mainScreen].scale;
blueLayer.contents = (__bridge id)[UIImage imageNamed:@"Circle.png"].CGImage;

```
- shadowPath: 如果图层是一个简单几何图形如矩形或者圆角矩形（假设不包含任何透明部分或者子图层），创建出一个对应形状的阴影路径就比较容易，而且Core Animation绘制这个阴影也相当简单，避免了屏幕外的图层部分的预排版需求。这对性能来说很有帮助。

#### 混合和过度绘制:不到必须时刻不要使用透明图层
- 给视图的backgroundColor属性设置一个固定的，不透明的颜色
- 设置opaque属性为YES

#### 减少图层数量
-  初始化图层，处理图层，打包通过IPC发给渲染引擎，转化成OpenGL几何图形，这些是一个图层的大致资源开销
- 对象回收:相似对象池。当一个对象的指定实例（本例子中指的是图层）结束了使命，你把它添加到对象池中。每次当你需要一个实例时，你就从池中取出一个。当且仅当池中为空时再创建一个新的。
- Core Graphics绘制:你可能仍然需要减少图层的数量。例如，如果你正在使用多个UILabel或者UIImageView实例去显示固定内容，你可以把他们全部替换成一个单独的视图，然后用-drawRect:方法绘制出那些复杂的视图层级

- renderInContext
	-   使用CALayer的-renderInContext:方法，你可以将图层及其子图层快照进一个Core Graphics上下文然后得到一个图片，它可以直接显示在UIImageView中，或者作为另一个图层的contents。不同于shouldRasterize —— 要求图层与图层树相关联 —— ，这个方法没有持续的性能消耗。+

-  当图层内容改变时，刷新这张图片的机会取决于你（不同于shouldRasterize，它自动地处理缓存和缓存验证），但是一旦图片被生成，相比于让Core Animation处理一个复杂的图层树，你节省了相当客观的性能。+












