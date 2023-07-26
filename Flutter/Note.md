# Note

## DarkMode

~~~dart
runApp(
        MaterialApp(
            darkTheme: ThemeData.dark().copyWith(),
            themeMode : ThemeMode.system,
            theme: ThemeData().copyWith()
        ),
    );
~~~



## Responsive & Adaptive

基本思想：媒体查询 + 判断切换



~~~dart

WidgetsFlutterBinding.ensureInitialized()； //主要用于确保 Flutter 框架已经初始化完成
SystemChrome.setPreferredOrientations([			//设置支持的朝向
    DeviceOrientation.portraitUp,
]).then((value) {
    runApp();
}
~~~



~~~dart
@override
    Widget build(BuildContext context) {
        final width = MediaQuery.of(context).size.width;			//媒体查询，获取当前widget的宽度 
        final keyboardSpace = MediaQuery.of(context).viewInsets.bottom; 	//获取当前widget的底部与其他组件重叠区域的高度
        return Scaffold(
            body : width  < 600 ?									//change widget dependent on the width
            Column(
                children: [
                    Chart(expenses: _registeredExpenses),
                    Expanded(child : mainContent,),
                ],
            ) :
            Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                    Expanded(child: Chart(expenses: _registeredExpenses)),
                    Expanded(child : mainContent,),
                ],
            )
        );
    }
~~~





cupertino提供IOS风格、行为的附件，`showDialog `-> `showCupertinoDialog`



查询特定平台：

~~~dart
Platform.isIOS		//boolean
~~~



## Internals

![image-20230715143558380](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230715143558380.png)

注意：当调用build函数（StatelessWidget只会调用一次build，这是一个优化点，尽可能最小化StatefulWidget的规模）改变Widget的属性时，Flutter会直接复用ElementTree中的元素，除非要删除与添加Widget。



Key属性可以唯一地表示一个Widget。考虑给一个表项排序的情景：

![image-20230715135941064](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230715135941064.png)

我们排序代码如下：

~~~dart
//要比较的对象
class CheckableTodoItem extends StatefulWidget {
    const CheckableTodoItem(this.text, this.priority, {super.key});
    final String text;
    final Priority priority;
    @override
    State<CheckableTodoItem> createState() => _CheckableTodoItemState();
}

class _CheckableTodoItemState extends State<CheckableTodoItem> {
    var _done = false;			//内部状态，表明是否被选中
    @override
    Widget build(BuildContext context) {
    	return Widget();
    }
}


class _KeysState extends State<Keys> {
    List<Todo> get _orderedTodos {
        final sortedTodos = List.of(_todos);
        sortedTodos.sort((a, b) {
              final bComesAfterA = a.text.compareTo(b.text);
              return _order == 'asc' ? bComesAfterA : -bComesAfterA;		//升序排序
        });
        return sortedTodos;
    }
    
    @override
    Widget build(BuildContext context) {
      	return Column(
            children: [
                // for (final todo in _orderedTodos) TodoItem(todo.text, todo.priority),
                for (final todo in _orderedTodos)
                    CheckableTodoItem (
                        todo.text,
                        todo.priority,
                    ),
            ],
        )；
    }
}
~~~



当重新排序时，Flutter会检查Widget类型与在ElementTree对应位置元素的类型是否一样。如果一样，那么就复用该ElementType，并刷新UI。但是这里有一个问题就是，ElementType中的内部状态会被保留，造成UI状态的不一致。

![image-20230715142531316](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230715142531316.png)

解决方案就是添加Key，那么Flutter在检查类型的同时也会检查Key值，只有都相等时才复用。

![image-20230715142436030](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230715142436030.png)

由两个方法可以生成key值

~~~dart
class Item extends StatelessWidget {
    Item({super.key});
}

Item(key : ValueKey())		//根据值来生成key（推荐）
Item(key : ObjectKey())		//根据对象实例来生成key
~~~





## GridView

