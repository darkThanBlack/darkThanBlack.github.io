# 最佳实践：命名空间

> Created: 2023/08/29
>
> Update: 2024/08/08    增补属性二次拆箱与 MutableWrapper 的说明



## Intro

本文主要是对 [DTBKit](https://github.com/darkThanBlack/DTBKit) 中 ``Base`` 和 ``Chain`` 模块的设计思路进行阐释。

这里暂不讨论命名空间在业务项目中广泛使用，以及依赖解除等设计模式上的问题，仅集中在代码实现上。



#### 上下文

``KingFisher`` 提供了一种在 ``Swift`` 中实现命名空间隔离的非常好的工程实践，不少人包括我自己也会尝试仿照它的模式进行基础库的搭建，如果你不知道我在说什么，那么阅读 ``KingFisher`` 的源码和一系列博文是必须的。

无论如何， ``KingFisher`` 本身的扩展主要集中在图片处理上，要想将这个模式推广到整个业务中去，需要比库本身的开发更为谨慎。尽管实现源码非常简单，但搭建过程中依然有很多细节和思辨没有体现出来，这就导致我看到的很多仿写库存在着各种各样的误区。



#### 目标

对以 ``KingFisher`` 为标杆的命名空间业务模式展开讨论并改进。



## Chain

> 链式语法的设计。



#### 一个典型的 func

先来看一个最符合直觉的接口设计。

```swift
public struct DTBKitWrapper<Base> {
    public let me: Base
    public init(_ base: Base) {
        self.me = base
    }
}

extension DTBKitWrapper where Base: UILabel {
    /// 新功能: 给 label 赋值
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
    label.dtb.setTextColor(.white)
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

进一步提问：如果被扩展的对象是值类型呢？

**结论：**

* 参照 ``KingFisher``，我们通过使用 ``wrapper`` 的方式，用户必须先调用 ``.dtb`` 才能访问到相关的自定义方法，也就是说成功地将方法作用域限制在了一个所谓的 "命名空间" 内。

* 但，每次用户调用 ``.dtb`` 方法，都会创建一个新的 ``wrapper``。虽然 ``struct`` 本身不应该有任何性能问题，但相较于如此大规模的调用次数，直觉上依然有必要加以控制。

**扩展结论：**

* 实际上，KF 的作者在最初对这一模式的性能影响确实有过调研，详情参见他的博客；尽管这额外的开销相较于优点来说值得承受，但这并不意味着扩展结构就可以被滥用，尤其是要推广到整个业务的时候，所以在设计过程中就应该设法引导开发者降低不必要的 ``struct`` 创建次数。
* 相较于引用类型，基础类型（数字，字符串等）本身使用就特别广泛，假如对基础类型做类似的扩展，从倍率来说相当于成倍地提升了空间消耗，越是基础的类型越要谨慎。



#### 改进版 func，支持链式调用

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
    	.setTextColor(.white)
    	.me
}
```

直接返回 ``Wrapper`` 对象来鼓励业务方使用链式语法，但也有缺点：这会强制调用者访问 ``me`` 属性来获得最终的引用，并且在只想更新某个对象的单个属性时让调用变得有些奇怪：

```swift
let _ = label.dtb.setText("test")
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

/// 编译器不会再提示警告了
label.dtb.setText("test")
```



#### 一个 func，需要返回一个类型相同的新对象时

```swift
extension DTBKitWrapper where Base: UIImage {
    /// Down sampling
    public func scale(to value: CGFloat) -> UIImage? {
		//do sth.
		return UIImage(cgImage: cg, scale: me.scale, orientation: me.imageOrientation)
    }
}
```

图片 [下采样](https://juejin.cn/post/6844903988077281288#heading-2) 必然会生成一个新的图片对象，如果需要后续操作，那必然需要重新创建新对象对应的结构体：

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
let chains = image.dtb.scale(to: 20.0).ci().smartColor().value
```

再回过头去看，假如我只想使用单次结果时：

```swift
let ci01 = image.dtb.ci()
let ci02 = image.dtb.ci().value
let ci03: CIImage = image.dtb.ci()
```

如果业务方只需要单次调用，那么由业务方多执行一次解包，虽然稍显啰嗦，但长度有限。



#### 一个 func，需要返回其他类型的对象时

这时其实是强业务相关，因为业务方拿到返回值后可能需要：

1. 返回值与当前对象无关；
2. 对返回值进行处理后，更新当前对象；
3. 对返回的其他类型进一步处理；
4. 以上情况兼有。

在设计上应当意识到，业务是**连续**的，所以应该让业务方通过拆箱方法获取原始值后自行调用，举个例子：

```swift
@discardableResult
public func sizeThatFits(_ size: CGSize, setter: ((_ base: Base, _ result: CGSize) -> Void)) -> Self {
    setter(me, me.sizeThatFits(size))
    return self
}

@discardableResult
public func layer(_ handler: ((DTBKitWrapper<CALayer>) -> Void)?) -> Self {
    handler?(me.layer.dtb)
    return self
}

@discardableResult
public func layer02(_ handler: ((DTBKitWrapper<CALayer>) -> Void)?) -> Self {
    handler?(me.layer.dtb)
    return self
}

func main() {
    let titleLabel = UILabel()
    let detailLabel = UILabel()
    
	// 情况 1, 拿返回值做其他事
    // 此时观察 tSize01 vs. tSize02，两者对业务方来说差别不大
    let tSize01 = titleLabel.sizeThatFits(bounds)
    let tSize02 = titleLabel.dtb.sizeThatFits(bounds).value
    detailLabel.frame = CGRect(
        x: tSize02.maxX, 
        y: tSize01.minY, 
        width: 10.0, 
        height: 10.0
    )
    
    // 情况 2, 需要拿返回值（size）继续更新 titleLabel 本身
    let tSize01 = titleLabel.sizeThatFits(bounds)
	titleLabel.frame = CGRect(origin: .zero, size: tSize01)
    // 等价于:
    titleLabel.dtb.sizeThatFits(bounds, setter: { value, size in
        value.frame = CGRect(origin: .zero, size: size)
    })
    // 是否提供 setter 差别依然不大，唯一好处是允许后续继续调用，但这可以通过某些通用方法来封装
    
    // 情况 3, 进一步处理
    detailLabel.layer.masksToBounds = true
    detailLabel.layer.cornerRadius = 5.0
    // 等价于:
    detailLabel.dtb
        .layer {
            $0.masksToBounds = true
            $0.cornerRadius = 5.0
		}
    	.sizeThatFits(bounds)
}
```

**结论：**

* 在需要返回**引用类型**的情况下，一般保持返回包装后的对象是更好的选择。



## Explore

> 列举调研过的方案。



#### explore: convenience init

把所有属性整合在一个函数里：

```swift
extension UILabel {
	
    convenience init(text: String? = nil, textColor: UIColor? = .white) {
        self.init(frame: .zero)
        self.text = text
        self.textColor = textColor
        // etc...
	}
}
```

一般我会将以下代码封装成 Xcode 代码片段用以代替：

```swift
private lazy var titleLabel = {
	let view = UILabel()
	view.text = "title"
	view.textColor = .white
	return view
}()
```

**结论：**

* 我有点懒得详细解释这种模式的劣势，这里只提一个点，在基于 Xcode 15.3 的开发过程中，函数的参数大约在超过 25 个的时候会被编译器禁止（类型推断超过了合理时间）。

**扩展结论：**

* 在处理 struct 的时候又会不得不回过头来考虑这种模式，参见后文说明。



#### explore: @dynamicMemberLookup

参见 [Swift 5.1: @dynamicMemberLookup](https://zhuanlan.zhihu.com/p/415217937)

先看微调后的核心代码：

```swift
@dynamicMemberLookup
struct DTBKitWrapper<Base> {
    
    subscript<Value>(dynamicMember keyPath: WritableKeyPath<Base, Value>) -> ((Value) -> DTBKitWrapper<Base>) {
        var subject = self.me
        return { value in
            subject[keyPath: keyPath] = value
            // 避免多次创建
            // return DTBKitWrapper(subject)
            return self
        }
    }
}
```

优势：

* 如文中所说，可以避免大量体力活
* 系统 API 变化或新增时可以直接兼容
* 对自定义类同样有效

劣势：

* 只能对属性生效
* 无法区分 get only，open，继承等语义
* IDE 自动完成的时候括号不会补全，而且方法容易变白
* ~~理论上基于闭包的方法应该和自定义方法良好共存，但事实上并非如此~~
* ~~结合闭包的结构体在内存管理上需要更多思考~~
* ~~对系统类的扩展必然需要大量声明~~

~~此思路作罢。~~

**结论**：作为引用类型的补充。



#### explore: another wrapper

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

业务层不需要感知具体的 wrapper，只注意返回值类型。



#### explore: any protocol

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
* 不强求扩展方法之间互相隔离，~~但提供空白操作符给业务层用来标明语义~~；
* 特殊的公有方法可以直接在 protocol 里实现，利用 where 隔离；

业务方调用：

```swift
UILabel().dtb.set.text("123")
UILabel().dtb.text("456")

UILabel().dtb.value.text
UILabel().dtb.get.text
```

**结论**：设计不同的 Wrapper 对象有它的用处，但没必要搞一堆无意义的关键字出来。



## Struct

> 处理结构体时需要额外考虑很多东西。



当 ``me`` 是一个 ``struct`` 的时候麻烦程度直线上升，因为修改 ``me`` 会受到限制，同时其他 ``protocol`` 相关的语法也需要调整。

第一反应的写法是 ``mutating``：

```swift
extension DTBKitMutableWrapper where Base == CGSize {
    @discardableResult
    public mutating func width(_ value: CGFloat) -> Self where Base: DTBKitChainable {
        me.width = value
        return self
    }
}

let nSize = CGSize.zero
nSize.width = 2.0
nSize.dtb.height(3.0)
```

然而，编译器会禁止如下写法：

```swift
let result = CGSize.zero.dtb.width(1.0).height(2.0).value  // build error
```

因为 ``wrapper`` 本身也是结构体，需要先由外层持有：

```swift
var size: CGSize = .zero
size.dtb.width(1.0)
```

这样的话用处就不大了，应该直接从 struct 的创建入手：

```swift
extension DTBKitStaticWrapper where T == CGSize {
    public func create(width: CGFloat, height: CGFloat) -> T {
        return CGSize(width: width, height: height)
    }
}
```

但 struct 本身有一个根据属性自动生成的 init 方法，属性完全相同的 ``create`` 方法唯一用处就只有省却参数：

```swift
extension DTBKitStaticWrapper where T == CGSize {
    /// 隐去参数说明
    public func create(_ width: CGFloat, _ height: CGFloat) -> T {
        return CGSize(width: width, height: height)
    }
}

// 等效
let edge1 = UIEdgeInsets(top: 0, left: 2, bottom: 0, right: 0)
let edge2 = UIEdgeInsets.dtb.create(left: 2)
let edge3 = UIEdgeInsets(0, 2, 0, 0)
```

还有另一种思路，即通过 class 进行转换，核心代码如下：

```swift
public struct DTBKitStaticWrapper<T> {}

extension DTBKitStaticWrapper where T: DTBKitStructable & DTBKitStructChainable {
    
    public var create: DTBKitMutableWrapper<T> {
        return DTBKitMutableWrapper(T.def_())
    }
}

public class DTBKitMutableWrapper<Base> {
    internal var me: Base
    public init(_ value: Base) { self.me = value }
}

extension Dictionary: DTBKitStructable, DTBKitStructChainable {
    public static func def_() -> Self {
        return [:]
    }
}
```

这种方案最大限度地保留了业务层语法的一致性，但也有非常致命的问题：

* 需要额外创建一个 class
* ``def_`` 方法无法对外隐藏

所以，一般的 struct 不应实现这个接口，它目前最大的用处在于将 key 值不定的字典构建转化成链式。

举个例子，平时的富文本字典使用存在以下痛点：

* 字典 value 都是 Any，缺乏类型推断检查；
* 字典 key 值都是 static let，往往还可能分散在各个 extension 中，只能看着文档去找。

在应用上述模式后，这两个问题都可以得到完美解决：

```swift
let attr = NSAttributedString(
    		string: "str",
    		attributes: .dtb.create
        					.font(.systemFont(ofSize: 13.0))
        					.foregroundColor(.white)
        					.value
		   )
```



## Wrapper

> 从内存管理角度来审视。



#### 知识储备

[01  美团：Swift 性能优化](https://tech.meituan.com/2018/11/01/swift-compile-performance-optimization.html)

[02  京东：二进制优化与 COW](https://juejin.cn/post/7191406877819797561)

[03  Swift-Regret-Weak-Vars-in-Structs](https://belkadan.com/blog/2021/12/Swift-Regret-Weak-Vars-in-Structs/#)



#### me: public vs. internal

~~将所有 extension 方法视为一个三方库，在前文的改造之后，外部业务必然出现大量对 ``me`` 属性的调用，这会使得我们难以区分对相应属性的修改来自于内部还是外部，所以我更倾向于将 ``me`` 看成是库的私有属性进行处理，同时另行暴露外部接口。~~

理想很丰满，但是 internal 以后自定义方法除非对 value 进行操作，否则无法修改 me 的属性；如果

```swift
import DTBKit

public typealias XMKitWrapper = DTBKit.DTBKitWrapper
public extension XMKitWrapper {
	var me: Base { return value }
}
```

在 Debug 模式下能通过编译，放到私有 Cocoapods 库中后，Release 模式会出现符号引用错误：

```tex
1.	Apple Swift version 5.10 (swiftlang-5.10.0.13 clang-1500.3.9.4)
2.	Compiling with the current language version
3.	While evaluating request ExecuteSILPipelineRequest(Run pipelines { PrepareOptimizationPasses, EarlyModulePasses, HighLevel,Function+EarlyLoopOpt, HighLevel,Module+StackPromote, MidLevel,Function, ClosureSpecialize, LowLevel,Function, LateLoopOpt, SIL Debug Info Generator } on SIL for XMSport)
4.	While running pass #33574 SILModuleTransform "PerformanceSILLinker".
5.	While deserializing SIL function "$s6DTBKit0A7WrapperV5XMKitAD12AlertCreaterCRbzlE5titleyACyAFGSSSgAFRszrlF"
6.	*** DESERIALIZATION FAILURE ***
*** If any module named here was modified in the SDK, please delete the ***
*** new swiftmodule files from the SDK and keep only swiftinterfaces.   ***
module 'XMKit', builder version '5.10(5.10)/Apple Swift version 5.10 (swiftlang-5.10.0.13 clang-1500.3.9.4)', built from source, non-resilient, loaded from '/Users/xuyiding/Library/Developer/Xcode/DerivedData/XMSport-fecvclpfuqscmegygspvgkdnntiq/Build/Products/Release-iphonesimulator/XMKit/XMKit.framework/Modules/XMKit.swiftmodule/x86_64-apple-ios-simulator.swiftmodule'
result is ambiguous (_)
Cross-reference to module 'DTBKit'
... DTBKitWrapper
... me
... with type τ_0_0

Command SwiftCompile failed with a nonzero exit code
```



#### me: let vs. var

只能另外创建一种 Wrapper ：

```swift
public struct DTBKitMutableWrapper<Base> {
    public var me: Base
    public init(_ base: Base) {
        self.me = base
    }
}
```



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

那么如果不考虑 ``optional`` 和置空，``removeFromSuperview`` 等类似的函数会不会造成引用计数和内存方面的问题？``me`` 属性本身是否需要 ``weak`` 修饰呢？参见"知识储备"链接 01。



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

从业务角度来说 ABC 三种处理方案都说得通，但除了方案 C 都需要 ``var me``；而再进一步思考，就是如何将 ``throw`` ， ``optional`` 以及异步处理也纳入链式语法设计中的问题了，先按下不表。



## Helper

> 辅助方法设计。



#### when



#### then



#### vs. PromiseKit




## Protocol

> 如何使用泛型约束。



#### where: Array

> 自带泛型的类本身难以约束。


