# Note3

[TOC]

## Animation

![image-20230717164514334](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230717164514334.png)



**`Animation<T>`是一个抽象类**，它本身和UI渲染没有任何关系，而它主要的功能是保存动画的插值和状态；



### 动画通知

在动画的每一帧中，我们可以通过`Animation`对象的`value`属性获取动画的当前状态值。

我们可以通过`Animation`来监听动画每一帧以及执行状态的变化，`Animation`有如下两个方法：

1. `addListener()`；它可以用于给`Animation`添加帧监听器，在每一帧都会被调用。
2. `addStatusListener()`；它可以给`Animation`添加“动画状态改变”监听器；动画开始、结束、正向或反向（见`AnimationStatus`定义）时会调用状态改变的监听器。



Flutter中，有四种动画状态，在`AnimationStatus`枚举类中定义，下面我们逐个说明：

| 枚举值      | 含义             |
| ----------- | ---------------- |
| `dismissed` | 动画在起始点停止 |
| `forward`   | 动画正在正向执行 |
| `reverse`   | 动画正在反向执行 |
| `completed` | 动画在终点停止   |

~~~dart
ontroller = AnimationController(
  duration: const Duration(seconds: 1), 
  vsync: this,
);
//图片宽高从0变到300
animation = Tween(begin: 0.0, end: 300.0).animate(controller);
animation.addStatusListener((status) {
  if (status == AnimationStatus.completed) {
    //动画执行结束时反向执行动画
    controller.reverse();
  } else if (status == AnimationStatus.dismissed) {
    //动画恢复到初始状态时执行动画（正向）
    controller.forward();
  }
});

//启动动画（正向）
controller.forward();
~~~



## Curve

动画过程可以是匀速的、匀加速的或者先加速后减速等。Flutter中通过`Curve`（曲线）来描述动画过程

我们可以通过`CurvedAnimation`（继承`Animation<double>`）包装`AnimationController`和`Curve`生成一个新的动画对象 ，我们正是通过这种方式来将动画和动画执行的曲线关联起来的。

