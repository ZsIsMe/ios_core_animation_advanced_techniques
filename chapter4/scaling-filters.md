# 拉伸过滤


&nbsp;&nbsp;&nbsp;&nbsp;最后我们再来谈谈`minificationFilter`和`magnificationFilter`属性。总得来讲，当我们视图显示一个图片的时候，都应该正确地显示这个图片（意即：以正确的比例和正确的1：1像素显示在屏幕上）。原因如下：

* 能够显示最好的画质，像素既没有被压缩也没有被拉伸。
* 能更好的使用内存，因为这就是所有你要存储的东西。
* 最好的性能表现，CPU不需要为此额外的计算。

&nbsp;&nbsp;&nbsp;&nbsp;不过有时候，显示一个非真实大小的图片确实是我们需要的效果。比如说一个头像或是图片的缩略图，再比如说一个可以被拖拽和伸缩的大图。这些情况下，为同一图片的不同大小存储不同的图片显得又不切实际。

&nbsp;&nbsp;&nbsp;&nbsp;当图片需要显示不同的大小的时候，有一种叫做*拉伸过滤*的算法就起到作用了。它作用于原图的像素上并根据需要生成新的像素显示在屏幕上。

&nbsp;&nbsp;&nbsp;&nbsp;事实上，重绘图片大小也没有一个统一的通用算法。这取决于需要拉伸的内容，放大或是缩小的需求等这些因素。`CALayer`为此提供了三种拉伸过滤方法，他们是：

* kCAFilterLinear
* kCAFilterNearest
* kCAFilterTrilinear

&nbsp;&nbsp;&nbsp;&nbsp;minification（缩小图片）和magnification（放大图片）默认的过滤器都是`kCAFilterLinear`，这个过滤器采用双线性滤波算法，它在大多数情况下都表现良好。双线性滤波算法通过对多个像素取样最终生成新的值，得到一个平滑的表现不错的拉伸。但是当放大倍数比较大的时候图片就模糊不清了。

&nbsp;&nbsp;&nbsp;&nbsp;`kCAFilterTrilinear`和`kCAFilterLinear`非常相似，大部分情况下二者都看不出来有什么差别。但是，较双线性滤波算法而言，三线性滤波算法存储了多个大小情况下的图片（也叫多重贴图），并三维取样，同时结合大图和小图的存储进而得到最后的结果。

&nbsp;&nbsp;&nbsp;&nbsp;这个方法的好处在于算法能够从一系列已经接近于最终大小的图片中得到想要的结果，也就是说不要对很多像素同步取样。这不仅提高了性能，也避免了小概率因舍入错误引起的取样失灵的问题

![图4.14](./4.14.png)

图4.14 对于大图来说，双线性滤波和三线性滤波表现得更出色

&nbsp;&nbsp;&nbsp;&nbsp;`kCAFilterNearest`是一种比较武断的方法。从名字不难看出，这个算法（也叫最近过滤）就是取样最近的单像素点而不管其他的颜色。这样做非常快，也不会使图片模糊。但是，最明显的效果就是，会使得压缩图片更糟，图片放大之后也显得块状或是马赛克严重。

![图4.15](./4.15.png)

图4.15 对于没有斜线的小图来说，最近过滤算法要好很多

&nbsp;&nbsp;&nbsp;&nbsp;总的来说，对于比较小的图或者是差异特别明显，极少斜线的大图，最近过滤算法会保留这种差异明显的特质以呈现更好的结果。但是对于大多数的图尤其是有很多斜线或是曲线轮廓的图片来说，最近过滤算法会导致更差的结果。换句话说，线性过滤保留了形状，最近过滤则保留了像素的差异。

&nbsp;&nbsp;&nbsp;&nbsp;让我们来实验一下。我们对第三章的时钟项目改动一下，用LCD风格的数字方式显示。我们用简单的像素字体（一种用像素构成字符的字体，而非矢量图形）创造数字显示方式，用图片存储起来，而且用第二章介绍过的拼合技术来显示（如图4.16）。

![图4.16](./4.16.png)

图4.16 一个简单的运用拼合技术显示的LCD数字风格的像素字体

&nbsp;&nbsp;&nbsp;&nbsp;我们在Interface Builder中放置了六个视图，小时、分钟、秒钟各两个，图4.17显示了这六个视图是如何在Interface Builder中放置的。如果每个都用一个淡出的outlets对象就会显得太多了，所以我们就用了一个`IBOutletCollection`对象把他们和控制器联系起来，这样我们就可以以数组的方式访问视图了。清单4.6是代码实现。

清单4.6 显示一个LCD风格的时钟

```objective-c
@interface ViewController ()

@property (nonatomic, strong) IBOutletCollection(UIView) NSArray *digitViews;
@property (nonatomic, weak) NSTimer *timer;
￼￼
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad]; //get spritesheet image
  UIImage *digits = [UIImage imageNamed:@"Digits.png"];

  //set up digit views
  for (UIView *view in self.digitViews) {
    //set contents
    view.layer.contents = (__bridge id)digits.CGImage;
    view.layer.contentsRect = CGRectMake(0, 0, 0.1, 1.0);
    view.layer.contentsGravity = kCAGravityResizeAspect;
  }

  //start timer
  self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];

  //set initial clock time
  [self tick];
}

- (void)setDigit:(NSInteger)digit forView:(UIView *)view
{
  //adjust contentsRect to select correct digit
  view.layer.contentsRect = CGRectMake(digit * 0.1, 0, 0.1, 1.0);
}

- (void)tick
{
  //convert time to hours, minutes and seconds
  NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier: NSGregorianCalendar];
  NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
  ￼
  NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];

  //set hours
  [self setDigit:components.hour / 10 forView:self.digitViews[0]];
  [self setDigit:components.hour % 10 forView:self.digitViews[1]];

  //set minutes
  [self setDigit:components.minute / 10 forView:self.digitViews[2]];
  [self setDigit:components.minute % 10 forView:self.digitViews[3]];

  //set seconds
  [self setDigit:components.second / 10 forView:self.digitViews[4]];
  [self setDigit:components.second % 10 forView:self.digitViews[5]];
}
@end
```

如图4.18，这样做的确起了效果，但是图片看起来模糊了。看起来默认的`kCAFilterLinear`选项让我们失望了。

![图4.18](./4.18.png)

图4.18 一个模糊的时钟，由默认的`kCAFilterLinear`引起

&nbsp;&nbsp;&nbsp;&nbsp;为了能像图4.19中那样，我们需要在for循环中加入如下代码：

```objective-c
view.layer.magnificationFilter = kCAFilterNearest;
```

![图4.19](./4.19.png)

图4.19 设置了最近过滤之后的清晰显示
