# contentsScale

&nbsp;&nbsp;&nbsp;&nbsp;`contentsScale`属性定义了寄宿图的像素尺寸和视图大小的比例，默认情况下它是一个值为1.0的浮点数。

&nbsp;&nbsp;&nbsp;&nbsp;`contentsScale`的目的并不是那么明显。它并不是总会对屏幕上的寄宿图有影响。如果你尝试对我们的例子设置不同的值，你就会发现根本没任何影响。因为`contents`由于设置了`contentsGravity`属性，所以它已经被拉伸以适应图层的边界。

&nbsp;&nbsp;&nbsp;&nbsp;如果你只是单纯地想放大图层的`contents`图片，你可以通过使用图层的`transform`和`affineTransform`属性来达到这个目的（见第五章『Transforms』，里面对此有解释），这(指放大)也不是`contengsScale`的目的所在.

&nbsp;&nbsp;&nbsp;&nbsp;`contentsScale`属性其实属于支持高分辨率（又称Hi-DPI或Retina）屏幕机制的一部分。它用来判断在绘制图层的时候应该为寄宿图创建的空间大小，和需要显示的图片的拉伸度（假设并没有设置`contentsGravity`属性）。UIView有一个类似功能但是非常少用到的`contentScaleFactor`属性。

&nbsp;&nbsp;&nbsp;&nbsp;如果`contentsScale`设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片，这就是我们熟知的Retina屏幕。（如果你对像素和点的概念不是很清楚的话，这个章节的后面部分将会对此做出解释）。

&nbsp;&nbsp;&nbsp;&nbsp;这并不会对我们在使用kCAGravityResizeAspect时产生任何影响，因为它就是拉伸图片以适应图层而已，根本不会考虑到分辨率问题。但是如果我们把`contentsGravity`设置为kCAGravityCenter（这个值并不会拉伸图片），那将会有很明显的变化（如图2.3）

![图2.3](./2.3.png)

图2.3 用错误的`contentsScale`属性显示Retina图片

&nbsp;&nbsp;&nbsp;&nbsp;如你所见，我们的雪人不仅有点大还有点像素的颗粒感。那是因为和UIImage不同，CGImage没有拉伸的概念。当我们使用UIImage类去读取我们的雪人图片的时候，他读取了高质量的Retina版本的图片。但是当我们用CGImage来设置我们的图层的内容时，拉伸这个因素在转换的时候就丢失了。不过我们可以通过手动设置`contentsScale`来修复这个问题（如2.2清单），图2.4是结果

```objective-c
@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad]; //load an image
  UIImage *image = [UIImage imageNamed:@"Snowman.png"]; //add it directly to our view's layer
  self.layerView.layer.contents = (__bridge id)image.CGImage; //center the image
  self.layerView.layer.contentsGravity = kCAGravityCenter;

  //set the contentsScale to match image
  self.layerView.layer.contentsScale = image.scale;
}

@end
```

![图2.4](./2.4.png)

图2.4 同样的Retina图片设置了正确的`contentsScale`之后

&nbsp;&nbsp;&nbsp;&nbsp;当用代码的方式来处理寄宿图的时候，一定要记住要手动的设置图层的`contentsScale`属性，否则，你的图片在Retina设备上就显示得不正确啦。代码如下：

```objective-c
layer.contentsScale = [UIScreen mainScreen].scale;
```
