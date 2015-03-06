# 锚点

&nbsp;&nbsp;&nbsp;&nbsp;之前提到过，视图的`center`属性和图层的`position`属性都指定了`anchorPoint`相对于父图层的位置。图层的`anchorPoint`通过`position`来控制它的`frame`的位置，你可以认为`anchorPoint`是用来移动图层的*把柄*。

&nbsp;&nbsp;&nbsp;&nbsp;默认来说，`anchorPoint`位于图层的中点，所以图层的将会以这个点为中心放置。`anchorPoint`属性并没有被`UIView`接口暴露出来，这也是视图的position属性被叫做“center”的原因。但是图层的`anchorPoint`可以被移动，比如你可以把它置于图层`frame`的左上角，于是图层的内容将会向右下角的`position`方向移动（图3.3），而不是居中了。

<img src="./3.3.jpeg" alt="图3.3" title="图3.3" width="700"/>

图3.3 改变`anchorPoint`的效果

&nbsp;&nbsp;&nbsp;&nbsp;和第二章提到的`contentsRect`和`contentsCenter`属性类似，`anchorPoint`用*单位坐标*来描述，也就是图层的相对坐标，图层左上角是{0, 0}，右下角是{1, 1}，因此默认坐标是{0.5, 0.5}。`anchorPoint`可以通过指定x和y值小于0或者大于1，使它放置在图层范围之外。

&nbsp;&nbsp;&nbsp;&nbsp;注意在图3.3中，当改变了`anchorPoint`，`position`属性保持固定的值并没有发生改变，但是`frame`却移动了。

&nbsp;&nbsp;&nbsp;&nbsp;那在什么场合需要改变`anchorPoint`呢？既然我们可以随意改变图层位置，那改变`anchorPoint`不会造成困惑么？为了举例说明，我们来举一个实用的例子，创建一个模拟闹钟的项目。

&nbsp;&nbsp;&nbsp;&nbsp;钟面和钟表由四张图片组成（图3.4），为了简单说明，我们还是用传统的方式来装载和加载图片，使用四个`UIImageView`实例（当然你也可以用正常的视图，设置他们图层的`contents`图片）。

<img src="./3.4.jpeg" alt="图3.4" title="图3.4" width="700"/>

图3.4 组成钟面和钟表的四张图片

&nbsp;&nbsp;&nbsp;&nbsp;闹钟的组件通过IB来排列（图3.5），这些图片视图嵌套在一个容器视图之内，并且自动调整和自动布局都被禁用了。这是因为自动调整会影响到视图的`frame`，而根据图3.2的演示，当视图旋转的时候，`frame`是会发生改变的，这将会导致一些布局上的失灵。

&nbsp;&nbsp;&nbsp;&nbsp;我们用`NSTimer`来更新闹钟，使用视图的`transform`属性来旋转钟表（如果你对这个属性不太熟悉，不要着急，我们将会在第5章“变换”当中详细说明），具体代码见清单3.1

<img src="./3.5.jpeg" alt="图3.5" title="图3.5" width="700"/>

图3.5 在Interface Builder中布局闹钟视图

清单3.1 **Clock**
```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *hourHand;
@property (nonatomic, weak) IBOutlet UIImageView *minuteHand;
@property (nonatomic, weak) IBOutlet UIImageView *secondHand;
@property (nonatomic, weak) NSTimer *timer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //start timer
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];
                  ￼
    //set initial hand positions
    [self tick];
}

- (void)tick
{
    //convert time to hours, minutes and seconds
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
    NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];
    CGFloat hoursAngle = (components.hour / 12.0) * M_PI * 2.0;
    //calculate hour hand angle //calculate minute hand angle
    CGFloat minsAngle = (components.minute / 60.0) * M_PI * 2.0;
    //calculate second hand angle
    CGFloat secsAngle = (components.second / 60.0) * M_PI * 2.0;
    //rotate hands
    self.hourHand.transform = CGAffineTransformMakeRotation(hoursAngle);
    self.minuteHand.transform = CGAffineTransformMakeRotation(minsAngle);
    self.secondHand.transform = CGAffineTransformMakeRotation(secsAngle);
}

@end
```

&nbsp;&nbsp;&nbsp;&nbsp;运行项目，看起来有点奇怪（图3.6），因为钟表的图片在围绕着中心旋转，这并不是我们期待的一个支点。

<img src="./3.6.jpeg" alt="图3.6" title="图3.6" width="700"/>

图3.6 钟面，和不对齐的钟指针

&nbsp;&nbsp;&nbsp;&nbsp;你也许会认为可以在Interface Builder当中调整指针图片的位置来解决，但其实并不能达到目的，因为如果不放在钟面中间的话，同样不能正确的旋转。

&nbsp;&nbsp;&nbsp;&nbsp;也许在图片末尾添加一个透明空间也是个解决方案，但这样会让图片变大，也会消耗更多的内存，这样并不优雅。

&nbsp;&nbsp;&nbsp;&nbsp;更好的方案是使用`anchorPoint`属性，我们来在`-viewDidLoad`方法中添加几行代码来给每个钟指针的`anchorPoint`做一些平移（清单3.2），图3.7显示了正确的结果。

清单3.2
```objective-c
- (void)viewDidLoad 
{
    [super viewDidLoad];
    // adjust anchor points

    self.secondHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
    self.minuteHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
    self.hourHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);


    // start timer
} 
```

<img src="./3.7.jpeg" alt="图3.7" title="图3.7" width="700"/>

图3.7 钟面，和正确对齐的钟指针
