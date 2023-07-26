# Note
[TOC]

## 布局

| Widget                        | 说明             | 用途                                                         |
| ----------------------------- | ---------------- | ------------------------------------------------------------ |
| LeafRenderObjectWidget        | 非容器类组件基类 | Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。 |
| SingleChildRenderObjectWidget | 单子组件基类     | 包含一个子Widget，如：ConstrainedBox、DecoratedBox等         |
| MultiChildRenderObjectWidget  | 多子组件基类     | 包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等 |

布局类组件就是指继承`SingleChildRenderObjectWidget` 和`MultiChildRenderObjectWidget`的Widget。不同的布局类组件所使用的布局算法不同，故对子组件排列（layout）方式不同



继承关系 Widget > RenderObjectWidget > (Leaf/SingleChild/MultiChild)RenderObjectWidget 。

关于`RenderObject`我们现在只需要知道它是最终布局、渲染UI界面的对象即可。如`Stack`（层叠布局，继承自SingleChildRenderObejctWidget）对应的`RenderObject`就是`RenderStack`，而层叠布局的实现就在`RenderStack`中。



Flutter 中有两种布局模型：

- 基于 RenderBox 的盒模型布局。
- 基于 Sliver ( RenderSliver ) 按需加载列表布局。



两种布局方式在细节上略有差异，但大体流程相同，布局流程如下：

1. 上层组件向下层组件传递约束（constraints）条件（最大/最小宽度，以及最大/最小高度）。
2. 下层组件确定自己的大小，然后告诉上层组件。**任何时候子组件都必须先遵守父组件的约束**，在此基础上再应用子组件约束。
3. 上层组件确定下层组件相对于自身的偏移（布局算法）和确定自身的大小（大多数情况下会根据子组件的大小来确定自身的大小）。

⚠️⚠️⚠️⚠️ **Flutter 的布局方式与 HTML 的布局差异相当大**

## Align

`Align` 组件可以调整子组件的位置，定义如下：

```dart
Align({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor,
  this.heightFactor,
  Widget child,
})
```

- `alignment` : 需要一个`AlignmentGeometry`类型的值，表示子组件在父组件中的起始位置。`AlignmentGeometry` 是一个抽象类，它有两个常用的子类：`Alignment`和 `FractionalOffset`，我们将在下面的示例中详细介绍。
- `widthFactor`和`heightFactor`是用于确定`Align` 组件本身宽高的属性；它们是两个缩放因子，会分别**乘以子元素的宽、高**，最终的结果就是`Align` 组件的宽高。如果值为`null`，则组件的宽高将会占用尽可能多的空间。



`Alignment`继承自`AlignmentGeometry`，表示矩形内的一个点，他有，`Alignment`定义如下：

```dart
Alignment(this.x, this.y)
```

- 两个属性`x`、`y`，分别表示在水平和垂直方向的偏移

- `Alignment` Widget会以**矩形的中心点作为坐标原点**
  $$
  offset_x = (Alignment.x * (parentWidth - childWidth) / 2 + (parentWidth - childWidth) / 2
  $$

  $$
  offset_y = Alignment.y * (parentHeight - childHeight) / 2 + (parentHeight - childHeight) / 2)
  $$

  

  如`Alignment(-1.0, -1.0)` 代表矩形的左侧顶点，而`Alignment(1.0, 1.0)`代表右侧底部终点



`FractionalOffset` 继承自 `Alignment`，它和 `Alignment`唯一的区别就是坐标原点不同！`FractionalOffset` 的坐标原点为矩形的左侧顶点
$$
FractionalOffset.x * (parentWidth - childWidth) \\
FractionalOffset.y * (parentHeight - childHeight)
$$


还有，熟悉Web开发的同学可能会发现`Align`组件的特性和Web开发中相对定位（`position: relative`）非常像，是的！在大多数时候，我们可以直接使用`Align`组件来实现Web中相对定位的效果，读者可以类比记忆。Align允许显示子Widget溢出的部分！但是有可能被其他Widget遮盖

~~~dart
runApp(MaterialApp(
    home : Scaffold(
      body : Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Align(
            heightFactor: 2,
            alignment: FractionalOffset(0.5, 6),
            child : FlutterLogo (
              size : 50
            )
          ),
          Container(
            color : Colors.red,
            height : 200,
            width : 200     //约束大小要求最小高度尽可能占用空间
          )
        ],
      )
    )
  ));
~~~

![屏幕截图 2023-07-18 145755](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\屏幕截图 2023-07-18 145755.png)





