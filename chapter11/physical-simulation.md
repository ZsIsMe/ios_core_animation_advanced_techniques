
##物理模拟

即使使用了基于定时器的动画来复制第10章中关键帧的行为，但还是会有一些本质上的区别：在关键帧的实现中，我们提前计算了所有帧，但是在新的解决方案中，我们实际上实在按需要在计算。意义在于我们可以根据用户输入实时修改动画的逻辑，或者和别的实时动画系统例如物理引擎进行整合。

###Chipmunk

我们来基于物理学创建一个真实的重力模拟效果来取代当前基于缓冲的弹性动画，但即使模拟2D的物理效果就已近极其复杂了，所以就不要尝试去实现它了，直接用开源的物理引擎库好了。

我们将要使用的物理引擎叫做Chipmunk。另外的2D物理引擎也同样可以（例如Box2D），但是Chipmunk使用纯C写的，而不是C++，好处在于更容易和Objective-C项目整合。Chipmunk有很多版本，包括一个和Objective-C绑定的“indie”版本。C语言的版本是免费的，所以我们就用它好了。在本书写作的时候6.1.4是最新的版本；你可以从http://chipmunk-physics.net下载它。

Chipmunk完整的物理引擎相当巨大复杂，但是我们只会使用如下几个类：

* `cpSpace` - 这是所有的物理结构体的容器。它有一个大小和一个可选的重力矢量
* `cpBody` - 它是一个固态无弹力的刚体。它有一个坐标，以及其他物理属性，例如质量，运动和摩擦系数等等。
* `cpShape` - 它是一个抽象的几何形状，用来检测碰撞。可以给结构体添加一个多边形，而且`cpShape`有各种子类来代表不同形状的类型。

在例子中，我们来对一个木箱建模，然后在重力的影响下下落。我们来创建一个`Crate`类，包含屏幕上的可视效果（一个`UIImageView`）和一个物理模型（一个`cpBody`和一个`cpPolyShape`，一个`cpShape`的多边形子类来代表矩形木箱）。

用C版本的Chipmunk会带来一些挑战，因为它现在并不支持Objective-C的引用计数模型，所以我们需要准确的创建和释放对象。为了简化，我们把`cpShape`和`cpBody`的生命周期和`Crate`类进行绑定，然后在木箱的`-init`方法中创建，在`-dealloc`中释放。木箱物理属性的配置很复杂，所以阅读了Chipmunk文档会很有意义。

视图控制器用来管理`cpSpace`，还有和之前一样的计时器逻辑。在每一步中，我们更新`cpSpace`（用来进行物理计算和所有结构体的重新摆放）然后迭代对象，然后再更新我们的木箱视图的位置来匹配木箱的模型（在这里，实际上只有一个结构体，但是之后我们将要添加更多）。

Chipmunk使用了一个和UIKit颠倒的坐标系（Y轴向上为正方向）。为了使得物理模型和视图之间的同步更简单，我们需要通过使用`geometryFlipped`属性翻转容器视图的集合坐标（第3章中有提到），于是模型和视图都共享一个相同的坐标系。

具体的代码见清单11.3。注意到我们并没有在任何地方释放`cpSpace`对象。在这个例子中，内存空间将会在整个app的生命周期中一直存在，所以这没有问题。但是在现实世界的场景中，我们需要像创建木箱结构体和形状一样去管理我们的空间，封装在标准的Cocoa对象中，然后来管理Chipmunk对象的生命周期。图11.1展示了掉落的木箱。

清单11.3 使用物理学来对掉落的木箱建模