~~~dart
final CurvedAnimation curve = CurvedAnimation(parent: controller, curve: Curves.easeIn);
~~~



 [Curves (opens new window)](https://docs.flutter.io/flutter/animation/Curves-class.html)类是一个预置的枚举类，定义了许多常用的曲线，下面列几种常用的：

| Curves曲线 | 动画过程                     |
| ---------- | ---------------------------- |
| linear     | 匀速的                       |
| decelerate | 匀减速                       |
| ease       | 开始加速，后面减速           |
| easeIn     | 开始慢，后面快               |
| easeOut    | 开始快，后面慢               |
| easeInOut  | 开始慢，然后加速，最后再减速 |

当然我们也可以创建自己Curve，例如我们定义一个正弦曲线：

~~~dart
class ShakeCurve extends Curve {
  @override
  double transform(double t) {
    return math.sin(t * math.PI * 2);
  }
}
~~~





## AnimationController

`AnimationController`（继承`Animation<double>`用于控制动画，它包含动画的启动`forward()`、停止`stop()` 、反向播放 `reverse()`等方法。默认情况下，`AnimationController`在给定的时间段内线性的生成从 0.0 到1.0（默认区间）的数字。

 例如，下面代码创建一个`Animation`对象（但不会启动动画）：

~~~dart
final AnimationController controller = AnimationController(
      duration: const Duration(milliseconds: 2000),
      vsync: this,
);
~~~

`AnimationController`生成数字的区间可以通过`lowerBound`和`upperBound`来指定，如：

~~~dart
final AnimationController controller = AnimationController( 
    duration: const Duration(milliseconds: 2000), 
    lowerBound: 10.0,
    upperBound: 20.0,
    vsync: this
);
~~~



当创建一个`AnimationController`时，需要传递一个`vsync`参数，它接收一个`TickerProvider`类型的对象，它的主要职责是创建`Ticker`，定义如下：

~~~dart
abstract class TickerProvider {
  //通过一个回调创建一个Ticker
  Ticker createTicker(TickerCallback onTick);
}
~~~

通常我们会将`SingleTickerProviderStateMixin`添加到`State`的定义中，然后将State对象作为`vsync`的值

> Flutter 应用在启动时都会绑定一个`SchedulerBinding`，通过`SchedulerBinding`可以给每一次屏幕刷新添加回调。而Ticker注册到SchedulerBinding中，这样每次屏幕刷新时都会调用Ticker中的TickerCallback回调方法。使用`Ticker`(而不是`Timer`)来驱动动画会防止屏幕外动画（动画的UI不在当前屏幕时，如锁屏时）消耗不必要的资源。



## Tween

Tween将AnimationController的value映射到你所需的数值以及类型。例如`Tween`生成[-200.0，0.0]的值

~~~dart
final Tween doubleTween = Tween<double>(begin: -200.0, end: 0.0);		//AnimiationController中的值为[0, 1]
final Tween colorTween = ColorTween(begin: Colors.transparent, end: Colors.black54);
~~~



通过`animate(AnimationController controller)`来使用Tween对象，下面给出一个用例：

~~~dart
final AnimationController controller = AnimationController(
  duration: const Duration(milliseconds: 500), 
  vsync: this,
);
final Animation curve = CurvedAnimation(parent: controller, curve: Curves.easeOut);
Animation<int> alpha = IntTween(begin: 0, end: 255).animate(curve);		//Animation默认值为[0, 1],这里IntTween将其映射到[0, 255]
~~~



## 动画的基本结构

~~~dart
class ScaleAnimationRoute extends StatefulWidget {
  const ScaleAnimationRoute({Key? key}) : super(key : key);

  @override
  _ScaleAnimationRouteState createState() => _ScaleAnimationRouteState();
}

class _ScaleAnimationRouteState extends State<ScaleAnimationRoute>
  with SingleTickerProviderStateMixin {

    late Animation<double> animation;
    late AnimationController controller;			//每个state都有属于自己的动画

    @override
    initState() {
      super.initState();
      //利用组合的思想生成动画
      controller = AnimationController(duration: const Duration(seconds : 2),vsync: this);
      animation = CurvedAnimation(parent: controller, curve: Curves.bounceIn);
      animation = Tween(begin : 0.0, end : 300.0).animate(animation)
      ..addListener(() {
          setState(() => {});
      });
	 //addListener()函数调用了setState()，所以每次动画生成一个新的数字时，当前帧被标记为脏(dirty)，这会导致widget的build()方法再次被调用
      controller.forward();
    }

    @override
    Widget build(BuildContext context) {
      return Center(
        child : Image.network(
          "https://book.flutterchina.club/assets/img/logo.png",
          width: animation.value,			//动画值
          height: animation.value,
        )
      );
    }

    @override
    dispose() {
      controller.dispose();
      super.dispose();
    }
}
~~~



我们可以通过AnimatedWidget将addListener()以及setState这一步部分逻辑提取出来

~~~diff
class AnimatedImage extends AnimatedWidget {			//AnimatedWidget是一个抽象类，必须继承使用
  const AnimatedImage({
    Key? key,
+    required Animation<double> animation,
+  }) : super(key: key, listenable: animation);		//向AnimatedWidget注册animation，

  @override
  Widget build(BuildContext context) {
+    final animation = listenable as Animation<double>;			//获取到动画
    return  Center(
      child: Image.asset(
        "imgs/avatar.png",
        width: animation.value,
        height: animation.value,
      ),
    );
  }
}


class _ScaleAnimationRouteState extends State<ScaleAnimationRoute1>
    with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;

  @override
  initState() {
    super.initState();
    controller =  AnimationController(
        duration: const Duration(seconds: 2), vsync: this);
    //图片宽高从0变到300
+    animation =  Tween(begin: 0.0, end: 300.0).animate(controller);
-    animation = Tween(begin : 0.0, end : 300.0).animate(animation)
-      ..addListener(() {
-          setState(() => {});
-      });
    //启动动画
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
+    return AnimatedImage(
+      animation: animation,
    );
  }

  @override
  dispose() {
    //路由销毁时需要释放动画资源
    controller.dispose();
    super.dispose();
  }
}
~~~

