
##减少图层数量

&nbsp;&nbsp;&nbsp;&nbsp;初始化图层，处理图层，打包通过IPC发给渲染引擎，转化成OpenGL几何图形，这些是一个图层的大致资源开销。事实上，一次性能够在屏幕上显示的最大图层数量也是有限的。

&nbsp;&nbsp;&nbsp;&nbsp;确切的限制数量取决于iOS设备，图层类型，图层内容和属性等。但是总得说来可以容纳上百或上千个，下面我们将演示即使图层本身并没有做什么也会遇到的性能问题。

###裁切

&nbsp;&nbsp;&nbsp;&nbsp;在对图层做任何优化之前，你需要确定你不是在创建一些不可见的图层，图层在以下几种情况下回事不可见的：

* 图层在屏幕边界之外，或是在父图层边界之外。
* 完全在一个不透明图层之后。
* 完全透明

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation非常擅长处理对视觉效果无意义的图层。但是经常性地，你自己的代码会比Core Animation更早地想知道一个图层是否是有用的。理想状况下，在图层对象在创建之前就想知道，以避免创建和配置不必要图层的额外工作。

&nbsp;&nbsp;&nbsp;&nbsp;举个例子。清单15.3 的代码展示了一个简单的滚动3D图层矩阵。这看上去很酷，尤其是图层在移动的时候（见图15.1），但是绘制他们并不是很麻烦，因为这些图层就是一些简单的矩形色块。

清单15.3 绘制3D图层矩阵

```objective-c
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>

#define WIDTH 10
#define HEIGHT 10
#define DEPTH 10
#define SIZE 100
#define SPACING 150
#define CAMERA_DISTANCE 500

@interface ViewController ()
￼
@property (nonatomic, strong) IBOutlet UIScrollView *scrollView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //set content size
    self.scrollView.contentSize = CGSizeMake((WIDTH - 1)*SPACING, (HEIGHT - 1)*SPACING);

    //set up perspective transform
    CATransform3D transform = CATransform3DIdentity;
    transform.m34 = -1.0 / CAMERA_DISTANCE;
    self.scrollView.layer.sublayerTransform = transform;

    //create layers
    for (int z = DEPTH - 1; z >= 0; z--) {
        for (int y = 0; y < HEIGHT; y++) {
            for (int x = 0; x < WIDTH; x++) {
                //create layer
                CALayer *layer = [CALayer layer];
                layer.frame = CGRectMake(0, 0, SIZE, SIZE);
                layer.position = CGPointMake(x*SPACING, y*SPACING);
                layer.zPosition = -z*SPACING;
                //set background color
                layer.backgroundColor = [UIColor colorWithWhite:1-z*(1.0/DEPTH) alpha:1].CGColor;
                //attach to scroll view
                [self.scrollView.layer addSublayer:layer];
            }
        }
    }
    ￼
    //log
    NSLog(@"displayed: %i", DEPTH*HEIGHT*WIDTH);
}
@end
```

![](./15.1.png)

图15.1 滚动的3D图层矩阵

&nbsp;&nbsp;&nbsp;&nbsp;`WIDTH`，`HEIGHT`和`DEPTH`常量控制着图层的生成。在这个情况下，我们得到的是10\*10\*10个图层，总量为1000个，不过一次性显示在屏幕上的大约就几百个。

&nbsp;&nbsp;&nbsp;&nbsp;如果把`WIDTH`和`HEIGHT`常量增加到100，我们的程序就会慢得像龟爬了。这样我们有了100000个图层，性能下降一点儿也不奇怪。

&nbsp;&nbsp;&nbsp;&nbsp;但是显示在屏幕上的图层数量并没有增加，那么根本没有额外的东西需要绘制。程序慢下来的原因其实是因为在管理这些图层上花掉了不少功夫。他们大部分对渲染的最终结果没有贡献，但是在丢弃这么图层之前，Core Animation要强制计算每个图层的位置，就这样，我们的帧率就慢了下来。

&nbsp;&nbsp;&nbsp;&nbsp;我们的图层是被安排在一个均匀的栅格中，我们可以计算出哪些图层会被最终显示在屏幕上，根本不需要对每个图层的位置进行计算。这个计算并不简单，因为我们还要考虑到透视的问题。如果我们直接这样做了，Core Animation就不用费神了。

&nbsp;&nbsp;&nbsp;&nbsp;既然这样，让我们来重构我们的代码吧。改造后，随着视图的滚动动态地实例化图层而不是事先都分配好。这样，在创造他们之前，我们就可以计算出是否需要他。接着，我们增加一些代码去计算可视区域这样就可以排除区域之外的图层了。清单15.4是改造后的结果。

清单15.4 排除可视区域之外的图层

