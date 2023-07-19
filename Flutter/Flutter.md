# Flutter

[TOC]

## 文档

https://docs.flutter.dev/ui/widgets?ref=embrace.io/blog&gclid=EAIaIQobChMI2evPoMKIgAMVOMFMAh1_CwmREAAYASAAEgKJgvD_BwE&gclsrc=aw.ds





## 快捷键 & 工具 & 第三方包

shift + alt + F 格式化代码

强烈推荐 Flutter DevTools，可以分析性能、网络延迟、实时布局修改、布局显示！

`ctrl + shift + P`打开命令面板，然后输入flutter devtool即可



生成唯一UID的包

~~~shell
flutter pub add uuid
~~~

```dart
import 'package:uuid/uuid.dart';
```



本地化包

~~~shell
flutter pub add intl
~~~



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



## 基础组件

### 文本

~~~dart
Text Text(
  String data, {				//字符串
  Key? key,
  TextStyle? style,				
  TextAlign? textAlign,			//对齐方式
  TextOverflow? overflow,		//指定截断方式
  int? maxLines,				//指定文本显示的最大行数
  Color? selectionColor,
})
~~~

- 截断方式`TextOverflow.ellipsis`，它会将多余文本截断后以省略符“...”表示；



~~~dart
Text("Hello world",
  style: TextStyle(
    color: Colors.blue,
    fontSize: 18.0,
    height: 1.2,  
    fontFamily: "Courier",
    background: Paint()..color=Colors.yellow,
    decoration:TextDecoration.underline,
    decorationStyle: TextDecorationStyle.dashed
  ),
);
~~~



![3-3](data:image/png;base64,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)

### 字体

要将字体文件打包到应用中，和使用其他资源一样，要先在`pubspec.yaml`中声明它。然后将字体文件复制到在`pubspec.yaml`中指定的位置，如：

~~~yaml
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: assets/fonts/Raleway-Regular.ttf
        - asset: assets/fonts/Raleway-Medium.ttf
          weight: 500
        - asset: assets/fonts/Raleway-SemiBold.ttf
          weight: 600
    - family: AbrilFatface
      fonts:
        - asset: assets/fonts/abrilfatface/AbrilFatface-Regular.ttf
~~~

~~~dart
// 声明文本样式
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
);

// 使用文本样式
var buttonText = const Text(
  "Use the font for this text",
  style: textStyle,
);
~~~





可以通过`flutter pub add google_fonts`命令，在项目中引入第三方包，然后直接使用即可

~~~dart
Text(
    currentQuestion.text,
    style: GoogleFonts.poppins(
        color : Colors.white,
        fontSize : 24,
        fontWeight: FontWeight.bold
    ),
    textAlign: TextAlign.center,
),
~~~





### 颜色

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



### 图片

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
//Image.network("http://...", width : 100.0);
    
    
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



### 按钮

#### TextButton

`TextButton`即文本按钮，默认背景透明并不带阴影。按下后，会有背景色

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

#### OutlineButton

`OutlinedButton`默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)

~~~dart
OutlinedButton(
    onPressed: () {}, 
    style : OutlinedButton.styleFrom(
        foregroundColor: Colors.white
    ),
    child: //....
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

#### ElevatedButton

`ElevatedButton` 即"漂浮"按钮，它默认带有阴影和灰色背景。按下后，阴影会变大

~~~dart
return ElevatedButton(
    onPressed: onTap, 
    child: Text(answerText,),
    style: ElevatedButton.styleFrom(
        padding : const EdgeInsets.symmetric(
            vertical: 10,
            horizontal: 40
        ),
        backgroundColor: const Color.fromARGB(255, 33, 1, 95),
        foregroundColor: Colors.white,
        shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(40)
        )
    ),
);
~~~



#### IconButton

`IconButton`是一个可点击的Icon，不包括文字，默认没有背景，点击后会出现背景，如图3-9所示：

![图3-9](data:image/png;base64,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)

代码如下：

```dart
IconButton(
  icon: Icon(Icons.thumb_up),
  onPressed: () {},
)
```







`ElevatedButton`、`TextButton`、`OutlinedButton`都有一个`icon` 构造函数，通过它可以轻松创建带图标的按钮，如图3-10所示：

