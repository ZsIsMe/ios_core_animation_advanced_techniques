# 属性动画

首先我们来探讨一下*属性动画*。属性动画作用于图层的某个单一属性，并指定了它的一个目标值，或者一连串将要做动画的值。属性动画分为两种：*基础*和*关键帧*。

###基础动画

动画其实就是一段时间内发生的改变，最简单的形式就是从一个值改变到另一个值，这也是`CABasicAnimation`最主要的功能。`CABasicAnimation`是`CAPropertyAnimation`的一个子类，而`CAPropertyAnimation`的父类是`CAAnimation`，`CAAnimation`同时也是Core Animation所有动画类型的抽象基类。作为一个抽象类，`CAAnimation`本身并没有做多少工作，它提供了一个计时函数（见第十章“缓冲”），一个委托（用于反馈动画状态）以及一个`removedOnCompletion`，用于标识动画是否该在结束后自动释放（默认`YES`，为了防止内存泄露）。`CAAnimation`同时实现了一些协议，包括`CAAction`（允许`CAAnimation`的子类可以提供图层行为），以及`CAMediaTiming`（第九章“图层时间”将会详细解释）。

`CAPropertyAnimation`通过指定动画的`keyPath`作用于一个单一属性，`CAAnimation`通常应用于一个指定的`CALayer`，于是这里指的也就是一个图层的`keyPath`了。实际上它是一个关键*路径*（一些用点表示法可以在层级关系中指向任意嵌套的对象），而不仅仅是一个属性的名称，因为这意味着动画不仅可以作用于图层本身的属性，而且还包含了它的子成员的属性，甚至是一些*虚拟*的属性（后面会详细解释）。

`CABasicAnimation`继承于`CAPropertyAnimation`，并添加了如下属性：

    id fromValue 
    id toValue 
    id byValue
    
从命名就可以得到很好的解释：`fromValue`代表了动画开始之前属性的值，`toValue`代表了动画结束之后的值，`byValue`代表了动画执行过程中改变的值。

通过组合这三个属性就可以有很多种方式来指定一个动画的过程。它们被定义成`id`类型而不是一些具体的类型是因为属性动画可以用作很多不同种的属性类型，包括数字类型，矢量，变换矩阵，甚至是颜色或者图片。

`id`类型可以包含任意由`NSObject`派生的对象，但有时候你会希望对一些不直接从`NSObject`继承的属性类型做动画，这意味着你需要把这些值用一个对象来封装，或者强转成一个对象，就像某些功能和Objective-C对象类似的Core Foundation类型。但是如何从一个具体的数据类型转换成id看起来并不明显，一些普通的例子见表8.1。

表8.1 用于`CAPropertyAnimation`的一些类型转换

Type          | Object Type | Code Example
--------------|-------------|-----------------------------------------------------
CGFloat       | NSNumber    | id obj = @(float);
CGPoint       | NSValue     | id obj = [NSValue valueWithCGPoint:point);
CGSize        | NSValue     | id obj = [NSValue valueWithCGSize:size);
CGRect        | NSValue     | id obj = [NSValue valueWithCGRect:rect);
CATransform3D | NSValue     | id obj = [NSValue valueWithCATransform3D:transform);
CGImageRef    | id          | id obj = (__bridge id)imageRef;
CGColorRef    | id          | id obj = (__bridge id)colorRef;

`fromValue`，`toValue`和`byValue`属性可以用很多种方式来组合，但为了防止冲突，不能一次性同时指定这三个值。例如，如果指定了`fromValue`等于2，`toValue`等于4，`byValue`等于3，那么Core Animation就不知道结果到底是4（`toValue`）还是5（`fromValue + byValue`）了。他们的用法在`CABasicAnimation`头文件中已经描述的很清楚了，所以在这里就不重复了。总的说来，就是只需要指定`toValue`或者`byValue`，剩下的值都可以通过上下文自动计算出来。

举个例子：我们修改一下第七章中的颜色渐变的动画，用显式的`CABasicAnimation`来取代之前的隐式动画，代码见清单8.1。

清单8.1 通过`CABasicAnimation`来设置图层背景色

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;
@property (nonatomic, strong) IBOutlet CALayer *colorLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create sublayer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer];
}

- (IBAction)changeColor
{
    ￼//create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
}

