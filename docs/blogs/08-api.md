# API 拾遗

> Created: 2024/01/09



#### Intro

制作 ``Chainable`` 需要遍历 API，将我没用过，不理解的，且认为有一定价值的 API 列出作为目录。



#### UITableView



```swift

		// 数据预加载，实现也不难，问题在于正常情况下性能瓶颈不在这里
    @available(iOS 10.0, *)
    weak open var prefetchDataSource: UITableViewDataSourcePrefetching?

    @available(iOS 15.0, *)
    open var isPrefetchingEnabled: Bool
		
  	// Drag & Drop，很重要，不多说
    @available(iOS 11.0, *)
    weak open var dragDelegate: UITableViewDragDelegate?

    @available(iOS 11.0, *)
    weak open var dropDelegate: UITableViewDropDelegate?
		
    // TODO: wwdc 2022
		@available(iOS 16.0, *)
    open var selfSizingInvalidation: UITableView.SelfSizingInvalidation

  	// 3D Touch 菜单
    @available(iOS 14.0, *)
    open var contextMenuInteraction: UIContextMenuInteraction? { get }
		
		// 不常用
		rect(for:)
		
		@available(iOS 15.0, *)
    open func reconfigureRows(at indexPaths: [IndexPath])
	
    open var sectionIndexMinimumDisplayRowCount: Int // show special section index list on right when row count reaches this value. default is 0
		
    @available(iOS 8.0, *)
    @NSCopying open var separatorEffect: UIVisualEffect? // effect to apply to table separators

    @available(iOS 9.0, *)
    open var cellLayoutMarginsFollowReadableWidth: Bool // if cell layout margins are derived from the width of the readableContentGuide. default is NO.
		
		// TODO: Focus
    @available(iOS 9.0, *)
    open var remembersLastFocusedIndexPath: Bool
    
    @available(iOS 14.0, *)
    open var selectionFollowsFocus: Bool

    @available(iOS 15.0, *)
    open var allowsFocus: Bool
    
    @available(iOS 15.0, *)
    open var allowsFocusDuringEditing: Bool
    
    UISpringLoadedInteractionSupporting
  
```

