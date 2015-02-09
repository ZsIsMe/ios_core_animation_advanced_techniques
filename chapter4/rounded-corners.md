# 圆角

&nbsp;&nbsp;&nbsp;&nbsp;圆角矩形是iOS的一个标志性审美特性。这在iOS的每一个地方都得到了体现，不论是主屏幕图标，还是警告弹框，甚至是文本框。按照这流行程度，你可能会认为一定有不借助Photoshop就能轻易创建圆角举行的方法。恭喜你，猜对了。

&nbsp;&nbsp;&nbsp;&nbsp;CALayer有一个叫做`conrnerRadius`的属性控制着图层角的曲率。它是一个浮点数，默认为0（为0的时候就是直角），但是你可以把它设置成任意值。默认情况下，这个曲率值只影响背景颜色而不影响背景图片或是子图层。不过，如果把`masksToBounds`设置成YES的话，图层里面的所有东西都会被截取。

&nbsp;&nbsp;&nbsp;&nbsp;我们可以通过一个简单的项目来演示这个效果。在Interface Builder中，我们放置一些视图，他们有一些子视图。而且这些子视图有一些超出了边界（如图4.1）。你可能无法看到他们超出了边界，因为在编辑界面的时候，超出的部分总是被Interface Builder裁切掉了。不过，你相信我就好了 :)

![图4.1](./4.1.png)

图4.1 两个白色的大视图，他们都包含了小一些的红色视图。

&nbsp;&nbsp;&nbsp;&nbsp;然后在代码中，我们设置角的半径为20个点，并裁剪掉第一个视图的超出部分（见清单4.1）。技术上来说，这些属性都可以在Interface Builder的探测板中分别通过『用户定义运行时属性』和勾选『裁剪子视图』(Clip Subviews)选择框来直接设置属性的值。不过，在这个示例中，代码能够表示得更清楚。图4.2是运行代码的结果

清单4.1 设置`cornerRadius`和`masksToBounds`

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;

@end

@implementation ViewController
- (void)viewDidLoad
{￼￼￼
  [super viewDidLoad];

  //set the corner radius on our layers
  self.layerView1.layer.cornerRadius = 20.0f;
  self.layerView2.layer.cornerRadius = 20.0f;

  //enable clipping on the second layer
  self.layerView2.layer.masksToBounds = YES;
}
@end
```

![图4.2](./4.2.png)

&nbsp;&nbsp;&nbsp;&nbsp;右图中，红色的子视图沿角半径被裁剪了

&nbsp;&nbsp;&nbsp;&nbsp;如你所见，右边的子视图沿边界被裁剪了。

&nbsp;&nbsp;&nbsp;&nbsp;单独控制每个层的圆角曲率也不是不可能的。如果想创建有些圆角有些直角的图层或视图时，你可能需要一些不同的方法。比如使用一个图层蒙板（本章稍后会讲到）或者是CAShapeLayer（见第六章『专用图层』）。