一个动画要分离出一个Widget，这样做并不是很优雅。通过AnimatedBuilder正是将渲染逻辑分离出来, 上面的 build 方法中的代码可以改为：

~~~diff
class _ScaleAnimationRouteState extends State<ScaleAnimationRoute>
  with SingleTickerProviderStateMixin {
    @override
    Widget build(BuildContext context) {
+      return AnimatedBuilder(
+        animation : animation,
+        child : Image.network("https://book.flutterchina.club/assets/img/logo.png",),
+        builder : ((context, child)  {
          return Center(
            child: SizedBox(
+              height: animation.value,
+              width: animation.value,
+              child : child
            ),
          );
        }),
      );
    }

}
~~~

- `child`看起来像被指定了两次。但实际发生的事情是：将外部引用`child`传递给`AnimatedBuilder`后，`AnimatedBuilder`再将其传递给`Center`，最终的结果是`AnimatedBuilder`返回的对象插入到 widget 树中。这样做能获得更好的性能，因为动画每一帧需要构建的 widget 的范围缩小了

## 自定义路由动画

使用组件中的动画

~~~dart
 Navigator.push(
     context, 
     CupertinoPageRoute(  			//这个Page包含了一些动画信息
   		builder: (context)=>PageB(),//只需要提供路由页面即可
 	)
);
~~~

使用PageRouteBuilder自定义路由切换动画

~~~dart
Navigator.push(
    context,
    PageRouteBuilder(
    	transitionDuration: Duration(milliseconds: 500), 	//动画时间为500毫秒
        
    	pageBuilder: (BuildContext context, Animation animation, Animation secondaryAnimation) {
    		return FadeTransition(				//内置的动画组件
    			opacity: animation,				//opaciry是一个Animation<double>类型
    			child: PageB(), //路由B
    		);
    	},
    ),
);
~~~



无论是`MaterialPageRoute`、`CupertinoPageRoute`，还是`PageRouteBuilder`，它们都继承自PageRoute类。下面我们介绍PageRoute

~~~dart
class FadeRoute extends PageRoute {
  FadeRoute({
    required this.builder,					//对外提供的builder，允许构建路由页面，但是不允许动画，动画由该Route类来定义
    this.transitionDuration = const Duration(milliseconds: 300),
    this.opaque = true,
    this.barrierDismissible = false,
    this.barrierColor,
    this.barrierLabel,
    this.maintainState = true,
  });

  final WidgetBuilder builder;

  @override
  final Duration transitionDuration;

  @override
  final bool opaque;

  @override
  final bool barrierDismissible;

  @override
  final Color barrierColor;

  @override
  final String barrierLabel;

  @override
  final bool maintainState;

  //Animation由PageRoute提供
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) => builder(context);

  
  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {
      //可以在这里进一步配置animation类
      //animation = Tween.(begin : 100, end : 100).animated(animation);
     return FadeTransition( 
       opacity: animation,			//传入动画
       child: builder(context),		//获取用户构建的页面
     );
  }
}
~~~

- `buildTransitions` 给child控件包裹一个或多个transition控件，来定义路由如何进入和离开屏幕。默认情况下，child（包含了buildPage返回的控件）不包裹任何transition控件。它和`buildPage`方法的区别就是，`buildTransitions`会在每次路由状态变化时都被调用（比如路由canPop的时候）。我们常见的创建`PageRouteBuilder`的时候，如果设置了transitionsBuilder属性，其实就是调用了该方法。

   `buildPage(BuildContext context, Animation<double> animation, Animation<double> secondaryAnimation)`
   定义路由的主页面内容。





## AnimatedSwitcher

`AnimatedSwitcher` 可以同时对其新、旧子元素添加显示、隐藏动画。也就是说在`AnimatedSwitcher`的子元素发生变化时，会对其旧元素和新元素做动画，我们先看看`AnimatedSwitcher` 的定义：