![图3-10](https://book.flutterchina.club/assets/img/3-10.6e8d650a.png)

代码如下：

```dart
ElevatedButton.icon(
  icon: Icon(Icons.send),
  label: Text("发送"),
  onPressed: _onPressed,
),
OutlinedButton.icon(
  icon: Icon(Icons.add),
  label: Text("添加"),
  onPressed: _onPressed,
),
TextButton.icon(
  icon: Icon(Icons.info),
  label: Text("详情"),
  onPressed: _onPressed,
),
```

### 透明



~~~dart
Opacity(
    opacity: 0.8,
    child : Image.asset(
        "assets\\images\\quiz-logo.png",
        width : 300
    ),
),
~~~





### Icon

~~~dart
Icon Icon(
  IconData? icon,
)
~~~

其中 IconData可以同Icons.XXXXX来获取内置的Icon



要使用自定义图标



## 表单以及输入框

`TextField`用于文本输入

~~~dart
const TextField({
  TextInputType keyboardType,
  InputDecoration decoration = const InputDecoration(),
  ValueChanged<String> onChanged,
  TextEditingController controller, 
})
~~~

- `decoration`：用于控制`TextField`的外观显示，如提示文本、背景颜色、边框等。

  ~~~dart
  decoration: InputDecoration(
      prefixText: '\$',			
      label: Text('Amount')		//提示文本
  ),
  ~~~

  

- `keyboardType`：用于设置该输入框默认的键盘输入类型，取值如下：

  | TextInputType枚举值 | 含义                                                |
  | ------------------- | --------------------------------------------------- |
  | text                | 文本输入键盘                                        |
  | multiline           | 多行文本，需和maxLines配合使用(设为null或大于1)     |
  | number              | 数字；会弹出数字键盘                                |
  | phone               | 优化后的电话号码输入键盘；会弹出数字键盘并显示“* #” |
  | datetime            | 优化后的日期输入键盘；Android上会显示“: -”          |
  | emailAddress        | 优化后的电子邮件地址；会显示“@ .”                   |
  | url                 | 优化后的url输入键盘； 会显示“/ .”                   |

- `onChange`：输入框内容改变时的回调函数

- `controller`：编辑框的控制器，通过它可以设置/获取编辑框的内容、选择编辑内容、监听编辑文本改变事件。大多数情况下我们都需要显式提供一个`controller`来与文本框交互。如果没有提供`controller`，则`TextField`内部会自动创建一个。

  ~~~dart
  final _titleController = TextEditingController();
  
  @override
  void dispose() {
      print(_titleController.text);
      _titleController.dispose();		//一定要主动删除它
      super.dispose();			
  }
  ~~~
  
  

## 可滚动组件

### SingleChildScrollView

```dart
SingleChildScrollView({
  //....
  this.child,
})
```

需要注意的是，**通常`SingleChildScrollView`只应在期望的内容不会超过屏幕太多时使用**，这是因为`SingleChildScrollView`不支持基于 Sliver 的延迟加载模型，所以如果预计视口可能包含超出屏幕尺寸太多的内容时，那么使用`SingleChildScrollView`将会非常昂贵（性能差），此时应该使用一些支持Sliver延迟加载的可滚动组件，如`ListView`。



### ListView

`ListView`是最常用的可滚动组件之一，它可以沿一个方向线性排布所有子组件，并且它也支持列表项懒加载（在需要时才会创建）。

`ListView.builder`适合列表项比较多或者列表项不确定的情况，下面看一下`ListView.builder`的核心参数列表：

~~~dart
ListView.builder({
  // ListView公共参数已省略  
  ...
  required IndexedWidgetBuilder itemBuilder,		//(BuildContext ctx, int index) => Widget
  int itemCount,
  ...
})
~~~

- `itemBuilder`：它是列表项的构建器，类型为`IndexedWidgetBuilder`，返回值为一个widget。当列表滚动到具体的`index`位置时，会调用该构建器构建列表项。
- `itemCount`：列表项的数量，如果为`null`，则为无限列表。



## 容器类组件

### Padding

`Padding`可以给其子节点添加填充（留白），和边距效果类似。

~~~dart
Padding({
  ...
  EdgeInsetsGeometry padding,
  Widget child,
})
~~~



### Scaffold 页面骨架

一般来说，总是定义一个 Scaffold 当作实参传入到 MaterialApp 的 home 属性。换句话说，一个 MaterialApp 总是绑定一个 Scaffold。Scaffold定义了一个 UI 框架，这个框架包含 头部导航栏，body，右下角浮动按钮，底部导航栏等。

~~~dart
Scaffold(
  appBar: AppBar( //导航栏
    title: Text("App Name"), 
    actions: <Widget>[ //导航栏右侧菜单，一般为按钮
      IconButton(icon: Icon(Icons.share), onPressed: () {}),
    ],
  ),
  drawer: MyDrawer(), //抽屉
  bottomNavigationBar: BottomNavigationBar( // 底部导航
    items: <BottomNavigationBarItem>[
      BottomNavigationBarItem(icon: Icon(Icons.home), title: Text('Home')),
      BottomNavigationBarItem(icon: Icon(Icons.business), title: Text('Business')),
      BottomNavigationBarItem(icon: Icon(Icons.school), title: Text('School')),
    ],
    currentIndex: _selectedIndex,		//当前高亮图标的索引
    fixedColor: Colors.blue,
    onTap: (index) {},					//index为 当前点击的Item的索引
  ),
  floatingActionButton: FloatingActionButton( //悬浮按钮
      child: Icon(Icons.add),
      onPressed:_onAdd
  ),
  body:			//主体部分，接受一个Widget
);
~~~

如果想要通过底部导航栏切换页面，那么要放在一个StatefulWidget中，并在onTap时触发setState，并根据index来决定body应用哪个Widget。

~~~dart
final _selectPageIndex = 0;
void _selectPage(int index) {
    setState(() {
        _selectPageIndex = index;
    });
}

@override
Widget build(BuildContext context) {
    Widget activePage = CategoriesScreen();
    if (_selectPageIndex == 1) {
        activePage = MealsScreen();
    }
    return Scaffold(
        body : activePage,
        bottomNavigationBar: BottomNavigationBar(
            onTap: _selectPage,
        ),
    );
}

~~~



### Container

`Container`是一个组合类容器，它本身不对应具体的`RenderObject`，它是`DecoratedBox`、`ConstrainedBox、Transform`、`Padding`、`Align`等组件组合的一个多功能容器，所以我们只需通过一个`Container`组件可以实现同时需要装饰、圆角、边距、变换、限制的场景。

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



~~~dart
Container(
    margin: EdgeInsets.only(top: 10),
    padding : EdgeInsets.all(20),
    decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(15),
        boxShadow: [
            BoxShadow(
                  color: Colors.black26,
                  blurRadius: 10.0,
                  spreadRadius: 2.0,
                  offset: Offset(2.0, 2.0), 
            )
        ]
    ),
);
~~~



