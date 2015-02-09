# 在动画过程中取消动画


之前提到过，你可以用`-addAnimation:forKey:`方法中的`key`参数来在添加动画之后检索一个动画，使用如下方法：

    - (CAAnimation *)animationForKey:(NSString *)key;

但并不支持在动画运行过程中修改动画，所以这个方法主要用来检测动画的属性，或者判断它是否被添加到当前图层中。

为了终止一个指定的动画，你可以用如下方法把它从图层移除掉：

    - (void)removeAnimationForKey:(NSString *)key;

或者移除所有动画：

    - (void)removeAllAnimations;
    
动画一旦被移除，图层的外观就立刻更新到当前的模型图层的值。一般说来，动画在结束之后被自动移除，除非设置`removedOnCompletion`为`NO`，如果你设置动画在结束之后不被自动移除，那么当它不需要的时候你要手动移除它；否则它会一直存在于内存中，直到图层被销毁。

我们来扩展之前旋转飞船的示例，这里添加一个按钮来停止或者启动动画。这一次我们用一个非`nil`的值作为动画的键，以便之后可以移除它。`-animationDidStop:finished:`方法中的`flag`参数表明了动画是自然结束还是被打断，我们可以在控制台打印出来。如果你用停止按钮来终止动画，它会打印`NO`，如果允许它完成，它会打印`YES`。

清单8.15是更新后的示例代码，图8.6显示了结果。

清单8.15 开始和停止一个动画

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, strong) CALayer *shipLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //add the ship
    self.shipLayer = [CALayer layer];
    self.shipLayer.frame = CGRectMake(0, 0, 128, 128);
    self.shipLayer.position = CGPointMake(150, 150);
    self.shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    [self.containerView.layer addSublayer:self.shipLayer];
}

- (IBAction)start
{
    //animate the ship rotation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"transform.rotation";
    animation.duration = 2.0;
    animation.byValue = @(M_PI * 2);
    animation.delegate = self;
    [self.shipLayer addAnimation:animation forKey:@"rotateAnimation"];
}

- (IBAction)stop
{
    [self.shipLayer removeAnimationForKey:@"rotateAnimation"];
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
{
    //log that the animation stopped
    NSLog(@"The animation stopped (finished: %@)", flag? @"YES": @"NO");
}

@end
```

<img src="./8.6.jpeg" alt="图8.6" title="图8.6" width="700"/>

图8.6 通过开始和停止按钮控制的旋转动画
