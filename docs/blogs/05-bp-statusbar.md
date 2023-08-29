# 最佳实践：statusBar

> Created: 2023/08/29



#### 前提

不考虑定制，屏幕旋转，pad 分屏等情况。



#### 上下文

你永远能在项目里看到诸如 ``ScreenWidth``，``StatusBarHeight``，``NavigationBarHeight`` 之类的东西，运气好的还有 ``isIPhoneX``。尽管五百年前就出现了 ``safeArea`` 的概念，但我很难在项目里看到标准的业务实践。



#### 目标

总而言之，本文尽量将讨论范围约束在 ``statusBar``，甚至只是 ``statusBarHeight`` 。网上大量的 ``safeArea`` 文章其实只介绍了安全区概念本身，和具体某一侧边缘的解决方案往往有差距。



### StatusBar 本身：Appearance

[随便一搜](https://juejin.cn/post/7083399385702203428) 就是，完善点的文章提得也比较全，这里稍微做个罗列。

#### NavigationBar.barStyle

只有使用原生导航栏时需要关注。

虽然在我看来大型项目依然使用原生导航栏本身就是问题，但这是另一个话题，略过。

#### Present

模态弹出时需要关注，纯业务。

#### Scene / Window

前面的链接里提到了他们碰到的一个相关问题，但我不认为是 ``window`` 本身层级，而是 SDK 业务代码导致，因为单纯自定义一个 ``window`` 加上去不会有问题，估计跟 ``window`` 上再挂载的 ``rootViewController`` 有关。

#### Hook

我本身就反对通过 ``hook`` 实现业务 UI，但的确存在一些诸如 [利用状态栏来实现网络状态检测](https://juejin.cn/post/6844903925678604295) 之类的特殊手段。



### StatusBar 相关：业务

你必须强迫自己去思考这项业务为什么必须要与状态栏本身的各种状态有所联系。

#### statusBarFrame

官方注释都提到状态栏隐藏时会返回 ``.zero``，但版本兼容后如果在 ``window.makeKeyAndVisable`` 方法调用前就去获取，结果依然是 ``.zero``，原因想来也不难理解。

#### AutoLayout

   ```swift
let father = UIView()
let son = UIView()

func setup() {
    [father, son].forEach { $0.translatesAutoresizingMaskIntoConstraints = false }
    
    if #available(iOS 11.0, *) {
      son.topAnchor.constraint(equalTo: father.safeAreaLayoutGuide.topAnchor, constant: 0.0).isActive = true
    } else {
      son.topAnchor.constraint(equalTo: father.topAnchor, constant: 0.0).isActive = true
    }
}
   ```

总之，需要最外层有一个整体的容器，由它来关注与处理安全区的间距差别。

也可以由内部撑开，营造一种类似于 ``padding`` 的效果。

这是 ``autoLayout`` 的原始写法，在 SDK 内就得这样写，换成 ``SnapKit`` 之类的语法糖思路也是一样的。

#### Frame

```swift
let father = UIView()
let son = UIView()

override var intrinsicContentSize: CGSize {
    return .zero  //
}

override func sizeThatFits(_ size: CGSize) -> CGSize {
    return .zero  // 
}

override func layoutSubviews() {
    super.layoutSubviews()
    
	if #available(iOS 11.0, *) {
        son.frame.origin.y = father.safeAreaInsets.top + 8.0
    } else {
        son.frame.origin.y = 8.0
    }
}
```

手动布局的最佳实践需要实现 ``layoutSubviews``， ``sizeThatFits:`` 和 ``intrinsicContentSize`` 三个方法，而其中是否需要关注 ``safeArea`` 由业务 UI 层级决定，不要弄混。

#### 状态栏背景：固定

如果是自定义导航栏的话，只需要业务 ``viewController`` 自行配合导航栏状态即可实现。

需要将自定义导航栏状态与状态栏的状态实现关联吗？

由于 ``preferredStatusBarStyle`` 是 ``override``，那要么在 ``style`` 的实现里与自定义导航栏对象发生关联，要么就需要 ``runtime`` 或者其他实现，所以总而言之我认为并无太大必要，因为一般不存在同个页面导航栏本身状态一直发生变化的业务场景。

#### 状态栏背景：动态

存在一些无边框设计，图片或者控件会直接顶到导航栏下方。

第一步，需要对状态栏的背景区域做截图或者重绘，拿到不包含状态栏本身 UI 的背景图片；注意不能是业务图片的整体色调或者色值，因为图片的一小部分可能就是很白或者很黑。

第二步，对这个特殊图片提主题色，拿到一个色值与纯白/纯黑比对，或者通过二值化等其他操作直接得出结论；最后做起来整体可能有一点麻烦，但并不难。