@end
```

运行程序，结果有点差强人意，点击按钮，的确可以使图层动画过渡到一个新的颜色，然动画结束之后又立刻变回原始值。

这是因为动画并没有改变图层的*模型*，而只是*呈现*（第七章）。一旦动画结束并从图层上移除之后，图层就立刻恢复到之前定义的外观状态。我们从没改变过`backgroundColor`属性，所以图层就返回到原始的颜色。

当之前在使用隐式动画的时候，实际上它就是用例子中`CABasicAnimation`来实现的（回忆第七章，我们在`-actionForLayer:forKey:`委托方法打印出来的结果就是`CABasicAnimation`）。但是在那个例子中，我们通过设置属性来打开动画。在这里我们做了相同的动画，但是并没有设置任何属性的值（这就是为什么会立刻变回初始状态的原因）。

把动画设置成一个图层的行为（然后通过改变属性值来启动动画）是到目前为止同步属性值和动画状态最简单的方式了，假设由于某些原因我们不能这么做（通常因为`UIView`关联的图层不能这么做动画），那么有两种可以更新属性值的方式：在动画开始之前或者动画结束之后。

动画之前改变属性的值是最简单的办法，但这意味着我们不能使用`fromValue`这么好的特性了，而且要手动将`fromValue`设置成图层当前的值。

于是在动画创建之前插入如下代码，就可以解决问题了

    animation.fromValue = (__bridge id)self.colorLayer.backgroundColor; 
    self.colorLayer.backgroundColor = color.CGColor;
    
这的确是可行的，但还是有些问题，如果这里已经正在进行一段动画，我们需要从*呈现*图层那里去获得`fromValue`，而不是模型图层。另外，由于这里的图层并不是`UIView`关联的图层，我们需要用`CATransaction`来禁用隐式动画行为，否则默认的图层行为会干扰我们的显式动画（实际上，显式动画通常会覆盖隐式动画，但在文章中并没有提到，所以为了安全最好这么做）。

更新之后的代码如下：
```objective-c
CALayer *layer = self.colorLayer.presentationLayer ?:
self.colorLayer;
 animation.fromValue = (__bridge id)layer.backgroundColor;
[CATransaction begin];
[CATransaction setDisableActions:YES];
self.colorLayer.backgroundColor = color.CGColor;
[CATransaction commit];
```

如果给每个动画都添加这些，代码会显得特别臃肿。幸运的是，我们可以从`CABasicAnimation`去自动设置这些。于是可以创建一个可复用的代码。清单8.2修改了之前的示例，通过使用`CABasicAnimation`的一个函数来避免在每次动画时候都重复那些臃肿的代码。

清单8.2 修改动画立刻恢复到原始状态的一个可复用函数

```objective-c
- (void)applyBasicAnimation:(CABasicAnimation *)animation toLayer:(CALayer *)layer
￼{

    //set the from value (using presentation layer if available)
    animation.fromValue = [layer.presentationLayer ?: layer valueForKeyPath:animation.keyPath];
    //update the property in advance
    //note: this approach will only work if toValue != nil 
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    [layer setValue:animation.toValue forKeyPath:animation.keyPath];
    [CATransaction commit];
    //apply animation to layer
    [layer addAnimation:animation forKey:nil];
}

- (IBAction)changeColor
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
    //apply animation without snap-back
    [self applyBasicAnimation:animation toLayer:self.colorLayer];
}
```

这种简单的实现方式通过`toValue`而不是`byValue`来处理动画，不过这已经是朝更好的解决方案迈出一大步了。你可以把它添加给`CALayer`作为一个分类，以方便更好地使用。

解决看起来如此简单的一个问题都着实麻烦，但是别的方案会更加复杂。如果不在动画开始之前去更新目标属性，那么就只能在动画完全结束或者取消的时候更新它。这意味着我们需要精准地在动画结束之后，图层返回到原始值之前更新属性。那么该如何找到这个点呢？

###CAAnimationDelegate

在第七章使用隐式动画的时候，我们可以在`CATransaction`完成块中检测到动画的完成。但是这种方式并不适用于显式动画，因为这里的动画和事务并没太多关联。

那么为了知道一个显式动画在何时结束，我们需要使用一个实现了`CAAnimationDelegate`协议的`delegate`。
