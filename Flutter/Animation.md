 

# Animation

[TOC]

## 



![image-20230806165051519](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20230806165051519.png)



怎么插值要回答以下两个问题

- 插值区间是什么？
- 如何插值，线性插值？ 曲线插值？

## 隐式动画



`AnimatedContainer`

~~~dart
AnimatedContainer(
	duration : Duration(seconds : 1),
    
)
~~~

这个Widget只会把动画效果过应用在本身的属性上，而不会应用在子Widget上



`AnimatedSwitcher`，在子Widget切换时，应用动画切换效果（默认的），新的Widget是应用正向的，而旧的Widget是应用反向的。注意diff算法而导致动画失效的问题，可以在子Widget构造时传入Key来解决这个问题

~~~dart
AnimatedSwitcher(
	duration: Duration(seconds : 2),
    child : Text(string, ValueKey(string)));
)
~~~

如果要应用其他动画切换效果，可以使用`transitionBuilder`属性

~~~dart
AnimatedSwitcher(
	transitionBuilder : (child, animation) {
        return ...
    }
)
~~~



自定义隐式动画 `TweenAnimationBuilder`：

~~~dart
TweenAnimationBuilder(
  duration: Duration(seconds: 1),
  tween: Tween<double>(begin: 0.0, end : 1.0),  //线性补间数值
  //当tween改变时，会从old_end -> new_end，而不是 begin -> new_end
  //更具体来说是从current_value -> new_end 因此动画在执行时Tween可能发生改变
  builder:(context, value, child) {
    //动画每一帧都会调用builder，并通过参数value传入补间值
    //child参数用于优化
    return Opacity(opacity: value);
  },
  child : ...
)
~~~

在自定义动画时经常用到的控件`Transform`

~~~c
Transform(
	transform: Matrix4()			//任何3D动画都可以通过4*4矩阵来表示
    child : ...
);
    
//不熟悉矩阵的话，可以使用以下构造函数
Transform.scale(
	scale : 0.5,
    child : ...
);
    
Transform.rotate(
	angle : 3.14					//3.14 = Pi 表示转半圈
    child : ...
);

Transform.translate(
	offset : Offset(10, -10)		 //向右边移动10像素、向上移动10像素
    child : ...
)
~~~



每个AnimatedXXX Widget都有一个curve属性，默认值为Curve.linear，更多内置动画曲线请见https://api.flutter.dev/flutter/animation/Curves-class.html

## 显式动画

一个动画要包括：控制器、插值（区间+方式）。

其中控制器本身就是`Animation`类型，而插值区间是`Animatable`，插值方式是`Curve`

基本框架

~~~dart
class CustomWidget extends StatefulWidget with SingleTrickerProviderStateMixin {
    late AnimationController _controller;
    
    @override
    void initState() {
        _controller = AnimationController(			//继承自Animation<Double>
        	duration : Duration(seconds : 1),		//动画持续时间
            lowerBound : 3.0,					   //插值的初始值，默认为0.0
            upperBound : 5.0,					   //插值的最终值，默认为1.0
            vsync : this						  //垂直同步，通过mixin SingleTrickerProviderStateMixin来获取相应数据
        );
        //_controller.forword()		//开始执行动画
        //_controller.reset();		//重置回初始状态
        //_controller.stop();		//停止动画
        _controller.addListener(() {
           //在动画的每一帧都会调用该函数 
        });
        super.initState();
    }
    @override
    void dispose(){
        _controller.dispose();		//回收资源
        super.initState();	
    }
    
    @override
  	Widget build(BuildContext context) {
        return RotationTransition(
        	turns : _controller,		//Flutter自带的显式动画控件，通过参数turns传入一个动画控制器
            child : ...
        )
    }
}
~~~



~~~c
FadeTransition(
	opacity : _controller,
    child : ...
)
    
ScaleTransition(
	scale : _controller,
    child : ...
)
    
SlideTransition(
	position : ,
)
~~~



> Flutter内置的显式动画控件，相比隐式动画控件来说，除了动画效果相同，还可以控制动画的播放行为（暂停、播放、重复）以及其他特性（事件监听）



通过`Tween`的`animate`方法或者`Animation`的`drive`方法，可以将动画控制器的插值区间映射到新的插值区间，并返回一个新的`Animation`

~~~dart
Center(
    child: SlideTransition(   
      position: _controller.drive(Tween(begin : Offset(0,0), end : Offset(1, 1))),
      position: Tween(begin : Offset(0, 0), end : Offset(1, 1)).animate(_controller),
      child: ...
    ),
  ),
~~~





~~~c
Tween(begin: Offset(0, 0), end : Offset(0, 0.1))
      .chain(CurveTween(curve: Curves.elasticInOut))
      .animate(_controller),
~~~

这段代码做了两件事

- 将动画_controller的区间映射到新的区间[begin, end]
- 以曲线的方式来插值

`chain`接受一个`Animatable<T>`参数，该参数与`this`将一起结合成新的插值

而这里的`Tween`与`CurveTween`都是`Animatable<T>`。此外要注意结合顺序的影响





显示动画控件如何曲线插值？

~~~dart
//推荐
final Animation curve = CurvedAnimation(parent : controller, curve : Cureves.easeOut);

//在某些场景下可以使用
Tween(begin : ..., end : ...)
    .chain(CurveTween(curve : ... ))
    .animate(_controller)
~~~





