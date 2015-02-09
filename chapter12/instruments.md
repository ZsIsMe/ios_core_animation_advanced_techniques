
##Instruments

&nbsp;&nbsp;&nbsp;&nbsp;Instruments是Xcode套件中没有被充分利用的一个工具。很多iOS开发者从没用过Instruments，或者只是用Leaks工具检测循环引用。实际上有很多Instruments工具，包括为动画性能调优的东西。

&nbsp;&nbsp;&nbsp;&nbsp;你可以通过在菜单中选择Profile选项来打开Instruments（在这之前，记住要把目标设置成iOS设备，而不是模拟器）。然后将会显示出图12.1（如果没有看到所有选项，你可能设置成了模拟器选项）。

<img src="./12.1.jpeg" title="图12.1" alt="图12.1" width="700" />

图12.1 Instruments工具选项窗口

&nbsp;&nbsp;&nbsp;&nbsp;就像之前提到的那样，你应该始终将程序设置成发布选项。幸运的是，配置文件默认就是发布选项，所以你不需要在分析的时候调整编译策略。

我们将讨论如下几个工具：

* **时间分析器** - 用来测量被方法/函数打断的CPU使用情况。

* **Core Animation** - 用来调试各种Core Animation性能问题。

* **OpenGL ES驱动** - 用来调试GPU性能问题。这个工具在编写Open GL代码的时候很有用，但有时也用来处理Core Animation的工作。

&nbsp;&nbsp;&nbsp;&nbsp;Instruments的一个很棒的功能在于它可以创建我们自定义的工具集。除了你初始选择的工具之外，如果在Instruments中打开Library窗口，你可以拖拽别的工具到左侧边栏。我们将创建以上我们提到的三个工具，然后就可以并行使用了（见图12.2）。

<img src="./12.2.jpeg" title="图12.2" alt="图12.2" width="700" />

图12.2 添加额外的工具到Instruments侧边栏

###时间分析器

&nbsp;&nbsp;&nbsp;&nbsp;时间分析器工具用来检测CPU的使用情况。它可以告诉我们程序中的哪个方法正在消耗大量的CPU时间。使用大量的CPU并*不一定*是个问题 - 你可能期望动画路径对CPU非常依赖，因为动画往往是iOS设备中最苛刻的任务。

&nbsp;&nbsp;&nbsp;&nbsp;但是如果你有性能问题，查看CPU时间对于判断性能是不是和CPU相关，以及定位到函数都很有帮助（见图12.3）。

<img src="./12.3.jpeg" title="图12.3" alt="图12.3" width="700" />

图12.3 时间分析器工具

&nbsp;&nbsp;&nbsp;&nbsp;时间分析器有一些选项来帮助我们定位到我们关心的的方法。可以使用左侧的复选框来打开。其中最有用的是如下几点：

* 通过线程分离 - 这可以通过执行的线程进行分组。如果代码被多线程分离的话，那么就可以判断到底是哪个线程造成了问题。

* 隐藏系统库 - 可以隐藏所有苹果的框架代码，来帮助我们寻找哪一段代码造成了性能瓶颈。由于我们不能优化框架方法，所以这对定位到我们能实际修复的代码很有用。

* 只显示Obj-C代码 - 隐藏除了Objective-C之外的所有代码。大多数内部的Core Animation代码都是用C或者C++函数，所以这对我们集中精力到我们代码中显式调用的方法就很有用。

###Core Animation

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation工具用来监测Core Animation性能。它给我们提供了周期性的FPS，并且考虑到了发生在程序之外的动画（见图12.4）。

<img src="./12.4.jpeg" title="图12.4" alt="图12.4" width="700" />

图12.4 使用可视化调试选项的Core Animation工具

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation工具也提供了一系列复选框选项来帮助调试渲染瓶颈：

* **Color Blended Layers** - 这个选项基于渲染程度对屏幕中的混合区域进行绿到红的高亮（也就是多个半透明图层的叠加）。由于重绘的原因，混合对GPU性能会有影响，同时也是滑动或者动画帧率下降的罪魁祸首之一。