## 布局类组件

### Expanded & Spacer

### Material widgets



~~~dart
MaterialApp(
    theme: ThemeData(useMaterial3: true),		//这需要你重新定义一遍所有的theme
    //ThemeData().copyWith()					//只需要定义需想要修改的即可，其他theme保持不动
    home : Expenses()
),
~~~



### SizedBox

`SizedBox`用于给子元素指定固定的宽高。SizedBox固定大小后，可以将子组件的溢出部分裁剪掉。

```dart
SizedBox(
  width: 80.0,
  height: 80.0,
  child: redBox
)
```

### Center

让子元素居中的同时，尽可能占据可用空间



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

- `mainAxisAlignment`：表示子组件在`Row`所占用的水平空间内对齐方式，

  - 如果`mainAxisSize`值为`MainAxisSize.min`，则此属性无意义，因为子组件的宽度等于`Row`的宽度。

  - `MainAxisAlignment.start`表示沿`textDirection`的初始方向对齐，如`textDirection`取值为`TextDirection.ltr`时，则`MainAxisAlignment.start`表示左对齐，`textDirection`取值为`TextDirection.rtl`时表示从右对齐。

  - `MainAxisAlignment.center`表示居中对齐

- `crossAxisAlignment`：它的取值和`MainAxisAlignment`一样(包含`start`、`end`、 `center`三个值)，
- 不同的是`crossAxisAlignment`的参考系是`verticalDirection`，即`verticalDirection`值为`VerticalDirection.down`时`crossAxisAlignment.start`指顶部对齐，`verticalDirection`值为`VerticalDirection.up`时，`crossAxisAlignment.start`指底部对齐；而



注意：如果当Row是Column的子元素时，Row的width尽可能大，而Column的width取决于Row的。这两者语义并不冲突，结果就是Column的width尽可能大



如果`Row`里面嵌套`Row`，或者`Column`里面再嵌套`Column`，那么只有最外面的`Row`或`Column`会占用尽可能大的空间，里面`Row`或`Column`所占用的空间为实际大小。可以用`Expanded`嵌套里面的`Row`即可。



