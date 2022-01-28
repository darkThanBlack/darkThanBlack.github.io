

# Learn:更轻量的 View Controllers



> 原文链接：[ObjC中国-更轻量的ViewControllers](https://objccn.io/issue-1-1/)
>
> 本文是对ObjC期刊一系列高质量博文所作的个人学习笔记。
>
> 本文约 3,580 字，可能需要 10 分钟时间阅读。
>
> 本文所用到的[示例DEMO](https://github.com/darkThanBlack/LearnObjcIO/tree/master/issue1-%E6%9B%B4%E8%BD%BB%E9%87%8F%E7%9A%84ViewControllers/issue1-1)。
>
> 
>
>
> created: 2018-12-16
>
> update: 2018-12-17  增加[issue1-2](https://github.com/darkThanBlack/LearnObjcIO/blob/master/issue1-%E6%9B%B4%E8%BD%BB%E9%87%8F%E7%9A%84ViewControllers/issue1-2/issue1-2.md)链接
>
> update: 2022-01-28  迁移至 github.io，调整链接，增补内容



## 2022-01-28：最佳实践

回头来看以前的垃圾文章，一堆废话，删就不删了，补充点东西。说那么多，本质是一个推导出 vc 写法最佳实践的过程。

#### protocol

现在主要使用 Swift 开发后，protocol 能力增强，cell 解耦的写法大大简化，目前项目组内原则上是要求任意一个 cell 都有对应的 protocol，而 vc 内的 delegate 和 dataSource 分离反而不常见，因为往往 delegate 和 dataSource 依赖 vc. viewModel，而我们会将绝大部分逻辑放在 vc.viewModel 内；分离做法也可以参见苹果官方 DEMO [AVCam: Building a Camera App](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/avcam_building_a_camera_app) 里 ``PhotoCaptureProcessor`` 类的实现。另一个容易搞混的点是 protocol 主要是为了复用还是解耦：我认为解耦的意义 >> 复用，一旦 vo 或者 vm 出现调整，protocol 形式的 cell 会好调整得多，这是使用 protocol 的主要目的；如果想做成公共组件，那就应该单独写，或者从业务模块里提出来，不要思考和实践业务模块里直接捞 cell 用的情况；高复用的组件往往包含了 protocol 模式，但不是用了 protocol 就等于可复用。


#### viewModel

原则上一个 vc 必须有一个对应的 vm，所有东西能移的都移到 viewModel 里，主要包含网络请求，页面数据和业务逻辑三块，vc 更多的是传参赋值操作。绝不完美，但肯定够用，一个移动端 SaaS 系统的业务复杂度不能算低了。

#### viewController 数据传递

传 id >> 传 model，进下个页面通过单独的 getDetail 请求去拿数据，这个应该算基础原则了，提一个点：

```swift
/// V1
class Acontroller: UIViewController {
    var uid: Int64?
    var userName: String?
}

/// V2
class Bcontroller: UIViewController {
	
    enum `Type` {
        case userLogin
        case userDetail(uid: Int64?, userName: String?)
    }
    var type: Type
}

/// V3
class Ccontroller: UIViewController {
	
    enum `Type` {
        case userLogin
        case userDetail
    }
    struct Params {
        let type: Ccontroller.Type
        var uid: Int64?
        var userName: String?
    }
    var param: Params?
}

```

V2 版本常见于一个 vc 处理多种场景类似的业务，定义枚举指代业务入口，枚举数据绑定来传参，但有一个问题是单个 case 绑定的数据也会出现膨胀。当然也可以写成``case userDetail(data: Params?)``这样来处理；还一个问题是项目里 ``if case .userDetail = self.type`` 之类的代码会变多，没法利用 switch-case 块来保证不可变；所以如果参数可能会超过2个，我会采用类似 V3 的写法，如有必要还可以向 ``Params`` 类添加方法来满足需要。

一些基本规则：

```swift
/// 单纯讲传参
class Controller: UIViewController {
    /// private >> public
    private var param: Params?
    /// init >> set
    required init(param: Params?) { ... }
    
    /// 确实有外部修改必要的，func >> var
    func updateProcessData(with userName: String?) {
        self.param.userName = userName
    }
}

/// 嵌套类定义过长可以用 extension 分开，或者不用嵌套类
extension Controller {
    enum `Type` {
        case userLogin
        case userDetail
    }
    struct Params {
        let type: Ccontroller.Type
        var uid: Int64?
        var userName: String?
    }
}

```

嵌套类有一个坑点，如果嵌套类继承自``NSObject``，必须用``@objc``标签重命名一个永远不会冲突的名字，否则打包的时候会因类名相同报错。



## 2018-12-16：原文

### 将DataSource作为一个单独的类分离出来

原文使用`UITableViewDataSource`做示例，并指出这种方法可以推及到像`UICollectionViewDataSource`等其他Protocols上，要理解这一点其实只要再仔细想一下Delegate的模式就可以明白——既然ViewController作为一个类可以实现委托方法，那么在ViewController过于臃肿的情况下，我们自然可以再让别的类来实现这些委托方法。

但要将这一思路应用在实际项目的重构上，相较于原作者DEMO的形式我认为还有几点需要更细致地说明。我们首先从最简单的形式开始，先实现一个最简单的TableView结构：

```objective-c
@interface LOStyle_1_View()<UITableViewDelegate>

@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) NSArray *cellDataArray;

@end

@implementation LOStyle_1_View

#pragma mark Getter

- (UITableView *)tableView
{
    if (!_tableView) {
        _tableView = [[UITableView alloc]init];
        _tableView.delegate = self;
        _tableView.dataSource = self.style_1_dataSource;
        _tableView.tableFooterView = [[UIView alloc]initWithFrame:CGRectZero];
        _tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
        
        [_tableView registerClass:[LOStyle_1_Cell class] forCellReuseIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    }
    return _tableView;
}

- (NSArray *)cellDataArray
{
    if (!_cellDataArray) {
        _cellDataArray = @[[UIColor greenColor]];
    }
    return _cellDataArray;
}

#pragma mark TableViewDataSource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.cellDataArray.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    LOStyle_1_Cell *cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    cell.backgroundColor = self.cellDataArray[indexPath.row];
    return cell;
}

```

现在这个是最简单的例子，没有Model，没有网络请求和回调，也没有数据绑定；然后我们假装业务庞杂，设置cell属性的代码开始变多：

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    LOStyle_1_Cell *cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    cell.backgroundColor = self.cellDataArray[indexPath.row];
    cell.textLabel.text = @"test...";
    //do sth...
    //...
    return cell;
}
```

当然我们可以使用Model，将具体的属性赋值操作转到cell内部来解决：

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    LOStyle_1_Cell *cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    cell.cellInfo = self.cellDataArray[indexPath.row];
    return cell;
}
```

但实际的代码可能还是看着庞杂，我们就先直接简单地新建一个类，把DataSource方法都移过去，这时候的代码可能类似这样：

```objective-c
@implementation LOStyle_1_View
- (void)initView {
    self.style_1_dataSource.cellDataArray = self.cellDataArray;
    self.tableView.dataSource = self.style_1_dataSource;
}
@end

@interface LOStyle_1_DataSource : NSObject<UITableViewDataSource>
@property (nonatomic, strong) NSArray<LOStyle_1_CellModel *> *cellDataArray;
@end
    
@implementation LOStyle_1_DataSource
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    LOStyle_1_Cell *cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    cell.cellInfo = self.cellDataArray[indexPath.row];  //Tag_1
    return cell;
}
@end
```

注意这里和原作者DEMO有所不同的是，原作者DEMO将`Tag_1`处的赋值代码用Block或代理又交回给了持有`TableView`的`LOStyle_1_View`；就目前的代码而言，我们好像觉得那样做还没什么必要。接下来，我们在`LOStyle_1_View`里放置一个按钮假装成网络请求的回调，按一下按钮，`self.cellDataArray`的值就会改变；这下我们必须保证`LOStyle_1_DataSource`持有的`cellDataArray`值先更新，然后才能去刷新`TableView`，代码可能会变成这样：

```objective-c
- (void)dataButtonEvent {
    //get new data...
    self.cellDataArray = [newDataArray copy];
    self.style_1_dataSource.cellDataArray = self.cellDataArray;
    [self.tableView reloadData];
}
```

这个时候我们应当注意到事情有些麻烦了：如果`style_1_dataSource`有多个属性，或者有多个网络请求等等多个落点的情况，写`self.style_1_dataSource`的地方就越来越多，可能还需要注意先后顺序，那再这样写肯定是难以接受的，怎么办？在目前的情况下，data和view都是由`LOStyle_1_View`来持有，那么我们可以仿照数据绑定的思路，请出一个`dataHelper`类来帮忙：`dataHelper`类管理所有数据并由`LOStyle_1_View`持有，这样`style_1_dataSource`只要从一开始取一下`dataHelper`就好，来看一下具体实现：

```objective-c
@interface LOStyle_1_DataHelper : NSObject
@property (nonatomic, strong) NSArray<LOStyle_1_CellModel *> *cellDataArray;
@end
    
@interface LOStyle_1_DataSource ()
@property (nonatomic, weak) LOStyle_1_DataHelper *dataHelper;  //Tag_2
@end

@implementation LOStyle_1_DataSource
//Tag_3
- (instancetype)initWithDataHelper:(LOStyle_1_DataHelper *)dataHelper
{
    if (self = [super init]) {
        self.dataHelper = dataHelper;
    }
    return self;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.dataHelper.cellDataArray.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath 
{
    LOStyle_1_Cell *cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LOStyle_1_Cell class])];
    cell.cellInfo = self.dataHelper.cellDataArray[indexPath.row];
    return cell;
}
@end

@interface LOStyle_1_View()<UITableViewDelegate>
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) LOStyle_1_DataSource *style_1_dataSource;
@property (nonatomic, strong) LOStyle_1_DataHelper *cellDataHelper;
@end

@implementation LOStyle_1_View

- (void)moreDataButtonEvent
{
	//get more data...
    self.cellDataHelper.cellDataArray = [tempArray copy];
    [self.tableView reloadData];
}
@end
```

注意这里`Tag_2`用了弱引用，因为`dataHelper`由`LOStyle_1_View`维护，数据更新时`dataHelper`的值自然会随之改变，`DataSource`类撒手不管了；由于只需在初始化的时候取一次值，我们遵循设计规范在`Tag_3`处给出自定义的初始化方法，对外不暴露属性。现在`dataHelper`甚至可以拿到cell实例，和cell相关的一些操作甚至都可以扔给`dataHelper`去做来进一步拆分，当然这都是后话，我们可以灵活选择使用。回到上面`Tag_1`处给cell赋值的操作，这里进一步的优化内容在[issue1-2](https://github.com/darkThanBlack/LearnObjcIO/blob/master/issue1-%E6%9B%B4%E8%BD%BB%E9%87%8F%E7%9A%84ViewControllers/issue1-2/issue1-2.md)，现在暂先不讨论；随着业务变化，极有可能出现多个不同样式的cell，我们回头看一下原作者`datasource`的初始化示例代码：

```objective-c
self.photosArrayDataSource = [[ArrayDataSource alloc] initWithItems:photos
                                  cellIdentifier:PhotoCellIdentifier
                                  configureCellBlock:configureCell];
self.tableView.dataSource = self.photosArrayDataSource;
```

可以看出他在设计上是偏向于想要将整个`ArrayDataSource`复用的，但我觉得一般`DataSource`类如果撇去和cell相关的操作其实本身代码量并不多，由于我们是先对ViewController瘦身，可以先把整个Protocols拆到`DataSource`类里，如果这样还是庞杂那再由`Helper`来完成拆分和转交，不用再递回上层去；当然实践当中这部分就非常灵活了，博主也只是举个例子讲解一下思路，[我的DEMO代码](https://github.com/darkThanBlack/LearnObjcIO/tree/master/issue1-%E6%9B%B4%E8%BD%BB%E9%87%8F%E7%9A%84ViewControllers/issue1-1)放在GitHub上，写文章的时候回去看了一下自己的项目，我自己是利用这个思路来处理`CollectionView`的代理方法，同时发现如果大家想看更复杂一点的例子，可以去看一下美洽客服（不是广告！）聊天页面cell的实现，他们是在`Helper`类里面用代理再拆分，cell则利用继承来在具体的子cell内部更新数据，希望对大家有所帮助。



### 将业务逻辑、网络请求移到 Model 层中，必要时创建 Store 类

这里多是原作者的一些经验之谈，我将它们合并成一句话，其实这里涉及到旷古已久的`胖Model`和`瘦Model`之争，包括之前流行的`MVVM`思想也有相似之处，我个人认为这些其实都是对`MVC`的补充，`iOS`的`C`似乎更容易变得臃肿庞大，那么大家就来对它来进行拆分，大体上都是按照功能逻辑来进行腾挪，但具体的界限各人都有自己的心得经验，我是觉得即使是同一个人面对两套不同的业务，也没有一成之法去说一定要怎样怎样划分，设计模式本就是为了应对业务和需求的不断变化而生的，不要被自己定的规矩套死。当然这里针对作者的例子我也想说一下我的经验，同样是`User`类，我们假设`User`有一个`time`属性，这种情况下我们从服务器接收到的数据一般是不能直接展示的，有不少常见的写法：

* 当场直接写：

```objective-c
  //do some format...
  NSString *formatTimeString = [NSString stringWithFormat:@"format=%@", self.time];
  view.timeLabel.text = formatTimeString;
```

* 当前模块刚好有工具类：

```objective-c
  #import "HomeViewHelper.h"
  HomeViewHelper *helper = [[HomeViewHelper alloc]initWithUser:self.user];
  view.timeLabel.text = [helper userFormatTime];
```

* 工厂模式，代码会像是：

```objective-c
  #import "LOIFactory.h"
  view.timeLabel.text = [LOIFactory formatTime:self.user.time];
```

* 升级一下，像原作者一样用扩展来做：

```objective-c
  #import "User+Helper.h"
  view.timeLabel.text = [self.user formatTime];
```

* 增加属性，甚至直接重写Getter：

```objective-c
  @interface User : NSObject
  @property (nonatomic, assign) NSInteger time;
  @property (nonatomic, strong) NSString *formatTime;
  @end
      
  @implementation User
  - (NSString *)formatTime {
      return [NSString stringWithFormat:@"format=%@", self.time];  //do some format...
  }
```

* 连根拔起：

```objective-c
  @interface User : NSObject
  @property (nonatomic, strong) NSString *time;
  @end
  
  @implementation DataManager
  - (void)loadUserData {
      self.user.time = [NSString stringWithFormat:@"format=%@", self.userData.time];
  }
```

列了这么多写法，我想说的其实是相对于这些写法之间的差异，`User`类的性质有的时候更能决定它们孰优孰劣，甚至工具类本身的影响都更大：`User+Helper.h`还有什么功能？哪些地方会用到？今后会不会因为`User+Helper.h`其他功能的改动而不得不拆分`time`属性？这个格式转换功能其他类的属性会不会用到？这是一个非常简单的例子，但前面提出的一些问题其实涉及到APP整体的一些思考，我觉得有时候多去从全局的角度考虑一下，决定`time`属性的具体写法或许会更加容易。



### 将DataSource作为一个单独的类分离出来

我的个人习惯是所有的ViewController都有一个View与之对应，所有子控件都在这个View上布局，利用Xcode自带的代码片段功能实现起来会更轻松一些，在前面的示例代码中也可以看出我的这一习惯。我认为在UI层要小心的是复用的程度，清晰的模块划分有时候比复用几个小控件更重要。



### 注意ViewController与其他对象的通讯

诚如作者所言，这是一个复杂的主题，我也简单地介绍一下我日常使用的一些准则：

* 信息尽量通过属性单向链式传递，传递时涉及的属性越少越好，举个例子：

```objective-c
  VC_A.user = self.user;
  VC_B.uid = self.uid;
```

如果情况允许我肯定是倾向于使用`VC_B`的传值方式；即使`user`可能是单例，如果`VC_B`和`VC_A`之间有逻辑关系，那还是让`VC_A`用传值的方式递交过来更好。

* 杜绝空值传递，合法性判断越早做越好。



### 总结

本篇文章多数是经验之谈，作者讲了很多实用的技巧，但经验类的知识要想真正地消化吸收为我所用，还是需要真正地通过项目和需求实在碰撞过后才会有自己的体悟，这也是我的第一篇长笔记，凛冽寒冬，一提笔竟然写了六个多小时，也衷心祝福有缘看到本文的朋友，努力共勉！