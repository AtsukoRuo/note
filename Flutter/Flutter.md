# Flutter

[TOC]

## 文档

https://docs.flutter.dev/ui/widgets?ref=embrace.io/blog&gclid=EAIaIQobChMI2evPoMKIgAMVOMFMAh1_CwmREAAYASAAEgKJgvD_BwE&gclsrc=aw.ds

## 快捷键 & 工具

shift + alt + F 格式化代码

## Flutter

在当前目录下运行`flutter create ${your_project_name}`，即可创建一个Flutter项目。

Material.io是Google Material Design的官方网站，上面有很多示例。在pubspec.yaml中管理、包，其中在dependencies项中添加所需的依赖。



Flutter Uis Are Built With Widgets。Widgets Tree is Combination (or nesting) of widgets 

![image-20230610142251776](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230610142251776.png)



一定要合理拆分Widget，提高代码的可读性以及复用性。下面给出一个简单的示例

~~~dart
void main() {
  runApp(
    const MaterialApp(
      home:  Scaffold(
        body : GradientContainer(
          colors : [Colors.black, 
                    Colors.black87])
      ),
    ),
  );
}
const startAlignment = Alignment.topLeft;
const endAlignment = Alignment.bottomRight;
class GradientContainer extends StatelessWidget {
  const GradientContainer({required this.colors, super.key});
  final List<Color> colors;

  @override
  Widget build(context) {
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: colors,
          begin: startAlignment,
          end: endAlignment,
        ),
      ),
      child: const Center(
        child: StyledText("Hello World", Colors.white, 28.0),
      ),
    );
  }
}

class StyledText extends StatelessWidget {
  final String text;
  final Color color;
  final double fontSize;

  const StyledText(this.text, this.color, this.fontSize, {super.key});

  @override
  Widget build(context) {
    return Text(text,
        style: TextStyle(color: color, fontSize: fontSize));
  }
}

~~~



## 文本

~~~dart
Text Text(
  String data, {				//字符串
  Key? key,
  TextStyle? style,				
  StrutStyle? strutStyle,
  TextAlign? textAlign,
  TextDirection? textDirection,
  Locale? locale,
  bool? softWrap,
  TextOverflow? overflow,
  double? textScaleFactor,
  int? maxLines,
  String? semanticsLabel,
  TextWidthBasis? textWidthBasis,
  TextHeightBehavior? textHeightBehavior,
  Color? selectionColor,
})
~~~





~~~dart
(const) TextStyle TextStyle({
  bool inherit = true,
  Color? color,
  Color? backgroundColor,
  double? fontSize,
  FontWeight? fontWeight,
  FontStyle? fontStyle,
  double? letterSpacing,
  double? wordSpacing,
  TextBaseline? textBaseline,
  double? height,
  TextLeadingDistribution? leadingDistribution,
  Locale? locale,
  Paint? foreground,
  Paint? background,
  List<Shadow>? shadows,
  List<FontFeature>? fontFeatures,
  List<FontVariation>? fontVariations,
  TextDecoration? decoration,
  Color? decorationColor,
  TextDecorationStyle? decorationStyle,
  double? decorationThickness,
  String? debugLabel,
  String? fontFamily,
  List<String>? fontFamilyFallback,
  String? package,
  TextOverflow? overflow,
})
~~~



## 颜色

~~~dart
Color Color.fromARGB(
  int a,
  int r,
  int g,
  int b,
)
    
Color Color(int value)
/*
The bits are interpreted as follows:
Bits 24-31 are the alpha value.
Bits 16-23 are the red value.
Bits 8-15 are the green value.
Bits 0-7 are the blue value.
*/
    
Colors.white;
~~~



## 添加图片

在pubspec.yaml文件中 flutter下的assets中添加图片在项目中的路径即可

~~~yaml
flutter:
  assets:
    - assets/images/dice-1.png
    - assets/images/dice-2.png
    - assets/images/dice-3.png
~~~



~~~dart
Image Image.asset(
  String name, {		//在pubspec中的路径
  double? scale,
  double? width,
  double? height,
})
~~~





`Image` widget 有一个必选的`image`参数，它对应一个 ImageProvider。下面我们分别演示一下如何从 asset 和网络加载图片。

~~~dart
Image(
  image: AssetImage("images/avatar.png"),
  width: 100.0
);

Image(
  image: NetworkImage(
      "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4"),
  width: 100.0,
)
    
    
const Image({
  ...
  this.width, //图片的宽
  this.height, //图片高度
  this.color, //图片的混合色值
  this.colorBlendMode, //混合模式
  this.fit,//缩放模式
  this.alignment = Alignment.center, //对齐方式
  this.repeat = ImageRepeat.noRepeat, //重复方式
  ...
})
~~~