## 约束

![Example 1 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-1.png)

```dart
Container(color: red)
```

整个屏幕作为 `Container` 的父级，并且强制 `Container` 变成和屏幕一样的大小（最小高度约束）。

所以这个 `Container` 充满了整个屏幕，并绘制成红色。







![Example 2 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-2.png)

```dart
Container(width: 100, height: 100, color: red)
```

红色的 `Container` 想要变成 100 x 100 的大小，但是它无法变成，因为屏幕强制它变成和屏幕一样的大小（最小高度约束）。所以 `Container` 充满了整个屏幕。





![Example 5 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-5.png)

```dart
Center(
  child: Container(
      width: double.infinity, height: double.infinity, color: red),
)
```

屏幕强制 `Center` 变得和屏幕一样大，所以 `Center` 充满屏幕。

然后 `Center` 告诉 `Container` 可以变成任意大小，但是不能超出屏幕。现在，`Container` 想要无限的大小，但是由于它不能比屏幕更大（最大约束），所以就仅充满屏幕。



![Example 6 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-6.png)

```dart
Center(
  child: Container(color: red),
)
```

屏幕强制 `Center` 变得和屏幕一样大，所以 `Center` 充满屏幕。

然后 `Center` 告诉 `Container` 可以变成任意大小，但是不能超出屏幕。由于 `Container` 没有子级而且没有固定大小，所以它决定能有多大就有多大，所以它充满了整个屏幕。但是，为什么 `Container` 做出了这个决定？非常简单，因为这个决定是由 `Container` widget 的创建者决定的。





![Example 7 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-7.png)

```dart
Center(
  child: Container(
    color: red,
    child: Container(color: green, width: 30, height: 30),
  ),
)
```

屏幕强制 `Center` 变得和屏幕一样大，所以 `Center` 充满屏幕。

然后 `Center` 告诉红色的 `Container` 可以变成任意大小，但是不能超出屏幕。由于 `Container` 没有固定大小但是有子级，所以它决定变成它 child 的大小。

然后红色的 `Container` 告诉它的 child 可以变成任意大小，但是不能超出屏幕。

而它的 child 是一个想要 30 × 30 大小绿色的 `Container`。由于红色的 `Container` 和其子级一样大，所以也变为 30 × 30。由于绿色的 `Container` 完全覆盖了红色 `Container`，所以你看不见它了。





**BoxConstraints 是盒模型布局过程中父渲染对象传递给子渲染对象的约束信息**，包含最大宽高信息，子组件大小需要在约束的范围内，BoxConstraints 默认的构造函数如下：

```dart
const BoxConstraints({
  this.minWidth = 0.0, //最小宽度
  this.maxWidth = double.infinity, //最大宽度
  this.minHeight = 0.0, //最小高度
  this.maxHeight = double.infinity //最大高度
})
```

`BoxConstraints.tight(Size size)`，它可以生成固定宽高的限制；



> 约定：为了描述方便，如果我们说一个组件不约束其子组件或者取消对子组件约束时是指对子组件约束的最大宽高为无限大，而最小宽高为0，相当于子组件完全可以自己根据需要的空间来确定自己的大小



`ConstrainedBox`用于对子组件添加额外的约束



