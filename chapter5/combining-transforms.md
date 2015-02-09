# 混合变换


Core Graphics提供了一系列的函数可以在一个变换的基础上做更深层次的变换，如果做一个既要*缩放*又要*旋转*的变换，这就会非常有用了。例如下面几个函数：

    CGAffineTransformRotate(CGAffineTransform t, CGFloat angle)
    CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy)
    CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)

当操纵一个变换的时候，初始生成一个什么都不做的变换很重要--也就是创建一个`CGAffineTransform`类型的空值，矩阵论中称作*单位矩阵*，Core Graphics同样也提供了一个方便的常量：

	CGAffineTransformIdentity

最后，如果需要混合两个已经存在的变换矩阵，就可以使用如下方法，在两个变换的基础上创建一个新的变换：

	CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);

我们来用这些函数组合一个更加复杂的变换，先缩小50%，再旋转30度，最后向右移动200个像素（清单5.2）。图5.4显示了图层变换最后的结果。

清单5.2 使用若干方法创建一个复合变换

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad]; //create a new transform
    CGAffineTransform transform = CGAffineTransformIdentity; //scale by 50%
    transform = CGAffineTransformScale(transform, 0.5, 0.5); //rotate by 30 degrees
    transform = CGAffineTransformRotate(transform, M_PI / 180.0 * 30.0); //translate by 200 points
    transform = CGAffineTransformTranslate(transform, 200, 0);
    //apply transform to layer
    self.layerView.layer.affineTransform = transform;
}
```

<img src="./5.4.jpeg" alt="图5.4" title="图5.4" width="700"/>

图5.4 顺序应用多个仿射变换之后的结果

图5.4中有些需要注意的地方：图片向右边发生了平移，但并没有指定距离那么远（200像素），另外它还有点向下发生了平移。原因在于当你按顺序做了变换，上一个变换的结果将会影响之后的变换，所以200像素的向右平移同样也被旋转了30度，缩小了50%，所以它实际上是斜向移动了100像素。

这意味着变换的顺序会影响最终的结果，也就是说旋转之后的平移和平移之后的旋转结果可能不同。
