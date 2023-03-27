# Java 文件与字符串

[TOC]

## 字符串

### 接口

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

	

### 字符串常量池

所有的字符串字面值都会被自动放置到这个特殊的存储空间中。

~~~java
String s1 = "Hello";						//在常量池中创建“Hello”
String s2 = "Hello";						
String s3 = "Hel" + "lo";				    //在常量池中创建“Hel”与“lo”，而“Hello”之前已被创建，故无需再次创建
String s4 = "Hel" + new String("lo");		//这里常量池中已创建“Hel”与“lo”，这里只需在堆中创建一个字符串对象即可
String s5 = new String("Hello");
String s6 = s5.intern();					//将s5对象中的字符串内容放在常量池中。
String s7 = "H";
String s8 = "ello";
String s9 = s7 + s8;					   //只会在堆中创建s9对象

// == ：比较两个对象是否为同一对象
System.out.println(s1 == s2);  // true
System.out.println(s1 == s3);  // true
System.out.println(s1 == s4);  // false
System.out.println(s1 == s9);  // false
System.out.println(s4 == s5);  // false
System.out.println(s1 == s6);  // true
//注：== 判断引用在栈中的值。
~~~

