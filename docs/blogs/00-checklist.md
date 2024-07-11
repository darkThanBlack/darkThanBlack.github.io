# 新电脑飞行清单

> Fork as [个人工作流](https://github.com/darkThanBlack/MOONWorkflow)



## Git

* [Sourcetree](https://www.sourcetreeapp.com/)

```shell
$ ls ~/.ssh/
$ ssh-keygen -t ed25519 -C "your_email@example.com"
$ open "https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent"

$ git config --global user.name "darkThanBlack"
$ git config --global user.email "331614794@qq.com"
```



## Pull

```shell
$ git clone --depth 1 --branch master git@github.com:darkThanBlack/MOONWorkflow.git
```



## SS + KcpTun

* ``~/.ShadowsocksX-NG/user-rule.txt``
* ``~/Library/Preferences/com.qiuyuzhou.ShadowsocksX-NG.plist``



## Browser

* [Google Chrome](https://www.google.com/intl/zh-CN/chrome/)



## Input

* [搜狗输入法](https://pinyin.sogou.com/mac/)

* 账号同步
* 偏好设置
  * 常用
    * 简中半角，全拼+超级简拼
    * 中文下使用英文标点：Xcode
    * 取消勾选其他所有选项
  * 按键
    * 禁用所有快捷键，除了：
      * 中英文，``Shift``；中英文标点，``Ctrl + . ``；
      * 候选翻页，``中括号 / Tab / Shift+Tab ``；
  * 外观
    * 横排，单行模式，9 候选词
  * 高级
    * 模糊音：``z=zh, c=ch, s=sh``
    * 关闭自定义表情，关闭 TouchBar，关闭版本更新



## App Store

* QQ
* 微信
* 钉钉
* WPS
* 网易云音乐
* Transporter



## Mac OS

* 升级至最新版

* Apple ID

  * 关闭 iCloud 云盘同步选项

* 程序坞与菜单栏

  * 显示建议和最近使用的 APP  关闭
  * 下载 右键 从程序坞移除

* Finder

  * 边栏：仅外接磁盘，并隐藏标签
  * 高级：显示所有文件扩展名，搜索当前文件夹

* Siri

  * 关闭

* 安全性与隐私

  ```shell
  # 任何来源
  $ sudo spctl --master-disable
  
  # Content 内容变化后报错
  $ sudo xattr -rd com.apple.quarantine /Applications/*.app
  
  # 重签名
  sudo xattr -cr /Applications/Sketch.app
  sudo codesign --force --deep --sign /Applications/Sketch.app
  ```

  * M1 芯片：注意 ``Rosetta``

* 软件更新

  * 取消自动更新

* 键盘

  * 键盘
    * 按键重复：拉到最快
    * 重复前延迟：拉到最短减1
    * TouchBar
      * 触控栏显示：App 控制
      * 从左至右，折叠情况下：截屏；亮度滑块；声音滑块；锁定屏幕
      * 从左至右，展开情况下：亮度（减，加）；调度中心；启动台；键盘亮度（减，加）；音量（静，减，加）；屏保；锁定屏幕
  * 文本
    * 取消所有选项的选择，删除所有快捷短语
  * 快捷键
    * 取消所有选项的选择，除了：
      * 调度中心：``control + 四方向键``
      * 键盘：焦点移动，``command + ` `` 和 ``option + command + ` ``
      * 输入法：快速选择，``^ + 空格键``
  * 输入法
    * 在菜单栏中显示
    * 不自动切换
  * 听写
    * 关闭

* 触控板

  * 光标与点按
    * 查询与数据检测器：单指用力点按，勾选触感反馈
    * 辅助点按：双指点按
    * 轻点来点按：取消勾选
    * 点按强度：强
    * 跟踪速度：中等
  * 滚动缩放
    * 全部勾选
  * 更多手势
    * 全部勾选，并调整：
      * 调度中心：三指向上
      * App Expose：三指向下

* 显示器

  * 自动调节亮度，原彩显示

* 电池

  * 电源适配器
    * 15分钟后关闭显示器
    * 防止自动进入睡眠
    * 唤醒以供网络访问

* 共享

  * 电脑名称



## iTerm2

* [下载](https://iterm2.com/downloads.html)
* 个人设置同步：``Preferences -> General -> Settings -> Load settings from a custom folder or URL``



## Karabiner

* [下载](https://karabiner-elements.pqrs.org/) 或 [GitHub](https://github.com/pqrs-org/Karabiner-Elements/releases)
* 个人设置同步
  * ``Setting -> Misc -> Open config folder``
  * ``~/.config/karabiner``



## Alfred

* [下载](https://macked.app/alfred.html)
* 访问权限：完全磁盘访问权限
* 个人设置同步：``Advanced -> Syncing``
* 应用搜索：``Features -> Search Scope`` 添加 ``Applications``



## Typora

* [中文官网下载](https://typoraio.cn/)

* [英文官网下载](https://typora.io/)

* 破解：

  ```shell
  # 找到文件: /Applications/Typora.app/Contents/Resources/TypeMark/page-dist/static/js/LicenseIndex.180dd4c7.6d698c41.chunk.js
  # 检查字段: hasActivated="true"
  ```



## PicGo

* [下载](https://picgo.github.io/PicGo-Doc/zh/guide/#%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85)
* [打不开/打开黑屏](https://github.com/Molunerfinn/PicGo/issues/781#issuecomment-1008603421)
* ``~/Library/Application Support/picgo/data.json``



## Charles

* [下载](https://www.charlesproxy.com/download/)



## VSCode

* [下载](https://code.visualstudio.com/)



## Xcode

* [下载](https://developer.apple.com/download/all/)

* [单独安装模拟器环境](https://developer.apple.com/documentation/xcode/installing-additional-simulator-runtimes)

  ```shell
  sudo xcode-select -s /Applications/Xcode-beta.app;
  sudo xcodebuild -runFirstLaunch;
  sudo xcrun simctl runtime add "~/Downloads/**.dmg"
  ```

  

* ``~/Library/Developer/Xcode/UserData``

  * CodeSnippets
    * 固定前缀：``MOON``
    * 片段文件名：OC 用``MOON__``开头，Swift 用``MOON_``开头以区分，快捷方式不变。
      * 新建文件：``MOON_New``
        * ``MOON_NewViewController``
        * ``MOON_NewView``
        * ``MOON_NewCell``
      * 对象声明：``MOON_GetLazy``
        * ``MOON_GetLazyUIView``
        * ``MOON_GetLazyUILabel``
        * ``MOON_GetLazyUIImageView``
        * ``MOON_GetLazyUIButton``
        * ``MOON_GetLazyUITableView``
      * 页面布局：``MOON_SnapKit``
      * 用户事件：``MOON_SingleTapGesture``
  * FontAndColorThemes
  * KeyBindings


* General
  * File Extensions: Show all
* Accounts
* Navigation
  * Command-click on Code: Jumps to Definition
* Themes
* Text Editing



## Env

首先需确认是否 ``Rosetta``



#### Homebrew

* [Website](https://brew.sh/)

```shell
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"

$ brew install xcodegen
```



#### RVM

* [Website](https://rvm.io/)

```shell
$ \curl -sSL https://get.rvm.io | bash -s stable

# 同时尝试安装 ruby
$ \curl -sSL https://get.rvm.io | bash -s stable --rails
# 或
$ rvm list known
$ rvm install 3.0.0 --disable-binary

$ rvm list
$ rvm use 3.0.0 --default
$ which ruby
```



#### GEM

```shell
$ ruby --version  # 3.0.0
$ gem --version  # 3.2.3

# 删除
$ gem list | cut -d " " -f1 | xargs sudo gem uninstall -aIx

$ gem search cocoapods
$ gem install cocoapods -v 1.15.2
```



#### Cocoapods

* [Website](https://cocoapods.org/)

```shell
$ pod --version
# 在工程目录下
$ pod setup --verbose
$ pod install --verbose
```



