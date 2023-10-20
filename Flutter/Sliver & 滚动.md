# Sliver & 滚动

[TOC]

## 概述

通常将滚动方向称为主轴，非滚动方向称为副轴

Flutter 中的可滚动组件主要由三个角色组成：Scrollable、Viewport 和 Sliver：

- Scrollable ：用于处理滑动手势，确定滑动偏移，滑动偏移变化时构建 Viewport 
- Viewport：显示的视窗，即列表的可视区域；
- Sliver：视窗里显示的元素



具体布局过程：

1. Scrollable 监听到用户滑动行为后，根据最新的滑动偏移构建 Viewport 。
2. Viewport 将当前视口信息和配置信息通过 SliverConstraints 传递给 Sliver。
3. Sliver 中对子组件（RenderBox）按需进行构建和布局，然后确认自身的位置、绘制等信息，保存在 geometry 中（一个 SliverGeometry 类型的对象）。



![](assets/6-1.7d0c763e.png)



## Scrollable

Scrollable 是一个可滚动的 Widget，它主要负责：

- 监听用户的手势，计算滚动状态发出 Notification
- 计算 offset 通知 listeners

Scrollable 本身不具有绘制内容的能力，它通过构造注入的 viewportBuilder 来创建一个 Viewport 来显示内容，当滚动状态变化的时候，Scrollable 就会不断的更新 Viewport 的 offset ，Viewport 就会不断的更新显示内容。

```dart
Scrollable({
  ...
  this.axisDirection = AxisDirection.down,
  this.controller,
  this.physics,
  required this.viewportBuilder,
})
```

- `axisDirection` 滚动方向。
- `physics`：此属性接受一个`ScrollPhysics`类型的对象，它决定可滚动组件如何响应用户操作。
- `controller`：控制滚动位置和监听滚动事件。
- `viewportBuilder`：当用户滑动时，Scrollable 会调用此回调构建新的 Viewport



Scrollable 主要结构如下：

~~~dart
Widget result = _ScrollableScope(
      scrollable: this,
      position: position,
      child: Listener(
        onPointerSignal: _receivedPointerSignal,
        child: RawGestureDetector(
          gestures: _gestureRecognizers,
          ...,
          child: Semantics(
            ...
            child: IgnorePointer(
			...
              child: widget.viewportBuilder(context, position),
            ),
          ),
        ),
      ),
    );
~~~

- _ScrollableScope 继承自 InheritedWidget，这样它的 children 可以方便的获取 scrollable 和 position；

- RawGestureDetector 负责手势监听，手势变化时会回调 _gestureRecognizers；

- viewportBuilder 会生成 viewport；

## Viewport

~~~dart
Viewport({
  Key? key,
  this.axisDirection = AxisDirection.down,
  this.crossAxisDirection,
  this.anchor = 0.0,
  required ViewportOffset offset, 			// 用户的滚动偏移

  this.center,
  this.cacheExtent, 						// 预渲染区域

  this.cacheExtentStyle = CacheExtentStyle.pixel, 
  this.clipBehavior = Clip.hardEdge,
  List<Widget> slivers = const <Widget>[], 	// 需要显示的 Sliver 列表
})
~~~

- `offset`：该参数为Scrollabel 构建 Viewport 时传入，它描述了滚动偏移

- `CacheExtentStyle`
  - `pixel `，预渲染区域的具体像素长度为$cacheExtent$
  - ` viewport`，预渲染区域的具体像素长度为$cacheExtent * viewport$



ViewPort 有一些重要属性：

~~~dart
class Viewport extends MultiChildRenderObjectWidget {
  /// 主轴方向
  final AxisDirection axisDirection;
  /// 纵轴方向
  final AxisDirection crossAxisDirection;
    
  /// center 决定 viewport 的 zero 基准线，也就是 viewport 从哪个地方开始绘制，默认是第一个 sliver
  /// center 必须是 viewport slivers 中的一员的 key
  final Key center;
  
/// 锚点，取值[0,1]，和 zero 的相对位置，比如 0.5 代表 zero 被放到了 Viewport.height / 2 处
  final double anchor;
    