```objective-c
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>

#define WIDTH 100
#define HEIGHT 100
#define DEPTH 10
#define SIZE 100
#define SPACING 150
#define CAMERA_DISTANCE 500
#define PERSPECTIVE(z) (float)CAMERA_DISTANCE/(z + CAMERA_DISTANCE)

@interface ViewController () <UIScrollViewDelegate>

@property (nonatomic, weak) IBOutlet UIScrollView *scrollView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //set content size
    self.scrollView.contentSize = CGSizeMake((WIDTH - 1)*SPACING, (HEIGHT - 1)*SPACING);
    //set up perspective transform
    CATransform3D transform = CATransform3DIdentity;
    transform.m34 = -1.0 / CAMERA_DISTANCE;
    self.scrollView.layer.sublayerTransform = transform;
}
￼
- (void)viewDidLayoutSubviews
{
    [self updateLayers];
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    [self updateLayers];
}

- (void)updateLayers
{
    //calculate clipping bounds
    CGRect bounds = self.scrollView.bounds;
    bounds.origin = self.scrollView.contentOffset;
    bounds = CGRectInset(bounds, -SIZE/2, -SIZE/2);
    //create layers
    NSMutableArray *visibleLayers = [NSMutableArray array];
    for (int z = DEPTH - 1; z >= 0; z--)
    {
        //increase bounds size to compensate for perspective
        CGRect adjusted = bounds;
        adjusted.size.width /= PERSPECTIVE(z*SPACING);
        adjusted.size.height /= PERSPECTIVE(z*SPACING);
        adjusted.origin.x -= (adjusted.size.width - bounds.size.width) / 2;
        adjusted.origin.y -= (adjusted.size.height - bounds.size.height) / 2;
        for (int y = 0; y < HEIGHT; y++) {
        //check if vertically outside visible rect
            if (y*SPACING < adjusted.origin.y || y*SPACING >= adjusted.origin.y + adjusted.size.height)
            {
                continue;
            }
            for (int x = 0; x < WIDTH; x++) {
                //check if horizontally outside visible rect
                if (x*SPACING < adjusted.origin.x ||x*SPACING >= adjusted.origin.x + adjusted.size.width)
                {
                    continue;
                }
                ￼
                //create layer
                CALayer *layer = [CALayer layer];
                layer.frame = CGRectMake(0, 0, SIZE, SIZE);
                layer.position = CGPointMake(x*SPACING, y*SPACING);
                layer.zPosition = -z*SPACING;
                //set background color
                layer.backgroundColor = [UIColor colorWithWhite:1-z*(1.0/DEPTH) alpha:1].CGColor;
                //attach to scroll view
                [visibleLayers addObject:layer];
            }
        }
    }
    //update layers
    self.scrollView.layer.sublayers = visibleLayers;
    //log
    NSLog(@"displayed: %i/%i", [visibleLayers count], DEPTH*HEIGHT*WIDTH);
}
@end
```

&nbsp;&nbsp;&nbsp;&nbsp;这个计算机制并不具有普适性，但是原则上是一样。（当你用一个`UITableView`或者`UICollectionView`时，系统做了类似的事情）。这样做的结果？我们的程序可以处理成百上千个『虚拟』图层而且完全没有性能问题！因为它不需要一次性实例化几百个图层。

###对象回收

&nbsp;&nbsp;&nbsp;&nbsp;处理巨大数量的相似视图或图层时还有一个技巧就是回收他们。对象回收在iOS颇为常见；`UITableView`和`UICollectionView`都有用到，`MKMapView`中的动画pin码也有用到，还有其他很多例子。

&nbsp;&nbsp;&nbsp;&nbsp;对象回收的基础原则就是你需要创建一个相似对象池。当一个对象的指定实例（本例子中指的是图层）结束了使命，你把它添加到对象池中。每次当你需要一个实例时，你就从池中取出一个。当且仅当池中为空时再创建一个新的。

&nbsp;&nbsp;&nbsp;&nbsp;这样做的好处在于避免了不断创建和释放对象（相当消耗资源，因为涉及到内存的分配和销毁）而且也不必给相似实例重复赋值。

&nbsp;&nbsp;&nbsp;&nbsp;好了，让我们再次更新代码吧（见清单15.5）

清单15.5 通过回收减少不必要的分配

