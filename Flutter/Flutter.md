# Flutter

[TOC]

## 快捷键 & 工具

shift + alt + F 格式化代码

## Flutter

在当前目录下运行`flutter create ${your_project_name}`，即可创建一个Flutter项目。

Material.io是Google Material Design的官方网站，上面有很多示例。在pubspec.yaml中管理、包，其中在dependencies项中添加所需的依赖。



Flutter Uis Are Built With Widgets。Widgets Tree is Combination (or nesting) of widgets 

![image-20230610142251776](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230610142251776.png)



一定要合理拆分Widget，提高代码的可读性以及复用性。

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
  String data, {
  Key? key,
});
~~~





~~~dart
TextStyle TextStyle({
  bool inherit = true,
  Color? color,
  double? fontSize,
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



## 按钮

### TextButton

~~~dart
(new) TextButton TextButton({
  required void Function()? onPressed,
  required Widget child,		//一般为Text
  ButtonStyle? style
})
    
TextButton(onPressed : () {}, child : Text("Hello World"), style: TextButton.styleFrom(textStyle : TextStyle(fontSize : 28)));
~~~



## 布局

### Scaffold

一般来说，总是定义一个 Scaffold 当作实参传入到 MaterialApp 的 home 属性。换句话说，一个 MaterialApp 总是绑定一个 Scaffold。Scaffold定义了一个 UI 框架，这个框架包含 头部导航栏，body，右下角浮动按钮，底部导航栏等。

- 

### Row()、Column()

Column()、Row()可以将多个子组件紧挨着放置在一起

![image-20230623153919423](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230623153919423.png)

~~~dart
child: Center(
  child: Column(mainAxisSize: MainAxisSize.min, children: [
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
        textStyle: const TextStyle(fontSize: 28)))
])),
~~~

推荐使用SizedBox来代替Padding。因为SizedBox固定大小后，可以将子组件的溢出部分裁剪掉。



## StatelessWidget & StatefulWidget

StatelessWidget用于渲染内容不改变的Widget。而StateWidget恰恰相反，用于渲染内容改变的Widget

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
class DiceRoller extends StatefulWidget {
  const DiceRoller({super.key});
  @override
  State<DiceRoller> createState() {
    return _DiceRolerState();
  }
}

class _DiceRolerState extends State<DiceRoller> {
  var activeDiceImage = 'assets/images/dice-2.png';
  void rollDice() {
    setState(() {
      activeDiceImage = 'assets/images/dice-4.png';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(mainAxisSize: MainAxisSize.min, children: [
      Image.asset(
        activeDiceImage,
        width: 200,
      ),
      const SizedBox(
        height: 20,
      ),
      TextButton(
        onPressed: rollDice,
        style: TextButton.styleFrom(
            foregroundColor: Colors.white,
            textStyle: const TextStyle(fontSize: 28)),
        child: const Text("Roll Dice"),
      ),
    ]);
  }
}

~~~

