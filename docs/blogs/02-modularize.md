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



## 语言特性

先考虑手动制作一个最通用的静态库。lipo 的合并命令使用非常简单，导致网上充斥着大量的垃圾教程。



纯 ObjC / 纯 Swift / ObjC - Swift 混编 / *.a - *.framework 重新封装 / *.bundle 处理 都必须考虑到。如果新业务涉及到的时候需要重新制作静态库，那就是你的问题。

要注意混编相关的多种场景：静态库本身是纯 ObjC / 纯 Swift，但会被另一种代码调用；静态库内部的代码混编了 ObjC 和 Swift。

*.bundle 可能需要单独处理，做完后一定要测一下。需要提供快速验证静态库调用本身 *.bundle 是否成功的手段。现在还可能有 *.docC。



## 选型

我现在敢说 Swift 最大的痛点就是静态库支持。基于较低版本 Swift 制作的静态库无法在任何更高版本的 Swift 环境中运行。



#### *.xcFramework

缺失关键特性。不可接受。不可用。









## 实践 03.19

* ``CFBundleSupportedPlatforms: iPhoneOS``
* ``xxx.frameworks/Headers/xxx-Swift.h`` merge
  * ``#if 0``
  * ``\#elif defined(__x86_64__) && __x86_64__``
  * ``\#elif defined(__arm64__) && __arm64__``
  * ``\#elif defined(__ARM_ARCH_7A__) && __ARM_ARCH_7A__``
  * 难点：
    ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios_16_20230330182417.png)

比如 ``Xcode 13.x`` 之前编译产物就不长这样，脚本需要适时调整。







## 日志



#### Xcode  14.x

* 本质原因：![image-20230329142235601](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230329142235601.png)

  ```markdown
  Failed to build module 'SnapKit'; this SDK is not supported by the compiler (the SDK is built with 'Apple Swift version 5.6 (swiftlang-5.6.0.323.62 clang-1316.0.20.8)', while this compiler is 'Apple Swift version 5.8 (swiftlang-5.8.0.124.1 clang-1403.0.22.11.100)'). Please select a toolchain which matches the SDK.
  ```
  
  

* pod 通用覆写代码：

  ```ruby
  post_install do |installer|
    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        # do sth.
      end
    end
  end
  ```



* development team required
  * 问题：
    ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230329110852.png)

    ```swift
    Signing for "mock" requires a development team. Select a development team in the Signing & Capabilities editor.
    ```
  
  * 解决：https://www.jianshu.com/p/21cf97fb7fc7
  
    ```ruby
    config.build_settings['EXPANDED_CODE_SIGN_IDENTITY'] = ""
    config.build_settings['CODE_SIGNING_REQUIRED'] = "NO"
    config.build_settings['CODE_SIGNING_ALLOWED'] = "NO"
    ```
  
    
  
* SwiftAlgorithms
    * Charts v3.6.0 -> 4.1.0
    
    * 问题：
      ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/image-20220929175129043.png)

    * 解决：https://github.com/apple/swift-algorithms/issues/189
    
      ```ruby
      config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      ```
    



* 问题：

    ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230328180702.png)

    ```swift
    Showing Recent Messages
    Cannot code sign because the target does not have an Info.plist file and one is not being generated automatically. Apply an Info.plist file to the target using the INFOPLIST_FILE build setting or generate one automatically by setting the GENERATE_INFOPLIST_FILE build setting to YES (recommended).
    ```
    
    * 解决1: https://juejin.cn/post/7197361396219772983  其实不是，但需要了解。

        ```ruby
        config.build_settings['GENERATE_INFOPLIST_FILE'] = "NO"
        ```
    
    * 解决2: 这里是因为 ``.xcodeproj`` 通过 ``xcodegen`` 生成，这个模块又没有放入 ``.plist`` 文件。



* 问题：

  ![image-20230403143608378](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230403143608378.png)

  ```swift
Explicit '@objc' on subclass of 'MobilePlayerViewController' requires iOS 13.0.0
  ```
  
  * OC 调用 Swift 知识：``@objc / @objcMembers`` 关键字，其实不是，但需要了解。
* 结论是需要处理 ``BUILD_LIBRARY_FOR_DISTRIBUTION``，[京东组件化](https://zhuanlan.zhihu.com/p/349967113) 里有提到。



* 耗时过长，拆开来写：

  ![image-20230404155213205](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16-20230404155213205.png)

  ```
  The compiler is unable to type-check this expression in reasonable time; try breaking up the expression into distinct sub-expressions.
  ```



* 编译通过，模拟器运行时，出现：
  ![image-20230405171937015](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_-20230405171937015.png)

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
  
  * 解决：两种原因，``_OBJC_CLASS_$_SLImageClipController`` 是因为我二进制真的做错了，重做即可；``Realm`` 比较复杂：
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



* 编译通过，真机运行时，出现：
  ```swift
  dyld[7510]: Library not loaded: @rpath/App.framework/App
    Referenced from: <A11A27F1-222B-3BA4-A371-268CE92F7FD3> /private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/XMApp
    Reason: tried: '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/preboot/Cryptexes/OS@rpath/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/usr/lib/swift/App.framework/App' (errno=2, not in dyld cache), '/private/preboot/Cryptexes/OS/usr/lib/swift/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/private/var/containers/Bundle/Application/B9B45F2E-20FC-49EB-B5BC-D0F08EF75FB2/XMApp.app/Frameworks/App.framework/App' (errno=2), '/System/Library/Frameworks/App.framework/App' (errno=2, not in dyld cache)
  ```
  
  * 解决：[issue-92896](https://github.com/flutter/flutter/issues/92896#issuecomment-999441920)，注意同时保证 cocoapods 和 ruby-macho 版本。



* 警告：

  ```swift
  Ignoring ffi-1.10.0 because its extensions are not built. Try: gem pristine ffi --version 1.10.0
  Ignoring redcarpet-3.4.0 because its extensions are not built. Try: gem pristine redcarpet --version 3.4.0
  Ignoring sqlite3-1.3.13 because its extensions are not built. Try: gem pristine sqlite3 --version 1.3.13
  ```
  
  * 解决：执行相应命令

* 静态库不制作 framework，直接导入？
