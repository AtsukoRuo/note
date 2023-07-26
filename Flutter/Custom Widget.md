# Custom Widget

在Flutter中自定义组件有三种方式

- 通过组合其他组件：StatefulWidget以及StatelessWidget

- 自绘：`CustomPaint`， 它只是为了方便开发者封装的一个代理类

- `RenderObject`：RenderObject是一个抽象类，它定义了一个抽象方法paint(...)：

  ~~~dart
  void paint(PaintingContext context, Offset offset)
  ~~~

  `PaintingContext`代表组件的绘制上下文，通过`PaintingContext.canvas`可以获得`Canvas`，而绘制逻辑主要是通过`Canvas` API来实现



## CustomPaint & Canvas

在Flutter中，提供了一个`CustomPaint` 组件（继承自singleChildRenderWidget），它可以结合画笔`CustomPainter`来实现自定义图形绘制。

~~~dart
CustomPaint({
  Key key,
  this.painter, 
  this.foregroundPainter,
  this.size = Size.zero, 
  this.isComplex = false, 
  this.willChange = false, 
  Widget child, //子节点，可以为空
})
~~~

- `painter`: 背景画笔。显示在子节点的后面
- `foregroundPainter`: 前景画笔。显示在子节点的前面
- `size`：
  - 当child为null时，代表默认绘制区域大小
  - 有child则忽略此参数，画布尺寸则为child尺寸。
- `isComplex`：是否复杂的绘制，如果是，Flutter会应用一些缓存策略来减少重复渲染的开销。
- `willChange`：和`isComplex`配合使用，当启用缓存时，该属性代表在下一帧中绘制是否会改变。



如果`CustomPaint`有子节点，为了避免子节点不必要的重绘并提高性能，通常情况下都会将子节点包裹在`RepaintBoundary`组件中，这样会在绘制时就会创建一个新的绘制层（Layer），其子组件将在新的Layer上绘制，而父组件将在原来Layer上绘制：

~~~dart
CustomPaint(
  size: Size(300, 300), //指定画布大小
  painter: MyPainter(),
  child: RepaintBoundary(child:...)), 
)
~~~





`CustomPainter`中提定义了一个虚函数`paint`：

~~~dart
@override
void paint(Canvas canvas, Size size);
~~~

通过size可以获取canvas的大小



canvas画布提供了以下方法：

| API名称    | 功能   |
| ---------- | ------ |
| drawLine   | 画线   |
| drawPoint  | 画点   |
| drawPath   | 画路径 |
| drawImage  | 画图像 |
| drawRect   | 画矩形 |
| drawCircle | 画圆   |
| drawOval   | 画椭圆 |
| drawArc    | 画圆弧 |

现在我们还需要一个画笔，Flutter提供了`Paint`类来实现画笔。在`Paint`中，我们可以配置画笔的各种属性如粗细、颜色、样式等。如：

~~~dart
var paint = Paint() //创建一个画笔并配置其属性
  ..isAntiAlias = true //是否抗锯齿
  ..style = PaintingStyle.fill //画笔样式：填充
  ..color=Color(0x77cdb175);//画笔颜色
~~~





一个例子

~~~dart
void main() {

  runApp(MaterialApp(
    home : Scaffold(
      body: CustomPaintRoute()
    )
  ));
}



class CustomPaintRoute extends StatelessWidget {
  const CustomPaintRoute({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          CustomPaint(
            size: Size(300, 300), //指定画布大小
            painter: MyPainter(),
          ),
          //添加一个刷新button
          ElevatedButton(onPressed: () {}, child: Text("刷新"))
        ],
      ),
    );
  }
}

class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    print("paint");
    var rect = Offset.zero & size;
    drawChessboard(canvas, rect);
    drawPieces(canvas, rect);
  }

  void drawChessboard(Canvas canvas, Rect rect) {
    var paint = Paint()
      ..isAntiAlias = true
      ..style = PaintingStyle.fill
      ..color = Color(0xFFDCC48C);

     //画棋盘网格
    paint
      ..style = PaintingStyle.stroke //线
      ..color = Colors.black38
      ..strokeWidth = 1.0;

    for (int i = 0; i <= 15; ++i) {
      double dy = rect.top + rect.height / 15 * i;
      canvas.drawLine(Offset(rect.left, dy), Offset(rect.right, dy), paint);
    }

    for (int i = 0; i <= 15; ++i) {
      double dx = rect.left + rect.width / 15 * i;
      canvas.drawLine(Offset(dx, rect.top), Offset(dx, rect.bottom), paint);
    }

  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;

  void drawPieces(Canvas canvas, Rect rect) {
    double eWidth = rect.width / 15;
    double eHeight = rect.height / 15;
    //画一个黑子
    var paint = Paint()
      ..style = PaintingStyle.fill
      ..color = Colors.black;
    //画一个黑子
    canvas.drawCircle(
      Offset(rect.center.dx - eWidth / 2, rect.center.dy - eHeight / 2),
      min(eWidth / 2, eHeight / 2) - 2,
      paint,
    );
    //画一个白子
    paint.color = Colors.white;
    canvas.drawCircle(
      Offset(rect.center.dx + eWidth / 2, rect.center.dy - eHeight / 2),
      min(eWidth / 2, eHeight / 2) - 2,
      paint,
    );
  }

}
~~~



### 绘制性能

绘制是比较昂贵的操作，所以我们在实现自绘控件时应该考虑到性能开销，下面是两条关于性能优化的建议：

- 尽可能的利用好`shouldRepaint`返回值；在UI树重新build时，控件在绘制前都会先调用该方法以确定是否有必要重绘；假如我们绘制的UI不依赖外部状态，即外部状态改变不会影响我们的UI外观，那么就应该返回`false`；如果绘制依赖外部状态，那么我们就应该在`shouldRepaint`中判断依赖的状态是否改变，如果已改变则应返回`true`来重绘，反之则应返回`false`不需要重绘。

- 绘制尽可能多的分层：优化的方法就是将棋盘单独抽为一个组件，并设置其`shouldRepaint`回调值为`false`，然后将棋盘组件作为背景。然后将棋子的绘制放到另一个组件中，这样每次落子时只需要绘制棋子。

- 防止意外重绘：点击按钮的时候发生了多次重绘。奇怪，`shouldRepaint` 我们返回的是false，并且点击刷新按钮也不会触发页面重新构建，那是什么导致的重绘呢？要彻底弄清楚这个问题得等到第十四章中介绍 Flutter 绘制原理时才行，现在读者可以简单认为，刷新按钮的画布和CustomPaint的画布是同一个。要解决这个问题的方案很简单，给刷新按钮 或 CustomPaint 任意一个添加一个 RepaintBoundary 父组件即可