```objective-c
@interface ViewController () <UIScrollViewDelegate>

@property (nonatomic, weak) IBOutlet UIScrollView *scrollView;
@property (nonatomic, strong) NSMutableSet *recyclePool;


@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad]; //create recycle pool
    self.recyclePool = [NSMutableSet set];
    //set content size
    self.scrollView.contentSize = CGSizeMake((WIDTH - 1)*SPACING, (HEIGHT - 1)*SPACING);
    //set up perspective transform
    CATransform3D transform = CATransform3DIdentity;
    transform.m34 = -1.0 / CAMERA_DISTANCE;
    self.scrollView.layer.sublayerTransform = transform;
}

- (void)viewDidLayoutSubviews
{
    [self updateLayers];
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    [self updateLayers];
}

- (void)updateLayers {
    ￼
    //calculate clipping bounds
    CGRect bounds = self.scrollView.bounds;
    bounds.origin = self.scrollView.contentOffset;
    bounds = CGRectInset(bounds, -SIZE/2, -SIZE/2);
    //add existing layers to pool
    [self.recyclePool addObjectsFromArray:self.scrollView.layer.sublayers];
    //disable animation
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    //create layers
    NSInteger recycled = 0;
    NSMutableArray *visibleLayers = [NSMutableArray array];
    for (int z = DEPTH - 1; z >= 0; z--)
    {
        //increase bounds size to compensate for perspective
        CGRect adjusted = bounds;
        adjusted.size.width /= PERSPECTIVE(z*SPACING);
        adjusted.size.height /= PERSPECTIVE(z*SPACING);
        adjusted.origin.x -= (adjusted.size.width - bounds.size.width) / 2; adjusted.origin.y -= (adjusted.size.height - bounds.size.height) / 2;
        for (int y = 0; y < HEIGHT; y++) {
            //check if vertically outside visible rect
            if (y*SPACING < adjusted.origin.y ||
                y*SPACING >= adjusted.origin.y + adjusted.size.height)
            {
                continue;
            }
            for (int x = 0; x < WIDTH; x++) {
                //check if horizontally outside visible rect
                if (x*SPACING < adjusted.origin.x ||
                    x*SPACING >= adjusted.origin.x + adjusted.size.width)
                {
                    continue;
                }
                //recycle layer if available
                CALayer *layer = [self.recyclePool anyObject]; if (layer)
                {
                    ￼
                    recycled ++;
                    [self.recyclePool removeObject:layer]; }
                else
                {
                    layer = [CALayer layer];
                    layer.frame = CGRectMake(0, 0, SIZE, SIZE); }
                //set position
                layer.position = CGPointMake(x*SPACING, y*SPACING); layer.zPosition = -z*SPACING;
                //set background color
                layer.backgroundColor =
                [UIColor colorWithWhite:1-z*(1.0/DEPTH) alpha:1].CGColor;
                //attach to scroll view
                [visibleLayers addObject:layer]; }
        } }
    [CATransaction commit]; //update layers
    self.scrollView.layer.sublayers = visibleLayers;
    //log
    NSLog(@"displayed: %i/%i recycled: %i",
          [visibleLayers count], DEPTH*HEIGHT*WIDTH, recycled);
}
@end
```

&nbsp;&nbsp;&nbsp;&nbsp;本例中，我们只有图层对象这一种类型，但是UIKit有时候用一个标识符字符串来区分存储在不同对象池中的不同的可回收对象类型。

&nbsp;&nbsp;&nbsp;&nbsp;你可能注意到当设置图层属性时我们用了一个`CATransaction`来抑制动画效果。在之前并不需要这样做，因为在显示之前我们给所有图层设置一次属性。但是既然图层正在被回收，禁止隐式动画就有必要了，不然当属性值改变时，图层的隐式动画就会被触发。

###Core Graphics绘制

&nbsp;&nbsp;&nbsp;&nbsp;当排除掉对屏幕显示没有任何贡献的图层或者视图之后，长远看来，你可能仍然需要减少图层的数量。例如，如果你正在使用多个`UILabel`或者`UIImageView`实例去显示固定内容，你可以把他们全部替换成一个单独的视图，然后用`-drawRect:`方法绘制出那些复杂的视图层级。

&nbsp;&nbsp;&nbsp;&nbsp;这个提议看上去并不合理因为大家都知道软件绘制行为要比GPU合成要慢而且还需要更多的内存空间，但是在因为图层数量而使得性能受限的情况下，软件绘制很可能提高性能呢，因为它避免了图层分配和操作问题。

&nbsp;&nbsp;&nbsp;&nbsp;你可以自己实验一下这个情况，它包含了性能和栅格化的权衡，但是意味着你可以从图层树上去掉子图层（用`shouldRasterize`，与完全遮挡图层相反）。

###-renderInContext: 方法

&nbsp;&nbsp;&nbsp;&nbsp;用Core Graphics去绘制一个静态布局有时候会比用层级的`UIView`实例来得快，但是使用`UIView`实例要简单得多而且比用手写代码写出相同效果要可靠得多，更边说Interface Builder来得直接明了。为了性能而舍弃这些便利实在是不应该。

&nbsp;&nbsp;&nbsp;&nbsp;幸好，你不必这样，如果大量的视图或者图层真的关联到了屏幕上将会是一个大问题。没有与图层树相关联的图层不会被送到渲染引擎，也没有性能问题（在他们被创建和配置之后）。

&nbsp;&nbsp;&nbsp;&nbsp;使用`CALayer`的`-renderInContext:`方法，你可以将图层及其子图层快照进一个Core Graphics上下文然后得到一个图片，它可以直接显示在`UIImageView`中，或者作为另一个图层的`contents`。不同于`shouldRasterize` —— 要求图层与图层树相关联 —— ，这个方法没有持续的性能消耗。

&nbsp;&nbsp;&nbsp;&nbsp;当图层内容改变时，刷新这张图片的机会取决于你（不同于`shouldRasterize`，它自动地处理缓存和缓存验证），但是一旦图片被生成，相比于让Core Animation处理一个复杂的图层树，你节省了相当客观的性能。
