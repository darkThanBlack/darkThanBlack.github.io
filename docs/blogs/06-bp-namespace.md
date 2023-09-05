# 最佳实践：命名空间

> Created: 2023/08/29



#### 前提

这里暂不讨论命名空间在业务项目中广泛使用，以及依赖解除等设计模式上的问题，仅集中在代码实现上。



#### 上下文

``KingFisher`` 提供了一种在 ``Swift`` 中实现命名空间隔离的非常好的工程实践，不少人包括我自己也会尝试仿照它的模式进行基础库的搭建，如果你不知道我在说什么，那么阅读 ``KingFisher`` 的源码和一系列博文是必须的。



#### 目标

无论如何， ``KingFisher`` 本身的扩展主要集中在图片处理上，要想将这个模式推广到整个业务中去，需要比库本身的开发更为谨慎。尽管实现源码非常简单，但搭建过程中依然有很多细节和思辨没有体现出来，这就导致我看到的很多仿写库存在着各种各样的误区。



#### 一个典型的 Wrapper

```swift
public struct DTBKitWrapper<Base> {
    public let me: Base
    public init(_ base: Base) {
        self.me = base
    }
}
```

首先，``struct`` 的创建一定会带来额外消耗，具体差别可以查看相关博文；尽管这额外的开销相较于优点来说值得承受，但这并不意味着扩展结构就可以被滥用，尤其是想要推广到整个业务的时候，调用量一定是远大于仅处理图片业务的，所以在接口层我们需要设法限制不必要的 ``struct`` 创建。基于这个思路，先来看一个符合直觉的接口设计。



#### 一个典型的 func，只改变引用对象的属性

```swift
extension DTBKitWrapper where Base: UILabel {
    public func setText(_ value: String?) {
		me.text = value
    }
}

func main() {
    let label = UILabel()
    label.text = "test"
    label.dtb.setText("test")
}
```

看起来好像没有任何问题，我们再增加一下调用，并观察调用者。

```swift
func main() {
    let label = UILabel()
    label.dtb.setText("test1")  // let w1 = label.dtb
    label.dtb.setText("test2")  // let w2 = label.dtb
}
```

提问：w1 和 w2 的地址是否相同？他们在内存中是如何分布的？

``me: Base`` 确实保证了 ``Wrapper`` 引用的是同一个对象，但结构体本身并没有复用。



```swift
func main() {
    let w1 = Int64(1).dtb
    let w2 = Int64(2).dtb
}
```

提问：如果扩展对象是数值类型呢？

相较于对象类型，基础类型本身的占用很小，使用又特别广泛，引入的结构体占用甚至比类型本身的占用还大，从倍率来说相当于成倍地提升了空间消耗，所以越是基础的类型反而越要慎用。



#### 改进版 func

```swift
extension DTBKitWrapper where Base: UILabel {
    public func setText(_ value: String?) -> DTBKitWrapper<Base> {
		me.text = value
        return self
    }
}

func main() {
    let label = UILabel()
    	.dtb
    	.setText("test01")
    	.setText("test02")
    	.me
}
```

直接返回 ``Wrapper`` 对象来鼓励业务方使用链式语法，但也有缺点：这会强制调用者访问 ``me`` 属性来获得最终的引用，并且在只想更新某个对象的单个属性时让调用变得有些奇怪：

```swift
func update(label: UILabel) {
    let _ = label.dtb.setText("test")
}
```

可以通过增加 ``@discardableResult`` 注解来保持一致性，同时简写回参：

```swift
extension DTBKitWrapper where Base: UILabel {
    @discardableResult
    public func setText(_ value: String?) -> Self {
        me.text = value
        return self
    }
}
```



#### me: public vs. internal

将所有 extension 方法视为一个三方库，在前文的改造之后，外部业务必然出现大量对 ``me`` 属性的调用，这会使得我们难以区分对相应属性的修改来自于内部还是外部，所以我更倾向于将 ``me`` 看成是库的私有属性进行处理，同时另行暴露外部接口。

```swift
public struct DTBKitWrapper<Base> {
    let me: Base
    public var done: Base {
        return me
    }
    public init(_ base: Base) {
        self.me = base
    }
}

func main() {
    let label = UILabel().dtb.setText("A").done
}
```



#### 一个典型的 func，返回一个类型相同的新对象

```swift
extension DTBKitWrapper where Base: UIImage {
    /// Down sampling
    public func scale(to value: CGFloat) -> UIImage? {
		//do sth.
		return UIImage(cgImage: cg, scale: me.scale, orientation: me.imageOrientation)
    }
}
```

图片[下采样](https://juejin.cn/post/6844903988077281288#heading-2)必然会生成一个新的图片对象，如果需要后续操作，那必然需要重新创建新对象对应的结构体：

```swift
func main() {
	let image = UIImage(named: "test")
    let thumb = image.dtb.scale(to: 20.0)
    let other01: CIImage = thumb.dtb.ci()
    let other02 = image.dtb.scale(to: 20.0).dtb.ci().dtb.smartColor()
}
```
