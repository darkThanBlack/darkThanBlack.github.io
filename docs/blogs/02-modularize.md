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









## 日志



#### Xcode  14.x

* 本质：![image-20230329142235601](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230329142235601.png)

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

    ```markdown
    Signing for "mock" requires a development team. Select a development team in the Signing & Capabilities editor.
    ```
  
  * 解决：https://www.jianshu.com/p/21cf97fb7fc7
  
    ```ruby
    config.build_settings['EXPANDED_CODE_SIGN_IDENTITY'] = ""
    config.build_settings['CODE_SIGNING_REQUIRED'] = "NO"
    config.build_settings['CODE_SIGNING_ALLOWED'] = "NO"
    ```
  
    
  
* SwiftAlgorithms
    * v3.6.0 -> 4.1.0
    
    * 问题：
      ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/image-20220929175129043.png)

    * 解决：https://github.com/apple/swift-algorithms/issues/189
    
      ```ruby
      config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      ```
    
      
    
* RealmSwift, Others
    * 10.25.1 -> 10.30.0
    
    * 问题：
    
      ``pod 'RealmSwift'`` 且 同时 ``pod 'WechatOpenSDK'``，并设置 Realm 的 ``MACH_O_TYPE`` 为 ``staticlib`` 时，报``Undefind Symbol``错误
    
    * 解决1：``staticlib`` -> ``mh_dylib``； issue 待咨询



* plist

    * 问题：

        ![img](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/ios16_20230328180702.png)

        ```markdown
        Showing Recent Messages
        Cannot code sign because the target does not have an Info.plist file and one is not being generated automatically. Apply an Info.plist file to the target using the INFOPLIST_FILE build setting or generate one automatically by setting the GENERATE_INFOPLIST_FILE build setting to YES (recommended).
        ```

    * 解决1: https://juejin.cn/post/7197361396219772983  其实不是，但需要了解。

        ```ruby
        config.build_settings['GENERATE_INFOPLIST_FILE'] = "NO"
        ```

    * 解决2: 这里是因为 ``.xcodeproj`` 通过 ``xcodegen`` 生成，这个模块又没有放入 ``.plist`` 文件。











