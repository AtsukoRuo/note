# Java 反射

[TOC]

## 概述

反射可以在程序运行时获取对象的类型信息，使我们摆脱了只能在编译时执行面向类型操作的限制。

下面我们看一个最基本的反射：在运行时确定对象的类型，检查所有的类型转换是否正确。

~~~java
abstract class Shape {}
class Circle extends Shape {}
class Triangle extends Shape {}

Circle c = new Circle();
Triangle t = new Triangle();
c = (Circle)t			//反射
Shape a = (Shape)c		//反射 + 多态
~~~



程序中的每一个类都有一个对应的Class对象。而这个特殊的Class对象保存了运行时的类型信息。每次编写并编译一个新类时，都会生成一个Class对象（并被相应地存储在同名的.class文件中）。为了生成这个对象，Java虚拟机（JVM）使用被称为类加载器（class loader）的子系统。

类加载器子系统实际上可以包含一条类加载器链，但里面只会有一个原始类加载器，它是JVM实现的一部分。原始类加载器通常从本地磁盘加载所谓的可信类，包括Java API类。通常来说我们不需要加载器链中的额外加载器，但对于特殊需要（例如以某种方式加载类以支持Web服务器应用程序，或通过网络来下载类），你可以引入额外的类加载器来实现。

当程序第一次引用该类的静态成员时，就会触发这个类的加载，也就是生成该类型的Class对象。构造器是类的一个静态方法，尽管没有明确使用static关键字。因此，使用new操作符创建类的新对象也算作对该类静态成员的引用，而构造器的初次使用会导致该类的加载。当该类的字节数据被加载时，它们会被验证，以确保没有被损坏，并且不包含恶意的Java代码（这是Java的众多安全防线里的一条）。一旦该类型的Class对象加载到内存中，它就会用于创建该类型的所有对象。

所以，Java程序在运行前并不会被完全加载，而是在必要时加载对应的部分。这与许多传统语言不同。这种动态加载能力使得Java可以支持很多行为，而它们在静态加载语言（如 C++）中很难复制，或根本不可能复制。

总结一下，在使用一个类之前，需要先执行以下3个步骤：

- 加载：这是由类加载器执行的。该步骤会先找到字节码（通常在类路径中的磁盘上，但也不一定），然后从这些字节码中创建一个Class对象。
- 链接：链接阶段会验证类中的字节码，为静态字段分配存储空间，并在必要时解析该类对其他类的所有引用。
- 初始化：如果有基类的话，会先初始化基类，执行静态初始化器和静态初始化块。





## 获取Class对象

### forName()

所有Class对象都属于Class类。可以通过静态的forName()方法获取到Class对象的引用：

~~~java
try {
	Class.forName("reflection.toys.Gum");
} catch(ClassNotFoundException e) {
    
}
~~~

注意：传递给forName()的字符串参数必须是类的完全限定名称。

### getClass()

如果已经有了一个你想要的类型的对象，就可以通过getClass()方法来获取Class引用，这个方法属于Object根类。它返回的Class引用表示了这个对象的**实际类型**

~~~java
Fruit fruit = new Apple();
Class c1 = fruit.getClass();		//实际上为Apple
~~~

### class字面值

~~~java
FancyToy.class
~~~

每个基本包装类都有一个名为TYPE的标准字段。TYPE字段表示一个指向和基本类型对应的Class对象的引用。

~~~java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
~~~

`int.class`等价于`Integer.TYPE`，`void.class`等价于`Void.TYPE`。其他基本类型类似。



使用.class语法来获取对Class对象的引用不会导致对应类型的初始化。而Class.forName()会立即初始化类。而且对于.class语法来说，如果一个static final字段的值是“编译时常量”，访问它并不会导致类的初始化。下面给出一个例子说明这一点：

~~~java
class Initable {
  static final int STATIC_FINAL = 47;
  static final int STATIC_FINAL2 =
    ClassInitialization.rand.nextInt(1000);			//不是编译期常量
  static {
    System.out.println("Initializing Initable");
  }
}

class Initable2 {
  static int staticNonFinal = 147;
  static {
    System.out.println("Initializing Initable2");
  }
}

class Initable3 {
  static int staticNonFinal = 74;
  static {
    System.out.println("Initializing Initable3");
  }
}

