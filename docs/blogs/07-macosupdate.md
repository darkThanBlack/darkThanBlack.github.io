# Mac OS 与 Xcode 升级方案

> Created: 2023/12/14



#### 问题描述

以 Mac OS Sonoma 为例，官网要求 Macbook Pro 至少为 2021 年款方可使用，而 Xcode 本身又与 Mac OS 版本深度绑定，Xcode 14.3 只能在 Ventura 上运行，而允许适配隐私政策清单的 Xcode 15 又需要 Sonoma，充分体现了苹果作为大企业的傲慢与丑恶。



#### 解决方案

利用 ``HackinTool`` 生成符合要求的序列号，再使用 ``OpenCore Legacy Pather`` （简称 ``OCLP``）将本机的序列号等信息替换掉，通过官方渠道完成系统升级，最后安装相关补丁。

[参考](https://www.joeyne.cool/os/macos/%e8%80%81%e6%ac%be%ef%bc%882017%e4%b9%8b%e5%89%8d%ef%bc%89macbook-pro%e5%a6%82%e4%bd%95%e5%8d%87%e7%ba%a7macos-ventura/)



## Sonoma



#### Wifi 无法使用

问题：Wifi 功能显示为不可用

解决：不支持 Broadcom（博通）网卡导致；OCLP 安装补丁现已包含在内，直接执行安装即可。



#### TouchBar 没有 Esc

问题：本机是 2017 的 TouchBar，Esc 是虚拟的；后来出的机器 Esc 独立出来变成了物理按键，系统升级后 TouchBar 上自然找不到 Esc 了。

临时解决：利用 Karabiner 的改键功能，将 bar 设定为 F1 - F12，并将 F1 映射成 Esc



#### ShackdowSocksX-NG 模式切换失效

问题：更新后软件无法自动修改 wifi 里的代理设置。

解决：需要在软件的相应设置中指明对哪几个网络链接应用代理





