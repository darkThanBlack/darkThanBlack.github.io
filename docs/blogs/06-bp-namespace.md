# 最佳实践：命名空间

> Created: 2023/08/29



#### 上下文

``KingFisher`` 提供了一种在 ``Swift`` 中实现命名空间隔离的非常好的工程实践，不少人包括我自己也会尝试仿照它的模式进行基础库的搭建，如果你不知道我在说什么，那么阅读 ``KingFisher`` 的源码和一系列博文是必须的。



#### 目标

尽管实现源码非常简单，但搭建过程中依然有很多细节和思辨是没有体现出来的，这就导致我看到的很多仿写库存在着各种各样的误区。



#### 一个典型的 Wrapper

```swift
public struct DTBKitWrapper<Base> {
    public let me: Base
    public init(_ base: Base) {
        self.me = base
    }
}
```

