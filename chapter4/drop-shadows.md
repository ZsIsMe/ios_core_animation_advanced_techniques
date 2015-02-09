## 阴影

&nbsp;&nbsp;&nbsp;&nbsp;iOS的另一个常见特性呢，就是阴影。阴影往往可以达到图层深度暗示的效果。也能够用来强调正在显示的图层和优先级（比如说一个在其他视图之前的弹出框），不过有时候他们只是单纯的装饰目的。

&nbsp;&nbsp;&nbsp;&nbsp;给`shadowOpacity`属性一个大于默认值（也就是0）的值，阴影就可以显示在任意图层之下。`shadowOpacity`是一个必须在0.0（不可见）和1.0（完全不透明）之间的浮点数。如果设置为1.0，将会显示一个有轻微模糊的黑色阴影稍微在图层之上。若要改动阴影的表现，你可以使用CALayer的另外三个属性：`shadowColor`，`shadowOffset`和`shadowRadius`。

&nbsp;&nbsp;&nbsp;&nbsp;显而易见，`shadowColor`属性控制着阴影的颜色，和`borderColor`和`backgroundColor`一样，它的类型也是`CGColorRef`。阴影默认是黑色，大多数时候你需要的阴影也是黑色的（其他颜色的阴影看起来是不是有一点点奇怪。。）。

&nbsp;&nbsp;&nbsp;&nbsp;`shadowOffset`属性控制着阴影的方向和距离。它是一个`CGSize`的值，宽度控制这阴影横向的位移，高度控制着纵向的位移。`shadowOffset`的默认值是 {0, -3}，意即阴影相对于Y轴有3个点的向上位移。

&nbsp;&nbsp;&nbsp;&nbsp;为什么要默认向上的阴影呢？尽管Core Animation是从图层套装演变而来（可以认为是为iOS创建的私有动画框架），但是呢，它却是在Mac OS上面世的，前面有提到，二者的Y轴是颠倒的。这就导致了默认的3个点位移的阴影是向上的。在Mac上，`shadowOffset`的默认值是阴影向下的，这样你就能理解为什么iOS上的阴影方向是向上的了（如图4.5）.

![图4.5](./4.5.png)

图4.5 在iOS（左）和Mac OS（右）上`shadowOffset`的表现。

&nbsp;&nbsp;&nbsp;&nbsp;苹果更倾向于用户界面的阴影应该是垂直向下的，所以在iOS把阴影宽度设为0，然后高度设为一个正值不失为一个做法。

&nbsp;&nbsp;&nbsp;&nbsp;`shadowRadius`属性控制着阴影的*模糊度*，当它的值是0的时候，阴影就和视图一样有一个非常确定的边界线。当值越来越大的时候，边界线看上去就会越来越模糊和自然。苹果自家的应用设计更偏向于自然的阴影，所以一个非零值再合适不过了。

&nbsp;&nbsp;&nbsp;&nbsp;通常来讲，如果你想让视图或控件非常醒目独立于背景之外（比如弹出框遮罩层），你就应该给`shadowRadius`设置一个稍大的值。阴影越模糊，图层的深度看上去就会更明显（如图4.6）.

![图4.6](./4.6.png)

### 阴影裁剪


&nbps;&nbsp;&nbsp;&nbsp;和图层边框不同，图层的阴影继承自内容的外形，而不是根据边界和角半径来确定。为了计算出阴影的形状，Core Animation会将寄宿图（包括子视图，如果有的话）考虑在内，然后通过这些来完美搭配图层形状从而创建一个阴影（见图4.7）。

![图4.7](./4.7.png)

图4.7 阴影是根据寄宿图的轮廓来确定的

&nbps;&nbsp;&nbsp;&nbsp;当阴影和裁剪扯上关系的时候就有一个头疼的限制：阴影通常就是在Layer的边界之外，如果你开启了`masksToBounds`属性，所有从图层中突出来的内容都会被才剪掉。如果我们在我们之前的边框示例项目中增加图层的阴影属性时，你就会发现问题所在（见图4.8）.

![图4.8](./4.8.png)

图4.8 `maskToBounds`属性裁剪掉了阴影和内容