### . SliverGridDelegateWithFixedCrossAxisCount

该子类实现了一个横轴为固定数量子元素的layout算法，其构造函数为：

```dart
SliverGridDelegateWithFixedCrossAxisCount({
  @required double crossAxisCount, 
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})
```

- `crossAxisCount`：横轴子元素的数量。此属性值确定后子元素在横轴的长度就确定了，即ViewPort横轴长度除以`crossAxisCount`的商。
- `mainAxisSpacing`：主轴方向的间距。
- `crossAxisSpacing`：横轴方向子元素的间距。
- `childAspectRatio`：子元素在横轴长度和主轴长度的比例。由于`crossAxisCount`指定后，子元素横轴长度就确定了，然后通过此参数值就可以确定子元素在主轴的长度。





上面我们介绍的GridView都需要一个widget数组作为其子元素，这些方式都会提前将所有子widget都构建好，所以只适用于子widget数量比较少时，当子widget比较多时，我们可以通过`GridView.builder`来动态创建子widget。`GridView.builder` 必须指定的参数有两个：

```dart
GridView.builder(
 ...
 required SliverGridDelegate gridDelegate, 
 required IndexedWidgetBuilder itemBuilder,
)
```

其中`itemBuilder`为子widget构建器。


## Gesture

`GestureDetector`是一个用于手势识别的功能性组件，我们通过它可以来识别各种手势。`GestureDetector` 内部封装了 Listener，用以识别语义化的手势，接下来我们详细介绍一下各种手势的识别。

GestureDetector没有用户交互效果，可以使用InkWell来代替

## Route

`Navigator`是一个路由管理的组件，通过一个栈来管理活动路由集合，通常当前屏幕显示的页面就是栈顶的路由。



 1. `Future push(BuildContext context, Route route)`

    将给定的路由入栈（即打开新的页面），返回值是一个`Future`对象，用以接收新路由出栈（即关闭）时的返回数据。



 2. `bool pop(BuildContext context, [ result ])`

    将栈顶路由出栈，`result` 为页面关闭时返回给上一个页面的数据。

 3. `Future pushReplacement(BuildContext context, Route route)`

    与push作用一样，除了它会替换栈顶元素



Navigator类中第一个参数为context的**静态方法**都对应一个Navigator的**实例方法**， 比如`Navigator.push(BuildContext context, Route route)`等价于`Navigator.of(context).push(Route route)` ，下面命名路由相关的方法也是一样的。

~~~dart
//in TipRoute.dart
Navigator.pop(context, "我是返回值"),



var result = await 
    Navigator.push(
        context,
        MaterialPageRoute(
            //从这里附加路由信息，并包裹着一个Widget
            builder: (context) {
                return TipRoute(		//这是一个StatelessWidget
                  text: "我是提示xxxx",
                );
        	},
    	),
    );
~~~

~~~dart
MaterialPageRoute({
    WidgetBuilder builder,
    RouteSettings settings,
    bool maintainState = true,
    bool fullscreenDialog = false,
});
~~~

- `builder` 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。
- `maintainState`：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置`maintainState`为 `false`。





我们可以先给路由起一个名字，然后就可以通过名字直接打开路由了，这就是命名路由。要想使用命名路由，我们必须先提供并注册一个**路由表（routing table）**，这样应用程序才知道哪个名字与哪个路由组件相对应：

~~~dart
Map<String, WidgetBuilder> routes;
~~~

- WidgetBuilder，是个`builder`回调函数



~~~dart
MaterialApp(
  title: 'Flutter Demo',
  initialRoute:"/", //名为"/"的路由作为应用的home(首页)
  //注册路由表
  routes:{
   "new_page":(context) => NewRoute(),
   "/":(context) => MyHomePage(title: 'Flutter Demo Home Page'), //注册首页路由
  } 
);

