## 异步绘制


&nbsp;&nbsp;&nbsp;&nbsp;UIKit的单线程天性意味着寄宿图通畅要在主线程上更新，这意味着绘制会打断用户交互，甚至让整个app看起来处于无响应状态。我们对此无能为力，但是如果能避免用户等待绘制完成就好多了。

&nbsp;&nbsp;&nbsp;&nbsp;针对这个问题，有一些方法可以用到：一些情况下，我们可以推测性地提前在另外一个线程上绘制内容，然后将由此绘出的图片直接设置为图层的内容。这实现起来可能不是很方便，但是在特定情况下是可行的。Core Animation提供了一些选择：`CATiledLayer`和`drawsAsynchronously`属性。

###CATiledLayer

&nbsp;&nbsp;&nbsp;&nbsp;我们在第六章简单探索了一下`CATiledLayer`。除了将图层再次分割成独立更新的小块（类似于脏矩形自动更新的概念），`CATiledLayer`还有一个有趣的特性：在多个线程中为每个小块同时调用`-drawLayer:inContext:`方法。这就避免了阻塞用户交互而且能够利用多核心新片来更快地绘制。只有一个小块的`CATiledLayer`是实现异步更新图片视图的简单方法。

###drawsAsynchronously

&nbsp;&nbsp;&nbsp;&nbsp;iOS 6中，苹果为`CALayer`引入了这个令人好奇的属性，`drawsAsynchronously`属性对传入`-drawLayer:inContext:`的CGContext进行改动，允许CGContext延缓绘制命令的执行以至于不阻塞用户交互。

&nbsp;&nbsp;&nbsp;&nbsp;它与`CATiledLayer`使用的异步绘制并不相同。它自己的`-drawLayer:inContext:`方法只会在主线程调用，但是CGContext并不等待每个绘制命令的结束。相反地，它会将命令加入队列，当方法返回时，在后台线程逐个执行真正的绘制。

&nbsp;&nbsp;&nbsp;&nbsp;根据苹果的说法。这个特性在需要频繁重绘的视图上效果最好（比如我们的绘图应用，或者诸如`UITableViewCell`之类的），对那些只绘制一次或很少重绘的图层内容来说没什么太大的帮助。