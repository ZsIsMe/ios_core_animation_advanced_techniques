##CAGradientLayer

`CAGradientLayer`是用来生成两种或更多颜色平滑渐变的。用Core Graphics复制一个`CAGradientLayer`并将内容绘制到一个普通图层的寄宿图也是有可能的，但是`CAGradientLayer`的真正好处在于绘制使用了硬件加速。

###基础渐变

我们将从一个简单的红变蓝的对角线渐变开始（见清单6.6）.这些渐变色彩放在一个数组中，并赋给`colors`属性。这个数组成员接受`CGColorRef`类型的值（并不是从`NSObject`派生而来），所以我们要用通过bridge转换以确保编译正常。

`CAGradientLayer`也有`startPoint`和`endPoint`属性，他们决定了渐变的方向。这两个参数是以单位坐标系进行的定义，所以左上角坐标是{0, 0}，右下角坐标是{1, 1}。代码运行结果如图6.6

清单6.6 简单的两种颜色的对角线渐变

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];
  //create gradient layer and add it to our container view
  CAGradientLayer *gradientLayer = [CAGradientLayer layer];
  gradientLayer.frame = self.containerView.bounds;
  [self.containerView.layer addSublayer:gradientLayer];

  //set gradient colors
  gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id)[UIColor blueColor].CGColor];

  //set gradient start and end points
  gradientLayer.startPoint = CGPointMake(0, 0);
  gradientLayer.endPoint = CGPointMake(1, 1);
}
@end
```

![图6.6](./6.6.png)

图6.6 用`CAGradientLayer`实现简单的两种颜色的对角线渐变

###多重渐变


如果你愿意，`colors`属性可以包含很多颜色，所以创建一个彩虹一样的多重渐变也是很简单的。默认情况下，这些颜色在空间上均匀地被渲染，但是我们可以用`locations`属性来调整空间。`locations`属性是一个浮点数值的数组（以`NSNumber`包装）。这些浮点数定义了`colors`属性中每个不同颜色的位置，同样的，也是以单位坐标系进行标定。0.0代表着渐变的开始，1.0代表着结束。

`locations`数组并不是强制要求的，但是如果你给它赋值了就一定要确保`locations`的数组大小和`colors`数组大小一定要相同，否则你将会得到一个空白的渐变。

清单6.7展示了一个基于清单6.6的对角线渐变的代码改造。现在变成了从红到黄最后到绿色的渐变。`locations`数组指定了0.0，0.25和0.5三个数值，这样这三个渐变就有点像挤在了左上角。（如图6.7）.

清单6.7 在渐变上使用`locations`

```objective-c
- (void)viewDidLoad {
  [super viewDidLoad];

  //create gradient layer and add it to our container view
  CAGradientLayer *gradientLayer = [CAGradientLayer layer];
  gradientLayer.frame = self.containerView.bounds;
  [self.containerView.layer addSublayer:gradientLayer];

  //set gradient colors
  gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id) [UIColor yellowColor].CGColor, (__bridge id)[UIColor greenColor].CGColor];

  //set locations
  gradientLayer.locations = @[@0.0, @0.25, @0.5];

  //set gradient start and end points
  gradientLayer.startPoint = CGPointMake(0, 0);
  gradientLayer.endPoint = CGPointMake(1, 1);
}
```

![图6.7](./6.7.png)

图6.7 用`locations`构造偏移至左上角的三色渐变
