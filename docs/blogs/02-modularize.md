# Modularize

简要记录模块化过程中个人思路和要点。

> Created: 2022/06/26



## 哲学

绝对的论断默认前提是尚未实现任何模块化，否则展开永远讲不完。下同。



强业务相关，随业务总量和人员规模演进，基于团队实力选型。

极度依赖主导者能力和团队水准，讨论前先确认好业务场景，前提和背景，否则鸡同鸭讲。

综上，具体实现并无定法，方案也往往无法套用，有起点无终点。

同理，缺乏业务和协作经验则注定无法理解，不看经历就在面试里问是在浪费时间。



技术演进的根本目的是提升业务效率，不结合业务的技术注定失败；

最影响效率的是代码编译速度，如果你认为不是，那么你不需要参与讨论；

编译加速的核心手段是静态化，任何不实现静态化的加速方案都不需要去考虑；

静态化后的代码无法被修改，所以需要区分出被修改的代码，和不被修改的代码。

在对代码进行区分过后，必然形成相对独立的多份代码，有人将这些代码称为模块，将区分的手段称为模块化。

静态化的代码越多，编译速度越快；静态化的代码越少，编译速度越慢。好的方案需要使可静态化的代码尽可能多。

代码之间的联系越多，修改时涉及到的代码也就越多，可静态化的代码就越少，编译速度越慢。好的方案需要使模块代码之间的关联尽可能少。

静态化手段本身具有成本。好的方案需要尽可能降低使用成本。



## 选型

我现在敢说 Swift 最大的痛点就是静态库支持。基于较低版本 Swift 制作的静态库无法在任何更高版本的 Swift 环境中运行。




## 我们的模式

// todo...



## 静态库不简单

阿猫阿狗都知道 lipo，导致网上充斥着大量的垃圾教程信息，谁来都能讲个一二三，但要讲全很难。我们需要仔细思考在 iOS 移动端技术栈的语境下，手动制作一个最通用的静态库，到底要考虑到哪些方面。

一开始先不要考虑包大小和架构拆分，先实现后优化。

### 目标工程

先考虑这个静态库会被什么样子的业务工程使用。

#### C++ 与 ObjC 混编

略

#### ObjC 与 Swift 混编

基础知识？

在 ObjC 中如何引用？

在 Swift 中如何引用？

#### 架构

Intel / M1 区别？为什么 M1 上的模拟器依然需要 x86_64？

#### 依赖

是否依赖于其他系统库/静态库运行？

如何处理，或向业务层体现出来？

依赖于某个特定版本吗？依赖库出现更新/废弃/升级时如何兼容？

我依赖的库本身是否也存在这些问题？

#### 兼容性

最低支持的 iOS 版本是？

是否能在更高/更低版本的环境下编译通过？

#### 更新

这里只看库本身发生变化时如何同步给业务工程。

具体的每个工具值得单独辨析。



### 静态库工程

再考虑要将什么样子的代码做成静态库，以及做成什么样子的静态库。

针对的代码（第三方 / 私有底层 / 业务层）不同，考量截然不同。

#### *.a / *.framework

基础知识？

两者如何互相转化？架构如何拆分？

多个静态库如何互相依赖？处理方式？

后续没有特殊说明，均针对 ``*.framework`` 展开讨论。

#### *.xcFramework