`fit`：该属性用于在图片的显示空间和图片本身大小不同时指定图片的适应模式。适应模式是在`BoxFit`中定义，它是一个枚举类型，有如下值：

- `fill`：会拉伸填充满显示空间，图片本身长宽比会发生变化，图片会变形。
- `cover`：会按图片的长宽比放大后居中填满显示空间，图片不会变形，超出显示空间部分会被剪裁。
- `contain`：这是图片的默认适应规则，图片会在保证图片本身长宽比不变的情况下缩放以适应当前显示空间，图片不会变形。
- `fitWidth`：图片的宽度会缩放到显示空间的宽度，高度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被剪裁。
- `fitHeight`：图片的高度会缩放到显示空间的高度，宽度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被剪裁。
- `none`：图片没有适应策略，会在显示空间内显示图片，如果图片比显示空间大，则显示空间只会显示图片中间部分。

![图3-12](https://book.flutterchina.club/assets/img/3-12.3ae1737a.png)

`color`和 `colorBlendMode`：在图片绘制时可以对每一个像素进行颜色混合处理，`color`指定混合色，而`colorBlendMode`指定混合模式，下面是一个简单的示例：

~~~dart
Image(
  image: AssetImage("images/avatar.png"),
  width: 100.0,
  color: Colors.blue,
  colorBlendMode: BlendMode.difference,
);
~~~

运行效果如图3-13所示（彩色）：

![图3-13](https://book.flutterchina.club/assets/img/3-13.feb336de.png)

通过这个可以透明化纯色图片

## 按钮

### TextButton

~~~dart
(new) TextButton TextButton({
  required void Function()? onPressed,
  required Widget child,		//一般为Text
  ButtonStyle? style
})
    
TextButton(
    onPressed : () {}, 
    child : Text("Hello World"), 
    style: TextButton.styleFrom(
        textStyle : TextStyle(fontSize : 28)
    )
);
~~~

### OutlineButton

~~~dart
OutlinedButton(
    onPressed: () {}, 
    style : OutlinedButton.styleFrom(
        foregroundColor: Colors.white
    ),
    child: 
)；
    
OutlinedButton.icon(
    onPressed: () {}, 
    style : OutlinedButton.styleFrom(
        foregroundColor: Colors.white
    ),
    icon : Icon(Icons.arrow_right_alt),
    label : const Text(				//相当于子元素
        "Start Quiz",
        style: TextStyle(

        ),
    ),
)
~~~

## 透明

~~~dart
Opacity(
    opacity: 0.8,
    child : Image.asset(
        "assets\\images\\quiz-logo.png",
        width : 300
    ),
),
~~~



## 布局

### center

让子元素居中的同时，尽可能占据可用空间

### Scaffold

一般来说，总是定义一个 Scaffold 当作实参传入到 MaterialApp 的 home 属性。换句话说，一个 MaterialApp 总是绑定一个 Scaffold。Scaffold定义了一个 UI 框架，这个框架包含 头部导航栏，body，右下角浮动按钮，底部导航栏等。

- 

### Row()、Column()

Column()、Row()可以将多个子组件紧挨着放置在一起

![image-20230623153919423](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230623153919423.png)

~~~dart
Row({
  ...  
  TextDirection textDirection,    
  MainAxisSize mainAxisSize = MainAxisSize.max,    
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  VerticalDirection verticalDirection = VerticalDirection.down,  
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  List<Widget> children = const <Widget>[],
})
~~~

- `mainAxisSize`：表示`Row`在主轴(水平)方向占用的空间，默认是`MainAxisSize.max`，表示尽可能多的占用水平方向的空间，此时无论子 widgets 实际占用多少水平空间，`Row`的宽度始终等于水平方向的最大宽度；而`MainAxisSize.min`表示尽可能少的占用水平空间，当子组件没有占满水平剩余空间，则`Row`的实际宽度等于所有子组件占用的水平空间；



~~~dart
child: Center(
  child: Column(
      mainAxisSize: MainAxisSize.min, 
      children: [
            Image.asset(
              'assets/images/dice-2.png',
              width: 200,
            ),
            const SizedBox(
              height: 20,
            ),
            TextButton(
                onPressed: rollDice,
                child: const Text("Roll Dice"),
                style: TextButton.styleFrom(
                    /*
                    padding: const EdgeInsets.only(
                      top: 20,
                    ),
                    padding: EdgeInsets.all()
                    */
                    foregroundColor: Colors.white,
                    textStyle: const TextStyle(fontSize: 28)
                )
            )，
        ]
  )
)
~~~

推荐使用SizedBox来代替Padding。因为SizedBox固定大小后，可以将子组件的溢出部分裁剪掉。



## Container

`Container`是一个组合类容器，它本身不对应具体的`RenderObject`，它是`DecoratedBox`、`ConstrainedBox、Transform`、`Padding`、`Align`等组件组合的一个多功能容器，所以我们只需通过一个`Container`组件可以实现同时需要装饰、变换、限制的场景。

下面是`Container`的定义：

~~~dart
Container({
  this.alignment,
  this.padding, //容器内补白，属于decoration的装饰范围
  Color color, // 背景色
  Decoration decoration, // 背景装饰
  Decoration foregroundDecoration, //前景装饰
  double width,//容器的宽度
  double height, //容器的高度
  BoxConstraints constraints, //容器大小的限制条件
  this.margin,//容器外补白，不属于decoration的装饰范围
  this.transform, //变换
  this.child,
  ...
})
~~~

- 容器的大小可以通过`width`、`height`属性来指定，也可以通过`constraints`来指定；如果它们同时存在时，`width`、`height`优先。实际上Container内部会根据`width`、`height`来生成一个`constraints`。
- `color`和`decoration`是互斥的，如果同时设置它们则会报错！实际上，当指定`color`时，`Container`内会自动创建一个`decoration`。

## StatelessWidget & StatefulWidget

StatelessWidget用于自身状态不改变的Widget。而StateWidget恰恰相反，用于自身状态改变的Widget

在StatelessWidget子类中必须覆写Widget build(BuildContext context)方法，它在Flutter框架首次渲染时调用该方法获取到要渲染的Widget

~~~dart
class GradientContainer extends StatelessWidget {
  const GradientContainer({super.key});
  @override
  Widget build(context) {
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: [color1, color2],
          begin: startAlignment,
          end: endAlignment,
        ),
      ),
      child: Center(child: DiceRoller()),
    );
  }
}

