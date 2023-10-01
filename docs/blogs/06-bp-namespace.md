# 最佳实践：命名空间

> Created: 2023/08/29



## Intro

这里暂不讨论命名空间在业务项目中广泛使用，以及依赖解除等设计模式上的问题，仅集中在代码实现上。



#### 上下文

``KingFisher`` 提供了一种在 ``Swift`` 中实现命名空间隔离的非常好的工程实践，不少人包括我自己也会尝试仿照它的模式进行基础库的搭建，如果你不知道我在说什么，那么阅读 ``KingFisher`` 的源码和一系列博文是必须的。

无论如何， ``KingFisher`` 本身的扩展主要集中在图片处理上，要想将这个模式推广到整个业务中去，需要比库本身的开发更为谨慎。尽管实现源码非常简单，但搭建过程中依然有很多细节和思辨没有体现出来，这就导致我看到的很多仿写库存在着各种各样的误区。



#### 目标

对以 ``KingFisher`` 为标杆的命名空间业务模式展开讨论并改进。



## Chain

> 链式语法的设计。



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
    label.dtb.setText("test1")
    label.dtb.setText("test2")
    let w1 = label.dtb
	let w2 = label.dtb
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

进一步提问：如果扩展对象是数值类型呢？

相较于对象类型，基础类型本身的占用很小，使用又特别广泛，假如对基础类型做类似的扩展，从倍率来说相当于成倍地提升了空间消耗，所以越是基础的类型越要谨慎。



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

通过增加 ``@discardableResult`` 注解来保持一致性，同时简写回参：

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
    let ciImage: CIImage = thumb.dtb.ci()
    let color = ciImage.dtb.smartColor()
    
    let chainResult = image.dtb.scale(to: 20.0).dtb.ci().dtb.smartColor()
}
```

在设计时直接将 ``UIImage`` 对象返回符合直觉，但观察业务方的调用，如果不使用中间变量的话链式语法就过于冗长了。假如遵照前文思路，直接返回结构体本身呢？

```swift
let chains = image.dtb.scale(to: 20.0).ci().smartColor().done
```

再回过头去看，假如我只想使用单次结果时：

```swift
let ci01 = image.dtb.ci()
let ci02 = image.dtb.ci().done
let ci03: CIImage = image.dtb.ci()
```

综上所述，无论业务方需要单次还是多次调用，直接返回 ``Wrapper`` 对象都是更好的选择。



#### Chainable setter

很容易想到，链式语法的应用场景之一就是改写一系列的赋值语句，让业务看起来更为紧凑，而其中最容易改写的就是大部分的成员变量赋值，实现也很简单，不再赘述：

```swift
UIView().dtb.text("title").textColor(.gray).backgroundColor(.white)
```

除此之外，我们还希望能够在代码里显式地标明哪些类已经支持了 "Chain" 语法，这里存在着多种思路：



#### setter: @dynamicMemberLookup

具体参见 [Swift 5.1: @dynamicMemberLookup](https://zhuanlan.zhihu.com/p/415217937)，总而言之，按这个思路最终改造后的代码如下：

```swift
// [Style3] key-path
extension DTBKitWrapper where Base == UIView {
	@dynamicMemberLookup
    subscript<T>(dynamicMember keyPath: WritableKeyPath<Base, T>) -> ((T) -> (DTBKitChainWrapper<Base>)) {
        var n = me
        return { value in
            n[keyPath: keyPath] = value
            return DTBKitChainWrapper(n)
        }
    }
}
```

这么做当然有一堆问题，比如

* 理论上基于闭包的方法应该和自定义方法良好共存，但事实上并非如此；
* 结合闭包的结构体在内存管理上需要更多思考；
* 对系统类的扩展必然需要大量的自定义方法；
* Base 是 class 还是 struct 的兼容；

此思路作罢。



#### setter: another wrapper

另一种思路是拆成多种 "Wrapper"，链式语法只在新的 wrapper 内实现，并定义一系列的操作符用来转换：

```swift
// [Style2] another wrapper
public protocol DTBKitChainable {
    associatedtype ChainT
    var obj: ChainT { get }
}

extension DTBKitChainable {
    ///
    public var set: DTBKitChainWrapper<ChainT> {
        get { return DTBKitChainWrapper(obj) }
        set { }
    }
}

///
public struct DTBKitChainWrapper<Base> {
    internal let me: Base
    public init(_ value: Base) { self.me = value }
}