&nbps;&nbsp;&nbsp;&nbsp;从技术角度来说，这个结果是可以是可以理解的，但确实又不是我们想要的效果。如果你想沿着内容裁切，你需要用到两个图层：一个只画阴影的空的外图层，和一个用`masksToBounds`裁剪内容的内图层。

&nbps;&nbsp;&nbsp;&nbsp;如果我们把之前项目的右边用单独的视图把裁剪的视图包起来，我们就可以解决这个问题（如图4.9）.

![图4.9](./4.9.png)

图4.9 右边，用额外的阴影转换视图包裹被裁剪的视图

&nbps;&nbsp;&nbsp;&nbsp;我们只把阴影用在最外层的视图上，内层视图进行裁剪。清单4.3是代码实现，图4.10是运行结果。

清单4.3 用一个额外的视图来解决阴影裁切的问题

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;
@property (nonatomic, weak) IBOutlet UIView *shadowView;

@end

@implementation ViewController
￼
- (void)viewDidLoad
{
  [super viewDidLoad];

  //set the corner radius on our layers
  self.layerView1.layer.cornerRadius = 20.0f;
  self.layerView2.layer.cornerRadius = 20.0f;

  //add a border to our layers
  self.layerView1.layer.borderWidth = 5.0f;
  self.layerView2.layer.borderWidth = 5.0f;

  //add a shadow to layerView1
  self.layerView1.layer.shadowOpacity = 0.5f;
  self.layerView1.layer.shadowOffset = CGSizeMake(0.0f, 5.0f);
  self.layerView1.layer.shadowRadius = 5.0f;

  //add same shadow to shadowView (not layerView2)
  self.shadowView.layer.shadowOpacity = 0.5f;
  self.shadowView.layer.shadowOffset = CGSizeMake(0.0f, 5.0f);
  self.shadowView.layer.shadowRadius = 5.0f;

  //enable clipping on the second layer
  self.layerView2.layer.masksToBounds = YES;
}

@end
```

![图4.10](./4.10.png)

图4.10 右边视图，不受裁切阴影的阴影视图。
### shadowPath属性


&nbsp;&nbsp;&nbsp;&nbsp;我们已经知道图层阴影并不总是方的，而是从图层内容的形状继承而来。这看上去不错，但是实时计算阴影也是一个非常消耗资源的，尤其是图层有多个子图层，每个图层还有一个有透明效果的寄宿图的时候。

&nbsp;&nbsp;&nbsp;&nbsp;如果你事先知道你的阴影形状会是什么样子的，你可以通过指定一个`shadowPath`来提高性能。`shadowPath`是一个`CGPathRef`类型（一个指向`CGPath`的指针）。`CGPath`是一个Core Graphics对象，用来指定任意的一个矢量图形。我们可以通过这个属性单独于图层形状之外指定阴影的形状。

图4.11 展示了同一寄宿图的不同阴影设定。如你所见，我们使用的图形很简单，但是它的阴影可以是你想要的任何形状。清单4.4是代码实现。

![图4.11](./4.11.png)

图4.11 用`shadowPath`指定任意阴影形状

清单4.4 创建简单的阴影形状

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //enable layer shadows
  self.layerView1.layer.shadowOpacity = 0.5f;
  self.layerView2.layer.shadowOpacity = 0.5f;

  //create a square shadow
  CGMutablePathRef squarePath = CGPathCreateMutable();
  CGPathAddRect(squarePath, NULL, self.layerView1.bounds);
  self.layerView1.layer.shadowPath = squarePath; CGPathRelease(squarePath);

  ￼//create a circular shadow
  CGMutablePathRef circlePath = CGPathCreateMutable();
  CGPathAddEllipseInRect(circlePath, NULL, self.layerView2.bounds);
  self.layerView2.layer.shadowPath = circlePath; CGPathRelease(circlePath);
}
@end
```

&nbsp;&nbsp;&nbsp;&nbsp;如果是一个矩形或者是圆，用`CGPath`会相当简单明了。但是如果是更加复杂一点的图形，`UIBezierPath`类会更合适，它是一个由UIKit提供的在CGPath基础上的Objective-C包装类。


图4.6 大一些的阴影位移和角半径会增加图层的深度即视感