~~~



而StatefulWidget子类必须覆写StatefulWidget createState()方法，其中`State<T>`中的泛型参数说明要被管理状态的Widget。一般必须有一个私有类继承自State<Type>，并覆写build方法，然后在StatefulWidget子类中的createState方法返回。如果要改变渲染内容，必须在State<Type>类中调用setState方法。该方法通知Flutter框架重新调用build方法来渲染该组件（要意识到多态机制在起作用）

~~~dart
//仅仅起到返回状态组件的作用
class DiceRoller extends StatefulWidget {
  const DiceRoller({super.key});
    
  @override
  State<DiceRoller> createState() {
    return _DiceRolerState();
  }
}

//惯用命名 _XXXState 其中_表示私有类
//实际上的组件
class _DiceRolerState extends State<DiceRoller> {
  var activeDiceImage = 'assets/images/dice-2.png';
  void rollDice() {
    setState(() {
      activeDiceImage = 'assets/images/dice-4.png';
    });
  }

  @override
  Widget build(BuildContext context) {
    return TextButton(
        onPressed: rollDice,
        child: const Text("Roll Dice"),
    );
  }
}

~~~





## Widget的生命周期

Every Flutter Widget has a built-in **lifecycle**: A collection of methods that are **automatically executed** by Flutter (at certain points of time).

There are **three** extremely important (stateful) widget lifecycle methods you should be aware of:

- `initState()`: Executed by Flutter when the StatefulWidget's State object is **initialized**
- `build()`: Executed by Flutter when the Widget is built for the **first time** AND after `setState()` was called
- `dispose()`: Executed by Flutter right before the Widget will be **deleted** (e.g., because it was displayed conditionally)



## 状态管理

一种很典型的状态管理模式：

1. 保存当前活跃屏幕到状态类的变量中
2. 封装一个方法（在其中调用setState方法）来更新活跃屏幕
3. 屏幕组件通过调用该方法来触发切换

~~~dart
class _QuizState extends State<Quiz> {
    Widget? activeScreen = null;
	//Widget activeScreen = StartScreen(switchScreen);		//不可以，原因见初始化顺序
    
    @override
    void initState() {
        super.initState();
        activeScreen = StartScreen(switchScreen);
    }

    void switchScreen() {
        setState(() {
            activeScreen = const QuestionsScreen();
        });
    }
}

class StartScreen extends StatelessWidget {
    const StartScreen(this.switchMethod, {super.key});
    final void Function() switchMethod;
}
~~~

