# Java 接口&抽象类

[TOC]

**接口和抽象类提供了一种更加结构化的方式来分离接口与实现。**其中抽象类反映子类共有的特性。而接口实现了类型的完全解耦！而多态只实现了在继承层次上的类型解耦

## 抽象类

通过`abstract`关键字创建抽象类：

~~~java
abstract class AbstractClass {}
~~~

**抽象类包含任意多个（包括0个）抽象方法，且抽象方法只能声明在抽象类中**。抽象方法（abstract method）是一个不完整方法，它只有声明没有方法体，这类似于C++中的纯虚函数（pure virtual function）。

因为抽象方法是一个不完整的方法，因此**你无法安全地创建一个抽象类的实例化对象**，但是可以接受向上转型对象：

~~~java
abstract class A {}
class B extends A {}
A a = new A();		//错误
A a = new B();		//正确
~~~



**子类必须覆写基类中所有抽象方法，否则它本身必须是一个抽象类。**

抽象类几乎对访问权限没有任何限制。但禁止使用`private abstract`，这样做也是有道理的：任何子类都不可能给`private`的方法提供一个定义。



**虽然抽象方法只提供了声明，但是它还实现了对基类方法的覆写**：

~~~java
class T {
	void f() {}
    final void g() {}
}
abstract class R extends T {
	@Override abstract void f();
    // void g(); 不允许覆写
}
~~~



## 接口

通过`interface`来定义接口：
~~~java
interface I {
    void f();
}
~~~

接口中的所有方法都是隐式的`public`（除非显式声明为private）。在实现接口时，记得接口方法的访问权限必须是public，因为Java不允许在继承时降低访问权限。

**而且接口中的方法（除static、default方法）都是隐式抽象方法。**



要创建一个符合特定接口（或一组接口）的类，请使用`implements`关键字：

~~~java
class A implements {
    @Override void f() {};
}
~~~

类必须覆写接口中所有方法（除static、default方法）。



### 默认方法与静态方法

通过关键字`default`创建一个默认方法

~~~java
interface I {
    default void f() {}
}
class A implements I {
    void g() { f(); }
    void h() { I.super.f(); }		//在多继承中解决默认方法签名冲突问题。
}
~~~

实现了该接口的类可以在不定义方法的情况下直接使用方法体，也可以覆写该方法。添加默认方法的一个令人信服的原因是，它允许向现有接口中添加方法，而不会破坏已经在使用该接口的所有代码。默认方法有时也称为防御方法（defender method）或虚拟扩展方法（virtual extension method）。



通过关键字`static`创建一个静态方法：

~~~java
interface I {
    static void f() {};
}
~~~

静态含义就是只跟接口有关的逻辑操作，与接口实例化对象无关。

### private

在JDK 9中，接口里的default和static方法都可以是private的。

~~~java
interface I {
	private static void f();
    private void f();			//默认是default的
    private default void f();	//错误
}
~~~



### 通过继承扩展接口

通过`extends`关键字，可继承多个接口：

~~~java
interface I1 extends I2, I3 {
    
}
~~~

此时会名字隐藏

在不同接口中使用相同的方法名称通常会导致代码可读性较差。应该努力避免这种情况。下面给出命名冲突的例子

~~~java
// interfaces/InterfaceCollision.java
// (c)2021 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

package interfaces;
interface I1 { void f(); }
interface I2 { int f(int i); }
interface I3 { int f(); }
class C { public int f() { return 1; } }

class C2 implements I1, I2 {
    @Override
    public void f() {}
    @Override
    public int f(int i) { return 1; } // Overloaded
}

class C3 extends C implements I2 {
    @Override
    public int f(int i) { return 1; } // Overloaded
}

class C4 extends C implements I3 {
    // Identical, no problem:


    public static void main(String[] args) {
        System.out.println("");
    }
}

// Methods differ only by return type:
//- class C5 extends C implements I1 {}
//- interface I4 extends I1, I3 {}

~~~



### 多重继承

Java严格来说是一种单继承语言：只能继承一个类（或抽象类）。你可以实现任意数量的接口。自从有了**接口的默认方法**，我们可以**继承来自多个基类型的行为**。因为接口仍然不允许包含字段（接口里只有静态字段，并不适用于我们这里讨论的场景），所以字段仍然只能来自单个基类或抽象类。也就是说，**你不能拥有状态的多重继承**。

**一个类只允许继承一个基类，但可以实现任意多个接口**



当实现多个接口时，多个默认方法的签名可能会冲突，此时编译器会报错：

~~~java
interface I1 {
	default void f() {}
}
interface I2 {
    default int f() { return 0; }
}
class A implements I1, I2 {
    //错误 I1中的f与I2中的签名相同
}
~~~

解决此类冲突的方法是覆写该方法：