  /// 滚动的累计值，确切的说是 viewport 从什么地方开始显示
  final ViewportOffset offset;
    
  /// 缓存区域，也就是相对有头尾需要预加载的高度
  final double cacheExtent;

  /// children widget
  List<Widget> slivers；
}

~~~

假设每个 sliver 的 height 都相等且等于屏幕高度的 ⅕

![img](assets/172087a995e51ba4tplv-t2oaga2asx-jj-mark3024000q75.webp)



## ScrollPostion

ScrollPosition包含了Viewport 的滚动信息，它的主要成员变量如下：

~~~dart
abstract class ScrollPosition extends ViewportOffset with ScrollMetrics {
  // 滚动偏移量
  double _pixels;
    
  // 设置滚动响应效果，比如滑动停止后的动画
  final ScrollPhysics physics;
    
  // 保存当前的滚动偏移量到 PageStore 中，当 Scrollable 重建后可以恢复到当前偏移量
  final bool keepScrollOffset;
    
  // 最小滚动值
  double _minScrollExtent;
    
  // 最大滚动值
  double _maxScrollExtent;
  ...
}
~~~

## SliverConstraints

与BoxConstraints 约束类似，Sliver 布局采用 SliverConstraints 作为约束。

~~~dart
class SliverConstraints extends Constraints {
  // 主轴方向
  final AxisDirection axisDirection;
    
  // 滚动的方向（正向/反向）
  final GrowthDirection growthDirection;
    
  // 当前Sliver已经滑出可视区域的总偏移
  final double scrollOffset;
    
  // 上一个 sliver 覆盖当前 sliver 的长度（重叠部分的长度）
  // 当过量滚动时，该属性就为负数  
  final double overlap;
    
  // 当前Sliver在Viewport中的最大可以绘制的区域
  // 在过量滚动时，该  
  final double remainingPaintExtent;
    
  // Viewport在主轴方向的长度
  final double viewportMainAxisExtent;
    

  ...
}
~~~

![img](assets/1720880655116400tplv-t2oaga2asx-jj-mark3024000q75.webp)



## SliverGeometry

Sliver 则通过 SliverGeometry 反馈给 Viewport 需要占用多少空间量。

~~~dart
class SliverGeometry extends Diagnosticable {
  // sliver滚动的范围
  final double scrollExtent;
    
  // 绘制起点，相对于布局起点的
  final double paintOrigin;
    
  // 绘制范围
  final double paintExtent;
    
  // 布局范围
  final double layoutExtent;
    
  // 最大绘制大小，设置为paintExtent值即可
  final double maxPaintExtent;
    
  // 点击有效区域的大小，默认为paintExtent
  final double hitTestExtent;
    
  ...
}
~~~



## Sliver

Sliver 主要作用是对子组件进行构建和布局

Sliver 的布局协议如下：

1. Viewport 将当前布局和配置信息通过 SliverConstraints 传递给已经进入了构建区域的Sliver。
2. Sliver 确定自身的位置、绘制等信息，保存在 geometry 中（一个 SliverGeometry 类型的对象）。
3. Viewport 读取 geometry 中的信息来对 Sliver 进行布局和绘制。



## ScrollController

~~~dart
ScrollController({
  double initialScrollOffset = 0.0, //初始滚动位置
  this.keepScrollOffset = true,//是否保存滚动位置
  ...
})
~~~

当keepScrollOffset为true时，每次滚动结束，可滚动组件都会将滚动位置`offset`存储到`PageStorage`中。此后，可滚动组件重新创建时，再从PageStorage中恢复。

~~~dart
ListView(key:PageStorageKey(1), ...);
~~~



它的一些常用属性和方法如下：

- `offset`：可滚动组件当前的滚动位置。只在控制器唯一绑定一个可滚动组件时，才可以访问这个属性。

- `positions`：`ScrollController`会获取每一个可滚动组件的`ScrollPosition`对象，然后将这些`ScrollPosition`保存在该属性中。

  假设一个`ScrollController`同时被两个可滚动组件使用，那么我们可以通过如下方式分别读取他们的滚动位置：

  ~~~dart
  controller.positions.elementAt(0).pixels
  controller.positions.elementAt(1).pixels
  ~~~