### Size Contraints

Size Constraints

Widgets get sized based on their size preferences & parent widget size constraints

![image-20230715123804209](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230715123804209.png)



> 注：以下讨论均是从感性角度出发的，正确性得不到保证！！！

在Flutter中布局渲染大小以及布局约束是分开的，这是Flutter布局系统的一个特点。如果子widget的布局大小没有满足父widget的布局约束,通常会出现以下几种情况：

- 子widget被强制调整大小以适应约束

- 布局发生异常

- 子widget隐藏或显示不全，或者根据子元素来调整大小

  



举个例子

~~~dart
runApp(
	Row(
    	children : [
            TextField(),
        ]
    )
)
~~~

首先考虑Width情况：Row的默认大小是Infinite，也就是说它会尽可能地占据父元素的大小。根Widget的布局约束会使得（1）。而Row的布局约束是不得超过本身Widget，而TextField的宽度大小是Infinite，但是对于Row来说，违反布局约束会（2），这是与根Widget的约束不同。

在高度上，Row并没有约束，但是它不会隐藏子Widget，反而会根据子元素来调整大小



## 其他组件

### 底部弹框

~~~dart
showModalBottomSheet(
    isScrollControlled: true,			//最大高度显示
    context: context, 
    builder: (ctx) {
        return NewExpense(onAddExpense : _addExpense);
    }
);			//这个不是一个Widget，直接调用即可，注意路由管理
~~~

可以通过调用

~~~dart
Navigator.pop(context);
~~~

来手动关闭底部弹窗



### 日期

~~~dart
void _presentDatePicker() async {
    final now = DateTime.now();
    final firstDate = DateTime(now.year - 1, now.month, now.day);
    final pickedDate = await showDatePicker(
        context: context, 
        initialDate: now, 
        firstDate: firstDate, 
        lastDate: now,
    );
    //这个不是一个Widget，直接调用即可，注意路由管理
    //code
}

~~~



~~~dart
void _presentDatePicker(){
    final now = DateTime.now();
    final firstDate = DateTime(now.year - 1, now.month, now.day);
    showDatePicker(
        context: context, 
        initialDate: now, 				//默认时间
        firstDate: firstDate, 			//起始时间
        lastDate: now,				   //终止时间
        //有效时间 = [起始时间，终止时间]
    ).then((value) => {
        //code
    });
}
~~~



### 下拉框

~~~dart
DropdownButton(
    value : _selectedCategory,					//当前下拉框选中的项名（字符串）
    items: Categoty.values.map((categoty) => 	 //项
        DropdownMenuItem(
            value: categoty,				   //项的标识，对用户不可见
            child: Text(categoty.name.toUpperCase())
        )
    ).toList(), 
    onChanged: (value) {
        
    }
    //这个是一个Widget
),
~~~



### 提示框

~~~dart
showDialog(context: context, builder: (ctx) => 
    AlertDialog(
        title: const Text("Invalid input"),				//标题
        content : const Text('Please make sure a valid title, amount, data and category was entered.'),											//内容
        actions: [
            TextButton(								//一般按钮
                onPressed: () {
                    Navigator.pop(ctx);
                },
                child: const Text("okey"),
            )
        ],
    )
);
~~~



### 滑动清除

~~~dart
Dismissible(
    key : ValueKey(expenses[index]),				//Widget的key
    child: ExpenseItem(expenses[index]),			//Removed widget
    background: Container(
        margin : EdgeInsets.symmetric(
            horizontal: Theme.of(context).cardTheme.margin!.horizontal
        ),
        color : Theme.of(context).colorScheme.error.withOpacity(0.75),
    ),
    onDismissed: (direction) {						//必须调用，否则会报错
        setState(() {
             _registeredExpenses.remove(expense);
        })
    },
)
~~~



### SnackBar

~~~dart
ScaffoldMessenger.of(context).clearSnackBars();			
ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
        duration: Duration(seconds: 3),
        content: Text("Expense deleted."),
        action: SnackBarAction(
            label: "Undo",
            onPressed: () {
                setState(() {
                  _registeredExpenses.insert(expenseIndex, expense);
                });
            },
        ),
    )
);
~~~



### 主题

通过定义 `Theme`，我们可以更好地复用颜色、字体样式以及设置子组件的默认样式，从而让整个 app 的设计看起来更一致。