![Example 9 layout](https://flutter.cn/assets/images/docs/ui/layout/layout-9.png)

```dart
ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 70,
    minHeight: 70,
    maxWidth: 150,
    maxHeight: 150,
  ),
  child: Container(color: red, width: 10, height: 10),
)
```

你可能会猜想 `Container` 的尺寸会在 70 到 150 像素之间，但并不是这样。 **`ConstrainedBox` 仅对其从其父级接收到的约束下施加其他约束**。

在这里，屏幕迫使 `ConstrainedBox` 与屏幕大小完全相同，因此它告诉其子 `Widget` 也以屏幕大小作为约束，**从而忽略了其 `constraints` 参数带来的影响**。





```dart
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: double.infinity, //宽度尽可能大
    minHeight: 50.0 //最小高度为50像素
  ),
  child: Container(
    height: 5.0, 
    child: redBox ,
  ),
)
```

可以看到，我们虽然将Container的高度设置为5像素，但是最终却是50像素，这正是ConstrainedBox的最小高度限制生效了。但是最小宽度在这里没有意义，因为屏幕的最大高度约束忽略了其 `constraints` 参数带来的影响。





`SizedBox`用于给子元素指定固定的宽高

~~~dart
SizedBox(
  width: 80.0,
  height: 80.0,
  child: redBox
)
~~~

实际上`SizedBox`只是`ConstrainedBox`的一个定制，上面代码等价于：

```dart
ConstrainedBox(
  constraints: BoxConstraints.tightFor(width: 80.0,height: 80.0),
  child: redBox, 
)
```

而`BoxConstraints.tightFor(width: 80.0,height: 80.0)`等价于：

```dart
BoxConstraints(minHeight: 80.0,maxHeight: 80.0,minWidth: 80.0,maxWidth: 80.0)
```



### UnconstrainedBox

~~~dart
/// A widget that imposes no constraints on its child, allowing it to render
/// at its "natural" size.
///
/// This allows a child to render at the size it would render if it were alone
/// on an infinite canvas with no constraints. This container will then attempt
/// to adopt the same size, within the limits of its own constraints. If it ends
/// up with a different size, it will align the child based on [alignment].
/// If the box cannot expand enough to accommodate the entire child, the
/// child will be clipped.
~~~

~~~dart
ConstrainedBox(
  constraints: BoxConstraints(minWidth: 60.0, minHeight: 100.0),  //父
  child: UnconstrainedBox( 
    child: ConstrainedBox(
      constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),//子
      child: redBox,
    ),
  )
)
~~~





## LayoutBuilder

通过 LayoutBuilder，我们可以在**布局过程**中拿到父组件传递的约束信息，然后我们可以根据约束信息**实时**（类似StatefulWidget）的构建不同的布局。注意context参数只能获取到当前组件的大小，而且LayoutBuilder会动态更新渲染UI的。

比如我们实现一个响应式的 Column 组件 ResponsiveColumn，它的功能是当当前可用的宽度小于 200 时，将子组件显示为一列，否则显示为两列。简单来实现一下：

~~~dart
class ResponsiveColumn extends StatelessWidget {
  const ResponsiveColumn({Key? key, required this.children}) : super(key: key);

  final List<Widget> children;

  @override
  Widget build(BuildContext context) {
    // 通过 LayoutBuilder 拿到父组件传递的约束，然后判断 maxWidth 是否小于200
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints constraints) {
        if (constraints.maxWidth < 200) {
          // 最大宽度小于200，显示单列
          return Column(children: children, mainAxisSize: MainAxisSize.min);
        } else {
          // 大于200，显示双列
          var _children = <Widget>[];
          for (var i = 0; i < children.length; i += 2) {
            if (i + 1 < children.length) {
              _children.add(Row(
                children: [children[i], children[i + 1]],
                mainAxisSize: MainAxisSize.min,
              ));
            } else {
              _children.add(children[i]);
            }
          }
          return Column(children: _children, mainAxisSize: MainAxisSize.min);
        }
      },
    );
  }
}

~~~





## Widget

Flutter 中的 widget 的概念更广泛，它不仅可以表示UI元素，也可以表示一些功能性的组件如：用于手势检测的 `GestureDetector` 、用于APP主题数据传递的 `Theme` 等等。

Widget实际上就是一个配置信息类，Flutter根据Widget的配置信息，渲染出相应的UI元素。

下面我们先来看一下 Widget 类的声明：

~~~dart
@immutable // 不可变的
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key? key;

  @protected
  @factory
  Element createElement();

  @override
  String toStringShort() {
    final String type = objectRuntimeType(this, 'Widget');
    return key == null ? type : '$type-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  @override
  @nonVirtual
  bool operator ==(Object other) => super == other;

  @override
  @nonVirtual
  int get hashCode => super.hashCode;

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
  ...
}
~~~

- `widget`类继承自`DiagnosticableTree`，`DiagnosticableTree`即“诊断树”，主要作用是提供调试信息。
- `Key`: 用于`canUpdate()`方法中
- `canUpdate(...)`,它主要用于在 widget 树重新`build`时复用旧的 widget ，其实具体来说，应该是：是否用新的 widget 对象去更新旧UI树上所对应的`Element`对象的配置；通过其源码我们可以看到，只要`newWidget`与`oldWidget`的`runtimeType`和`key`同时相等时就会用`new widget`去更新`Element`对象的配置（然后依次考察它的子孩子），否则就会创建新的`Element`（此时旧Element的孩子Element都被抛弃掉）。这样可以**复用Element节点**。
- `@immutable`代表 Widget 是不可变的，也就是说Widget 中定义的属性（即配置信息）必须是不可变的（final）





## Tree

Flutter 框架的处理流程是这样的：

