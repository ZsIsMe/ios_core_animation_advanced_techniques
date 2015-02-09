# maskToBounds

&nbsp;&nbsp;&nbsp;&nbsp;现在我们的雪人总算是显示了正确的大小，不过你也许已经发现了另外一些事情：他超出了视图的边界。默认情况下，UIView仍然会绘制超过边界的内容或是子视图，在CALayer下也是这样的。

&nbsp;&nbsp;&nbsp;&nbsp;UIView有一个叫做`clipsToBounds`的属性可以用来决定是否显示超出边界的内容，CALayer对应的属性叫做`masksToBounds`，把它设置为YES，雪人就在边界里啦～（如图2.5）

![图2.5](./2.5.png)

图2.5 使用`masksToBounds`来修建图层内容