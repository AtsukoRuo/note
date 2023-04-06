# Java 面向对象

在Java中**一切皆为对象**

[TOC]

## 基本语法

除基本类型外，Java中的对象都是通过**引用（reference）**来操作，例如

~~~java
Person person = new Person("AtsukoRuo", 18);
~~~

这里的`person`标识符实际上就是一个引用，它指向位于堆中的实际对象。



定义类

~~~java
class ClassName { }
~~~

通过new关键字创建该类的对象

~~~java
ClassName a = new ClassName();
~~~

当定义一个类时，你可以为其定义两种元素：**字段**（有时叫作“数据成员”）和**方法**（有时叫作“成员函数”）。



访问一个对象的字段或方法

~~~java
a.b = 10;		
a.c();
d();			//直接访问本类中的方法（隐式使用this参数）
this.e;			//通过this参数访问本类中的字段
~~~



### static

然而在两种情况下，这种只有通过对象调用方法的做法会显得不合时宜。

- 第一种情况是，有时候我们需要一小块共享空间来保存某个特定的字段，而并不关心创建多少个对象，甚至有没有创建对象。
- 第二种情况是，你需要使用一个类的某个方法，而该方法和具体的对象无关；换句话说，你希望即便没有生成任何该类的对象，依然可以调用其方法。生命周期持续到程序结束。



static关键字（源自 C++）为以上两个问题提供了解决方案。有些面向对象编程语言也会使用**“类数据”（class data）**和**“类方法”（class method）**来表示该数据或方法只服务于类，而非特定的对象。



> 注：我们平常说字段根据上下文区分是否包括static字段。



访问一个对象的static字段或static方法：

~~~java
A a = new A();
A.staticMethod();			//通过类名访问（推荐）
a.staticMethod();			//通过对象名访问
staticMethod();				//直接访问本类中的静态方法
~~~



**static字段是基于类创建的，而非static字段是基于对象创建的**。因此**不可在static方法中直接访问本类的非static字段、非static方法，以及使用this参数**，但是在非static方法中**可以**直接访问static字段。



**static具有全局方法的语义！**，因此有些人认为静态方法不符合面向对象的思想。关于它是否是“正确的OOP”就留给理论家们来争辩吧。



### this

~~~java
Banana a = new Banana(), b = new Banana();
a.peel(1);
b.peel(1);
~~~

编译器怎么知道`peel`方法是被`a`调用还是被`b`调用。每个对象方法（除static方法）都有一个隐藏参数this，位于所有显示参数之前，代表着被操作对象的引用。编译后的内部表现形式为：
~~~java
Banana.peel(a, 1);
Banana.peel(b, 1);
~~~

你不能这样编写代码，并试图通过编译，但它可以让你了解一些内部实际发生的事情。



如果你想在一个方法里获得对当前对象的引用，那么就使用this关键字：

~~~java
Banana getBanana() {
	return this;
}
~~~

如果从类的一个方法中调用该类的另一个方法或字段，那就没必要使用this，直接调用即可。

~~~java
class Car {
	int velocity;
    void speedDown(int velocity) { this.velocity -= velocity; }		//注意：局部变量名屏蔽了字段名，此时必须通过this引用该字段
    void stop() { 
        speedDown(velocity);
    }
}
~~~



> 有些人会痴迷于把this放在每个方法调用和字段引用的前面，认为可以使代码“更清晰、更明确”。不要这样做。我们使用高级语言是有原因的，它们可以帮助我们处理这些细节。如果在不必要的时候使用了this，就会让所有阅读代码的人感到困惑和烦恼。



### final

Java的final关键字在不同的上下文环境里含义可能会略有不同，但一般来说，它表示“这是无法更改的”。阻止更改可能出于两个原因：设计或效率。



#### 数据final

常量可以分为：

- **编译时常量**：对编译时常量来说，编译器可以将常量值“折叠”到计算中。这节省了一些运行时开销。**在Java里，这些常量必须是基本类型，在定义常量时必须初始化**。
- **运行时常量**：运行时常量允许延迟初始化。对于基本类型，它的值不变；一旦引用被初始化为一个对象，它就永远不能被更改为指向另一个对象了。但是，对象本身是可以修改的。Java没有提供使对象恒定不变的方法。**总而言之，final确保栈上的值保持不变**



