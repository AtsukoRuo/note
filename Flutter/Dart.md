# Dart

[TOC]



## 包管理

`import` 的唯一参数是用于指定代码库的 URI，对于 Dart 内置的库，使用 `dart:xxxxxx` 的形式。



导入本项目的其他文件`import "package:${your_project}/${filepath}"`; 或者` import "${relative_path}"`

### 导入包

~~~dart
import 'package:flutter/material.dart';
import "package:flutter/material.dart";
//双引号或单引号均可
~~~





~~~dart
void main() {		//程序入口点
    print("Hello World!");
}
~~~

### 指定库前缀

如果你导入的两个代码库有冲突的标识符，你可以为其中一个指定前缀。比如如果 library1 和 library2 都有 Element 类，那么可以这么处理：

~~~dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// Uses Element from lib1.
Element element1 = Element();

// Uses Element from lib2.
lib2.Element element2 = lib2.Element();
~~~

### 导入库的一部分

如果你只想使用代码库中的一部分，你可以有选择地导入代码库。例如：

~~~dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
~~~









## 函数

Dart 支持顶级函数。同时函数是对象



### Positional & Named Function

**Positional**: The position of an argument determines which parameter receives the value

~~~dart
void add(a, b) {
	print(a + b); 
}
add(5, 10); 
~~~

**Named**: The name of an argument determines which parameter receives the value

~~~dart
void add({a, b}) { 
	print(a + b);
}
add(b: 5, a: 10);
~~~



By default, positional parameters are required and must not be omitted - on the other hand, named arguments are optional and can be omitted（此时值为null）

Positional arguments can be made optional by wrapping them with square brackets (`[]`):

~~~dart
void add(a, [b])
~~~

Once a parameter is optional, you can also assign a **default value** to it - this value would be used if no value is provided for the argument:

~~~dart
void add(a, [b = 5])
~~~

Default values can also be assigned to named parameters - which are optional by default:

~~~dart
void add({a, b = 5})
~~~

You can also make named parameters required by using the built-in `required` keyword:

~~~dart
void add({required a, required b})
~~~

如果混用named与 position特性，那么named放在参数列表的最后面：

~~~dart
void add(c, {a, b})
~~~



![image-20230623155621502](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230623155621502.png)

## const & final & static

如果**你不想更改一个变量**，可以使用关键字 `final` 或者 `const` 修饰变量，这两个关键字可以替代 `var` 关键字或者加在一个具体的类型前。



**final变量是在运行时执行初始化，并且状态在运行时不可更改**

**const变量是在编译期执行初始化，并且状态在运行时不可更改**。



`const` 关键字不仅仅可以用来定义常量，还可以用来创建 **常量值**，该常量值可以赋予给任何变量。你也可以将构造函数声明为 const 的，这种类型的构造函数创建的对象是不可改变的。

~~~dart
var foo = const [];
final bar = const [];
const baz = []; // Equivalent to `const []`
~~~

在 **常量上下文** 场景中，你可以省略掉构造函数或字面量前的 `const` 关键字：

~~~dart
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};

const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
~~~





const成员必须为static的

~~~dart
static const Alignment topLeft = Alignment(-1.0, -1.0);
/// The center point along the top edge.
static const Alignment topCenter = Alignment(0.0, -1.0);
/// The top right corner.
static const Alignment topRight = Alignment(1.0, -1.0); // const Alignment（1.0，-1.0）

~~~







一般对象是在运行时期生成，所以常量构造函数的条件还是比较苛刻的：

- 常量构造函数需以 const 关键字修饰

- const 构造函数必须用于成员变量都是 final 的类（这是很重要的前置条件，即对象在初始化后不可变）

- 常量的不可变性是具有依赖关系

  ~~~dart
  final size = 12.0;
  const Text(
    "Hello",
    style: TextStyle(
      fontSize: size,    // error
    ),
  )
  ~~~

   Dart 也会尝试将 `TextStyle` 创建为常量，因为它知道 `TextStyle` 必须为常量才满足 `const Text()` 的调用。但是由于 `TextStyle` 依赖于变量 size，所以会直接报错。要解决此问题，必须替换 size 为常量或者直接使用数字。



常量构造函数不能有函数体

~~~dart
const MyClass(a, b);
~~~

而且常量构造函数只有在const语境中会调用。在非const语境中，与非const构造函数的作用一样

~~~dart
A a = const A();
const A a = A();
A a = A();		//不是常量构造函数，退化成final的情况。因为具有常量构造函数的类，它的成员都是final的
~~~



## 类型系统

Dart 支持顶级 **变量**

