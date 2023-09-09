# Java 内部类 

[TOC]

## 内部类的作用

使用内部类一般出于以下原因：

- 隐藏实现
- 实现多继承
- 简化代码编写（用Lambda表达式更好）





### 隐藏实现

~~~java
public interface Contents {
    int value();
}
public interface Destination {
    String readLabel();
}
public class Parcel {
    //内部类实现接口，权限为private，外部访问不到
    private class PContents implements Contents {
        private int i = 11;
        @Override
        public int value() {
            return i;
        }
    }
	
    //内部类实现接口，权限为protected，外部访问不到
    protected final class PDestination implements Destination {
        private String label;
        private PDestination(String whereTo) {
            label = whereTo;
        }
        @Override
        public String readLabel() {
            return label;
        }
    }
	
    //通过方法返回接口的引用，而且内部类通过向上转型到接口类型。这样隐藏内部实现。更加安全
    public Destination destination(String s) {
        return new PDestination(s);
    }
    public Contents contents() {
        return new PContents();
    }

    public static void main(String[] args) {
        Parcel3 parcel3 = new Parcel3();
        Contents contents = parcel3.contents();
        //Destination destination = parcel3.new PDestination("1"); 只有在本类中可用
        Destination destination1 = parcel3.destination("2");
    }

~~~



内部类为类的设计者提供了一种方式，可以完全阻止任何与类型相关的编码依赖（通过接口与向上转型实现），并且可以完全隐藏实现细节。此外，从客户程序员的角度来看，因为无法访问接口之外的任何方法，所以接口的扩展对他们而言并没有什么用处。

### 多继承

通常情况下，内部类可以继承自某个类或实现某个接口。内部类提供了进入其外部类的某种窗口，这样内部类中的代码可以操作外部类对象。

每个内部类都可以独立地继承自一个实现。因此，外部类是否已经继承了某个实现，对内部类并没有限制。

如果没有内部类提供的这种事实上能继承多个具体类或抽象类的能力，有些设计或编程问题会非常棘手。所以从某种角度上讲，内部类完善了多重继承问题的解决方案。接口解决了一部分问题，但内部类实际上支持了“多重继承”（包括状态上的）。例子：

~~~java
class D {}
abstract class E {}
class Z extends D {
    class F extends E {}
    E makeE() { return new F(); }		
}
~~~



## 成员内部类

创建一个内部类，只需在类中再定义即可。此时内部类的访问权限是任意的，而不仅仅是`public`或`default`。而且**内部类并不被子类继承**。

~~~java
package innerclasses.nesting;

public class A {
    private class B {
        protected class C {
            class D {
          	}
        }
    }
}
~~~



在外部类通过`OuterClassName.InnerClassName1.InnerClassName2...`来访问内部类。其中在本类中`OuterClassName`可以省略。例如：

~~~java
A.B 	b = new B();
B 		b = new B();
A.B.C 	c = b.new C();
B.C 	c = b.new C();
C 		c = b.new C();		//错误的
~~~



如何实例化出一个内部类对象？

- 直接调用构造函数。有限制，下面会说明。
- 通过语法`对象.new 构造器()`

当创建一个内部类时，这个内部类的对象中**必须**有一个隐含引用，指向用于创建该对象的外围对象。编译器会为你处理所有这些细节，如果编译器无法访问这个引用，它就会报错。下面给出一个例子：

~~~java
public class A {
    public class B {
        public class C {}
    }
    
    B f() { return new B(); }			//正确的，通过隐含this指针可以获取外围对象
    static B g() { return new B(); }	//错误的，没有办法获取到this指针
    B.C g() { return new B.C(); }		//错误的，此时没有类型为B的外部对象
    public static void main(String[] args) {
        A  a   = new A();
        B  b0  = new B();			//错误的，在静态方法里没有this，无法获取到外围对象
        B  b   = a.new B();
        B  b1  = a.f();				//通过构造器返回一个引用，很常用
    }
}
~~~



共享数据问题：Java中内部类机制并不复制外围对象的当前状态，而是与其他对象一直共享。

~~~java
public class Parcel12 {
    int j = 12;
    class A {
        void f() {
            System.out.println(j++);
        }
    }
    public static void main(String[] args) {
        Parcel12 p = new Parcel12();
        A a = p.new A();
        A a1 = p.new A();
        a.f();
        a1.f();						   //与其他内部类共享
        System.out.println(p.j++);		//与外围对象共享
        a1.f();
    }
}
/*
12
13
14
15
*/
~~~



**内部类与外部类都有对象所有字段或方法的访问权限，即使是private的**

~~~java
public class A {
    private class B {
        private class C {
            private void f() { g(); }			//访问外围对象的private方法
        }
    }
    private void g() { System.out.println("A::Private"); }