```objective-c
#import "ViewController.h" 
#import <QuartzCore/QuartzCore.h>
#import "chipmunk.h"

@interface Crate : UIImageView

@property (nonatomic, assign) cpBody *body;
@property (nonatomic, assign) cpShape *shape;

@end

@implementation Crate

#define MASS 100

- (id)initWithFrame:(CGRect)frame
{
    if ((self = [super initWithFrame:frame])) {
        //set image
        self.image = [UIImage imageNamed:@"Crate.png"];
        self.contentMode = UIViewContentModeScaleAspectFill;
        //create the body
        self.body = cpBodyNew(MASS, cpMomentForBox(MASS, frame.size.width, frame.size.height));
        //create the shape
        cpVect corners[] = {
            cpv(0, 0),
            cpv(0, frame.size.height),
            cpv(frame.size.width, frame.size.height),
            cpv(frame.size.width, 0),
        };
        self.shape = cpPolyShapeNew(self.body, 4, corners, cpv(-frame.size.width/2, -frame.size.height/2));
        //set shape friction & elasticity
        cpShapeSetFriction(self.shape, 0.5);
        cpShapeSetElasticity(self.shape, 0.8);
        //link the crate to the shape
        //so we can refer to crate from callback later on
        self.shape->data = (__bridge void *)self;
        //set the body position to match view
        cpBodySetPos(self.body, cpv(frame.origin.x + frame.size.width/2, 300 - frame.origin.y - frame.size.height/2));
    }
    return self;
}

- (void)dealloc
{
    //release shape and body
    cpShapeFree(_shape);
    cpBodyFree(_body);
}

@end

@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, assign) cpSpace *space;
@property (nonatomic, strong) CADisplayLink *timer;
@property (nonatomic, assign) CFTimeInterval lastStep;

@end

@implementation ViewController

#define GRAVITY 1000

- (void)viewDidLoad
{
    //invert view coordinate system to match physics
    self.containerView.layer.geometryFlipped = YES;
    //set up physics space
    self.space = cpSpaceNew();
    cpSpaceSetGravity(self.space, cpv(0, -GRAVITY));
    //add a crate
    Crate *crate = [[Crate alloc] initWithFrame:CGRectMake(100, 0, 100, 100)];
    [self.containerView addSubview:crate];
    cpSpaceAddBody(self.space, crate.body);
    cpSpaceAddShape(self.space, crate.shape);
    //start the timer
    self.lastStep = CACurrentMediaTime();
    self.timer = [CADisplayLink displayLinkWithTarget:self
                                             selector:@selector(step:)];
    [self.timer addToRunLoop:[NSRunLoop mainRunLoop]
                     forMode:NSDefaultRunLoopMode];
}

void updateShape(cpShape *shape, void *unused)
{
    //get the crate object associated with the shape
    Crate *crate = (__bridge Crate *)shape->data;
    //update crate view position and angle to match physics shape
    cpBody *body = shape->body;
    crate.center = cpBodyGetPos(body);
    crate.transform = CGAffineTransformMakeRotation(cpBodyGetAngle(body));
}

- (void)step:(CADisplayLink *)timer
{
    //calculate step duration
    CFTimeInterval thisStep = CACurrentMediaTime();
    CFTimeInterval stepDuration = thisStep - self.lastStep;
    self.lastStep = thisStep;
    //update physics
    cpSpaceStep(self.space, stepDuration);
    //update all the shapes
    cpSpaceEachShape(self.space, &updateShape, NULL);
}

@end
```

<img src="./11.1.jpeg" alt="图11.1" title="图11.1" width="700">

图11.1 一个木箱图片，根据模拟的重力掉落


###添加用户交互

下一步就是在视图周围添加一道不可见的墙，这样木箱就不会掉落出屏幕之外。或许你会用另一个矩形的`cpPolyShape`来实现，就和之前创建木箱那样，但是我们需要检测的是木箱何时离开视图，而不是何时碰撞，所以我们需要一个空心而不是固体矩形。

我们可以通过给`cpSpace`添加四个`cpSegmentShape`对象（`cpSegmentShape`代表一条直线，所以四个拼起来就是一个矩形）。然后赋给空间的`staticBody`属性（一个不被重力影响的结构体）而不是像木箱那样一个新的`cpBody`实例，因为我们不想让这个边框矩形滑出屏幕或者被一个下落的木箱击中而消失。

同样可以再添加一些木箱来做一些交互。最后再添加一个加速器，这样可以通过倾斜手机来调整重力矢量（为了测试需要在一台真实的设备上运行程序，因为模拟器不支持加速器事件，即使旋转屏幕）。清单11.4展示了更新后的代码，运行结果见图11.2。

由于示例只支持横屏模式，所以交换加速计矢量的x和y值。如果在竖屏下运行程序，请把他们换回来，不然重力方向就错乱了。试一下就知道了，木箱会沿着横向移动。

清单11.4 使用围墙和多个木箱的更新后的代码

```objetive-c
- (void)addCrateWithFrame:(CGRect)frame
{
    Crate *crate = [[Crate alloc] initWithFrame:frame];
    [self.containerView addSubview:crate];
    cpSpaceAddBody(self.space, crate.body);
    cpSpaceAddShape(self.space, crate.shape);
}

- (void)addWallShapeWithStart:(cpVect)start end:(cpVect)end
{
    cpShape *wall = cpSegmentShapeNew(self.space->staticBody, start, end, 1);
    cpShapeSetCollisionType(wall, 2);
    cpShapeSetFriction(wall, 0.5);
    cpShapeSetElasticity(wall, 0.8);
    cpSpaceAddStaticShape(self.space, wall);
}

- (void)viewDidLoad
{
    //invert view coordinate system to match physics
    self.containerView.layer.geometryFlipped = YES;
    //set up physics space
    self.space = cpSpaceNew();
    cpSpaceSetGravity(self.space, cpv(0, -GRAVITY));
    //add wall around edge of view
    [self addWallShapeWithStart:cpv(0, 0) end:cpv(300, 0)];
    [self addWallShapeWithStart:cpv(300, 0) end:cpv(300, 300)];
    [self addWallShapeWithStart:cpv(300, 300) end:cpv(0, 300)];
    [self addWallShapeWithStart:cpv(0, 300) end:cpv(0, 0)];
    //add a crates
    [self addCrateWithFrame:CGRectMake(0, 0, 32, 32)];
    [self addCrateWithFrame:CGRectMake(32, 0, 32, 32)];
    [self addCrateWithFrame:CGRectMake(64, 0, 64, 64)];
    [self addCrateWithFrame:CGRectMake(128, 0, 32, 32)];
    [self addCrateWithFrame:CGRectMake(0, 32, 64, 64)];
    //start the timer
    self.lastStep = CACurrentMediaTime();
    self.timer = [CADisplayLink displayLinkWithTarget:self
                                             selector:@selector(step:)];
    [self.timer addToRunLoop:[NSRunLoop mainRunLoop]
                     forMode:NSDefaultRunLoopMode];
    //update gravity using accelerometer
    [UIAccelerometer sharedAccelerometer].delegate = self;
    [UIAccelerometer sharedAccelerometer].updateInterval = 1/60.0;
}

- (void)accelerometer:(UIAccelerometer *)accelerometer didAccelerate:(UIAcceleration *)acceleration
{
    //update gravity
    cpSpaceSetGravity(self.space, cpv(acceleration.y * GRAVITY, -acceleration.x * GRAVITY));
}
```