* **ColorHitsGreenandMissesRed** - 当使用`shouldRasterizep`属性的时候，耗时的图层绘制会被缓存，然后当做一个简单的扁平图片呈现。当缓存再生的时候这个选项就用红色对栅格化图层进行了高亮。如果缓存频繁再生的话，就意味着栅格化可能会有负面的性能影响了（更多关于使用`shouldRasterize`的细节见第15章“图层性能”）。

* **Color Copied Images** - 有时候寄宿图片的生成意味着Core Animation被强制生成一些图片，然后发送到渲染服务器，而不是简单的指向原始指针。这个选项把这些图片渲染成蓝色。复制图片对内存和CPU使用来说都是一项非常昂贵的操作，所以应该尽可能的避免。

* **Color Immediately** - 通常Core Animation Instruments以每毫秒10次的频率更新图层调试颜色。对某些效果来说，这显然太慢了。这个选项就可以用来设置每帧都更新（可能会影响到渲染性能，而且会导致帧率测量不准，所以不要一直都设置它）。

* **Color Misaligned Images** - 这里会高亮那些被缩放或者拉伸以及没有正确对齐到像素边界的图片（也就是非整型坐标）。这些中的大多数通常都会导致图片的不正常缩放，如果把一张大图当缩略图显示，或者不正确地模糊图像，那么这个选项将会帮你识别出问题所在。

* **Color Offscreen-Rendered Yellow** - 这里会把那些需要离屏渲染的图层高亮成黄色。这些图层很可能需要用`shadowPath`或者`shouldRasterize`来优化。

* **Color OpenGL Fast Path Blue** - 这个选项会对任何直接使用OpenGL绘制的图层进行高亮。如果仅仅使用UIKit或者Core Animation的API，那么不会有任何效果。如果使用`GLKView`或者`CAEAGLLayer`，那如果不显示蓝色块的话就意味着你正在强制CPU渲染额外的纹理，而不是绘制到屏幕。

* **Flash Updated Regions** - 这个选项会对重绘的内容高亮成黄色（也就是任何在软件层面使用Core Graphics绘制的图层）。这种绘图的速度很慢。如果频繁发生这种情况的话，这意味着有一个隐藏的bug或者说通过增加缓存或者使用替代方案会有提升性能的空间。

&nbsp;&nbsp;&nbsp;&nbsp;这些高亮图层的选项同样在iOS模拟器的调试菜单也可用（图12.5）。我们之前说过用模拟器测试性能并不好，但如果你能通过这些高亮选项识别出性能问题出在什么地方的话，那么使用iOS模拟器来验证问题是否解决也是比真机测试更有效的。

<img src="./12.5.jpeg" title="图12.5" alt="图12.5" width="700" />

图12.5 iOS模拟器中Core Animation可视化调试选项

###OpenGL ES驱动

&nbsp;&nbsp;&nbsp;&nbsp;OpenGL ES驱动工具可以帮你测量GPU的利用率，同样也是一个很好的来判断和GPU相关动画性能的指示器。它同样也提供了类似Core Animation那样显示FPS的工具（图12.6）。

<img src="./12.6.jpeg" title="图12.6" alt="图12.6" width="700" />

图12.6 OpenGL ES驱动工具

侧栏的邮编是一系列有用的工具。其中和Core Animation性能最相关的是如下几点：

* **Renderer Utilization** - 如果这个值超过了~50%，就意味着你的动画可能对帧率有所限制，很可能因为离屏渲染或者是重绘导致的过度混合。

* **Tiler Utilization** - 如果这个值超过了~50%，就意味着你的动画可能限制于几何结构方面，也就是在屏幕上有太多的图层占用了。


##一个可用的案例

&nbsp;&nbsp;&nbsp;&nbsp;现在我们已经对Instruments中动画性能工具非常熟悉了，那么可以用它在现实中解决一些实际问题。