~~~java
int i = 10;
final int k = 10;		//编译时常量
final int j = i;		//运行时常量
~~~



**空白final**是没有初始值的final字段。编译器会确保在使用前初始化这个空白final字段。而且必须在构造器执行完之前完成初始化工作。

#### 方法final

将一个类定义为final，就阻止了该方法的继承

使用final方法的原因有两个：

- 设计上：防止继承类通过重写来改变该方法的含义

- 效率上：

	但在大多数情况下，它不会对程序的整体性能产生什么影响，**因此最好仅将final用作设计决策，而不是尝试用它提高性能**。长期以来，Java都不鼓励使用final来进行优化。

	- 在Java的早期实现中，如果创建了一个final方法 ，编译器可以将任何对该方法的调用转换为内联调用，即通过复制方法体中实际代码的副本来代替方法调用。而正常的方法调用方式则是将参数压入栈，跳到方法代码处并执行，然后跳回并清除栈上的参数，最后处理返回值。这节省了方法调用的开销，但会让代码开始膨胀。
	- 它告诉编译器自己不需要动态绑定。这让编译器可以为final方法的调用生成更为高效的代码。



注：**类中的任何private方法都是隐式的final。**

#### 类final

将整个类定义为final时（通过在其定义前加上final关键字），就阻止了该类的所有继承。它的所有方法都是隐式final的。因此无法重写它们。

## 构造器 & 初始化

> “不安全”的编程是导致编程成本高昂的罪魁祸首之一。初始化（initialization）和清理（cleanup）正是导致“不安全”编程的其中两个因素。

使用构造器保证对象的初始化。构造器是一种特殊的方法，它**没有返回类型**，与返回类型为空（void）有着明显的不同。而且**构造器是static方法**！



创建一个构造器：

~~~java
class ClassName {
    ClassName(/*args*/) {}
}
~~~

当创建对象时：

~~~java
new ClassName();
~~~

会自动调用这个类的构造器。



### 默认构造器

不带参数的构造器称为**默认构造器（default consttructor）**或者称为**无参构造器（no-arg constructor）**。当你未显式创建任何构造器时，编译器会为你创建一个隐式的无参构造器。





### 构造器中调用构造器

this + 参数列表

~~~java
class Phone {
    String model;
	Phone(String model) { this.model = model; }
    Phone() { this("Android"); }
    Phone() { this("Android"); this("iOS"); }	// 不能同时调用两个构造器
    Phone() { model = ""; this("iOS"); }	    // this()必须是第一条语句
    void f() { this("Android"); }			   // 不能在非构造器中使用
}
~~~



### 初始化 & 初始化中的前向引用问题

默认初始化：

~~~java
public class InitialValues {
    boolean t;
    char c;
    InitialValues reference;
}
~~~



赋值初始化：

~~~java
public class InitialValues {
    //int i = j;		  禁止前向引用
    int j = 10;
    boolean t = false;
    char c = 'c';
    InitialValues reference = new InitialValues();
}
~~~



方法初始化：

~~~java
public class MethodInit {
    //int j = g(i);			禁止前向引用
    int i = f();
    int k = g(i);
    
    int f() { return 0; }
    int g(int j) { return j * 2; }
   	 
    static int i = h();
    //static int i = f();			不能调用非static方法
    static int h () { return 0; }
}
~~~

注意这里有个陷阱：

~~~java
public class MethodInit {
    int i = f();			// i = 0;
    int j = 10;
    int f() { return j; }
}
~~~

这是因为在调用`f()`时，`j`的值为`0`，还未执行到`int j = 10`这条初始化语句。



静态初始化：

~~~java
public class Spoon {
    static int i = 0;
    static {
        i = 47, j = 48; k = 49;		//规避掉了前向引用，十分不推荐这么做。
        System.out.println("Satic {}");
    }
    static int j;
    static int k = 0;				//重新赋值
    public static void main(String[] args) {
        System.out.println(j);
        System.out.println(k);
    }
}
/*
Satic {}
48
0
*/
~~~

注意：**静态字段只会被初始化一次**



实例初始化（instance initialization）：

~~~java
class Mug {
    Mug mug1, mug2;
    {
        mug1 = new Mug();
        mug2 = new Mug();
    }
}
~~~



