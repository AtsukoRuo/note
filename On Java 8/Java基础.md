# Java基础

一个重要事实：在Java中**一切皆为对象**









除基本类型外，Java中的对象都是通过**引用（reference）**来操作，例如

~~~java
Person person = new Person("AtsukoRuo", 18);
~~~

这里的`person`标识符实际上就是一个引用，它指向位于堆中的实际对象。



必须对栈中的数据进行初始化工作，否则编译器会报错。而数组中的元素以及类中的字段





通过new关键字创建对象



字符串的初始化

- 字面值初始化

	~~~java
	String str = "Java Note";
	~~~

	> 注：在java中并没有c++的运算符重载。而这里String对象可以使用"="运算符是因为，编译器会为其生成特殊的代码。

- new操作符

	~~~java
	String str = new String("Java Note")
	~~~

	

Java中数据存储方式

- **寄存器（register）**：数据直接保存在中央处理器（CPU）中，因此这是访问速度最快的数据存储方式。然而寄存器的数量是有限的，所以只能按需分配。同时你不能直接控制寄存器的分配，甚至你在程序中都找不到寄存器存在过的证据（ C 和 C++ 是例外，它们允许你向编译器申请分配寄存器）。

- **栈（stack）**：数据存储在**随机存取存储器（random-access memory, RAM）**里，处理器可以通过栈指针（stack pointer）直接操作该数据。具体来说，栈指针向下移动将申请一块新的内存，向上移动则会释放这块内存。只不过 Java 系统在创建应用程序时就必须明确栈上所有对象的生命周期。这种限制约束了程序的灵活性。引用以及基本类型会保存在栈上

- **堆（heap）**：用于存放所有 Java 对象。与栈不同的是，编译器并不关心位于堆上的对象需要存在多久。因此，堆的使用是非常灵活的。然而这种灵活性是有代价的：分配和清理堆存储要比栈存储花费更多的时间。但你并不需要太过关注此类问题。

- **常量存储（constant storage）**：常量通常会直接保存在程序代码中，因为它们的值不会改变，所以这样做是安全的。

- **非RAM存储（non-RAM storage）**：这些数据的生命周期独立于应用程序的。其中最典型的例子：

	- **序列化对象（serialized object）**：转换为字节流并可以发送至其他机器的对象。网络
	- **持久化对象（persistent object）**：保存在磁盘上的对象，而这些对象即便在程序结束运行之后也依然能够保持其状态。数据库

	

一个常量存储的例子是字符串资源池。所有的字符串常量都会被自动放置到这个特殊的存储空间中。

~~~java
String s1 = "Hello";
String s2 = "Hello";
String s3 = "Hel" + "lo";
String s4 = "Hel" + new String("lo");
String s5 = new String("Hello");
String s6 = s5.intern();
String s7 = "H";
String s8 = "ello";
String s9 = s7 + s8;

// == ：比较两个对象是否为同一对象
System.out.println(s1 == s2);  // true
System.out.println(s1 == s3);  // true
System.out.println(s1 == s4);  // false
System.out.println(s1 == s9);  // false
System.out.println(s4 == s5);  // false
System.out.println(s1 == s6);  // true
~~~





对于**基本类型（primitive type）**，Java会在栈上直接创建一个**自动变量（automatic variable）**，而不是引用！

- 所有数值类型都是有符号的

- 并未对boolean类型的空间大小做出规定，而且其对象只能赋值为true或false



Java 还为基本类型提供了对应的**“包装类”（wrapper class）**，通过包装类可以将位于栈上的自动变量转换为位于堆上的对象。

自动装箱：

~~~java
~~~

自动拆箱：