MaterialApp(
  title: 'Flutter Demo',
  //注册路由表
  routes:{
   "new_page":(context) => NewRoute(),
    ... // 省略其他路由注册信息
  } ,
  home: MyHomePage(title: 'Flutter Demo Home Page'),
);
~~~



要通过路由名称来打开新路由，可以使用`Navigator` 的`pushNamed`方法：

~~~dart
Future pushNamed(BuildContext context, String routeName,{Object arguments})
~~~

在路由页通过`RouteSetting`对象获取路由参数：

~~~dart
class EchoRoute extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    //获取路由参数  
    var args=ModalRoute.of(context).settings.arguments;
    //...省略无关代码
  }
}
~~~

在打开路由时传递参数：

~~~dart
Navigator.of(context).pushNamed("new_page", arguments: "hi");
~~~



## Stack

![image-20230716121542795](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230716121542795.png)



## FadeInImage

在图片加载时，淡入

~~~dart
FadeInImage(
    placeholder: placeholder, 
    image: image
)
~~~

- placeholder：加载时的初始状态。我们可以使用这个包`dart pub add transparent_image`



~~~dart
FadeInImage(
    placeholder: MemoryImage(
        kTransparentImage			//transparent_iamge包中的
    ), 
    image: NetworkImage(
        meal.imageUrl
    ),
)
~~~



## Stack & Positioned

Flutter中使用`Stack`和`Positioned`这两个组件来配合实现绝对定位。`Stack`允许子组件堆叠，而`Positioned`用于根据`Stack`的四个角来确定子组件的位置。



~~~dart
Stack({
  this.alignment = AlignmentDirectional.topStart,
  this.fit = StackFit.loose,
  this.clipBehavior = Clip.hardEdge,
  List<Widget> children = const <Widget>[],
})
~~~

- `fit`：此参数用于确定**没有定位**的子组件如何去适应`Stack`的大小。`StackFit.loose`表示使用子组件的大小，`StackFit.expand`表示扩伸到`Stack`的大小。
- `clipBehavior`：此属性决定对超出`Stack`显示空间的部分如何剪裁，Clip枚举类中定义了剪裁的方式，Clip.hardEdge 表示直接剪裁，不应用抗锯齿
- `alignment`：此参数决定如何去对齐没有定位（没有使用`Positioned`）或部分定位的子组件。所谓部分定位，在这里**特指没有在某一个轴上定位：**`left`、`right`为横轴，`top`、`bottom`为纵轴，只要包含某个轴上的一个定位属性就算在该轴上有定位。



```dart
const Positioned({
  Key? key,
  this.left, 
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  required Widget child,
})
```

`left`、`top` 、`right`、 `bottom`分别代表离`Stack`左、上、右、底四边的距离。`width`和`height`用于指定需要定位元素的宽度和高度。`width`、`height` 和其他地方的意义稍微有点区别，此处用于配合`left`、`top` 、`right`、 `bottom`来定位组件，如指定`left`和`width`后，`right`会自动算出(`left`+`width`)





一般builder要求传入一个context目的是为了获取WidgetTree，找到正在构建Widget的父Widget。

Navigator.of(context)是为了获取NavigatorWidget的信息，而不是调用Widget

Navigator的push方法不会在当前Widget下创建子Widget,而是会在Navigator所在的Scaffold下创建子Widget。

```
Scaffold
  |--AppBar
  |--Drawer
  |--Body
     |--Navigator
        |--Page1 
        |--Page2
```



## ListTile

ListTile 通常用于在 Flutter 中填充 ListView。

https://juejin.cn/post/6844903796099776526

## Drawer

~~~dart
Scaffold(
  drawer: Drawer(
    child: // Populate the Drawer in the next step.
  ),
);
~~~



DrawerHeader 是 Drawer 最上方用来显示基本信息的控件

- decoration：通常用来设置背景颜色或者背景图片
- duration 和 curve：如果 decoration 发生了变化，则会使用 curve 设置的变化曲线和 duration 设置的动画时间来做一个切换动画
- child: Header 里面所显示的内容控件
- padding: Header 里面内容控件的 padding 值