1. 根据 Widget 树生成一个 Element 树，Element 树中的节点都继承自 `Element` 类。

2. 根据 Element 树生成 Render 树（渲染树），渲染树中的节点都继承自`RenderObject` 类。

3. 根据渲染树生成 Layer 树，然后上屏显示，Layer 树中的节点都继承自 `Layer` 类。

   

![图2-2](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\2-2.59d95f72.png)

真正的布局和渲染逻辑在 Render 树中，Element 是 Widget 和 RenderObject 的粘合剂，可以理解为一个中间代理。

WidgetsTree与ElementTree是一一对应的。

这里说明一下WidgetTree何时更新，如果一个父StatefulWidget触发setState后，Flutter框架开始执行diff算法，比较旧Widget与新Widget之间的差别。如果canUpdate为true，那么就仅仅根据Widget更新Element即可，然后再次依次考察它的子孩子。如果是false，那么就用新的Widget更新WidgetTree

## Context

[BuildContext] objects are actually [Element] objects. The [BuildContext] ,interface is used to discourage direct manipulation of [Element] objects.

下面是在子树中获取父级 widget 的一个示例：

~~~dart
class ContextRoute extends StatelessWidget  {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Context测试"),
      ),
      body: Container(
        child: Builder(builder: (context) {
          // 在 widget 树中向上查找最近的父级`Scaffold`  widget 
          Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
          // 直接返回 AppBar的title， 此处实际上是Text("Context测试")
          return (scaffold.appBar as AppBar).title;
        }),
      ),
    );
  }
}
~~~

![](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\2-4.63daca67.png)



Elment 是继承自 BuildContext ，我们甚至可以通过 context 来直接刷新 Element 的状态，如下：

~~~dart
(context as Element).markNeedsBuild();
~~~

这样就可以直接对当前的 Element 进行刷新，而不必去通过 SetState，但是这种做法是极其的不推荐的。

其实在 SetState 中，最终也是调用的 `markNeedsBuild` 方法



- (context as Element).findAncestorStateOfType()

  沿着当前的 Element 向上寻找，直到直到一个特定的类型之后，将他的 State 返回

- (context as Element).findRenderObject()

  获取 Element 渲染的对象



## Key

the category of key can be :

- LocalKey，require that the key should be unique between those Widget who has same father Widget. 

  - #### `ValueKey`：使用特定类型的值来标识自身的键

  - #### `ObjectKey`：使用对象的内存地址来比较

  - #### `UniqueKey`

- GlobalKey：Key的作用域是全局的

  

## Stateful Widget

1. Stateful widget 至少由两个类组成：
   - 一个`StatefulWidget`类。
   
   - 一个 `State`类；  **`StatefulWidget`类本身是不变的**，但是`State`类中持有的状态在 widget 生命周期中可能会发生变化。一个 StatefulWidget 类会对应一个 State 类，State表示与其对应的 StatefulWidget 要维护的状态，State 中的保存的状态信息可以：
   
     

`setState`方法的作用是通知 Flutter 框架，有状态发生了改变，Flutter 框架收到通知后，会执行 `build` 方法来根据新的状态重新构建界面

~~~dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key? key, required this.title}) : super(key: key);
  final String title;
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
 ...
}
~~~