### 初始化顺序

1. 当类首次被加载时，按照声明顺序初始化的static字段、以及执行静态初始化。

2. 在构造对象时，为对象分配的存储空间会被初始化为二进制零。这相当于对字段执行默认初始化。

3. 然后按照声明顺序初始化非static成员、以及执行实例初始化。

4. 最后执行构造器

如果在继承体系中，会从基类到子类递归地重复（1）或者 （2）~（4）。



## 清理对象

Java有**垃圾收集器（Garbage Collector）**来回收不再使用的对象内存！**注意它只回收对象的内存资源**。如果一个对象分配了一块“特殊”的内存，例如在本地方法（C++）中通过malloc分配内存、持有文件句柄、持有socket，那么GC对释放对象的这块“特殊”内存无能为力，这得由程序员手动释放这块特殊内存！



> 注：在java16中已经完全废用finalize方法。推荐显式调用对象的释放资源的方法，惯用命名有close()、dispose()、clean()等

为此Java在Object类中提供了finalize()的方法，任何类都可以重载此方法。当GC释放对象时，它会调用该对象的finalize()方法。何时调用取决于JVM所采用的策略。

但是finalize()方法绝不等同于C++中的析构函数。因为C++在销毁对象时必须调用这个函数。而在Java中，对象并不总是被回收。你也许会发现即使程序的某个对象不再使用，但它的存储空间一直没有被释放，直到在程序退出时这些存储空间才会全部归还给操作系统。这个策略是恰当的，因为垃圾收集本身也有开销，如果没有做过垃圾收集，那就不用承担这部分开销了。所以finalize()的调用是不可预测的，而且很危险但没必要的。**推荐显式调用对象的释放资源的方法，惯用命名有close()、dispose()、clean()等**，即清理工作由程序员负责！

~~~java
class TerminationCondition {
    @SuppressWarnings("deprecation")
    @Override
    public void finalize() {
        System.out.println("finalize()");
    }
    
    public static void main(String[] args) {
		new TerminationCondition();		//创建后立即丢掉该对象的引用
         System.gc();		//执行垃圾收集
         sleep(1000);		//伪方法，休眠1s，给垃圾收集器足够的时间确保执行finalize方法
    }
    
}
/*
	finalize()
*/
~~~



### 清理顺序

先按初始化相反的顺序清理本类的成员（以防对象依赖于其他对象），再清理基类（清理时可能会调用基类的方法，此时要求基类组件处于存活状态）。下面给出一个例子：

~~~java
class Shape {
    void dispose() {}
}
class Line extends Shape {
    void clean() {super.dispose();}
}
class Triangle extends Shape {
    void clean() {super.dispose();}	//一定记得执行基类清理工作
}
class CADSystem extends Shape {
	Triangle t = new Triangle();
    Line l = new Line();
    void close() {
        l.clean();
        t.clean();
        super.dispose();
    }
}
~~~



如果其中某个成员对象被其他对象所共享，则问题会变得更加复杂，此时就不能简单地调用`dispose()`。在这里，可能需要使用**引用计数**来跟踪访问共享对象的对象数量。下面是相关的示例：

~~~java
class Shared {
    private static long refCount = 0;
    public void addRef() { refCount += 1; }
    
    public void dispose() {
        if (reCount == 0) {
            /*清理工作*/
        } else refCount -= 1;
    }
}

class Composing {
    private Shared shared;
    Composing(Shared shared) { this.shared = shared; shared.addRef(); }
    protected void dispose() { shared.dispose(); }
}

class ReferenceCounting {
    public static void main(String[] args) {
		Shared shared = new Shared();
         Composing[] composing = { 
             new Composing(shared);
             new Composing(shared);
             new Composing(shared);
         } 
        for (Composing c : composing) {
            c.dispose();
        }
    }
}
~~~



## 复用——继承与组合

在新类中创建现有类的对象。这称为**组合（composition）**。你复用的是代码的功能，而不是其形式。

它直接复制了现有类的形式，然后向其中添加代码，而没有修改现有类。这种技术叫作**继承（inheritance）**。它的大部分工作是由编译器完成的。