- `jumpTo(double offset)`、`animateTo(double offset,...)`：这两个方法用于跳转到指定的位置。

  offset可以设置为负数，列表则会跳转至顶部后过量滚动，产生触顶动画并自动纠正至0.0

- `addListener`：在滚动值发生变化时调用

  ~~~dart
  _controller.addListener((){ print("现在的位置: ${_controller.offset}");});
  ~~~





我们来看一下如何向`ScrollController`注册滚动组件（一般不会用到）：
~~~dart
ScrollPosition createScrollPosition(
    ScrollPhysics physics,
    ScrollContext context,
    ScrollPosition oldPosition);

void attach(ScrollPosition position) ;
void detach(ScrollPosition position) ;
~~~

可滚动组件首先会调用`ScrollController`的`createScrollPosition()`方法来创建一个`ScrollPosition`来存储滚动位置信息。接着，可滚动组件会调用`attach()`方法，将创建的`ScrollPosition`添加到`ScrollController`的`positions`属性中，只有注册后`animateTo()` 和 `jumpTo()`才可以被调用。当可滚动组件销毁时，会调用`ScrollController`的`detach()`方法，将其`ScrollPosition`对象从`ScrollController`的`positions`属性中移除。

## ScrollConfiguration

`ScrollConfiguraion`和`Theme`一样，都是`inheritedWidget`。ScrollConfiguration 用于控制子控件的滚动行为

~~~dart
ScrollConfiguration(
  behavior: ScrollBehavior(),
  child: ...
}
~~~



## NotificationListener

 Scrollable 组件在滑动时就会发送**滚动通知**（ScrollNotification）。滚动通知的类型包括：

- ScrollStartNotification：滚动开始通知。
- UserScrollNotification：用户滚动通知，通常在用户改变滚动方向时触发。
- ScrollUpdateNotification：滚动更新通知。
- ScrollEndNotification：滚动终止通知
- OverscrollNotification：过度滚动通知



在接收到滚动事件时，参数类型为`ScrollNotification`，它包括一个`metrics`属性，它的类型是`ScrollMetrics`，该属性包含当前ViewPort及滚动位置等信息：

- pixels：当前绝对位置

- atEdge：是否在顶部或底部

- axis：垂直或水平滚动

- axisDirection：滚动方向描述是down还是up，这个受列表reverse影响，正序就是down倒序就是up，并不代表列表是上滑还是下滑

- extentAfter：视口底部距离列表底部有多大

- extentBefore：视口顶部距离列表顶部有多大

- extentInside：视口范围内的列表长度

- maxScrollExtent：最大滚动距离，列表长度-视口长度

- minScrollExtent：最小滚动距离

- viewportDimension：沿滚动方向视口的长度

- outOfRange：是否越过边界