`StatefulWidget`的类定义：

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);
    
  @override
  StatefulElement createElement() => StatefulElement(this);
    
  @protected
  State createState();
}
```

- `createState()` 用于创建和StatefulWidget相关的状态，它在StatefulWidget 的生命周期中可能会被多次调用。例如，当一个 StatefulWidget 插入到 widget 树的多个位置时，Flutter 框架就会调用该方法为每一个位置生成一个独立的State实例，其实，本质上就是一个`StatefulElement`对应一个State实例，而且这些State实例可以共享用一个StatefulWidget中的属性。下面给出一个例子来说明一下：

  ~~~dart
  void main() {
    Widget box = Box();
    runApp(MaterialApp(
      home : Scaffold(
        body : Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[box,box,box,],)),));
  }
  class Box extends StatefulWidget {
    int _count = 0;
    @override _BoxState createState() => _BoxState();
  }
  
  class _BoxState extends State<Box> {
    int _state_count = 0;
    @override
    Widget build(BuildContext context) {
      return GestureDetector(
        child : Container(
          width : 100,
          height: 100,
          color : Colors.red,
          alignment : Alignment.center,
          child : Text(
            widget._count.toString() + " " + _state_count.toString(), 
            style: TextStyle(fontSize:30)
          ),
        ),
        onTap: () => setState(() => ++widget._count, ++_state_count),
      );
    }
  }
  ~~~

  ![1689819015304-screenshot](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\1689819015304-screenshot.png)



## State

State 中有两个常用属性：

1. `widget`，它表示与该 State 实例关联的StatefulWidget 实例，由Flutter 框架**动态**设置。这样就可以访问StatefulWidget中的属性与方法了
2. `context`。StatefulWidget对应的 BuildContext，作用同StatelessWidget 的BuildContext。



~~~dart
class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    //初始化状态
    _counter = widget.initValue;
    print("initState");
  }

  @override
  Widget build(BuildContext context) {
    print("build");
    return Scaffold(
      body: Center(
        child: TextButton(
          child: Text('$_counter'),
          //点击后计数器自增
          onPressed: () => setState(
            () => ++_counter,
          ),
        ),
      ),
    );
  }

  @override
  void didUpdateWidget(CounterWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print("didUpdateWidget ");
  }

  @override
  void deactivate() {
    super.deactivate();
    print("deactivate");
  }

  @override
  void dispose() {
    super.dispose();
    print("dispose");
  }

  @override
  void reassemble() {
    super.reassemble();
    print("reassemble");
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("didChangeDependencies");
  }
}
~~~



下面我们来看看各个回调函数：

- `initState`：当 widget 第一次插入到 widget 树时会被调用，对于每一个State对象，Flutter 框架只会调用一次该回调，所以，通常在该回调中做一些一次性的操作，如状态初始化、订阅子树的事件通知等。不能在该回调中调用`BuildContext.dependOnInheritedWidgetOfExactType`（该方法用于在 widget 树上获取离当前 widget 最近的一个父级`InheritedWidget`，关于`InheritedWidget`我们将在后面章节介绍），原因是在初始化完成后， widget 树中的`InheritFrom widget`也可能会发生变化，所以正确的做法应该在在`build（）`方法或`didChangeDependencies()`中调用它。

- `didChangeDependencies()`：当State对象的依赖发生变化时会被调用；例如：在之前`build()` 中包含了一个`InheritedWidget` （第七章介绍），然后在之后的`build()` 中`Inherited widget`发生了变化，那么此时`InheritedWidget`的子 widget 的`didChangeDependencies()`回调都会被调用。典型的场景是当系统语言 Locale 或应用主题改变时，Flutter 框架会通知 widget 调用此回调。需要注意，组件第一次被创建后挂载的时候（包括重创建）对应的`didChangeDependencies`也会被调用。

- `build()`：此回调读者现在应该已经相当熟悉了，它主要是用于构建 widget 子树的，会在如下场景被调用：
  
  1. 在调用`initState()`之后。
  
  2. 在调用`didUpdateWidget()`之后。
  
  3. 在调用`setState()`之后。
  
  4. 在调用`didChangeDependencies()`之后。
  
  5. 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其他位置之后。
  
     
  
- `reassemble()`：此回调是专门为了开发调试而提供的，在热重载(hot reload)时会被调用，此回调在Release模式下永远不会被调用。
  
- `didUpdateWidget ()`：在 widget 重新构建时，Flutter 框架会调用`widget.canUpdate`来检测 widget 树中同一位置的新旧节点，然后决定是否需要更新，如果`widget.canUpdate`返回`true`则会调用此回调。正如之前所述，`widget.canUpdate`会在新旧 widget 的 `key` 和 `runtimeType` 同时相等时会返回true，也就是说在在新旧 widget 的key和runtimeType同时相等时`didUpdateWidget()`就会被调用。
  
- `deactivate()`：当 State 对象从树中被移除时，会调用此回调。在一些场景下，Flutter 框架会将 State 对象重新插到树中，如包含此 State 对象的子树在树的一个位置移动到另一个位置时（可以通过GlobalKey 来实现）。如果移除后没有重新插入到树中则紧接着会调用`dispose()`方法。
  
- `dispose()`：当 State 对象从树中被永久移除时调用；通常在此回调中释放资源。
  

![图2-5](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\2-5.a59bef97.jpg)







## Design Style

Flutter 提供了一套丰富、强大的基础组件，在基础组件库之上 Flutter 又提供了一套 Material 风格（ Android 默认的视觉风格）和一套 Cupertino 风格（iOS视觉风格）的组件库

- 基础组件：`Text` `Row` `Stack` `Container`

- ### Material

- ### Cupertino

