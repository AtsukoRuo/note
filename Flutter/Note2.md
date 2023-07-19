| Widget                        | 说明             | 用途                                                         |
| ----------------------------- | ---------------- | ------------------------------------------------------------ |
| LeafRenderObjectWidget        | 非容器类组件基类 | Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。 |
| SingleChildRenderObjectWidget | 单子组件基类     | 包含一个子Widget，如：ConstrainedBox、DecoratedBox等         |
| MultiChildRenderObjectWidget  | 多子组件基类     | 包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等 |

布局类组件就是指继承`SingleChildRenderObjectWidget` 和`MultiChildRenderObjectWidget`的Widget。不同的布局类组件对子组件排列（layout）方式不同



继承关系 Widget > RenderObjectWidget > (Leaf/SingleChild/MultiChild)RenderObjectWidget 。

关于`RenderObject`我们现在只需要知道它是最终布局、渲染UI界面的对象即可。如`Stack`（层叠布局）对应的`RenderObject`就是`RenderStack`，而层叠布局的实现就在`RenderStack`中。



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

你可能会猜想 `Container` 的尺寸会在 70 到 150 像素之间，但并不是这样。 `ConstrainedBox` 仅对其从其父级接收到的约束下施加其他约束。

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

