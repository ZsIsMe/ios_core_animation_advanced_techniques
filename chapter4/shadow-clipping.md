# 阴影裁剪


&nbps;&nbsp;&nbsp;&nbsp;和图层边框不同，图层的阴影继承自内容的外形，而不是根据边界和角半径来确定。为了计算出阴影的形状，Core Animation会将寄宿图（包括子视图，如果有的话）考虑在内，然后通过这些来完美搭配图层形状从而创建一个阴影（见图4.7）。

![图4.7](./4.7.png)

图4.7 阴影是根据寄宿图的轮廓来确定的

&nbps;&nbsp;&nbsp;&nbsp;当阴影和裁剪扯上关系的时候就有一个头疼的限制：阴影通常就是在Layer的边界之外，如果你开启了`masksToBounds`属性，所有从图层中突出来的内容都会被才剪掉。如果我们在我们之前的边框示例项目中增加图层的阴影属性时，你就会发现问题所在（见图4.8）.

![图4.8](./4.8.png)

图4.8 `maskToBounds`属性裁剪掉了阴影和内容

&nbps;&nbsp;&nbsp;&nbsp;从技术角度来说，这个结果是可以是可以理解的，但确实又不是我们想要的效果。如果你想沿着内容裁切，你需要用到两个图层：一个只画阴影的空的外图层，和一个用`masksToBounds`裁剪内容的内图层。

&nbps;&nbsp;&nbsp;&nbsp;如果我们把之前项目的右边用单独的视图把裁剪的视图包起来，我们就可以解决这个问题（如图4.9）.

![图4.9](./4.9.png)

图4.9 右边，用额外的阴影转换视图包裹被裁剪的视图

&nbps;&nbsp;&nbsp;&nbsp;我们只把阴影用在最外层的视图上，内层视图进行裁剪。清单4.3是代码实现，图4.10是运行结果。

清单4.3 用一个额外的视图来解决阴影裁切的问题

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;
@property (nonatomic, weak) IBOutlet UIView *shadowView;

@end

@implementation ViewController
￼
- (void)viewDidLoad
{
  [super viewDidLoad];

  //set the corner radius on our layers
  self.layerView1.layer.cornerRadius = 20.0f;
  self.layerView2.layer.cornerRadius = 20.0f;

  //add a border to our layers
  self.layerView1.layer.borderWidth = 5.0f;
  self.layerView2.layer.borderWidth = 5.0f;

  //add a shadow to layerView1
  self.layerView1.layer.shadowOpacity = 0.5f;
  self.layerView1.layer.shadowOffset = CGSizeMake(0.0f, 5.0f);
  self.layerView1.layer.shadowRadius = 5.0f;

  //add same shadow to shadowView (not layerView2)
  self.shadowView.layer.shadowOpacity = 0.5f;
  self.shadowView.layer.shadowOffset = CGSizeMake(0.0f, 5.0f);
  self.shadowView.layer.shadowRadius = 5.0f;

  //enable clipping on the second layer
  self.layerView2.layer.masksToBounds = YES;
}

@end
```

![图4.10](./4.10.png)

图4.10 右边视图，不受裁切阴影的阴影视图。