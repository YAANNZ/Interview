### UIView和CALayer的关系
1. 表面区别
* CALayer 基于 QuartzCore 框架，UIView 基于 UIKit 框架
* CALayer是直接继承自NSObject的,而UIView是直接继承自UIResponder的。

当我们展示出来的东西需要实现和用户交互的时候去使用UIView、而不需要的交互的时候CALayer和UIView都可以。
由于CALayer不需要处理交互事件、所以是轻量级的、性能要比UIView高。

2. View和CALayer的Frame映射及View如何创建CALayer
* 一个 Layer 的 frame 是由它的 anchorPoint,position,bounds,和 transform 共同决定的，而一个 View 的 frame 只是简单的返回 Layer的 frame，同样 View 的 center和 bounds 也是返回 Layer 的position和bounds。

3. UIView主要是对显示内容的管理而 CALayer 主要侧重显示内容的绘制
* UIView 是 CALayer 的CALayerDelegate，大概是在代理方法内部[UIView(CALayerDelegate) drawLayer:inContext]调用 UIView 的 DrawRect方法，从而绘制出了 UIView 的内容。



### 总结
1. 每个 UIView 内部都有一个 CALayer 在背后提供内容的绘制和显示，并且 UIView 的尺寸样式都由内部的 Layer 所提供。两者都有树状层级结构，layer 内部有 SubLayers，View 内部有 SubViews.但是 Layer 比 View 多了个AnchorPoint
2. 在 View显示的时候，UIView 做为 Layer 的 CALayerDelegate,View 的显示内容由内部的 CALayer 的 display
3. CALayer 是默认修改属性支持隐式动画的，在给 UIView 的 Layer 做动画的时候，View 作为 Layer 的代理，Layer 通过 actionForLayer:forKey:向 View请求相应的 action(动画行为)
4. layer 内部维护着三分 layer tree,分别是 presentLayer Tree(动画树),modeLayer Tree(模型树), Render Tree (渲染树),在做 iOS动画的时候，我们修改动画的属性，在动画的其实是 Layer 的 presentLayer的属性值,而最终展示在界面上的其实是提供 View的modelLayer
5. 两者最明显的区别是 View可以接受并处理事件，而 Layer 不可以

[参考1](https://www.jianshu.com/p/079e5cf0f014)<br>
[参考2](https://www.wncblog.top/posts/8da6bc0c/)


#### 代码示例
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.aView = [[UIView alloc] init];
    self.aLayer = [[CALayer alloc] init];
    
    //比较UIView对象个CALayer对象在内存中的大小
    NSLog(@"self.aView对象实际需要的内存大小: %zd", class_getInstanceSize([self.aView class]));//480
    NSLog(@"self.aView对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(self.aView)));//480
    NSLog(@"self.aLayer对象实际需要的内存大小: %zd", class_getInstanceSize([self.aLayer class]));//24
    NSLog(@"self.aLayer对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(self.aLayer)));//32

    NSLog(@"------");
    //针对于UILabel 和 CATextLayer内存对比
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(30, 300, 300, 44)];
    label.text = @"label";
    [self.view addSubview:label];
    
    CATextLayer *textLayer = [[CATextLayer alloc] init];
    textLayer.frame = CGRectMake(30, 400, 300, 44);
    textLayer.string = @"CATextLayer";
    [self.view.layer addSublayer:textLayer];
    
    //比较UILabel对象个CATextLayer对象在内存中的大小
    NSLog(@"label对象实际需要的内存大小: %zd", class_getInstanceSize([label class]));//744
    NSLog(@"label对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(label)));//752
    NSLog(@"textLayer对象实际需要的内存大小: %zd", class_getInstanceSize([textLayer class]));//32
    NSLog(@"textLayer对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(textLayer)));//32
    
    NSLog(@"------");
    //添加了一些设置, 比如字体大小, 颜色, 换行, 字体长度变化
    label.text = @"123456789012345678901234567890";
    label.font = [UIFont fontWithName:@"PingFangSC-Regular" size:12];
    label.textColor = [UIColor redColor];
    
    textLayer.string = @"123456789012345678901234567890";
    textLayer.font =  (__bridge CFTypeRef _Nullable)(@"PingFangSC-Regular");
    textLayer.fontSize = 12;
    textLayer.foregroundColor = [UIColor redColor].CGColor;
    //比较UILabel对象个CATextLayer对象在内存中的大小
    NSLog(@"label对象实际需要的内存大小: %zd", class_getInstanceSize([label class]));//744
    NSLog(@"label对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(label)));//752
    NSLog(@"textLayer对象实际需要的内存大小: %zd", class_getInstanceSize([textLayer class]));//32
    NSLog(@"textLayer对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(textLayer)));//32
}
```

打印结果

```
2019-09-12 11:06:33.017943+0800 Demo[7030:27412457] self.aView对象实际需要的内存大小: 480
2019-09-12 11:06:33.018123+0800 Demo[7030:27412457] self.aView对象实际分配的内存大小: 480
2019-09-12 11:06:33.018249+0800 Demo[7030:27412457] self.aLayer对象实际需要的内存大小: 24
2019-09-12 11:06:33.018367+0800 Demo[7030:27412457] self.aLayer对象实际分配的内存大小: 32
2019-09-12 11:06:33.018478+0800 Demo[7030:27412457] ------
2019-09-12 11:06:33.019261+0800 Demo[7030:27412457] label对象实际需要的内存大小: 744
2019-09-12 11:06:33.019357+0800 Demo[7030:27412457] label对象实际分配的内存大小: 752
2019-09-12 11:06:33.019443+0800 Demo[7030:27412457] textLayer对象实际需要的内存大小: 32
2019-09-12 11:06:33.019542+0800 Demo[7030:27412457] textLayer对象实际分配的内存大小: 32
2019-09-12 11:06:33.019651+0800 Demo[7030:27412457] ------
2019-09-12 11:06:33.023849+0800 Demo[7030:27412457] label对象实际需要的内存大小: 744
2019-09-12 11:06:33.024001+0800 Demo[7030:27412457] label对象实际分配的内存大小: 752
2019-09-12 11:06:33.024106+0800 Demo[7030:27412457] textLayer对象实际需要的内存大小: 32
2019-09-12 11:06:33.085927+0800 Demo[7030:27412457] textLayer对象实际分配的内存大小: 32
```

* 可以看得出来, 在对象的内存分配中, CATextLayer的内存大小明显要比UIView要轻量的多.
* 对于一些只是用来展示的控件,比如是UIlabel,或者是ImageView. 完全可以用Layer来代替.

### 参考
* [参考](https://www.jianshu.com/p/41c34316444d)
* [详细介绍](https://www.cnblogs.com/mawenqiangios/p/5884960.html)