~~~java
class A implements I1, I2 {
    @Override public int f() { return I2.super.f(); }
}
~~~

调用特定接口中的默认方法：`接口名.super.方法名`。

注意：super只能引用直接父类，而不能引用祖先！给出一个例子：

~~~java
interface I1 {
    default void f() {}
}
interface I2 extends I1 { }
class A implements I1, I2 {
    public void g() { 
    	//I1.super.f() 错误
        I2.super.f()
    }
}
~~~





接口中的方法（除static、default方法）都是隐式抽象方法，所以它还可以覆写父接口的方法：

~~~java
interface I1{
    void f();
    default void g() {};
}
interface I2 extends I1 {
    @Override void f();
    @Override void g();
}
~~~





子类从基类继承过来的方法，可以覆写接口中的方法：

~~~java
class A {
	void f() {}
}
interface I {
    void f();
}
class B extends A implements I{
    //OK
}
~~~



多个接口中的抽象方法签名相同，只需覆写其中一个即可：

~~~java
interface I1 {
    void g();
}
interface I2{
    void g();
}
abstract class B {
    abstract void g();
}

class A extends B implements I1, I2{
    @Override public void g() { }
}
~~~





### 字段

接口中的所有字段默认是`public static final`的。

由于接口中没有构造器，所以不允许空白`final`。

字段支持方法初始化，但不支持静态初始化：

~~~java
interface I1 {
    int i = f();
    static int f() { return 1; }
}
~~~



### 嵌套接口

Java为了语法一致性允许接口中嵌套接口，这里用例子呈现几个有趣的特性：

~~~java
// interfaces/nesting/NestingInterfaces.java
// (c)2021 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
package interfaces.nesting;

class A {
    //内部接口，具有default权限
    interface B {
        void f();
    }
    
    //内部类
    public class BImp implements B {
        @Override public void f() {}
    }
    
    private class BImp2 implements B {
        @Override public void f() {}
    }
    
    //内部接口 具有public权限
    public interface C {
        void f();
    }
    
    class CImp implements C {
        @Override public void f() {}
    }
    private class CImp2 implements C {
        @Override public void f() {}
    }
    
    //内部接口，具有private权限
    private interface D {
        void f();
    }
    private class DImp implements D {
        @Override public void f() {}
    }
    public class DImp2 implements D {
        @Override public void f() {}
    }
    
    //通过public方法操纵D接口，任何人都无法访问到此接口
    public D getD() { return new DImp2(); }
    private D dRef;
    public void receiveD(D d) {
        dRef = d;
        dRef.f();
    }
}

interface E {
    interface G {
        void f();
    }
    
    // Redundant "public":
    public interface H {
        void f();
    }
    void g();
    
    // Cannot be private within an interface:
    //- private interface I {}
}

public class NestingInterfaces {
    public class BImp implements A.B {
        @Override public void f() {}
    }
    class CImp implements A.C {
        @Override public void f() {}
    }
    
    // Cannot implement a private interface except
    // within that interface's defining class:
    //- class DImp implements A.D {
    //-  public void f() {}
    //- }
    
    class EImp implements E {
        @Override public void g() {}
        //不必嵌套实现子接口
    }
    class EGImp implements E.G {
        @Override public void f() {}
    }
    
    class EImp2 implements E {
        @Override public void g() {}
        class EG implements E.G {
            @Override public void f() {}
        }
    }
    
    public static void main(String[] args) {
        A a = new A();
        // Can't access A.D:
        //- A.D ad = a.getD();
        // Doesn't return anything but A.D:
        //- A.DImp2 di2 = a.getD();
        // Cannot access a member of the interface:
        //- a.getD().f();
        // Only another A can do anything with getD():
        A a2 = new A();
        a2.receiveD(a.getD());
    }
}
~~~





## 密封类

密封类限制了能派生出哪些类。这让我们可以对固定的一组类型进行建模。（枚举限制了一组固定的值）

通过sealed与permits关键字使用密封类：
~~~java
import D1; import D2;
sealed class Base permits D1, D2 {}
~~~

注意：

1. D1、D2必须继承Base，否则编译器报错：一条无效permits子句。
2. 如果省略permits语句，那么只有同一个文件中的类可以（不是必须）继承该类
3. **The sealed class and its permitted subclasses must belong to the same module, and, if declared in an unnamed module, the same package.**



sealed类的子类只能通过下面的某个修饰符来定义：

- `final`：不允许有进一步的子类。
- `sealed`：允许有一组密封子类。
- `non-sealed`：一个新关键字，允许未知的子类来继承它。放开限制

这样使得sealed的子类保持了对层次结构的严格控制：



还可以密封接口和抽象类：

~~~Java
sealed interface Ifc permits Imp1, imp2 {}
~~~





