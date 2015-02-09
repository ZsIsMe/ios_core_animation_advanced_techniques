# 图层蒙板

&nbsp;&nbsp;&nbsp;&nbsp;通过`masksToBounds`属性，我们可以沿边界裁剪图形；通过`cornerRadius`属性，我们还可以设定一个圆角。但是有时候你希望展现的内容不是在一个矩形或圆角矩形。比如，你想展示一个有星形框架的图片，又或者想让一些古卷文字慢慢渐变成背景色，而不是一个突兀的边界。

&nbsp;&nbsp;&nbsp;&nbsp;使用一个32位有alpha通道的png图片通常是创建一个无矩形视图最方便的方法，你可以给它指定一个透明蒙板来实现。但是这个方法不能让你以编码的方式动态地生成蒙板，也不能让子图层或子视图裁剪成同样的形状。

&nbsp;&nbsp;&nbsp;&nbsp;CALayer有一个属性叫做`mask`可以解决这个问题。这个属性本身就是个CALayer类型，有和其他图层一样的绘制和布局属性。它类似于一个子图层，相对于父图层（即拥有该属性的图层）布局，但是它却不是一个普通的子图层。不同于那些绘制在父图层中的子图层，`mask`图层定义了父图层的部分可见区域。

&nbsp;&nbsp;&nbsp;&nbsp;`mask`图层的`Color`属性是无关紧要的，真正重要的是图层的轮廓。`mask`属性就像是一个饼干切割机，`mask`图层实心的部分会被保留下来，其他的则会被抛弃。（如图4.12）

&nbsp;&nbsp;&nbsp;&nbsp;如果`mask`图层比父图层要小，只有在`mask`图层里面的内容才是它关心的，除此以外的一切都会被隐藏起来。

![图4.12](./4.12.png)

图4.12 把图片和蒙板图层作用在一起的效果

&nbsp;&nbsp;&nbsp;&nbsp;我们将代码演示一下这个过程，创建一个简单的项目，通过图层的`mask`属性来作用于图片之上。为了简便一些，我们用Interface Builder来创建一个包含UIImageView的图片图层。这样我们就只要代码实现蒙板图层了。清单4.5是最终的代码，图4.13是运行后的结果。

清单4.5 应用蒙板图层

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *imageView;
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create mask layer
  CALayer *maskLayer = [CALayer layer];
  maskLayer.frame = self.layerView.bounds;
  UIImage *maskImage = [UIImage imageNamed:@"Cone.png"];
  maskLayer.contents = (__bridge id)maskImage.CGImage;

  //apply mask to image layer￼
  self.imageView.layer.mask = maskLayer;
}
@end
```

![图4.13](./4.13.png)

图4.13 使用了`mask`之后的UIImageView

&nbsp;&nbsp;&nbsp;&nbsp;CALayer蒙板图层真正厉害的地方在于蒙板图不局限于静态图。任何有图层构成的都可以作为`mask`属性，这意味着你的蒙板可以通过代码甚至是动画实时生成。