&nbsp;&nbsp;&nbsp;&nbsp;我们创建一个简单的显示模拟联系人姓名和头像列表的应用。注意即使把头像图片存在应用本地，为了使应用看起来更真实，我们分别实时加载图片，而不是用`–imageNamed:`预加载。同样添加一些图层阴影来使得列表显示得更真实。清单12.1展示了最初版本的实现。

清单12.1 使用假数据的一个简单联系人列表

```objective-c
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>

@interface ViewController () <UITableViewDataSource>

@property (nonatomic, strong) NSArray *items;
@property (nonatomic, weak) IBOutlet UITableView *tableView;

@end

@implementation ViewController

- (NSString *)randomName
{
    NSArray *first = @[@"Alice", @"Bob", @"Bill", @"Charles", @"Dan", @"Dave", @"Ethan", @"Frank"];
    NSArray *last = @[@"Appleseed", @"Bandicoot", @"Caravan", @"Dabble", @"Ernest", @"Fortune"];
    NSUInteger index1 = (rand()/(double)INT_MAX) * [first count];
    NSUInteger index2 = (rand()/(double)INT_MAX) * [last count];
    return [NSString stringWithFormat:@"%@ %@", first[index1], last[index2]];
}

- (NSString *)randomAvatar
{
    NSArray *images = @[@"Snowman", @"Igloo", @"Cone", @"Spaceship", @"Anchor", @"Key"];
    NSUInteger index = (rand()/(double)INT_MAX) * [images count];
    return images[index];
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    //set up data
    NSMutableArray *array = [NSMutableArray array];
    for (int i = 0; i < 1000; i++) {
        ￼//add name
        [array addObject:@{@"name": [self randomName], @"image": [self randomAvatar]}];
    }
    self.items = array;
    //register cell class
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"Cell"];
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return [self.items count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    //dequeue cell
    UITableViewCell *cell = [self.tableView dequeueReusableCellWithIdentifier:@"Cell" forIndexPath:indexPath];
    //load image
    NSDictionary *item = self.items[indexPath.row];
    NSString *filePath = [[NSBundle mainBundle] pathForResource:item[@"image"] ofType:@"png"];
    //set image and text
    cell.imageView.image = [UIImage imageWithContentsOfFile:filePath];
    cell.textLabel.text = item[@"name"];
    //set image shadow
    cell.imageView.layer.shadowOffset = CGSizeMake(0, 5);
    cell.imageView.layer.shadowOpacity = 0.75;
    cell.clipsToBounds = YES;
    //set text shadow
    cell.textLabel.backgroundColor = [UIColor clearColor];
    cell.textLabel.layer.shadowOffset = CGSizeMake(0, 2);
    cell.textLabel.layer.shadowOpacity = 0.5;
    return cell;
}

@end
```

&nbsp;&nbsp;&nbsp;&nbsp;当快速滑动的时候就会非常卡（见图12.7的FPS计数器）。

<img src="./12.7.jpeg" title="图12.7" alt="图12.7" width="700" />

图12.7 滑动帧率降到15FPS

&nbsp;&nbsp;&nbsp;&nbsp;仅凭直觉，我们猜测性能瓶颈应该在图片加载。我们实时从闪存加载图片，而且没有缓存，所以很可能是这个原因。我们可以用一些很赞的代码修复，然后使用GCD异步加载图片，然后缓存。。。等一下，在开始编码之前，测试一下假设是否成立。首先用我们的三个Instruments工具分析一下程序来定位问题。我们推测问题可能和图片加载相关，所以用Time Profiler工具来试试（图12.8）。

<img src="./12.8.jpeg" title="图12.8" alt="图12.8" width="700" />

图12.8 用The timing profile分析联系人列表

&nbsp;&nbsp;&nbsp;&nbsp;`-tableView:cellForRowAtIndexPath:`中的CPU时间总利用率只有~28%（也就是加载头像图片的地方），非常低。于是建议是CPU/IO并不是真正的限制因素。然后看看是不是GPU的问题：在OpenGL ES Driver工具中检测GPU利用率（图12.9）。