    public static void main(String[] args) {
        A a  = new A();
        B b = a.new B();
        B.C c = b.new C();		//private类也是可以使用的
        c.f();					//访问内部类的private方法
        //f()  不能直接访问f，也没办法访问
    }
}
/*
A::Private
*/
~~~



通过`.this`关键字**解决命名冲突**以及**引用外部对象的方法与字段**：

~~~java
class A {
    void f() {System.out.println("A::f()"); }
    private class B {
        private class C {
            private void f() { A.this.f(); }		//通过this访问外部类的方法或字段
        }
    }

    public static void main(String[] args) {
        A a  = new A();
        B b = a.new B();
        B.C c = b.new C();
        c.f();					
    }
}

~~~



## 局部内部类

内部类与局部内部类区别

- **作用域的限制**：局部内部类超出作用域外就不可访问。

- **局部内部类可以在静态方法中创建**，因为局部内部类是定义在方法中的，它的作用域仅限于方法内部，不依赖于外部类的实例。因此，在静态方法中可以直接创建局部内部类的对象，而不需要创建外部类的实例。**此时只能访问外部类的静态方法以及静态字段**。

	~~~java
	class A {
	    class D {}
	    
		static int i = 0;
	    int j = 11;
	    static void f() {
	        class B {}
	        B b = new B();
	        D d = new D();		//错误
	    }
	}
	~~~

	

局部内部类可以在任意作用域内创建：

~~~java
public class Parcel6 {
    boolean aBoolean;
    private void internalTracking(final boolean b) {
        if(b) {
            //一个局部内部类
            class TrackingSlip {
                private String id;
                TrackingSlip(String s) {
                    id = s;
                }
                String getSlip() {
                    boolean bb =  b;
                    Parcel6.this.aBoolean = true;
                    aBoolean = true;
                    return id;
                }
            }
            TrackingSlip ts = new TrackingSlip("slip");
            String s = ts.getSlip();
        }
        // Can't use it here! Out of scope:
        //- TrackingSlip ts = new TrackingSlip("x");
    }
    public void track() { internalTracking(true); }
    public static void main(String[] args) {
        Parcel6 p = new Parcel6();
        p.track();
    }
}
~~~



虽然外部类无法访问局部内部类（超出了作用域范围），但是局部内部类可以通过`.this`语法（非静态方法中）访问到外部类：

~~~java
class B {}
public class A {
    int j = 10;
    B g() {
		return new B() {
            @Override public void h() { 
                A.this.j = 10;
            }
        }
    }
    
    static int i = 10;
    static void f() {
        class B {
			void g() {
                i = 10;		//OK
                j = 11;		//Error，在静态方法中不可访问非静态字段
            }
        }
    }
}
~~~



### final

我们先看一段代码：

~~~java
int sum = 0;
list.forEach(e -> { sum += e.num; }); // ERROR
~~~

这段代码会引发多线程中的竞争条件问题。

为了解决这个问题，Java规定如果局部内部类要捕获局部变量，那么该局部变量必须是final或者 effectively final（非final，但不修改它的值），否则将会报错。

但是可以作为一个对象（数组）的成员来避免这一点。



## 匿名内部类

在通过new创建对象时，可以创建一个匿名类，下面是一个例子：

~~~java
abstract class B {
    public B(int i) { }
    abstract public void h();
}
public class A {
    static void f(B b) {}

    B g() {
        return new B(0) {
            @Override
            public void h() {

            }
        };
    };