~~~dart
NotificationListener<ScrollNotification>(
onNotification: (notification) {
  ScrollMetrics metrics = notification.metrics;
  print('ScrollNotification####################');
  print('pixels = ${metrics.pixels}');
  print('atEdge = ${metrics.atEdge}');
  print('axis = ${metrics.axis}');
  print('axisDirection = ${metrics.axisDirection}');
  print('extentAfter = ${metrics.extentAfter}');
  print('extentBefore = ${metrics.extentBefore}');
  print('extentInside = ${metrics.extentInside}');
  print('maxScrollExtent = ${metrics.maxScrollExtent}');
  print('minScrollExtent = ${metrics.minScrollExtent}');
  print('viewportDimension = ${metrics.viewportDimension}');
  print('outOfRange = ${metrics.outOfRange}');
  print('ScrollNotification####################');
  return false;
},
~~~



## ScrollBar

Scrollbar组件可以为大部分滚动列表添加滚动条，若需要在任何设备上都显示iOS风格的滚动条，则可以直接使用CupertinoScrollbar组件。

`thumbVisbility`属性设置滚动条是否一直可见。



## ListView

`ListView`是最常用的可滚动组件之一，它可以沿一个方向线性排布所有子组件，并且它也支持列表项懒加载。



它会尽量扩展到父Widget所允许的最大尺寸，而且将自己在滚动副轴上的长度，以紧约束的形式传递给它的子Widget。如果父约束是无边界（例如，其父Widget为Column），那么将会抛出异常。

在Flutter框架中，使用ListView组件的默认构造函数，会使其立即初始化children列表，即不支持懒加载，不推荐使用。

~~~dart
ListView({
  ...  
  //可滚动widget公共参数
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController? controller,
  bool? primary,
  ScrollPhysics? physics,
  EdgeInsetsGeometry? padding,
  
  //ListView各个构造函数的共同参数  
  double? itemExtent,
  Widget? prototypeItem, //列表项原型，后面解释
  bool shrinkWrap = false,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double? cacheExtent, // 预渲染区域长度
    
  //子widget列表
  List<Widget> children = const <Widget>[],
})
~~~

- `itemExtent`：该参数如果不为`null`，则会强制`children`的“长度”为`itemExtent`的值；这里的“长度”是指滚动方向上子组件的长度。

  >固定子组件的主轴尺寸可在大幅跳转时提升性能。某程序可通过滚动控制器实现一键跳转x逻辑像素的功能。若ListView无法提前确定每个元素的高度，则跳转时它必须依次加载这些元素并完成布局测量。然而，若每个元素的高度是固定的，则只需简单计算便可得知一共需跳过个元素

- `cacheExtent`：在ListView动态加载与回收元素时，除了屏幕上可见的子组件外，ListView还会在视窗范围外额外加载x个像素作为缓冲

- `scrollDirection`：设置滚动的方向。Axis.horizontal水平滚动

- `shrinkWrap`：该属性表示是否根据子组件的总长度来设置`ListView`的长度，默认值为`false`。

- `padding`：内边距

- `physics`，可以设置列表滚动的行为

- `addAutomaticKeepAlives`：ListView 会为每一个列表项添加一个 AutomaticKeepAlive 父组件。可以缓存列表项，保存其状态



推荐使用支持懒加载的`ListView.builder()`：

~~~java
ListView.builder({
  // ListView公共参数已省略  
  ...
  required IndexedWidgetBuilder itemBuilder,
  int itemCount,
  ...
})
 
~~~

- `itemBuilder`：它是列表项的构建器，类型为`IndexedWidgetBuilder`，返回值为一个widget。当列表滚动到具体的`index`位置时，会调用该构建器构建列表项。

- `itemCount`：列表项的数量，如果为`null`，则为无限列表。

  


`ListView.separated()`可以在生成的列表项之间添加一个分割组件，它比`ListView.builder`多了一个`separatorBuilder`参数，该参数是一个分割组件生成器。



`ListView.custom()`可以自己指定一个委托：

~~~dart
const ListView.custom({
  // 暂略其他无关内容...
  required this.childrenDelegate,
  // 暂略其他无关内容...
}) : assert(childrenDelegate != null),
~~~







在ListView组件对元素的动态加载与资源回收机制的作用下，移出屏幕的元素会被摧毁，其内部状态也一并销毁。有两种解决方案：

- 状态提升：采纳前端网页React框架中著名的Lift State Up（状态提升）思路，把列表中每个子组件的状态都提升到列表之上。不过在构建列表项时，需要根据状态来构造。

- KeepAlive：将列表项所在的State类融合AutomaticKeepAliveClientMixin类，最后覆写wantKeepAlive函数。如果返回true，那么将保存其内部状态，否则直接销毁。别忘了在`build()`内调用`super.build(context)`

  ~~~dart
  class _CounterState extends State<Counter> with AutomaticKeepAliveClientMixin {
      int _count = 0;
      
      @override
      Widget build(BuildContext context) {
          //...
      }
      
      @override
      bool get wantKeepAlive => _count != 0;
  }
  ~~~

  

## AnimatedList

AnimatedList 和 ListView 的功能大体相似，不同的是， AnimatedList 可以在列表中插入或删除节点。并且执行一个动画。



AnimatedList 是一个 StatefulWidget，它对应的 State 类型为 AnimatedListState，添加和删除元素的方法位于 AnimatedListState 中：

~~~dart
void insertItem(int index, { Duration duration = _kDuration });

void removeItem(int index, AnimatedListRemovedItemBuilder builder, { Duration duration = _kDuration }) ;
~~~



~~~dart
  const AnimatedList({
    super.key,
    required super.itemBuilder,
    super.initialItemCount = 0,
    super.scrollDirection = Axis.vertical,
    super.reverse = false,
    super.controller,
    super.primary,
    super.physics,
    super.shrinkWrap = false,
    super.padding,
    super.clipBehavior = Clip.hardEdge,
  }) : assert(initialItemCount >= 0);
~~~

- `initialItemCount`属性，可以设置初始时表项的数量。

- `itemBuilder`：

  ~~~dart
  itemBuilder: (
      BuildContext context,
      int index,
      Animation<double> animation,
  );
  ~~~

一个简单的使用示例：

~~~dart
class _MyHomePage extends State<MyHomePage> {
    final globalKey = GlobalKey<AnimatedListState>();
    var data = <String>[];				//data[i]表示第i个表项的数据
    
    @override
    Widget build(countext) {
        return AnimatedList(
          key: globalKey,	//需要绑定一个key，来访问这个AnimatedList的State
          initialItemCount: data.length,
          itemBuilder: (
            BuildContext context,
            int index,
            Animation<double> animation,
          ) {
            return FadeTransition(
              opacity: animation,
              child: buildItem(context, index),		//最好将表项抽象成根据index构造的表项，原因见下
            );
          },
        ),
    }
    
    Widget buildItem(Context, index) {
    	return Widget();
  	}
}

globalKey.currentState.insertItem(index);			
globalKey.currentState.removeItem(index, (context, animation) {
    return FadeTransition(
        child : buildItem(context, index);			//原表项被直接删除，所以这里需要一个Widget来应用删除动画。
    )
})
~~~

我们需要特别注意地是：调用 AnimatedListState 的插入和移除方法相当于发送一个通知：在什么位置执行插入或移除动画，它并不执行滚动操作。并且我们列表的渲染区域是由数据驱动的，这个数据是由我们单独维护的。

## ReorderableListView

该滚动列表支持拖拽排序

## GridView

GridView组件是一个可将元素显示为二维网格状的列表组件，并支持主轴方向滚动。它会尽量扩展到父Widget所允许的最大尺寸。

~~~dart
  GridView({
    Key? key,
    Axis scrollDirection = Axis.vertical,
    bool reverse = false,
    ScrollController? controller,
    bool? primary,
    ScrollPhysics? physics,
    bool shrinkWrap = false,
    EdgeInsetsGeometry? padding,
    required this.gridDelegate,  //下面解释
    bool addAutomaticKeepAlives = true,
    bool addRepaintBoundaries = true,
    double? cacheExtent, 
    List<Widget> children = const <Widget>[],
    ...
  })
~~~

`GridView`和`ListView`的大多数参数都是相同的。其中我们需要关注SliverGridDelegate属性，它说明网格该如何构建。Flutter框架已经提供了该委托的两种实现方式，分别是：

- SliverGridDelegateWithFixedCrossAxisCount（交叉轴方向固定数量的委托）

  ~~~dart
  SliverGridDelegateWithFixedCrossAxisCount({
    @required double crossAxisCount, 
    double mainAxisSpacing = 0.0,
    double crossAxisSpacing = 0.0,
    double childAspectRatio = 1.0,
  })
  ~~~

  - `crossAxisCount`：副轴子元素的数量
  - `mainAxisSpacing`：主轴方向的间距
  - `crossAxisSpacing`：副轴方向子元素的间距
  - `childAspectRatio`：子元素在副轴长度和主轴长度的比例

  子元素的大小是通过`crossAxisCount`和`childAspectRatio`两个参数共同决定的

- SliverGridDelegateWithMaxCrossAxisExtent（交叉轴方向限制最大长度的委托）

  ~~~dart
  SliverGridDelegateWithMaxCrossAxisExtent({
    double maxCrossAxisExtent,
    double mainAxisSpacing = 0.0,
    double crossAxisSpacing = 0.0,
    double childAspectRatio = 1.0,
  })
  ~~~
  
  - `maxCrossAxisExtent`：设置每个元素的在副轴上的最大尺寸。
  
    在不考虑间距的情况下，假设GridView在副轴上的长度为$L$，而在副轴上的子元素有$x$个，它的在副轴上的宽度为$a$，那么必须满足$x*a \leq L < (x+1)*a$。



`GridView.count`构造函数内部使用了`SliverGridDelegateWithFixedCrossAxisCount`，我们通过它可以快速的创建副轴固定数量子元素的`GridView`。

`GridView.extent`构造函数内部使用了`SliverGridDelegateWithMaxCrossAxisExtent`，我们通过它可以快速的创建副轴子元素为固定最大长度的`GridView`。



~~~dart
GridView.builder(
 ...
 required SliverGridDelegate gridDelegate, 
 required IndexedWidgetBuilder itemBuilder,
)
~~~



## PageView

Tab 换页效果、图片轮动以及TikTok上下滑页切换视频功能等等，这些都可以通过 PageView 轻松实现。



PageView会尽量扩展到父Widget所允许的最大尺寸，而且它的每个子组件的尺寸都会被紧约束为父级允许的最大尺寸。

~~~dart
PageView({
  Key? key,
  this.scrollDirection = Axis.horizontal, // 滑动方向
  PageController? controller,
  this.physics,
  List<Widget> children = const <Widget>[],
  this.onPageChanged,
  this.pageSnapping = true,
})
~~~

- pageSnapping：如需允许用户停留在相邻页面之间的任意位置，则可传入pageSnapping：false实现
- onPageChanged：无论pageSnapping是否开启，页码变化的回传函数都会在翻页过半时触发，而不是在动作完成后触发
- controller：接受一个`PageController（页面控制器）`，它继承于ScrollController（滚动控制器。jumpTo（300）可以跳转300个逻辑像素（受pageSnapping属性影响），jumpToPage（3）则可以直接跳转到第4个页面。还支持previousPage（前一页）和nextPage（后一页）方法

PageView.builder支持懒加载。

~~~dart
pageView.builder(
	itemCount : 20,
     itemBuilder(context, index) {
        return Widget()
    }
)
~~~



PageView 并没有缓存功能，一旦页面滑出屏幕它就会被销毁。可以使用`AutomaticKeepAliveClientMixin`来解决这一问题。



## SingleChildScrollView

通常`SingleChildScrollView`只应在期望的内容不会超过屏幕太多时使用。这是因为`SingleChildScrollView`不支持基于 Sliver 的延迟加载模型。

它将父Widget的约束传递给子组件，然后它会尽可能地匹配子组件的大小。当子组件中的溢出时，就会提供一个滚动条。当Column作为SingleChildScrollView的孩子时，会强制设置`mainAxisSize：MainAxisSize.min`

~~~dart
SingleChildScrollView({
  this.scrollDirection = Axis.vertical, //滚动方向，默认是垂直方向
  this.padding, 
  bool primary, 
  this.physics, 
  this.controller,
  this.child,
})
~~~



## CustomScrollView

`CustomScrollView` 组件来帮助我们创建一个公共的 Scrollable 和 Viewport ，然后它的 slivers 参数接受一个 Sliver 数组。为所有子 Sliver 提供一个共享的 Scrollable，然后统一处理指定滑动方向的滑动事件。

~~~dart
CustomScrollView(
    slivers: [
        SliverFixedExtentList(),
        SliverFixedExtentList(),
    ],
);
~~~



![图6-24](assets/6-24.37e61c9d.png)



用户体验上就是Sliver一个接着一个地滚动下来，而不是分别滚动。

![图6-22](assets/6-22.8b8283a7.gif)

![图6-23](assets/6-23.74a615bf.gif)

| Sliver名称                | 功能                               | 对应的可滚动组件                 |
| ------------------------- | ---------------------------------- | -------------------------------- |
| SliverList                | 列表                               | ListView                         |
| SliverFixedExtentList     | 高度固定的列表                     | ListView，指定`itemExtent`时     |
| SliverAnimatedList        | 添加/删除列表项可以执行动画        | AnimatedList                     |
| SliverGrid                | 网格                               | GridView                         |
| SliverPrototypeExtentList | 根据原型生成高度固定的列表         | ListView，指定`prototypeItem` 时 |
| SliverFillViewport        | 包含多个子组件，每个都可以填满屏幕 | PageView                         |

还有一些用于对 Sliver 进行布局、装饰的组件，它们的子组件必须是 Sliver

| Sliver名称                      | 对应 RenderBox      |
| ------------------------------- | ------------------- |
| SliverPadding                   | Padding             |
| SliverVisibility、SliverOpacity | Visibility、Opacity |
| SliverFadeTransition            | FadeTransition      |
| SliverLayoutBuilder             | LayoutBuilder       |

还有一些其他常用的 Sliver：

| Sliver名称             | 说明                                                |
| ---------------------- | --------------------------------------------------- |
| SliverAppBar           | 对应 AppBar，主要是为了在 CustomScrollView 中使用。 |
| SliverToBoxAdapter     | 一个适配器，可以将 RenderBox 适配为 Sliver          |
| SliverPersistentHeader | 滑动到顶部时可以固定住                              |

### SliverFixedExtentList

~~~dart
SliverFixedExtentList(
    itemExtent: 56, //列表项高度固定
    delegate: SliverChildBuilderDelegate(
        (_, index) => ListTile(title: Text('$index')),
        childCount: 10,
    ),
);
~~~

### SliverToBoxAdapter

`SliverToBoxAdapter `组件可以将 RenderBox 适配为 Sliver，这样我们可以往 CustomScrollView 中添加一些非Sliver版本的组件。如果该SliverToBoxAdapter的孩子为可滚动组件，那么该孩子不会与CustomScrollView共享一套滚动逻辑。

### SliverFillRemaining

SliverFillRemaining会自动充满视图的全部空间，通常用于slivers的最后一个。

### SliverPersistentHeader


~~~dart
const SliverPersistentHeader({
  Key? key,
  required SliverPersistentHeaderDelegate delegate, 	// 构造 header组件的委托
  this.pinned = false, 				
  this.floating = false, 
})
~~~

- `floating`：

  - false：与其他Sliver一样，滚动到相应位置时才显示。
  - true：当用户开始向下滚动时，立即滚动下来。

- `pinned`：是否固定在视窗顶部。

  - false：滚动减少到`minExtent`时，继续滚动
  - true：滚动减少到`minExtent`时，吸附到顶部。

  对于HeaderB来说，如果已经有floating属性为false的、固定在顶部的HeaderA，那么这个HeaderA的底部就是当前HeaderB的顶部。

- `delegate`：用于生成 header 的委托

  ~~~dart
  abstract class SliverPersistentHeaderDelegate {
    double get maxExtent;
    double get minExtent;
    Widget build(BuildContext context, double shrinkOffset, bool overlapsContent);
    bool shouldRebuild(covariant SliverPersistentHeaderDelegate oldDelegate;
                       
    //其他属性
  }
  ~~~
  
  - `maxExtent`、`minExtent`：这个伸缩头部的大小在`[minExtent, maxExtent]`内变化
  - `shouldRebuild`：是否需要重新构建
  - `build`：构建Header里的子组件。当开始滚动伸缩时，不断调用build函数。子组件的最大约束为Header的大小。
    	-  `shrinkOffset`参数：当Header位于顶部时，其滚动偏移量，范围在$[0, maxExtent]$。



### SliverLayoutBuilder

这个组件是LayoutBuilder组件的Sliver版本，可用于在程序运行时获取上级的SliverConstraints约束信息。

~~~dart
SliverLayoutBuilder(
	builder : (BuildContext context, SliverConstraints constraints) => SliverToBoxAdapter();
)
~~~