如果想在 DrawerHeader 中显示用户账户信息，比如类似于 Gmail 的 联系人头像、用户名、Email 等信息，则可以使用 UserAccountsDrawerHeader 这个特殊的 DrawerHeader。

## WillPopScope

为了避免用户误触返回按钮而导致 App 退出，在很多 App 中都拦截了用户点击返回键的按钮，然后进行一些防误触判断。Flutter中可以通过`WillPopScope`来实现返回按钮拦截，我们看看`WillPopScope`的默认构造函数：

~~~dart
const WillPopScope({
  ...
  required WillPopCallback onWillPop,
  required Widget child
})
~~~

注意：不仅返回按钮点击会触发onWillPop,在代码中手动调用Navigator.pop（出onWillPop中）导航返回也会触发onWillPop回调

onWillPop返回`false`来通知平台我们已经处理了返回操作，防止平台再次执行返回操作。

## SwitchListTitle

![image-20230716170925030](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230716170925030.png)

~~~dart
SwitchListTile(
    value: _glutenFreeFilterSet, 				//设置开关的状态
    onChanged: (isChecked) {					//状态改变时
        setState(() {
        _glutenFreeFilterSet = isChecked;
        });
    },
    title : Text(							//标题
        "Gluten-free",
        style : Theme.of(context).textTheme.titleLarge!.copyWith(
            color : Theme.of(context).colorScheme.onBackground,
        ),
    ),
    subtitle : Text(							//子标题
        "Only include gluten-free meals.",
        style : Theme.of(context).textTheme.titleMedium!.copyWith(
            color : Theme.of(context).colorScheme.onBackground,
        ),
    ),
    activeColor: Theme.of(context).colorScheme.tertiary,	//开的颜色
    contentPadding: const EdgeInsets.only(left : 34, right : 22),
),
~~~





## 状态管理

用一张图来说明状态管理复杂性所在

![image-20230716171659324](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230716171659324.png)

如果两个Widget想要共享一些状态，那么这些状态必须交给共同的父Widget来管理。同时为了修改状态，父Widget需要提供钩子给这两个Widget调用。

由于父Widget每次build时，都会重新创建子Widget，所以子Widget的状态每次都要刷新初始化。因此

- 如果状态是用户数据，如复选框的选中状态、滑块的位置，则该状态最好由父 Widget 管理。
- 如果状态是有关界面外观效果的，例如颜色、动画，那么状态最好由 Widget 本身来管理。
- 如果某一个状态是不同 Widget 共享的则最好由它们共同的父 Widget 管理。



有一些第三方包可以帮助我们简化状态管理的工作，这里推荐Rivepod。使用Rivepod后，只有与动画有关的状态需要用到StatefulWidget，其他都可以使用StatelessWidget。因为

![image-20230716174313691](C:\Users\AtsukoRuo\Desktop\note\Flutter\assets\image-20230716174313691.png)



一个简单的示例：
~~~dart
class TabsScreen extends ConsumerStatefulWidget {		//注意继承的类
    const TabsScreen({super.key});
    @override
    ConsumerState<TabsScreen> createState() {
        return _TabsScreenState();
    }
}

class _TabsScreenState extends ConsumerState<TabsScreen> {
    @override
    Widget build(BuildContext context) {
    	final meals = ref.watch(mealsProvider);		//ref是由ConsumerState提供的，watch监听某个Provider，并返回Provider所提供的值。若Provider发生变化，那么就会执行build
    }
}


final mealsProvider = Provider((ref) {
    return dummyMeals;
});

void main() {
  runApp(const ProviderScope(
    child : App()
  ));
}
~~~


再给出一个示例：