使用`AnimatedBuilder`控件来自定义显式动画：

~~~dart
AnimatedBuilder(
  animation: _controller,
  //当监听到_controller的值发生变化时，即动画执行一帧
  //就会调用builder函数
  builder: (context, child) {
    //AnimatedBuilder中的child属性会传入到当前child参数中
    //这样就不会再重新绘制child参数部分，可以优化性能
    return Opacity(
      opacity: _controller.value,	//获取到当前插值
      
      //height : Tween(begin : 100.0, end : 200.0),evaluate(_controller);
      child: child				   
    );
  }
  child : ...
)
~~~



有时候，我们在AnimatedBuilder中获取到动画插值后，还要再次计算出我们想要的预期值。大部分的计算公式都可以用Tween中的evaluate方法来表示：

~~~dart
AnimatedBuilder(
	builder : (ctx, child) {
        return Container(
        	height : Tween(begin : ... , end : ...)
            	.chain(CurveTween(curve: Curves.bounceIn))
            	.evaluate(_controller);
        )
    }
)
    
//可选方案    
final Animation heightAnimation = Tween(begin : ..., end : ...).animate(_controller);

AnimatedBuilder(
	bhilder : (ctx, bhild) {
        return Container(
        	height : heightAnimation.value
        )
    }
)
    
//多个动画可以使用同一个控制器！
~~~



AnimatedBuilder、TweenAnimationBuilder以及官网提供的动画控件，实际上都是继承自StatefulWidget。也就是说动画每一帧的刷新都是Ticker回调setState函数。下面来演示如何不用Animation Controller来完成动画效果

~~~dart
class _MyHomePageState extends State<MyHomePage>
    with SingleTickerProviderStateMixin {
  double _height = 300;

  @override
  void initState() {
    Ticker _ticker = new Ticker((elapsed) {
      setState(() {
        _height--;
        if (_height <=0 ) _height = 300;
      });
    });
    //每次设备屏幕刷新时，就会调用回调函数
    _ticker.start();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
 
    return Center(
          child: AnimatedContainer(
              duration: Duration(seconds: 1),
              width: 300,
              height: _height,
    );
  }
}
~~~



实际上不推荐使用这种方案，因为Ticker是根据设备屏幕的刷新率来执行回调函数的，这个动画在60Hz、120Hz的屏幕上有着不同的动画执行时长。



### 交织动画



在多个控件上执行动画序列，可以让多个Animation使用同一个controller，但是要配合Interval一起使用

~~~dart
Animation animation1 = CurveTween(curve : Interval(0.0, 0.2)).animate(_controller);
Animation animation2 = CurveTween(curve : Interval(0.2, 0.4)).animate(_controller);
...
Animation animation5 = CurveTween(curve : Interval(0.8, 1.0)).animate(_controller);
~~~



交织动画还可以通过控制时长的方式来完成，我在js中就是使用这种方式来进行交织动画的。

AnimationController可以修改duration属性，但一般要配合`delayed`方法来使用

```dart
controller.duration = Duration(seconds : 4);
controller.forward();
await Future.delayed(Duration(second : 4));		//必须有这么一步，否则可能直接回溯播放7s
controller.duration = Duration(second : 7);
controller.reverse();
```



在单一控件上执行多个动画序列，还可以通过TweenSequence控件来完成



### Hero动画

只需使用Hero控件即可。在切换Widget时，Flutter会获取tag相同Widget，并在新的控件上应用Hero动画（旧的控件立即消失）。



### Painter

一般都是封装好CustomPainter，然后调用AnimatedBuilder，将动画参数传入到Painter中

## Other Widget

`Duration`的构造函数

~~~dart
const Duration({
      int days = 0,
      int hours = 0,
      int minutes = 0,
      int seconds = 0,
      int milliseconds = 0,
      int microseconds = 0
}) : this._microseconds(microseconds +
            microsecondsPerMillisecond * milliseconds +
            microsecondsPerSecond * seconds +
            microsecondsPerMinute * minutes +
            microsecondsPerHour * hours +
            microsecondsPerDay * days);

Duration(seconds : 0.5)			//Error
Duration(seconds : 120)			//OK
~~~



`CircularProgressIndicator`

~~~dart
child : CircularProgressIndicator()
~~~





`Opacity`



`BoxDecoration`与`Container`的区别









## Demo

### 计数器

~~~dart
class AnimatedCounter extends StatelessWidget {
  const AnimatedCounter({
    super.key,
    required this.counter,
    required this.duration
  });

  final int counter;
  final Duration duration;
  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder(
      duration:  duration,
      tween : Tween(end : counter.toDouble()),
      builder: (context, value, child) {
        value = value % 10;
        final whole = value ~/ 1;       //结果只保留整数部分
        final decimal= value - whole;
        return Stack (
          children: [
            Positioned(
              top : -100 * decimal,   // 0 -> -100
              child: Opacity(
                opacity: 1.0 - decimal,
                child: Text(
                  "$whole",
                  style : TextStyle(fontSize : 100),
                ),
              ),
            ),
            Positioned(
              top : 100 - 100 * decimal,   // 100 -> 0
              child: Opacity(
                opacity: decimal,
                child: Text(
                  "${whole + 1}",
                  style : TextStyle(fontSize : 100),
                ),
              ),
            ),
          ],
        );    
      }
    );
  }
}
~~~

> https://pub.dev/packages/animated_flip_counter
