# 最佳实践：keyWindow

> Created: 2023/08/28



#### 上下文

总有些东西会充斥在所有搜索引擎能搜到的平台上，并且所有人都宣称他是对的——别人是错的。自从 iOS 13 之后，``keyWindow`` 也加入到这一阵列里来，在 [StackOverflow](https://stackoverflow.com/questions/57134259) 上可以看到各路人马纷纷下场，包括 ``Mattt`` 之类的大神都吵得头破血流。总之，你需要先通读一遍这个链接里的高赞答案。



#### 目标

本文不会重新发起一个宣称，而是着力于指出各方案间的细微差别和各自局限，进而证明 ``windows`` 的管理本质上是业务特化的，并不存在一个通解。



#### UIApplication.shared.windows

是一种原初的兼容方案，在 iOS 15 惨遭废弃。



#### UIScene.keyWindow

仅 iOS 15.x 可用。



#### UIScene.activationState == .foregroundActive

先用 ``UIApplication.shared.connectedScenes`` 没有太多疑问，但是否需要做 filter？就我的实践来看是不能做的，因为刚刚冷启动/进后台/接电话/推送等场景都可能影响 state 状态，而 topMost 应当作为另一种不相干的业务来考虑。

例如，项目里可能需要获取 ``statusBarHeight`` 的绝对值，这是需要用到 scene 的，滤掉 foreground 状态在首页就会出问题，而判断其他状态问题就更大了。



#### windowLevel

也是关于是否需要做 filter 的考量，理由同上。层级是非常业务的。



#### Custom window for drift view

很多调试控件喜欢通过浮窗形式展现，而浮窗的直接实现就是通过 window，比如 [DoraemonKit](https://github.com/didi/DoKit) 是很难舍弃的；另外有一些特殊需求也会用到浮窗，这时候相当于 windows 发生了变化，where 查询自然失效，反而废弃的 ``UIApplication.shared.keyWindow`` 能取值正确。



#### .first vs .last

原楼层里也有相应的讨论，但在我看来本质上还是有场景使得 ``windows.count > 1`` 导致。如果会出现多个 window，就很难确保方案不会随着 Apple 版本更新或者业务变化而失效。



#### 我的看法

* 控制范围
    * 大面积出现 ``topMost`` 或者获取 ``window`` 的行为是不合理的。
* 自己封一层
    * ``URLNavigator`` 的 ``topMost`` 扩展相对流行一些，但最好是自己实现接口方便扩展。
* 不惮于 tag
    * 我的业务项目里确实会涉及多个浮窗，最终决定使用 tag 来管理。即使 ``keyWindow`` 本身的对象都发生了变化也没事。坏处自然是全局依赖，但这毕竟也属于业务。