/// Syntax candy
extension DTBKitChainWrapper {
    public var then: DTBKitWrapper<Base> { return DTBKitWrapper(me) }
    public var unBox: Base { return me }
    public func done() {}
}
```

这种写法的好处是进一步隔离了各扩展方法，并且强制业务方使用语义调用：

```swift
UIView().dtb.set.text("title").textColor(.gray)
UIView().dtb.text("")  // syntax error!
```

但这同样会导致业务方无法混用，废话连词变多：

```swift
let res = UIImage().set.base64("123").dtb.zipTo(0.7).set.tintColor(.gray).value
```

业务层不应该感知具体的 wrapper，而是只注意返回值类型。



#### setter: any protocol

继续基于另起 wrapper 的思路往下看，原有的 wrapper 必须要有个方法来转换到新的 wrapper：

```swift
public struct DTBKitWrapper<Base> {
	public func set() -> Self where Self: DTBKitChainable { return self }
}
```

调用者需要带方法括号不太美观，能不能省去：

```swift
public var set: any DTBKitChainable { return self as DTBKitChainable }
```

再去查一下 ``any protocol`` 实现，可以知道这又是在内存管理上不太好的做法。既然业务和内存两方面都存在问题，那这个思路也走不通。

退而求其次，

* 依然通过新的 protocol 来标明哪些类支持相应操作；
* 不强求扩展方法之间互相隔离，但提供空白操作符给业务层用来标明语义；
* 特殊的公有方法可以直接在 protocol 里实现，利用 where 隔离；

业务方调用时，

```swift
UILabel().dtb.set.text("123")
UILabel().dtb.text("456")

UILabel().dtb.value.text
UILabel().dtb.get.text
```

都是等效的，业务方可以根据自己的习惯使用。



## Definition

> 内存管理与声明语义。



#### 先看一些与 struct 相关的知识储备

[二进制优化与 COW](https://juejin.cn/post/7191406877819797561)



#### me: let vs. var

考虑到语言本身就设计了 ``mutating`` 关键字来显式地表明对成员变量的修改，所以更直觉的选择是另起一个 Wrapper ：

```swift
public struct DTBKitMutableWrapper<Base> {
    internal var me: Base
    public init(_ base: Base) {
        self.me = base
    }
}
```



#### me: optional

在处理 ``me`` 成员变量本身业务时当然有可能出现各种错误：

```swift
extension DTBKitMutableWrapper where Base == String {
    func sub() {
      if let c = me.first(where: { $0 == "a" }) {
      	me = String(c)
      } else {
        // A. me = nil
        // B. me = ""
        // C. me = me
      }
    }
}
```

从业务角度来说 ABC 三种处理方案都说得通，但明显 A 方案需要修改声明为 `` var me: Base?``；而再进一步思考，就是如何将 ``throw`` 和 ``optional`` 也纳入链式语法设计中的问题了，先按下不表。



#### me: weak

考虑以下代码：

```swift
extension DTBKitMutableWrapper where Base: UIView {
	  @discardableResult
    public func removeFromSuperview() -> Self {
        me.removeFromSuperview()
        return self
    }
}
```

一般业务实践中基本上和置空操作在一起：

```swift
class Test {
	let buttons: [UIView] = [button01, button02, button03]
	
	func reset() {
		buttons.map{ $0.removeFromSuperView() }
		buttons.removeAll()
		// do sth.
	}
}
```

一番改造后的 Wrapper 可能类似于：

```swift
extension DTBKitMutableOptionalWrapper where Base: UIView {
	  @discardableResult
    public func removeFromSuperview() -> Self {
        me?.removeFromSuperview()
        me = nil
        return self
    }
}
```

那么如果不用 ``optional`` 和置空操作，``removeFromSuperview`` 等类似的函数会不会造成引用计数和内存方面的问题？``me`` 属性本身是否需要 ``weak`` 修饰呢？

[Swift-Regret-Weak-Vars-in-Structs](https://belkadan.com/blog/2021/12/Swift-Regret-Weak-Vars-in-Structs/#)



## Semantic Candy

> 创建语法糖来增强语义。




## Extension - where

> 泛型约束使用。



#### where: class vs. func

> 方法本身的约束...



#### where: Array

> 带泛型的类本身难以约束...



#### 难解的 static func

> 静态方法...



#### Property:  Singleton vs. Objc runtime

> 不得不需要添加属性时...



## Wrapper



#### me: let vs. var

> 处理基础类型/结构体/指针...