我们可以在MaterialApp中定义样式：

~~~dart
MaterialApp(
  title: appName,
  theme: ThemeData(
    // Define the default brightness and colors.
    brightness: Brightness.dark,
    primaryColor: Colors.lightBlue[800],
    // Define the default font family.
    fontFamily: 'Georgia',
  ),
);
~~~



创建一个 `Theme` Widget，对局部进行样式修改

~~~dart
Theme(
  // Create a unique theme with `ThemeData`
  data: ThemeData(
    splashColor: Colors.yellow,
    cardTheme: const CardTheme().copyWith(		//Theme Widget 会将这个修改后的 ThemeData 在它的 child 子树中进行传递。
    )
  ),
  child: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
);


//从父级 Theme 扩展
Theme(
  // Find and extend the parent theme using `copyWith`. See the next
  // section for more info on `Theme.of`.
  data: Theme.of(context).copyWith(splashColor: Colors.yellow),
  child: const FloatingActionButton(
    onPressed: null,
    child: Icon(Icons.add),
  ),
);
~~~



接下来就要使用样式了：

~~~dart
//应用局部样式
Container(
    color: Theme.of(context).colorScheme.secondary,		//应用局部样式，或者全局样式中其他组件的样式（不一定是默认样式）
    child: Container(
        margin : EdgeInsets.symmetric(
        horizontal: Theme.of(context).cardTheme.margin!.horizontal,
        child : Text(
            'Text with a background color',
            style: Theme.of(context).textTheme.titleLarge.copyWith(),		//你甚至可以在特殊化一下
    	),
    ),
    
),


//应用全局样式（修改了默认样式）
MaterialApp(
  theme: ThemeData(
    // Define the default `TextTheme`. Use this to specify the default
    // text styling for headlines, titles, bodies of text, and more.
    textTheme: const TextTheme(
      displayLarge: TextStyle(fontSize: 72.0, fontWeight: FontWeight.bold),
      titleLarge: TextStyle(fontSize: 36.0, fontStyle: FontStyle.italic),
      bodyMedium: TextStyle(fontSize: 14.0, fontFamily: 'Hind'),
    ),
  ),
);
~~~

`Theme.of(context)` 会查询 widget 树，并返回其中最近的 `Theme`。所以他会优先返回我们之前定义过的一个独立的 `Theme`，如果找不到，它会返回全局 theme。



### Card

~~~dart
Card(
    margin : const EdgeInsets.all(8),
    shape:  RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
    ),
    clipBehavior: Clip.antiAliasWithSaveLayer,		//设置圆框后，如何剪裁溢出的子组件
    elevation: 16,
)
~~~



## 状态
响应式的编程框架中都会有一个永恒的主题——“状态(State)管理”

### StatelessWidget & StatefulWidget

StatelessWidget用于初始化后自身状态不改变的Widget。而StatefulWidget恰恰相反，用于自身状态改变的Widget。注意：StatelessWidget父亲可以有StatefulWidget孩子

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
//一般都是通过DiceRoller与其他组件打交道，而真正的组件_DiceRollerState被其封装起来
class DiceRoller extends StatefulWidget {
  const DiceRoller({super.key});
    
  @override
  State<DiceRoller> createState() {
    return _DiceRolerState();				//返回具有状态的组件
  }
    
  void f() {}
}

//惯用命名 _XXXState 其中_表示私有类
class _DiceRollerState extends State<DiceRoller> {
  var activeDiceImage = 'assets/images/dice-2.png';
  void rollDice() {
    widget.f()					//widget是由State<DiceRoller>提供的getter、便于访问代理对象DiceRoller中的成员
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





### Widget的生命周期

Every Flutter Widget has a built-in **lifecycle**: A collection of methods that are **automatically executed** by Flutter (at certain points of time).

There are **three** extremely important (stateful) widget lifecycle methods you should be aware of:

- `initState()`: Executed by Flutter when the StatefulWidget's State object is **initialized**
- `build()`: Executed by Flutter when the Widget is built for the **first time** AND after `setState()` was called
- `dispose()`: Executed by Flutter right before the Widget will be **deleted** (e.g., because it was displayed conditionally)



### 状态管理

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





### 其他属性

- Key，唯一标识一个Widget。大部分情况下，你并不需要为Widget设置Key，直接为null即可。
- context：Widget的元信息，例如属性值、位置等