~~~dart
//泛型为被管理数据的类型
class FavoriteMealsNotifier extends StateNotifier<List<Meal>> {
    FavoriteMealsNotifier() : super([]);        //super传入初始值
    bool toggleMealFavoriteStatus(Meal meal) {
        //通过state访问你要管理的数据
        //必须要求是被管理的对象List<Meal>以不可变的方式处理
        final mealIsFavorite = state.contains(meal);
        if (mealIsFavorite) {
            state = state.where((m) => meal.id != m.id).toList();
            return false;
        } else {
            state = [...state, meal];
            return true;
        }
    }

}

//如果提供的数据要改变，那么必须用StateNotifierProvider。
final favoriteMealsPovider = 
    StateNotifierProvider<FavoriteMealsNotifier, List<Meal>>(
        (ref) {
            return FavoriteMealsNotifier();
        }
);

//不可变数据可以放在Provider中


//一个Provider依赖于另一个Provider，数据处理逻辑可以抽象出来放在Provider中
final filteredMealsProvider = Provider((ref) {
    final meals = ref.watch(mealsProvider);
    final activeFilters = ref.watch(filtersProvider);
    
    return meals.where((meal) {
            if (activeFilters[Filter.glutenFree]! && !meal.isGlutenFree
            || activeFilters[Filter.lactoseFree]! && !meal.isLactoseFree
            || activeFilters[Filter.vegan]! && !meal.isVegan
            || activeFilters[Filter.vegetarian]! && !meal.isVegetarian) 
                return false;
            return true;
        }).toList();
});




class TabsScreen extends ConsumerStatefulWidget {
    const TabsScreen({super.key});
    @override
    ConsumerState<TabsScreen> createState() {
        return _TabsScreenState();
    }
}

class _TabsScreenState extends ConsumerState<TabsScreen> {
    @override
    Widget build(BuildContext context) {
        //监听值是否发生改变，如果改变了，那么就重新执行该build
        final meals = ref.watch(mealsProvider);		//这个ref再ConsumerState中
    }
}


//注意继承的类
class MealDetailsScreen extends ConsumerWidget {
    @override
    Widget build(BuildContext context, WidgetRef ref) {		//这里要提供一个ref参数
        final wasAdded = ref.read(favoriteMealsPovider.notifier).toggleMealFavoriteStatus(meal);
        //调用对象上的方法，来修改该数据。read方法不具备监听，适合用于StatelessWidget中
    }
}

~~~



在需要状态变化驱动UI更新的场景中,通常使用 watch(),其他情况可以使用 read()。

如果是一个页面退出销毁之后，它共享状态被其他Widget改变。在重新创建该页面时，可以使用read()。



总结来说,watch() 和 read() 在 Riverpod 中担任着不同的角色:

- watch() 是驱动重建的主力,是响应式编程的核心。
- read() 仅用于简单读取状态,更轻量,可以避免不必要的重建。



在 Riverpod 框架中,当使用 watch 方法观察一个 Provider 时,如果这个 Provider 的状态发生了变化,那么是会重新调用 Build 方法的。

watch 的工作流程是:

1. watch 方法会在 Widget 首次构建时立即读取 Provider 当前状态,并添加一个监听器(Subscriber)。
2. 当 Provider 的状态发生变化时,会通知 Riverpod 框架,该 Provider 有监听器需要更新。
3. Riverpod 框架将这个 Provider 使用到的所有 Widget 标记为"脏数据"(dirty)。
4. 在下一帧重绘时,Flutter 框架会重新构建这些脏数据 Widget。
5. 重新构建时,会再次执行 Widget 的 Build 方法,在 Build 方法中又会调用 watch 重新读取 Provider 的最新状态。
6. 使用最新状态构建Widget子树。

所以 watch 方法的确会在 Provider 状态变化时,引起 Widget 的 Build 方法被再次调用,来构建一个使用了最新状态的 Widget 子树。这就是 Riverpod 使用 Provider + Consumer 的方式来实现状态变化驱动 UI 更新的核心机制。开发者只需要关注状态管理,UI 可以自动更新。





