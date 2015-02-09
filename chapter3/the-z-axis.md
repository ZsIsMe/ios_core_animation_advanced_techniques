# Z坐标轴


&nbsp;&nbsp;&nbsp;&nbsp;和`UIView`严格的二维坐标系不同，`CALayer`存在于一个三维空间当中。除了我们已经讨论过的`position`和`anchorPoint`属性之外，`CALayer`还有另外两个属性，`zPosition`和`anchorPointZ`，二者都是在Z轴上描述图层位置的浮点类型。

&nbsp;&nbsp;&nbsp;&nbsp;注意这里并没有更*深*的属性来描述由宽和高做成的`bounds`了，图层是一个完全扁平的对象，你可以把它们想象成类似于一页二维的坚硬的纸片，用胶水粘成一个空洞，就像三维结构的折纸一样。

&nbsp;&nbsp;&nbsp;&nbsp;`zPosition`属性在大多数情况下其实并不常用。在第五章，我们将会涉及`CATransform3D`，你会知道如何在三维空间移动和旋转图层，除了做变换之外，`zPosition`最实用的功能就是改变图层的*显示顺序*了。

&nbsp;&nbsp;&nbsp;&nbsp;通常，图层是根据它们子图层的`sublayers`出现的顺序来类绘制的，这就是所谓的*画家的算法*--就像一个画家在墙上作画--后被绘制上的图层将会遮盖住之前的图层，但是通过增加图层的`zPosition`，就可以把图层向相机方向*前置*，于是它就在所有其他图层的*前面*了（或者至少是小于它的`zPosition`值的图层的前面）。

&nbsp;&nbsp;&nbsp;&nbsp;这里所谓的“相机”实际上是相对于用户是视角，这里和iPhone背后的内置相机没任何关系。

图3.8显示了在Interface Builder内的一对视图，正如你所见，首先出现在视图层级绿色的视图被绘制在红色视图的后面。

<img src="./3.8.jpeg" alt="图3.8" title="图3.8" width="700"/>

图3.8 在视图层级中绿色视图被绘制在红色视图的后面

&nbsp;&nbsp;&nbsp;&nbsp;我们希望在真实的应用中也能显示出绘图的顺序，同样地，如果我们提高绿色视图的`zPosition`（清单3.3），我们会发现顺序就反了（图3.9）。其实并不需要增加太多，视图都非常地薄，所以给`zPosition`提高一个像素就可以让绿色视图前置，当然0.1或者0.0001也能够做到，但是最好不要这样，因为浮点类型四舍五入的计算可能会造成一些不便的麻烦。

清单3.3

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *greenView;
@property (nonatomic, weak) IBOutlet UIView *redView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    ￼
    //move the green view zPosition nearer to the camera
    self.greenView.layer.zPosition = 1.0f;
}
@end
```

<img src="./3.9.jpeg" alt="图3.9" title="图3.9" width="700"/>

图3.9 绿色视图被绘制在红色视图的前面