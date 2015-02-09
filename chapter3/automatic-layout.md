# 自动布局


&nbsp;&nbsp;&nbsp;&nbsp;你可能用过`UIViewAutoresizingMask`类型的一些常量，应用于当父视图改变尺寸的时候，相应`UIView`的`frame`也跟着更新的场景（通常用于横竖屏切换）。

&nbsp;&nbsp;&nbsp;&nbsp;在iOS6中，苹果介绍了*自动排版*机制，它和自动调整不同，并且更加复杂。

&nbsp;&nbsp;&nbsp;&nbsp;在Mac OS平台，`CALayer`有一个叫做`layoutManager`的属性可以通过`CALayoutManager`协议和`CAConstraintLayoutManager`类来实现自动排版的机制。但由于某些原因，这在iOS上并不适用。

&nbsp;&nbsp;&nbsp;&nbsp;当使用视图的时候，可以充分利用`UIView`类接口暴露出来的`UIViewAutoresizingMask`和`NSLayoutConstraint`API，但如果想随意控制`CALayer`的布局，就需要手工操作。最简单的方法就是使用`CALayerDelegate`如下函数：

    - (void)layoutSublayersOfLayer:(CALayer *)layer;

&nbsp;&nbsp;&nbsp;&nbsp;当图层的`bounds`发生改变，或者图层的`-setNeedsLayout`方法被调用的时候，这个函数将会被执行。这使得你可以手动地重新摆放或者重新调整子图层的大小，但是不能像`UIView`的`autoresizingMask`和`constraints`属性做到自适应屏幕旋转。

&nbsp;&nbsp;&nbsp;&nbsp;这也是为什么最好使用视图而不是单独的图层来构建应用程序的另一个重要原因之一。