Dart 是类型安全的编程语言：Dart 使用静态类型检查（编译器检查）和 [运行时检查](https://dart.cn/guides/language/type-system#runtime-checks) （向下转型、强制类型转换时）的组合来确保变量的值始终与变量的静态类型或其他安全类型相匹配。

在Dart一切皆为对象。即使是内置类型int、double也是对象，继承自Number类，同时函数也为对象。



var关键字根据初始化表达式来自动推断类型。如果没有初始化表达式，那么推断为dynamic。同时在函数参数列表中没有类型，那么认为是dynamic。

~~~dart
void f(a, b) {}		//type of a and b is dynamic 
~~~

而dynamic相当于C语言中的void *指针，可以存储任何类型的值

### 初始化 & 默认值

可空类型默认为null。

若你启用了空安全，你必须在使用变量前初始化它的值：

~~~dart
int lineCount = 0;
~~~

你并不需要在声明变量时初始化，只需在第一次用到这个变量前初始化即可：

~~~dart
int lineCount;

if (weLikeToCount) {
  lineCount = countLines();
} else {
  lineCount = 0;
}

print(lineCount);
~~~



通常 Dart 的语义分析会在一个已声明为非空的变量被使用前检查它是否已经被赋值，但有时这个分析会失败。例如：在检查顶级变量和实例变量时，分析通常无法判断它们是否已经被初始化，因此不会进行分析。

如果你确定这个变量在使用前就已经被声明，但 Dart 判断失误的话，你可以在声明变量的时候使用 `late` 修饰来解决这个问题。

~~~dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
~~~

若 `late` 标记的变量在使用前没有初始化，在变量被使用时会抛出运行时异常。

~~~dart
late int i;
void main() {
  print(i != null);		//Uncaught Error: LateInitializationError: Field 'i' has not been initialized.
}
~~~



如果一个 `late` 修饰的变量在声明时就指定了初始化方法，那么它实际的初始化过程会发生在第一次被使用的时候。这样的延迟初始化在以下场景中会带来便利：

- Dart 认为这个变量可能在后文中没被使用，而且初始化时将产生较大的代价。

在下面这个例子中，如果 `temperature` 变量从未被使用的话，那么 `readThermometer()` 将永远不会被调用：

~~~dart
// This is the program's only call to readThermometer().
late String temperature = readThermometer(); // Lazily initialized.
~~~



### 空安全

有了健全的空安全体系，变量默认是「非空」的：它们可以赋予与定义的类型相同类型的任意值（例如 `int i = 42`），且永远不能被设置为 `null`。



若你想让变量可以为 `null`，只需要在类型声明后加上 `?`：

~~~dart
int? aNullableInt = null;
~~~





### 操作符

`? :`

## 面向对象机制



Dart 没有类似于 Java 那样的 `public`、`protected` 和 `private` 成员访问限定符。如果一个标识符以下划线 (`_`) 开头则表示该标识符在库内是私有的。



当且仅当命名冲突时使用 `this` 关键字才有意义，否则 Dart 会忽略 `this` 关键字。



语法糖：

~~~dart
  const StyledText(this.text, {super.key});
  const StyledText(text) text = text, super(key : key);
~~~



### 构造函数

Dart构造函数有4种格式：

- `ClassName(...) //普通构造函数`
- `Classname.identifier(...) //命名构造函数`
- `const ClassName(...) //常量构造函数`
- `factroy ClassName(...) //工厂构造函数`



用法如下

~~~dart
var p1 = Point(2, 2); //Dart2中，可以省略构造函数前的new
var p2 = Point.fromJson({'x': 1, 'y': 2});
var p = const ImmutablePoint(2, 2); //常量构造函数，用来创建编译期常量

~~~



初始化列表在方法体之前执行，不能在构造函数方法体中初始化final成员，而可以在初始化列表中初始化final成员。

~~~dart
class Point {
  final num x;
  final num y;
  final num distanceFromOrigin;

  Point(x, y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}
~~~



### 初始化顺序

1. 静态字段和静态方法

2. 普通字段，不支持实例初始化。除可空类型以及late修饰的字段，其他字段必须要在构造函数执行完毕前进行初始化。不同语言由于设计宗旨不同,在实例初始化上做出了不同选择。

3. 初始化列表

4. 构造函数体

   



> Java和Dart在类属性和方法的初始化顺序上有区别,导致了某些代码在两种语言中行为不一致。需要注意语言之间这些细节差异。

~~~dart
class QuizQuestion {
    int f() {
        return 10;
    }
    int aaa = f();					//错误的，在Java中是可以的，注意虚函数的绑定发生在运行期中。
}
~~~



以下划线开头的类名、字段名、方法名是私有的





### getter、setter

是getXXX、setXXX的语法糖

~~~dart
class A {
    final int _data = 0;
    int getData() {
        return _data;
    }
}

void main() {
    A a = A();
    a.getData()
}
~~~



~~~dart
class A {
    final int _data = 0;
   	int get data {
        return _data;
    }
}

void main() {
    A a = A();
    a.data;
}
~~~



### 内部类

在Dart中**不存在内部类**

### Mixin（混合）

> Mixins are a way of reusing a class’s code in multiple class hierarchies.
>
> Mixins 是一种在多个类层次结构中重用类代码的方法。

>In object-oriented programming languages, a mixin (or mix-in) is a class that contains methods for use by other classes without having to be the parent class of those other classes. How those other classes gain access to the mixin's methods depends on the language. Mixins are sometimes described as being "included" rather than "inherited".
>
>Mixins encourage code reuse and can be used to avoid the inheritance ambiguity that multiple inheritance can cause (the "diamond problem"), or to work around lack of support for multiple inheritance in a language. A mixin can also be viewed as an interface with implemented methods. This pattern is an example of enforcing the dependency inversion principle.
>
>在面向对象的编程语言中，mixin（或mix-in）是一个类，其中包含供其他类使用的方法，而不必成为其他类的父类。 这些其他类如何获得对mixin方法的访问权限取决于语言。 混合素有时被描述为“包含”而不是“继承”。
>
>Mixins鼓励代码重用，并且可用于避免多重继承可能导致的继承歧义（“钻石问题”），或解决语言中对多重继承的支持不足的问题。 混合也可以看作是已实现方法的接口。 此模式是强制执行依赖关系反转原理的示例。



这就是Java中的interface + default方法

~~~dart
class Person  {
  void eat() {
    print("person eat");
  }
}

mixin Dance {
  void dance() {
    print("Dance dance");
  }
}

mixin Sing {

  void sing() {
    print("Sing int");
  }
}

mixin Code on Person {				//只有Person与其子类可以with这个mixin类
  void code() {
    print("Code code");
  }
}

class A extends Person with Dance, Sing {}

class B extends Person with Sing, Code {}

class C extends Person with Code, Dance{}

class Dog with Code {}			//Error


~~~

 如何处理多个类有同一方法的情况？就一条规则进行混合的多个类是线性覆盖的

~~~dart
class A {
  void say() {print("A");}
}

mixin B {
  void say() {print("B"); }
}

mixin C {
  void say() {print("C"); }
}

class D extends A with B, C{
  @override
  void say() {
    print("D");
    super.say();
  }
}

void main()
{
    D d = new D();
  	d.say();
  	return;
}

/**
flutter: D
flutter: C
*/
~~~



## 函数式编程

通过 void Funtion()来声明Lambda表达式，而不是像Java那么要声明一个函数式接口。在Dart中，函数是第一等成员，因此有些微妙的问题又在初始化中产生：

~~~dart
class _QuizState extends State<Quiz> {
    Widget? activeScreen = StartScreen(switchScreen);			//Dart不支持实例初始化
    void switchScreen() {
        setState(() {
            activeScreen = const QuestionsScreen();
        });
    }
}
~~~



```dart
Object Function() a = () {return Object(); }
a = () => Object();
```



## 过程式语句

you may also use `if` inside of lists to conditionally add items to lists:

```dart
final myList = [  1,  2,  if (condition)    3];
final myList = [1,2,if (condition) 3 else 4];
final myList = [1,2,condition ? 3 : 4];
```

you can also use the `for` keyword to add multiple items into a list:

~~~dart
final myList = [
  1,
  2,
  for (final num in numbers)
    num
];
~~~



## 其他

### 列表

~~~dart
var list = [1, 2, 3, 4];
var list2 = [2, ...list, 3];		//把列表展开
list.map((item) {
   return item; 
});				//不会修改原列表

list.shuffle()		//洗牌算法	返回值为void，会修改原list
    
list2 = List.of(list)		//深拷贝
~~~



### Map

~~~dart
Map<String, Object> map = {
    "id" : 1,
    "username" : "AtsukoRuo",
    "pwd" : "2020204427"
}
~~~



### 字符串

字符串插值

~~~dart
var diceRoll = 0;
"assets/image/dice-$diceRoll.png"
~~~

Dart可以重载运算符

### 逗号

在函数参数列表或者初始化语句中可以有多余的逗号，一方面方便格式化，另一方面便于维护



### 枚举

~~~dart
enum State {
    Sleep,
    Work,
    Run,
    DoLove
}

State.values	//List<State>
State.Sleep.name //toString(); "Sleep"
~~~

