# 布局

&nbsp;&nbsp;&nbsp;&nbsp;`UIView`有三个比较重要的布局属性：`frame`，`bounds`和`center`，`CALayer`对应地叫做`frame`，`bounds`和`position`。为了能清楚区分，图层用了“position”，视图用了“center”，但是他们都代表同样的值。

&nbsp;&nbsp;&nbsp;&nbsp;`frame`代表了图层的外部坐标（也就是在父图层上占据的空间），`bounds`是内部坐标（{0, 0}通常是图层的左上角），`center`和`position`都代表了相对于父图层`anchorPoint`所在的位置。`anchorPoint`的属性将会在后续介绍到，现在把它想成图层的中心点就好了。图3.1显示了这些属性是如何相互依赖的。

<img src="./3.1.jpeg" alt="图3.1" title="图3.1" width="700"/>

图3.1 `UIView`和`CALayer`的坐标系

&nbsp;&nbsp;&nbsp;&nbsp;视图的`frame`，`bounds`和`center`属性仅仅是*存取方法*，当操纵视图的`frame`，实际上是在改变位于视图下方`CALayer`的`frame`，不能够独立于图层之外改变视图的`frame`。

&nbsp;&nbsp;&nbsp;&nbsp;对于视图或者图层来说，`frame`并不是一个非常清晰的属性，它其实是一个虚拟属性，是根据`bounds`，`position`和`transform`计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值

&nbsp;&nbsp;&nbsp;&nbsp;记住当对图层做变换的时候，比如旋转或者缩放，`frame`实际上代表了覆盖在图层旋转之后的整个轴对齐的矩形区域，也就是说`frame`的宽高可能和`bounds`的宽高不再一致了（图3.2）

<img src="./3.2.jpeg" alt="图3.2" title="图3.2" width="700"/>

图3.2 旋转一个视图或者图层之后的`frame`属性