    public static void main(String[] args) {
        f(new B(0) {
            @Override
            public void h(){}
        });
    }
}
~~~



匿名内部类的限制比较多，例如

- 因为匿名类没有名字，所以不可能有命名的构造器。实例初始化部分就是匿名内部类的构造器。不过它也有局限性——我们无法重载实例初始化部分，所以只能有一个这样的构造器。
- 与普通的继承相比，匿名内部类有些局限性，因为它们要么是扩展一个类，要么是实现一个接口，但是两者不可兼得。而且就算要实现接口，也只能实现一个。

匿名类相对局部内部类唯一的优势：在仅需使用一次类的地方可简化代码编写。Lambda表达式在这方面会做得更好。



### 双括号初始化

双括号初始化就是实例初始化 + 匿名类，下面举一个例子：

~~~java
Map<Integer, Integer> map = HashMap<>() {{
    put(1, 2);
    put(3, 5);
}}
~~~

实际上就是：

~~~java
Map<Integer, Integer> map = HashMap<>() {
    //实例初始化
    {
        put(1, 2);
        put(3, 5);
	}
    
    //你甚至可以覆写一些方法
    @override
    public boolean put(E e, V v) {
        
    } 
}//匿名类
~~~



## 嵌套类（静态内部类）

如果**不需要内部类对象和外部类对象之间的连接**，可以将内部类设置为static的。我们通常称之为**嵌套类**。此时只能访问外部类的静态字段或静态方法。

~~~java
class A {
    static class B {
        static class C {
            
        }
    }
    public static void main(String[] args) {
        B.C c = new B.C();	//OK 如果C不是嵌套类就不可以
    }
}
~~~



此时可以在静态语义下，直接创建该嵌套类对象：

~~~java
class A {
    private int i = 10;
    private static int j = 10;
	static class B {
        void g() {
            i = 10;			//Error
            A a = new A();
            a.i = 10;		//OK
            j = 10; 		//OK
            
        }
    }
    class C {}
    static void f() {
        C c = new C(); 	//错误
        B b = new B();	//正确
    }
}
~~~



曾建议在每个类中都写一个main()，用来测试这个类。这样做有个潜在的缺点，测试设施会暴露在交付的产品中。如果这是个问题，可以使用一个**静态嵌套类**来存放测试代码：

~~~java
public class Bed {
    public void f() {
        System.out.println("f()");
    }
    public static class Tester {
        public static void main(String[] args) {
            Bed b = new Bed;
            b.f();
        }
    }
}
~~~

这会生成一个叫`Bed$Tester`的独立的类。可以使用这个类来做测试，但是不必将其包含在交付的产品中。可以在打包之前删除`Bed$Tester.class`。



## 继承内部类

因为内部类的构造器必须附着到一个指向其包围类的对象的引用上，所以当你要继承内部类时，事情就稍微有点复杂了。这是因为在子类中并没有默认的外围对象供其附着。你必须使用一种特殊的语法来明确地指出这种关联：

~~~java
package innerclasses;
class WithInner {
    WithInner() {}
    WithInner(int i) { System.out.println("WithInner");}
    class Inner {
        Inner (int i) {System.out.println("Inner");}		//输出这一句
    }
}

public class InheritInner extends WithInner.Inner {
    InheritInner(WithInner wi) {
        wi.super(31);				//不是调用外围类的构造器，而是调用附着在外围对象上内部类的构造器
       	//这里wi.super()正是指代Inner()
    }

    public static void main(String[] args) {
        WithInner wi = new WithInner();
        InheritInner i = new InheritInner(wi);
    }
}
/**
Inner
*/
~~~



先在某个外围类中创建一个内部类，然后新创建一个类，使其继承该外围类，并在其中重新定义之前的内部类，这会发生什么呢？

~~~java
class Egg {
    private Yolk y;
    protected class Yolk {
        public Yolk() {
            System.out.println("Egg.Yolk()");
        }
    }
    Egg() {
        System.out.println("New Egg()");
        y = new Yolk();
    }
}

public class BigEgg extends Egg {
    public class Yolk {
        public Yolk() {
            System.out.println("BigEgg.Yolk()");
        }
    }
    public static void main(String[] args) {
        new BigEgg();
    }
}
/* Output:
New Egg()
Egg.Yolk()
*/
~~~



当继承外围类时，内部类并没有额外的特殊之处。这两个内部类是完全独立的实体，分别在自己的命名空间中。然而，显式地继承某个内部类也是可以的：

~~~java
class Egg2 {

  protected class Yolk {
    public Yolk() {
      System.out.println("Egg2.Yolk()");
    }
    public void f() {
      System.out.println("Egg2.Yolk.f()");
    }
  }
  private Yolk y = new Yolk();
  Egg2() { System.out.println("New Egg2()"); }
  public void insertYolk(Yolk yy) { y = yy; }
  public void g() { y.f(); }
}

public class BigEgg2 extends Egg2 {
  public class Yolk extends Egg2.Yolk {
    public Yolk() {
      System.out.println("BigEgg2.Yolk()");
    }
    @Override public void f() {
      System.out.println("BigEgg2.Yolk.f()");
    }
  }
  public BigEgg2() { insertYolk(new Yolk()); }
  public static void main(String[] args) {
    Egg2 e2 = new BigEgg2();
    e2.g();
  }
}
/* Output:
Egg2.Yolk()
New Egg2()
Egg2.Yolk()
BigEgg2.Yolk()
BigEgg2.Yolk.f()
*/
~~~