<img src="./12.9.jpeg" title="图12.9" alt="图12.9" width="700" />

图12.9 OpenGL ES Driver工具显示的GPU利用率

&nbsp;&nbsp;&nbsp;&nbsp;渲染服务利用率的值达到51%和63%。看起来GPU需要做很多工作来渲染联系人列表。

&nbsp;&nbsp;&nbsp;&nbsp;为什么GPU利用率这么高呢？我们来用Core Animation调试工具选项来检查屏幕。首先打开Color Blended Layers（图12.10）。

<img src="./12.10.jpeg" title="图12.10" alt="图12.10" width="700" />

图12.10 使用Color Blended Layers选项调试程序

&nbsp;&nbsp;&nbsp;&nbsp;屏幕中所有红色的部分都意味着字符标签视图的高级别混合，这很正常，因为我们把背景设置成了透明色来显示阴影效果。这就解释了为什么渲染利用率这么高了。

&nbsp;&nbsp;&nbsp;&nbsp;那么离屏绘制呢？打开Core Animation工具的Color Offscreen - Rendered Yellow选项（图12.11）。

<img src="./12.11.jpeg" title="图12.11" alt="图12.11" width="700" />

图12.11 Color Offscreen–Rendered Yellow选项

&nbsp;&nbsp;&nbsp;&nbsp;所有的表格单元内容都在离屏绘制。这一定是因为我们给图片和标签视图添加的阴影效果。在代码中禁用阴影，然后看下性能是否有提高（图12.12）。

<img src="./12.12.jpeg" title="图12.12" alt="图12.12" width="700" />

图12.12 禁用阴影之后运行程序接近60FPS

&nbsp;&nbsp;&nbsp;&nbsp;问题解决了。干掉阴影之后，滑动很流畅。但是我们的联系人列表看起来没有之前好了。那如何保持阴影效果而且不会影响性能呢？

&nbsp;&nbsp;&nbsp;&nbsp;好吧，每一行的字符和头像在每一帧刷新的时候并不需要变，所以看起来`UITableViewCell`的图层非常适合做缓存。我们可以使用`shouldRasterize`来缓存图层内容。这将会让图层离屏之后渲染一次然后把结果保存起来，直到下次利用的时候去更新（见清单12.2）。

清单12.2 使用`shouldRasterize`提高性能

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
￼{
    //dequeue cell
    UITableViewCell *cell = [self.tableView dequeueReusableCellWithIdentifier:@"Cell"
                                                                 forIndexPath:indexPath];
    ...
    //set text shadow
    cell.textLabel.backgroundColor = [UIColor clearColor];
    cell.textLabel.layer.shadowOffset = CGSizeMake(0, 2);
    cell.textLabel.layer.shadowOpacity = 0.5;
    //rasterize
    cell.layer.shouldRasterize = YES;
    cell.layer.rasterizationScale = [UIScreen mainScreen].scale;
    return cell;
}
```

&nbsp;&nbsp;&nbsp;&nbsp;我们仍然离屏绘制图层内容，但是由于显式地禁用了栅格化，Core Animation就对绘图缓存了结果，于是对提高了性能。我们可以验证缓存是否有效，在Core Animation工具中点击Color Hits Green and Misses Red选项（图12.13）。

<img src="./12.13.jpeg" title="图12.13" alt="图12.13" width="700" />

图12.13 Color Hits Green and Misses Red验证了缓存有效

&nbsp;&nbsp;&nbsp;&nbsp;结果和预期一致 - 大部分都是绿色，只有当滑动到屏幕上的时候会闪烁成红色。因此，现在帧率更加平滑了。

&nbsp;&nbsp;&nbsp;&nbsp;所以我们最初的设想是错的。图片的加载并不是真正的瓶颈所在，而且试图把它置于一个复杂的多线程加载和缓存的实现都将是徒劳。所以在动手修复之前验证问题所在是个很好的习惯！
