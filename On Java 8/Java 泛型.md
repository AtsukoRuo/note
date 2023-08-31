# Java 泛型

[TOC]

## 概述

[Java 不能实现真正泛型的原因是什么？](https://www.zhihu.com/question/28665443/answer/1873474818)

事实上，对于泛型的翻译有两种策略：

- **同构翻译（homogeneous translation）**：一个所有类型共享一种实现
- **异构翻译heterogeneous translation）**：一个为每种类型组合都创建一份特化

C++ 和Java 的策略分别处于异构翻译和同构翻译的极端。而 C#/CLR 处于两者中间，为共享布局的引用类型同构翻译，为值类型异构翻译。

这样任何是用Java泛型的地方都可以是用Object类型代替。但是使用泛型语法可以提供编译时类型检查。**具体来说提供参数与返回类型的类型检查**。



类型擦除的问题：

- **获取泛型参数的具体值。**程序员可能会想要知道一个 `List` 到底是 `List<String>` 还是 `List<Integer>`，想要拿到实际泛型参数相关的信息，而因为类型擦除，实际上并不能做到这一点。可以通过传递Class对象补偿这一点损失
- **布局特化。**当前基于擦除的实现要求泛型参数类型必须拥有公共运行时表示（common runtime representation），在 Java 里这意味着只能是一组引用类型，而不能为原始类型。这导致我们只能用 `List<Integer>`，而用不了针对 `int` 进行布局特化的 `List<int>`，底层存放的全是对 `Integer` 对象的引用，这造成了巨大的内存浪费，同时对现代 CPU 的缓存策略极端不友好，大量间接寻址产生大量 cache miss，产生大量的性能下降。这也是当前 Java 泛型擦除最大的问题。
- **运行时类型检查。**因为泛型实际参数会被擦除，`List<String>` 会被擦除为 `List`，所以当通过一些手段（强制转换，raw type 等）将其他类型的值放入这个 List 的时候并不会出错，直到实际访问时才会发生问题。



模板（异构翻译）的问题：

- 模板的每个实例都要有着不同的代码，这意味着模板展开会导致**代码膨胀**。产生更大的硬盘和内存占用



类型擦除的优势：

- **兼容性**。它完全维护了二进制兼容性和源代码兼容性。不会因引入泛型后让社区生态产生巨大的分歧

- **类型系统的自由度**：虽然 Java 和 JVM 常常被绑定在一起，但它们是各自独立的，有着各自的规范。由于通过类型擦除实现泛型，像 Scala 这样的语言可以以与 Java 泛型高度协同的方案实现远远超出 Java 类型系统表达能力的类型系统，同时保持高度的互操作性。

## 泛型的基本语法

1. 定义泛型类

   ~~~java
   class GenericClass<T> {
     private T element;
   
     public T getElement() {
       return element; 
     }
     
     public static T getElement() {}		//错误的，静态方法不能使用类的泛型参数
   }
   ~~~

2. 定义泛型方法

   ~~~java
   class GenericMethod {
     public static <T> T identity(T t) {
       return t;
     }
   }
   ~~~

3. 泛型类不支持继承具体类型

   ~~~java
   class Child extends GenericClass<String> //错误
   ~~~



下面看一个类型擦除的潜在问题：

~~~java
public static <U extends Comparable<U>> int foo(Vector<U> u) {}
foo(new Vector<Integer>());					//java.lang.ClassCastException
~~~

这段代码在编译时无报错。但是在运行时，由于类型擦除，解释器试图将Object转换为Comparable时抛出异常。即使这个Object的真实类型为Integer，原因未知。解决方案：

~~~java
public static <U extends Object & Comparable<U>> int foo(Vector<U> u) {}
~~~



原生类型的问题

~~~java
class List<T> {
    public Vector<Integer> getVector() {}
}

List list = new List();
for (Integer v : list.getVector()) 		//错误的，因为这里使用了原生类型List，所以Vector<Integer>变成了Vector<Object>。原因未知
~~~



## 泛型数组

[一个很好解释泛型数组的帖子](https://www.zhihu.com/question/300950759/answer/528504962)

类型擦除是在编译之后才丢失类型信息的。而在编译过程中编译器掌握完整的、为丢失泛型类型信息。

Java在类型系统中留下了一个缺陷：协变数组类型。也就是可以将派生类型数组赋值给基类数组的引用 。我们看这段代码：

~~~java
String[] strArr = new String[10];
Object[] arr = strArr;
arr[0] = 0;
~~~

因为 Java 中的数组类型是协变的，所以这段代码能通过编译。strArr是一个字符串数组，本不应该放入Integer，但是这里通过Object数组规避了这一点。因此Java只能把检查这种类型错误的工作放在运行时，并通过抛出ArrayStroreException来通知这个错误。

明白了这一点后，我们再来看这个例子：

~~~java
List<String>[] strListArr = new List<String>[10];
Object[] arr = strListArr;
arr[0] = new ArrayList<Integer>();
~~~

由于泛型类型擦除， 在运行时允许将`ArrayList<Integer>`放入到`List<String>[]`类型的数组中。**堆污染（Heap pollution）**就这样在编译和运行时都没有警告的时候发生了。这种情况是不被 Java 所允许的，所以 Java 禁止了一些泛型数组的相关操作（譬如通过new操作符来创建泛型数组） `new T[size]; //Error`。Java设计者让用户必须使用类型不安全的手段（例如，原生类型`ArrayList`、强制类型转换），让开发者自己保证类型安全。



我们可以通过三种方法创建一个与泛型数组作用相同的数组：

- Object[]，在Object[]与泛型类型数组T[]进行强制类型转换。（不考虑通配符/边界的情况下）
- 原生类型强制转换
- 使用原生类型ArrayList。



## 边界 & 通配符

### 边界

边界让你在使用泛型的时候，可以在参数类型上增加限制。此时你可以调用边界类型上的方法了。

边界的语法规则：

- 只能有一个具体类，而且具体类（如果有的话）必须放在最前面
- 可以有任意个接口

~~~java
class WithColor<T extends HasColor> {}
class WithColorCoord<T extends Coord & HasColor>
~~~



在继承层次中，边界的语法：

~~~java
class A { }
interface B { }
interface C { }
class I1<T> { }
class I2<T extends A & B> extends I1<T> { }
class I3<T extends A & B & C, U, W> extends I2<T> { }
class I4 <T extends A & C> extends I2<T> { }			//Type parameter 'T' is not within its bound; should implement 'B'
~~~

在继承时，如果子类的泛型参数`T`要实现父类的泛型参数`U`，那么`T`必须拥有`U`的全部边界。



**通配符只限于在单边界中使用**：

~~~java
List<? extends SuperHearing> audioPeople;
List<? extends SuperHearing & SuperSmell> dogPeople;		//语法错误
~~~

而且不能在泛型声明中使用：

~~~java
class A <? extends Object> {
    //语法错误
}
~~~



### 通配符

我们在泛型数组中讨论过协变数组的问题：

~~~java
String[] strArr = new String[10];
Object[] arr = strArr;
arr[0] = 0;								//运行时错误，编译通过
~~~

但是泛型并没有内建的协变性，因此下面代码在编译时就不通过：

~~~java
List<Fruit> flist = new ArrayList<Apple>();		//Compile Error: incompatible types
List<Fruit> flist = new ArrayList<Fruit>();		//OK
~~~

问题本质在于我们讨论的是**集合自身的类型，而不是它所持有的元素类型**。

不过有时你想在这两者之间建立某种向上转型的关系。通配符可以达到这个目的。也就是说**通配符在一定程度上实现了泛型协变**。

~~~java
List<? extends Fruit> flist = new ArraytList<Apple>();
~~~

协变引入了类型安全问题，下面给出一个例子说明这一点：

~~~java
List<Apple> apples = new ArrayList<>();
List<? extends Fruit> fruits = apples; 
fruits.add(new Orange());				//Orange类型放入了List<Apple>中，这只能在运行时检查出。
~~~

为了避免这种运行时安全问题，Java设计者规定：一旦进行了这种“向上转型”，**你便失去了向泛型参数传递任何对象的能力**。例如`List`有这样一个方法`add(T element)`，那么`flist.add(new Fruit())`便会被编译器阻止。但是调用一个返回`Fruit`类型的方法是安全的，因为`List`中的任何元素至少都必须是`Fruit`类型的，所以编译器允许这么做。



解决“无法写入”问题：

- 通过将泛型参数改为`Object`类型

-  使用边界代替通配符
	~~~java
	public static void print(List<? extends Fruit> list) {
	    //list.add(new Fruit());
	}
	~~~

	改为

	~~~java
	public static <T extends Fruit> void print(List<T> list) {
	    list.add(new Fruit());
	}
	~~~

	

### 逆变性 超类通配符

超类通配符的例子：

~~~java
List<? super Apple> apples = new ArrayList<Fruit>();
//apples = new ArrayList<Orange>();
~~~

apples是由某种Apple的基类组成的List，因此你知道**可以安全地向其中添加Apple类型或其子类型**。但是如果调用返回类型为泛型类型的方法，则必须进行强制转换，这是因为读操作是不安全的。



### List\<?\>、List\<? extends Obejct\>、List、List\<Object\>的区别

List<?>与List<? extends Object>是等价的表述，它们在添加元素时有限制（原因见上述讨论），而`List`、`List<Object>`可以随意添加元素。





它们之间赋值的例子：

~~~java
List<Apple> list1 = new ArrayList<Apple>();
List<? extends Object> list2 = list1;
List<? extends Apple> list3 = list2;		//语法错误
/**
Required type: List <? extends Apple>
Provided: List <? extends Object>
*/

List<Apple> list4 = list2;					//语法错误
/**
Required type: List <Apple>
Provided: List <? extends Object>
*/

List<Object> list5;
List<T> list6 = list5;					//语法错误
/**
Required type: List <T>
Provided: List <Object>
*/
~~~

但是原生类型可以随意赋值给任意泛型变量，但是这是一个未检查（unchecked）赋值，有潜在的类型转换安全问题：

~~~java
List list = new ArrayList<Apple>();
List<Orange> list2 = list;
~~~



## 自限定类型

首先给出一个难以理解的泛型声明——自限定类型：

~~~java
class SelfBounded<T extends SelfBounded<T>> {}
~~~



要理解这种泛型使用方法，我们要理解奇异递归类型，它的代码范式如下：

~~~java
class GenericType<T> {}
public class CuriouslyRecurringGeneric extends GenericType<CuriouslyRecurringGeneric> {}
~~~

继Jim Coplien在C++领域提出**奇异递归模板模式（Curiously Recurring Template Pattern）**后，这种方式可以称为**奇异递归泛型（curiously recurring generics, CRG）**。其中“奇异递归”指的是子类在基类中出现的现象。

CRG的精髓在于基类用子类替换了其参数。这意味着**泛型基类变成了一种为其子类实现通用功能的模板**，但是所实现的功能会将派生类型用于所有的参数和返回值。也就是说，基类中使用的是具体子类类型，而不是基类。

下面给出一个例子：

~~~java
public class BasicHolder<T> {
    T element;
    void set(T arg) { element = arg; }
    T get() { return element; }
}
class Subtype extends BasicHolder<Subtype> {
    public static void main(String[] args) {
        Subtype st1 = new Subtype(), st2 = new Subtype();
        s1.set(st2);
        Subtype s3= s1.get();
    }
}
~~~

BasicHolder可以将任何类型作为其泛型参数。例如：

~~~java
class Other {}
class BasicOther extends BasicHolder<Other> {}
~~~

通过自限定可以将条件加强：作为泛型参数的**类类型必须处于指定的继承关系**中。

~~~java
class SelfBounded<T extends SelfBounded<T>> {
    
}
class A extends SelfBounded<A> {}
class B extends SelfBounded<B> {}
class D {}
class F extends SelfBounded<D> {}		//Error

class Other<T extends Compare<T>> {}	//子类必须实现Compare接口
~~~

`<T extends SelfBounded<T>>`中的`SelfBounded<T>`是CRG，而`T extends U`是自限定，两者结合起来的意思是**泛型参数必须继承自边界`U`，而这个边界一般是子类的模板类（应用了模板设计模式）！**



自限定类型还可以实现参数协变性。这里不再介绍，需要的时候再补充。