<img src="./11.2.jpeg" alt="图11.2" title="图11.2" width="700">

图11.1 真实引力场下的木箱交互

###模拟时间以及固定的时间步长

对于实现动画的缓冲效果来说，计算每帧持续的时间是一个很好的解决方案，但是对模拟物理效果并不理想。通过一个可变的时间步长来实现有着两个弊端：

* 如果时间步长不是固定的，精确的值，物理效果的模拟也就随之不确定。这意味着即使是传入相同的输入值，也可能在不同场合下有着不同的效果。有时候没多大影响，但是在基于物理引擎的游戏下，玩家就会由于相同的操作行为导致不同的结果而感到困惑。同样也会让测试变得麻烦。

* 由于性能故常造成的丢帧或者像电话呼入的中断都可能会造成不正确的结果。考虑一个像子弹那样快速移动物体，每一帧的更新都需要移动子弹，检测碰撞。如果两帧之间的时间加长了，子弹就会在这一步移动更远的距离，穿过围墙或者是别的障碍，这样就丢失了碰撞。

我们想得到的理想的效果就是通过固定的时间步长来计算物理效果，但是在屏幕发生重绘的时候仍然能够同步更新视图（可能会由于在我们控制范围之外造成不可预知的效果）。

幸运的是，由于我们的模型（在这个例子中就是Chipmunk的`cpSpace`中的`cpBody`）被视图（就是屏幕上代表木箱的`UIView`对象）分离，于是就很简单了。我们只需要根据屏幕刷新的时间跟踪时间步长，然后根据每帧去计算一个或者多个模拟出来的效果。

我们可以通过一个简单的循环来实现。通过每次`CADisplayLink`的启动来通知屏幕将要刷新，然后记录下当前的`CACurrentMediaTime()`。我们需要在一个小增量中提前重复物理模拟（这里用120分之一秒）直到赶上显示的时间。然后更新我们的视图，在屏幕刷新的时候匹配当前物理结构体的显示位置。

清单11.5展示了固定时间步长版本的代码

清单11.5 固定时间步长的木箱模拟
```objective-c
#define SIMULATION_STEP (1/120.0)

- (void)step:(CADisplayLink *)timer
{
    //calculate frame step duration
    CFTimeInterval frameTime = CACurrentMediaTime();
    //update simulation
    while (self.lastStep < frameTime) {
        cpSpaceStep(self.space, SIMULATION_STEP);
        self.lastStep += SIMULATION_STEP;
    }
    ￼
    //update all the shapes
    cpSpaceEachShape(self.space, &updateShape, NULL);
}
```

###避免死亡螺旋

当使用固定的模拟时间步长时候，有一件事情一定要注意，就是用来计算物理效果的现实世界的时间并不会加速模拟时间步长。在我们的例子中，我们随意选择了120分之一秒来模拟物理效果。Chipmunk很快，我们的例子也很简单，所以`cpSpaceStep()`会完成的很好，不会延迟帧的更新。

但是如果场景很复杂，比如有上百个物体之间的交互，物理计算就会很复杂，`cpSpaceStep()`的计算也可能会超出1/120秒。我们没有测量出物理步长的时间，因为我们假设了相对于帧刷新来说并不重要，但是如果模拟步长更久的话，就会延迟帧率。

如果帧刷新的时间延迟的话会变得很糟糕，我们的模拟需要执行更多的次数来同步真实的时间。这些额外的步骤就会继续延迟帧的更新，等等。这就是所谓的死亡螺旋，因为最后的结果就是帧率变得越来越慢，直到最后应用程序卡死了。

我们可以通过添加一些代码在设备上来对物理步骤计算真实世界的时间，然后自动调整固定时间步长，但是实际上它不可行。其实只要保证你给容错留下足够的边长，然后在期望支持的最慢的设备上进行测试就可以了。如果物理计算超过了模拟时间的50%，就需要考虑增加模拟时间步长（或者简化场景）。如果模拟时间步长增加到超过1/60秒（一个完整的屏幕更新时间），你就需要减少动画帧率到一秒30帧或者增加`CADisplayLink`的`frameInterval`来保证不会随机丢帧，不然你的动画将会看起来不平滑。


# 物理模拟

