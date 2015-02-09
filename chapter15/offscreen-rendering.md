## 离屏渲染

&nbsp;&nbsp;&nbsp;&nbsp;当图层属性的混合体被指定为在未预合成之前不能直接在屏幕中绘制时，屏幕外渲染就被唤起了。屏幕外渲染并不意味着软件绘制，但是它意味着图层必须在被显示之前在一个屏幕外上下文中被渲染（不论CPU还是GPU）。图层的以下属性将会触发屏幕外绘制：

* 圆角（当和`maskToBounds`一起使用时）
* 图层蒙板
* 阴影

&nbsp;&nbsp;&nbsp;&nbsp;屏幕外渲染和我们启用光栅化时相似，除了它并没有像光栅化图层那么消耗大，子图层并没有被影响到，而且结果也没有被缓存，所以不会有长期的内存占用。但是，如果太多图层在屏幕外渲染依然会影响到性能。

&nbsp;&nbsp;&nbsp;&nbsp;有时候我们可以把那些需要屏幕外绘制的图层开启光栅化以作为一个优化方式，前提是这些图层并不会被频繁地重绘。

&nbsp;&nbsp;&nbsp;&nbsp;对于那些需要动画而且要在屏幕外渲染的图层来说，你可以用`CAShapeLayer`，`contentsCenter`或者`shadowPath`来获得同样的表现而且较少地影响到性能。

### CAShapeLayer

&nbsp;&nbsp;&nbsp;&nbsp;`cornerRadius`和`maskToBounds`独立作用的时候都不会有太大的性能问题，但是当他俩结合在一起，就触发了屏幕外渲染。有时候你想显示圆角并沿着图层裁切子图层的时候，你可能会发现你并不需要沿着圆角裁切，这个情况下用`CAShapeLayer`就可以避免这个问题了。

&nbsp;&nbsp;&nbsp;&nbsp;你想要的只是圆角且沿着矩形边界裁切，同时还不希望引起性能问题。其实你可以用现成的`UIBezierPath`的构造器`+bezierPathWithRoundedRect:cornerRadius:`（见清单15.1）.这样做并不会比直接用`cornerRadius`更快，但是它避免了性能问题。

清单15.1 用`CAShapeLayer`画一个圆角矩形

```objective-c
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>

@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //create shape layer
    CAShapeLayer *blueLayer = [CAShapeLayer layer];
    blueLayer.frame = CGRectMake(50, 50, 100, 100);
    blueLayer.fillColor = [UIColor blueColor].CGColor;
    blueLayer.path = [UIBezierPath bezierPathWithRoundedRect:
    CGRectMake(0, 0, 100, 100) cornerRadius:20].CGPath;
    ￼
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```

###可伸缩图片

&nbsp;&nbsp;&nbsp;&nbsp;另一个创建圆角矩形的方法就是用一个圆形内容图片并结合第二章『寄宿图』提到的`contensCenter`属性去创建一个可伸缩图片（见清单15.2）.理论上来说，这个应该比用`CAShapeLayer`要快，因为一个可拉伸图片只需要18个三角形（一个图片是由一个3*3网格渲染而成），然而，许多都需要渲染成一个顺滑的曲线。在实际应用上，二者并没有太大的区别。

清单15.2 用可伸缩图片绘制圆角矩形

```objective-c
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //create layer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50, 50, 100, 100);
    blueLayer.contentsCenter = CGRectMake(0.5, 0.5, 0.0, 0.0);
    blueLayer.contentsScale = [UIScreen mainScreen].scale;
    blueLayer.contents = (__bridge id)[UIImage imageNamed:@"Circle.png"].CGImage;
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```

&nbsp;&nbsp;&nbsp;&nbsp;使用可伸缩图片的优势在于它可以绘制成任意边框效果而不需要额外的性能消耗。举个例子，可伸缩图片甚至还可以显示出矩形阴影的效果。

###shadowPath

&nbsp;&nbsp;&nbsp;&nbsp;在第2章我们有提到`shadowPath`属性。如果图层是一个简单几何图形如矩形或者圆角矩形（假设不包含任何透明部分或者子图层），创建出一个对应形状的阴影路径就比较容易，而且Core Animation绘制这个阴影也相当简单，避免了屏幕外的图层部分的预排版需求。这对性能来说很有帮助。

&nbsp;&nbsp;&nbsp;&nbsp;如果你的图层是一个更复杂的图形，生成正确的阴影路径可能就比较难了，这样子的话你可以考虑用绘图软件预先生成一个阴影背景图。

