# 飞行清单 - 新电脑



#### [个人工作流](https://github.com/darkThanBlack/MOONWorkflow)

* 先处理 SS



#### Mac OS

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
* 键盘快捷键
    * 取消所有选项的选择，除了：
        * 显示：摇动鼠标指针以定位
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



#### App Store

* QQ，微信，钉钉，WPS Office，网易云音乐，Transporter
* Xcode 和 模拟器环境 单独下载 / 用 U 盘



#### 软件

* [Google Chrome](https://www.google.com/intl/zh-CN/chrome/)
    * 账号同步
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
* [Charles](https://www.charlesproxy.com/download/)
    * 配置：
* [VSCode](https://code.visualstudio.com/)