继承一般表示 `is-a`的关系，而组合表示`has-a`的关系。而且组合允许其状态发生改变，继承允许行为发生改变。

### 继承

其实当创建一个类时，总是在继承。除非明确指定了要继承某个类，否则将隐式继承Java的标准根类Object。

通过extends关键字来实现继承：

~~~java
class Sub extends Super {}
~~~



### 继承与初始化基类

当创建子类对象时，它里面包含了一个基类的**子对象（subobject）**。正确初始化基类的子对象至关重要，我们只有一种方法可以保证这一点：在子类构造器中调用基类构造器来执行初始化，它具有执行基类初始化所需的全部信息和权限。

编译器会为你隐式调用基类的无参构造器。如果基类没有无参构造器，或者如果你必须要调用具有参数的基类构造器，那么就要使用super关键字和相应的参数列表，来显式调用基类构造器：

~~~java
class A {
    A();
}
class B extends A {
    B(int i) { }		//隐式调用A()
}
class C extends B {
    C() { super(1); }
    //C() {}      			//基类没有无参构造器，必须显式调用基类的构造器
}
~~~

此外，调用基类的构造器必须是子类构造器的第一个操作！这与`this`构造器冲突，因此必须二选一！

### 委托

**委托（delegation）**。它介于继承和组合之间。虽然Java里没有提供直接支持，但是你可以在新类中创建现有类的对象（组合），同时又在新类里公开了成员对象的部分方法，以及适当添加一些操作（类似继承）。下面给出一个例子：

~~~java
public class SpaceShipControls {
    void up(int velocity) {};
    void down(int velocity) {};
    void left(int velocity) {};
    void right(int velocity) {};
}

public class SpaceShipDelegation {
    SpaceShipControls controls = new SpaceShipControls();
    public void up(int velocity) {
		controls.up(velocity);
    }
    public void up(int velocity) {
         log.info("up:" + velocity);		//扩展操作
		controls.up(velocity);
    }
    //暴露部分方法
}
~~~



### 继承中的名字隐藏

对于private、字段、static方法都会名字隐藏，此时要调用基类的版本（允许的话）要使用super关键字。

对于非private的final方法，它不会名字隐藏，而且明确要求子类不允许继承该方法：

~~~java
class C1{
    final void f() {};
    private void g() {};
}
class C2 extends C1 {
    //void f() {};		不允许覆写
    void g() {};
}
~~~



## 隐藏 & 封装

面向对象设计的一个主要考虑是**“将变化的事物与保持不变的事物分离”**。这也是设计模式中的核心思想！

### 访问权限修饰符

|             | 任何人 | 同一个包 | 子类 | 本类 |
| :---------: | :----: | :------: | :--: | :--: |
|  `public`   |   ✔️    |    ✔️     |  ✔️   |  ✔️   |
| `protected` |        |    ✔️     |  ✔️   |  ✔️   |
|  `default`  |        |          |  ✔️   |  ✔️   |
|  `private`  |        |          |      |  ✔️   |

注意：

- `default`访问权限并无对应的关键字

- 类本身的访问权限只能是`public`或`default`。实际上内部类可以是`private`或`protected`的，这会在内部类一节中介绍。

- **在继承期间不允许降低方法的可访问性**

- **`private`是隐式`final`的**



再此强调，private仅仅是语言层面上的隐藏，而不是二进制层面上的！这意味着private并不能够抵抗反编译。private仅仅是将底层实现与接口分离，当有能力改变底层实现时，你不仅有了改进设计的自由，还有了犯错误的自由。同时对客户程序员而言也是一种服务，因为这样的话他们就可以轻松地了解到什么对他们重要，什么又是他们可以忽略的。这简化了他们对类的理解。

### private构造器

如果无参构造器是唯一定义的构造器，并且它是private的，那么这就阻止了该类的继承！



private构造器可以实现单例模式或者限制对象的创建个数。

~~~java
class Soup1 {
    private Soup1() {}
    static int i = 0;
    public Soup1 getSoup() {
        return i > 10 ? null : new Soup1();
    }
}

class Soup2 {
    private Soup2() {}
    private static Soup2 soup2 = new Soup2();
    public Soup2 getSoup2() { return soup2; }
}
~~~



### 类的包访问权限与公共方法