[介绍](https://github.com/bielikb/xcframeworks)

参考 链接文章内的``Troubleshooting`` 一节，这个方案目前（2023.04）对业务层缺失关键特性，既不可用也无法接受。

第三方工具/库开始逐渐应用，如 Flutter Engine 已弃用 ``*.framework``，会有相应问题。

#### 纯 ObjC / 纯 Swift

编译产物与包结构上有何区别？如何适配？

两者在接口定义的暴露方法上有何区别？最佳实践是什么？

具体产物后续在编译选项中单独辨析。

#### ObjC 与 Swift 混编

如何在静态库内部实现混编？结构与代码如何调整？

#### 宏

这里只看如何避免冲突。

具体的宏定义和使用需要单独辨析。

#### Swift Nested Types

[Language Guide](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/nestedtypes)

* 如果**嵌套类继承**了任何 ObjC 类型，例如 ``UIView`` 或 ``NSObject``，必须利用 ``@objc()`` 将编译产物内的类名进行重写，避免同名冲突。
  * 否则调试正常，正式打包报错，合理怀疑是语言自身处理方式问题。

#### 命名空间与重名冲突

略。

#### 资源

这里只看静态库内部代码如何调用所需的资源文件，即便如此，很多第三方库的源码实现也并不完善，去取 ``Bundle`` 时需要考虑很多情况。

具体使用何种形式管理资源文件值得单独辨析。

#### *.plist

* **ITMS-90542** 要求 ``CFBundleSupportedPlatforms`` 字段的值为 ``iPhoneOS``

#### *.docC / Tutorials

[介绍](https://developer.apple.com/documentation/docc)



### 编译选项：

#### [Build Settings](https://developer.apple.com/documentation/xcode/build-settings-reference)

####  $xcodebuild

不了解制作过程中涉及的每一个编译选项，就意味着要用自己的血泪去浸润它。



## Troubleshooting: Guide

#### 核心原因

如果无法制作二进制接口稳定的静态库，那么就只能在每次 Swift 大版本升级时，用新工具链重新打包制作所有静态库。

业务层使用的工具链版本需要和制作 ``*.framework`` 时的版本严格相同。

```markdown
Failed to build module 'SnapKit'; this SDK is not supported by the compiler (the SDK is built with 'Apple Swift version 5.6 (swiftlang-5.6.0.323.62 clang-1316.0.20.8)', while this compiler is 'Apple Swift version 5.8 (swiftlang-5.8.0.124.1 clang-1403.0.22.11.100)'). Please select a toolchain which matches the SDK.
```

![image-20230329142235601](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230329142235601.png)




#### Podfile

这段 Ruby 代码极为常用。

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      # do sth.
    end
  end
end
```



#### 合并 ``*-Swift.h`` 文件

编译产物，源文件中存在 Swift 文件时出现，路径为 ``xxx.frameworks/Headers/xxx-Swift.h``。

不同的 Swift 版本之间生成结果会发生变化，截止目前（Swift version 5.8）依然不稳定，脚本代码需要进行相应处理。

合并后的目标文件中，源码结构大致如下：

```swift
#if 0
#elif defined(__x86_64__) && __x86_64__
// ...
#elif defined(__arm64__) && __arm64__
// ...
#elif defined(__ARM_ARCH_7A__) && __ARM_ARCH_7A__
// ...
#endif
```

``#if 0`` 大概是他们的最佳实践。

**Xcode 14.3, Swift 5.8**

在文件末尾增加了 3 行代码。

![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios_16_20230330182417.png)



## Troubleshooting: Issues

#### MOON__ISSUE_001

```swift
Signing for "mock" requires a development team. Select a development team in the Signing & Capabilities editor.
```

![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230329110852.png)

解决：[issues-11402](https://github.com/CocoaPods/CocoaPods/issues/11402#issuecomment-1379702414)

```ruby
config.build_settings['EXPANDED_CODE_SIGN_IDENTITY'] = ""
config.build_settings['CODE_SIGNING_REQUIRED'] = "NO"
config.build_settings['CODE_SIGNING_ALLOWED'] = "NO"
```



#### MOON__ISSUE_002

将 ``Charts`` 从 ``3.6.0`` 升级至 ``4.1.0``，``Charts`` 依赖于 ``SwiftAlgorithms``，编译报错。

![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/image-20220929175129043.png)

解决：[issues-189](https://github.com/apple/swift-algorithms/issues/189)

 ``BUILD_LIBRARY_FOR_DISTRIBUTION``



#### MOON__ISSUE_003

``*.plist`` 文件缺失。

```swift
Showing Recent Messages
Cannot code sign because the target does not have an Info.plist file and one is not being generated automatically. Apply an Info.plist file to the target using the INFOPLIST_FILE build setting or generate one automatically by setting the GENERATE_INFOPLIST_FILE build setting to YES (recommended).
```

![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230328180702.png)

解决：``GENERATE_INFOPLIST_FILE``

项目里是因为 ``xcodegen`` 未配置。

参考：[Xcode 14 Display Name](https://juejin.cn/post/7197361396219772983)



#### MOON__ISSUE_004

此处业务是在 ``ObjC`` 模块中，需要继承 ``Swift`` 静态库的类。


```swift
Explicit '@objc' on subclass of 'MobilePlayerViewController' requires iOS 13.0.0
```

![image-20230403143608378](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230403143608378.png)

解决： ``BUILD_LIBRARY_FOR_DISTRIBUTION``

报错提示与 OC / Swift 混编相关，相关资料会提到 ``@objc / @objcMembers`` 关键字。

项目里是因为制作业务静态库时编译选项导致，与上述知识无关。

参考：[京东组件化](https://zhuanlan.zhihu.com/p/349967113)



#### MOON__ISSUE_005

```
The compiler is unable to type-check this expression in reasonable time; try breaking up the expression into distinct sub-expressions.
```

![image-20230404155213205](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230404155213205.png)

解决：略。



#### MOON__ISSUE_006

``Realm`` 相关，``RealmSwift`` 依赖于 ``Realm``，将两者均制作成静态库后报错。

  ```swift
  Undefined symbols for architecture x86_64:
    "_$s10RealmSwift0A14CollectionImplPAAE12makeIteratorAA11RLMIteratorVy7ElementQzGyF", referenced from:
        _$s11Persistence12RealmManagerC7objects2of9predicateSayxGSgxm_q_tSo0B11SwiftObjectCRbzr0_lF in Persistence(RealmManager.o)
        _$s11Persistence12RealmManagerC10allObjects2ofSayxGSgxm_tSo0B11SwiftObjectCRbzlF in Persistence(RealmManager.o)
    "_$s10RealmSwift7ResultsVyxGAA0A14CollectionImplAAMc", referenced from:
        _$s11Persistence12RealmManagerC7objects2of9predicateSayxGSgxm_q_tSo0B11SwiftObjectCRbzr0_lF in Persistence(RealmManager.o)
        _$s11Persistence12RealmManagerC10allObjects2ofSayxGSgxm_tSo0B11SwiftObjectCRbzlF in Persistence(RealmManager.o)
    "_OBJC_CLASS_$_SLImageClipController", referenced from:
        objc-class-ref in HomeSchool(JPCropViewController.o)
  ld: symbol(s) not found for architecture x86_64
  clang: error: linker command failed with exit code 1 (use -v to see invocation)
  ```

 ![image-20230405171937015](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_-20230405171937015.png)



解决：通过 ``lipo -info`` 检查 ``_OBJC_CLASS_$_SLImageClipController`` 发现有误，略过。

对于 ``Realm``，


  * 首先：
    * [检查](https://www.mongodb.com/docs/realm/sdk/swift/install/#prerequisites)
    * 旧版的 [在 Objective-C 和 Swift 混合工程中使用 Realm](https://www.mongodb.com/docs/legacy/realm/objc/latest.html#using-the-realm-framework), 来自 [issue-7803](https://github.com/realm/realm-swift/issues/7803#issuecomment-1135568615) 的提示
  * 搜索讨论串，怀疑：
    *  [Intel / M1 Mac](https://www.mongodb.com/community/forums/t/installing-realmswift-on-an-intel-mac-installs-different-librealm-monorepo-a-binary-than-when-installed-on-an-m1-mac/191764/4)
    * [版本问题](https://www.mongodb.com/community/forums/t/trouble-archiving-because-of-undefined-realm-realmswift-symbols/205450)
  * 查 issues：
    * 受 [issues-7114](https://github.com/realm/realm-swift/issues/7114#issuecomment-1328242905) 启发，尝试 ``ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES``
  * 根据老代码，调整 ``mach-o type``
  * 暂定结论是 ``Realm`` 需要 ``mh_dylib``， ``RealmSwift`` 保持 ``staticlib``



#### MOON__ISSUE_007

编译通过，模拟器运行通过，真机运行时出现。
```swift
dyld[7510]: Library not loaded: @rpath/App.framework/App
  Referenced from: <A11A27F1-222B-3BA4-A371-268CE92F7FD3> /private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/XMApp
  Reason: tried: '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/preboot/Cryptexes/OS@rpath/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/System/Library/Frameworks/App.framework/App' (errno=2, not in dyld cache)
```

解决：[issue-92896](https://github.com/flutter/flutter/issues/92896#issuecomment-999441920)

升级 ``cocoapods`` 和 ``ruby-macho``至指定版本以上。



#### MOON__ISSUE_008

执行 ``pod install --verbose`` 时出现警告。

```swift
Ignoring ffi-1.10.0 because its extensions are not built. Try: gem pristine ffi --version 1.10.0
Ignoring redcarpet-3.4.0 because its extensions are not built. Try: gem pristine redcarpet --version 3.4.0
Ignoring sqlite3-1.3.13 because its extensions are not built. Try: gem pristine sqlite3 --version 1.3.13
```

解决：略。



#### MOON__ISSUE_009

```swift
Synchronous URL loading of xxx should not occur on this application's main thread as it may lead to UI unresponsiveness. Please switch to an asynchronous networking API such as URLSession.
```

![image-20230413185356587](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230413185356587.png)

解决：略



#### MOON__ISSUE_010

执行 CI 时报错，本质是 Archive 时报错。

![截屏2023-04-20 15.09.09](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230420150909.png)

查看该脚本具体内容，本质是执行以下命令：

```shell
"${PODS_ROOT}/Target Support Files/Pods-XMApp/Pods-XMApp-frameworks.sh"
```

查看 CI 日志输出，得到：

![截屏2023-04-20 15.10.24](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230420151024.png)

```swift
rsync error: some files could not be transferred (code 23)
```

解决： [issue-11868](https://github.com/CocoaPods/CocoaPods/issues/11868)，[pull-11828](https://github.com/CocoaPods/CocoaPods/pull/11828)

升级 cocoapods ``1.11.2`` 至 ``1.12.0``



## Troubleshooting: Flutter

本质：

在整体升级后，``~/Documents/XiaoMai/b/flutterPack/.ios/`` 工程执行 ``pod install`` 时极有可能报错，但 ``Flutter`` 本身无法 cover 这部分问题，需要自己手动去执行并检查，同时注意各种脚本如 ``podHelper.rb`` 源码



#### MOON__ISSUE_999

找不到 ``Flutter.h `` 头文件，一般报在某个插件内。

```swift
Flutter.h not found
```

* 观察  ``~/Documents/XiaoMai/b/flutterPack/.ios/Flutter`` [1]，发现 ``engine`` 有问题
* 前者应来自于 ``~/fvm/versions/2.0.6/bin/cache/artifacts/engine`` [2] 下的 ``ios`` 文件夹
* 从旧资料/电脑上可以发现正常情况下 [1] 处存在 ``Flutter.framework``，[参考1](https://stackoverflow.com/questions/64973346/error-flutter-flutter-h-file-not-found-when-flutter-run-on-ios?noredirect=1&lq=1)，[参考2](https://stackoverflow.com/questions/50671286/flutter-h-not-found-error)，实际一般无效
* 手动复制 ``Flutter.framework`` 置入不可行，因为会被 ``pod install`` 动作覆盖
* [参考3](https://blog.csdn.net/qq_32792839/article/details/111247075) 提到 ``*.framework`` 已经变成 ``*.xcframework``
* [参考4](https://juejin.cn/post/7202541740933759031) 提到 ``*.podspec`` 内 ``.vendored_frameworks`` 发生变化，但 ``.dependency`` 对我的场景未必管用
* 暂先直接改回 [2] 处 ``*.podspec`` 的 ``.vendored_frameworks`` 属性，``pod install`` 后发现 ``*.xcframework`` 拷贝成功



#### MOON__ISSUE_998

![image-20230417125907563](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230417125907563.png)

```shell
# 自定义脚本
set -e
set -u
source "${SRCROOT}/../flutterPack/.ios/Flutter/flutter_export_environment.sh"
"$FLUTTER_ROOT"/packages/flutter_tools/bin/xcode_backend.sh build
```

无任何具体报错信息。

解决：见 ``flutterpack`` 工程的 ``README``

``TREE_SHAKE_ICONS``

参考：[Flutter Tree Shaking](https://zhuanlan.zhihu.com/p/272567200)



#### MOON__ISSUE_997

![1681711867644](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/iOS16-1681711867644.jpg)

解决：Xcode Clean

先编译了自定义字段 ``Enterprise_Release``，又编译了 ``Release`` 导致



## Troubleshooting: ITMS

#### ITMS-90542

确保工程中所有 ``*.plist`` 文件中的 ``CFBundleSupportedPlatforms`` 字段的值为 ``iPhoneOS``，按具体情况处理。



## Playground



选型: 静态库不再重复制作 framework，直接导入



https://blog.csdn.net/weixin_43843218/article/details/124846510

