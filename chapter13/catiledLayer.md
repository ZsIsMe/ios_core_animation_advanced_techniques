
##矢量图形

&nbsp;&nbsp;&nbsp;&nbsp;我们用Core Graphics来绘图的一个通常原因就是只是用图片或是图层效果不能轻易地绘制出矢量图形。矢量绘图包含一下这些：

* 任意多边形（不仅仅是一个矩形）
* 斜线或曲线
* 文本
* 渐变

&nbsp;&nbsp;&nbsp;&nbsp;举个例子，清单13.1 展示了一个基本的画线应用。这个应用将用户的触摸手势转换成一个`UIBezierPath`上的点，然后绘制成视图。我们在一个`UIView`子类`DrawingView`中实现了所有的绘制逻辑，这个情况下我们没有用上view controller。但是如果你喜欢你可以在view controller中实现触摸事件处理。图13.1是代码运行结果。

清单13.1 用Core Graphics实现一个简单的绘图应用

```objective-c
#import "DrawingView.h"

@interface DrawingView ()

@property (nonatomic, strong) UIBezierPath *path;

@end

@implementation DrawingView

- (void)awakeFromNib
{
    //create a mutable path
    self.path = [[UIBezierPath alloc] init];
    self.path.lineJoinStyle = kCGLineJoinRound;
    self.path.lineCapStyle = kCGLineCapRound;
    ￼
    self.path.lineWidth = 5;
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

    //redraw the view
    [self setNeedsDisplay];
}

- (void)drawRect:(CGRect)rect
{
    //draw path
    [[UIColor clearColor] setFill];
    [[UIColor redColor] setStroke];
    [self.path stroke];
}
@end
```

![图13.1](./13.1.png)

图13.1 用Core Graphics做一个简单的『素描』

&nbsp;&nbsp;&nbsp;&nbsp;这样实现的问题在于，我们画得越多，程序就会越慢。因为每次移动手指的时候都会重绘整个贝塞尔路径（`UIBezierPath`），随着路径越来越复杂，每次重绘的工作就会增加，直接导致了帧数的下降。看来我们需要一个更好的方法了。

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation为这些图形类型的绘制提供了专门的类，并给他们提供硬件支持（第六章『专有图层』有详细提到）。`CAShapeLayer`可以绘制多边形，直线和曲线。`CATextLayer`可以绘制文本。`CAGradientLayer`用来绘制渐变。这些总体上都比Core Graphics更快，同时他们也避免了创造一个寄宿图。

&nbsp;&nbsp;&nbsp;&nbsp;如果稍微将之前的代码变动一下，用`CAShapeLayer`替代Core Graphics，性能就会得到提高（见清单13.2）.虽然随着路径复杂性的增加，绘制性能依然会下降，但是只有当非常非常浮躁的绘制时才会感到明显的帧率差异。

清单13.2 用`CAShapeLayer`重新实现绘图应用

```objective-c
#import "DrawingView.h"
#import <QuartzCore/QuartzCore.h>

@interface DrawingView ()

@property (nonatomic, strong) UIBezierPath *path;

@end
￼
@implementation DrawingView

+ (Class)layerClass
{
    //this makes our view create a CAShapeLayer
    //instead of a CALayer for its backing layer
    return [CAShapeLayer class];
}

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
@end
```