当定义一个具有包访问权限的类时，你可以给它一个`public`或`protected`方法。这样做的原因是这个类不可被其他包访问，但是可能被其他包中的类间接继承，此时这些`public`、`protected`方法可以被子类覆写！

~~~java
package hiding.access;
class PublicMethod {
	public void method() {} ;
}

public A extends PublicMethod { }

//hiding
package hiding;
import hiding.access.A;
public B extends A {
    @Override public void method() {};
}
~~~



### 封装

访问控制常常被称为实现隐藏。将数据和方法包装在类中，并与实现隐藏相结合，称为**封装（encapsulation）**。



## 多态

封装将实现与接口解耦。而多态（也称动态绑定、后期绑定或运行时绑定）实现类型之间的解耦，即让一段代码同等地适用于所有这些不同的类型，同时在行为上又有所不同。这改善了代码的组织结构和可读性，并且还能创建**可扩展**的程序。多态是“将变化的事物与不变的事物分离”的一项重要技术。因此大部分面向对象的设计模式依赖于多态机制！



### 转型

向上转型总是安全的，因为你是从更具体的类型转为更通用的类型。在向上转型期间，子类接口“缩小”至基类的接口。

因为向上转型会丢失特定类型的信息，所以我们自然就可以通过向下转型（downcast）来重新获取类型信息。而向下转型不总是安全的，因为它可能转型为错误的类型，而向该对象发送它无法接受的信息，从而在运行时抛出`ClassCastException`

### 绑定（覆写）

将一个方法调用和一个方法体关联起来的动作称为**绑定**。在程序运行之前执行绑定（如果存在编译器和链接器的话，由它们来实现），称为前期绑定。

**Java中的所有方法绑定都是后期绑定**，除非方法是static或final的（private方法隐式为final）。例子：

~~~java
class A {
    void f() { g(); }	//g()不一定是A中的f()方法，还可能是B中的
    void g() {}
}
class B extends A {
    @Override void g() {}
}
~~~

**在JAVA中，一定要注意在本类中调用的方法可能动态绑定到了子类的方法上 **，对于private、static、final方法则无需担心这一点。



请注意只有普通方法是多态的。而final方法、静态方法以及字段会在编译时解析的。一个副作用就是可以名字隐藏，想要访问基类的字段或静态方法必须使用super关键字。例子：

~~~java
class Super {
    public int field = 0;
}
class Sub extends Super {
    public int field = 1;
    public int getField() { return field; }			    //0
    public int getSuperField() { return super.field; }	//1
}
~~~

我们一般不会为基类字段与子类字段指定相同的名称，因为这会造成混淆。

### 协变返回类型

**协变返回类型（covariant return type）**，这表示子类中重写方法的返回值可以是基类方法返回值的子类型。这对于基本类型并不成立。

 在基类构造器中，整个子类对象还只是部分构造，只能知道基类对象是已完全初始化的。由于动态绑定，基类构造器可能调用子类的方法，此时可能有访问尚未初始化的子类字段，这会带来难以发现的错误！下面给出一个例子：

~~~java
class Glyph {
    void draw() { System.out.println("Glyph.draw()"); }
    Glyph() {
        System.out.println("Glyph() before draw()");
        draw();
        System.out.println("Glyph() after draw()");
    }
}

class RoundGlyph extends Glyph {
    private int radius = 1;
    RoundGlyph(int r) {
        radius = r;
        System.out.println(
            "RoundGlyph.RoundGlyph(), radius = " + radius);
    }
    @Override void draw() {
        System.out.println(
            "RoundGlyph.draw(), radius = " + radius);
    }
}

public class PolyConstructors {
    public static void main(String[] args) {
        new RoundGlyph(5);
    }
}
/* Output:
Glyph() before draw()
RoundGlyph.draw(), radius = 0		//期望是radius = 1
Glyph() after draw()
RoundGlyph.RoundGlyph(), radius = 5
*/
~~~



因此，编写构造器时有一个很好的准则：“用尽可能少的操作使对象进入正常状态，如果可以避免的话，请不要调用此类中的任何其他方法。”只有基类中的final、static方法可以在构造器中安全调用（这也适用于private方法，它们默认就是final的）。这些方法不能被重写，因此不会产生这种令人惊讶的问题。