~~~dart
const AnimatedSwitcher({
  Key? key,
  this.child,			 //优化性能的child
  required this.duration, // 新child显示动画时长
  this.reverseDuration,// 旧child隐藏的动画时长
  this.switchInCurve = Curves.linear, // 新child显示的动画曲线
  this.switchOutCurve = Curves.linear,// 旧child隐藏的动画曲线
  this.transitionBuilder = AnimatedSwitcher.defaultTransitionBuilder, // 动画构建器
  this.layoutBuilder = AnimatedSwitcher.defaultLayoutBuilder, //布局构建器
})
    
    
//默认的动画构建器
Widget defaultTransitionBuilder(Widget child, Animation<double> animation) {
  return FadeTransition(
    opacity: animation,
    child: child,
  );
}
~~~



对旧child，绑定的动画会反向执行（reverse）

对新child，绑定的动画会正向指向（forward）



另外，Flutter SDK中还提供了一个`AnimatedCrossFade`组件，它也可以切换两个子元素，切换过程执行渐隐渐显的动画，和`AnimatedSwitcher`不同的是`AnimatedCrossFade`是针对两个子元素，而`AnimatedSwitcher`是在一个子元素的新旧值之间切换。



如果想要打破这种动画的对称性，可以自己编写TransitionBuilder:

~~~dart
class MySlideTransition extends AnimatedWidget {
  const MySlideTransition({
    Key? key,
    required Animation<Offset> position,
    this.transformHitTests = true,
    required this.child,
  }) : super(key: key, listenable: position);

  final bool transformHitTests;

  final Widget child;

  @override
  Widget build(BuildContext context) {
    final position = listenable as Animation<Offset>;
    Offset offset = position.value;
    if (position.status == AnimationStatus.reverse) {
      offset = Offset(-offset.dx, offset.dy);
    }
    return FractionalTranslation(
      translation: offset,
      transformHitTests: transformHitTests,
      child: child,
    );
  }
}


AnimatedSwitcher(
  duration: Duration(milliseconds: 200),
  transitionBuilder: (Widget child, Animation<double> animation) {
    var tween=Tween<Offset>(begin: Offset(1, 0), end: Offset(0, 0))
     return MySlideTransition(
      child: child,
      position: tween.animate(animation),
    );
  },
  ...//省略
)
~~~





## 动画过渡组件

Flutter SDK中也预置了很多动画过渡组件，实现方式和大都和`AnimatedDecoratedBox`差不多，如表9-1所示：

| 组件名                   | 功能                                                         |
| ------------------------ | ------------------------------------------------------------ |
| AnimatedPadding          | 在padding发生变化时会执行过渡动画到新状态                    |
| AnimatedPositioned       | 配合Stack一起使用，当定位状态发生变化时会执行过渡动画到新的状态。 |
| AnimatedOpacity          | 在透明度opacity发生变化时执行过渡动画到新状态                |
| AnimatedAlign            | 当`alignment`发生变化时会执行过渡动画到新的状态。            |
| AnimatedContainer        | 当Container属性发生变化时会执行过渡动画到新的状态。          |
| AnimatedDefaultTextStyle | 当字体样式发生变化时，子组件中继承了该样式的文本组件会动态过渡到新样式。 |

## Mateiral 转场模式

四种主要的 Material 转场模式如下：

- **容器转换**：用于包含容器的界面元素之间的过渡；通过将一个元素无缝转换为另一个元素，在两个不同的界面元素之间创造可视化的连接。

![11807bdf36c66657.gif](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\11807bdf36c66657.gif)

- **共享轴**：用于具有空间或导航关系的界面元素之间的过渡；让元素在转换时共用 x 轴、y 轴或 z 轴，用以强调元素间的关系。

![71218f390abae07e.gif](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\71218f390abae07e.gif)

- **淡出后淡入**：用于彼此之间没有密切关系的界面元素之间的过渡；使用依序淡出和淡入的效果，并会对转入的元素进行缩放。

![385ba37b8da68969.gif](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\385ba37b8da68969.gif)

- **淡出**：用于进入或退出屏幕画面范围的界面元素。

![cfc40fd6e27753b6.gif](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\cfc40fd6e27753b6.gif)