public class ClassInitialization {
  public static Random rand = new Random(47);
  public static void
  main(String[] args) throws Exception {
    Class initable = Initable.class;
    System.out.println("After creating Initable ref");
    // Does not trigger initialization:
    System.out.println(Initable.STATIC_FINAL);
    // Triggers initialization:
    System.out.println(Initable.STATIC_FINAL2);
    // Triggers initialization:
    System.out.println(Initable2.staticNonFinal);
    Class initable3 = Class.forName("Initable3");
    System.out.println("After creating Initable3 ref");
    System.out.println(Initable3.staticNonFinal);
  }
}
/* Output:
After creating Initable ref
47
Initializing Initable
258
Initializing Initable2
147
Initializing Initable3
After creating Initable3 ref
74
*/
~~~



## Class对象的方法

Class对象具有以下方法

- getInterfaces()来查询Class对象的所有接口，返回了一个Class对象数组
- getSuperclass()来查询Class对象的直接基类，返回Class对象
- Class的newInstance()调用类的无参构造器。现已被弃用，推荐使用Constructor.newInstance()来代替
- getSimpleName()返回类名
- getCanonicalName()返回完全限定名



## Class对象与泛型

你可以使用泛型语法来限制Class引用的类型：

~~~java
Class intClass = int.class;
intClass = double.class;
Class<Integer> genericIntClass = int.class;
genericIntClass = Integer.class; // Same thing
// genericIntClass = double.class; // Illegal


Class<Number> genericNumberClass = int.class //(1)
~~~

对于（1）似乎有道理的，因为Integer继承了Number。但是要考虑泛型本身的类型，`Class<Number>`与`Class<Integer>`之间并不存在继承关系。可以通过通配符`<? extends Number>`来解决这个问题。



下面我们看一个例子：

~~~java
Class<FancyToy> ftc = FancyToy.class;
Class<? super FancyToy> up = ftc.getSuperclass();
// This won't compile:	Required type:Class<Toy>    Provided:Class<? super FancyToy>
//because  public native Class<? super T> getSuperclass();
Class<Toy> up2 = ftc.getSuperclass();
~~~


## instanceof关键字

关键字instanceof，它返回一个boolean值，表明一个对象是否是特定类型的实例：

~~~java
Animal x = new Dog();
x instanceof Dog		//true
x instanceof Animal		//true
~~~

它对根据对象实际运行类型来判断的

`Class.isInstance()`方法提供另一种动态验证对象类型的方式。



## 运行时的类信息

Class类和java.lang.reflect库一起支持了反射，这个库里包含Field、Method以及Constructor类（每个都实现了Member接口）。这些类型的对象是由JVM在运行时创建的，用来表示类中对应的成员。这样你就可以使用Constructor来创建新的对象，使用get()和set()方法来读取和修改与Field对象关联的字段，使用invoke()方法调用与Method对象关联的方法。另外，你还可以很便捷地调用getFields()、getMethods()和getConstructors()等方法，以返回表示字段、方法和构造器的对象数组。这样，类信息可以在运行时才完全确定下来，而在编译时就不需要知道任何信息。



### Field

表示一个对象的字段

- getFields()：获取所有public字段,包括父类的public字段
- getDeclaredFields()：获取所有字段,public和protected和private,但是不包括父类字段

### Method

表示一个对象的方法

-  Object invoke(Object obj, Object... args)
  - obj：被调用的对象，将this指针设置为obj。
  - args：方法参数

### Constructor

表示一个对象的构造器



## 动态代理

（proxy）是基本的设计模式之一。它是为了代替“实际”对象而插入的一个对象，从而提供额外的或不同的操作。这些操作通常涉及与“实际”对象的通信，因此代理通常充当中间人的角色。下面是一个用来展示代理结构的简单示例：

~~~java
interface Interface {
  void doSomething();
  void somethingElse(String arg);
}

class RealObject implements Interface {
  @Override public void doSomething() {
    System.out.println("doSomething");
  }
  @Override public void somethingElse(String arg) {
    System.out.println("somethingElse " + arg);
  }
}
~~~



~~~java
//代理对象
class SimpleProxy implements Interface {
  private Interface proxied;
  SimpleProxy(Interface proxied) {
    this.proxied = proxied;
  }
  @Override public void doSomething() {
    System.out.println("SimpleProxy doSomething");		//做一些额外的事情
    proxied.doSomething();							//再调用实际对象的功能
  }
  @Override public void somethingElse(String arg) {
    System.out.println(
      "SimpleProxy somethingElse " + arg);			//做一些额外的事情
    proxied.somethingElse(arg);
  }
}

class SimpleProxyDemo {
  public static void consumer(Interface iface) {
    iface.doSomething();
    iface.somethingElse("bonobo");
  }
  public static void main(String[] args) {
    consumer(new RealObject());
    consumer(new SimpleProxy(new RealObject()));
  }
}
/* Output:
doSomething
somethingElse bonobo
SimpleProxy doSomething
doSomething
SimpleProxy somethingElse bonobo
somethingElse bonobo
*/
~~~



Java的**动态代理（dynamic proxy）**比代理更进一步，它可以动态地创建代理，并动态地处理对所代理方法的调用。在动态代理上进行的所有调用都会被重定向到一个**调用处理器（invocation handler）**上，这个调用处理器的工作就是发现这是什么调用，然后决定如何处理它。

~~~java
class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;     //实际对象
    DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //proxy 代理的对象
        //method 当前调用的方法（声明在某一接口中）
        //args 方法参数
        if (method.getName().equals("interesting")) {
            return null;		//过滤某个方法
        }
        return method.invoke(proxied, args);
    }
}

public class SimpleDynamicProxy {
    public static void main(String[] args) {
        RealObject real = new RealObject();
        Interface proxy = (Interface) Proxy.newProxyInstance(
                Interface.class.getClassLoader(),		//类加载器，由于本地类加载是唯一的，所以通常可以从一个已经加载的对象里获取其类加载器，然后传递给它就可以了。
                new Class[] { Interface.class},			//一个希望代理实现的接口列表（不是类或抽象类）
                new DynamicProxyHandler(real)			//以及InvocationHandler接口的一个实现
        );
        proxy.doSomething();					//动态代理会将所有调用重定向到调用处理器，因此调用处理器的构造器通常会获得“实际”对象的引用，以便它在执行完自己的中间任务后可以转发请求。

    }
}
~~~



动态代理与静态代理唯一的区别在于：**在编译期是否知道要被代理的对象！**

## Optional



没有返回值这一事实可用null对象、标记接口等表示。而Optional提供了一种统一的、约定俗成的方式来表示没有返回值，并且提供了一系列API来方便的处理各种情况。但是在实践中，Optional并没有达到原本的设计目标。

Optional 是 Java 8 引进的一个新特性，我们通常认为Optional是用于解决Java臭名昭著的空指针异常问题。但是Brian Goetz （Java语言设计架构师）对Optional设计意图的原话如下：

> Optional is intended to provide a limited mechanism for library method return types where there needed to be a clear way to represent “no result," and using null for such was overwhelmingly likely to cause errors.

Optional的机制类似于 Java 的检查异常，**强迫API调用者面对没有返回值的现实**。**不要**将Optiona对象作为字段、参数，以及用它处理空指针异常问题或实现if-else逻辑。



当我们在自己的代码中生成一个 **Optional**对象时，可以使用下面 3 个静态方法：

- `empty()`：生成一个内容为null的 **Optional**对象（简称为空Optional）。相当于Optional.ofNullable(null)。
- `of(value)`：将值保存在Optional中，若为null，则抛出异常。
- `ofNullable(value)`： 将值保存在Optional中，可保存null值。



当你接收到 **Optional** 对象时，应首先调用 `isPresent()` 检查内容是否为null。可使用 `get()` 获取Optional的内容。



有许多便利函数可以解包 **Optional** ，这简化了上述“对所包含的对象的检查和执行操作”的过程：

- `ifPresent(Consumer)`：当值不为null时调用 **Consumer**，否则什么也不做。
- `orElse(otherObject)`：如果值不为null则直接返回，否则生成 **otherObject**。
- `orElseGet(Supplier)`：如果值不为null则直接返回，否则使用 **Supplier** 函数生成一个可替代对象。
- `orElseThrow(Supplier)`：如果值存在直接返回，否则使用 **Supplier** 函数生成一个异常。





**Optional** 的后续能做更多的操作

- `filter(Predicate)`：对 **Optional** 中的内容应用**Predicate** 并将结果返回。如果 **Optional** 不满足 **Predicate** ，将 **Optional** 转化为空 **Optional** 。如果 **Optional** 已经为空，则直接返回空**Optional** 。
- `map(Function<? super T, ? extends U> mapper)`：如果 **Optional** 不为空，应用 **Function** 于 **Optional** 中的内容，并返回结果。否则直接返回空Optional。
- `flatMap(Function<? super T, ? extends Optional<? extends U>)`：如果 **Optional** 不为空，应用 **Function** 于 **Optional** 内容中的内容，并返回结果。否则直接返回空Optional。
