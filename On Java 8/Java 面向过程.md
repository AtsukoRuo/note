# Java 面向过程

[TOC]

## 方法

方法最基础的几个部分包括：**方法名**、**参数列表**、**返回值**，以及**方法体（method body）**：

~~~java
ReturnType methodName(/*参数列表*/)
{
    //方法体
}
~~~

方法名和参数列表共同构成了方法的**签名（signature）**，方法签名即该方法的唯一标识符。



在Java 中的方法只能作为类的一部分而存在。例如：

~~~java
ObjectReference.methodName(arg1, arg2, arg3);
~~~

这种调用方法的行为也被描述为“向一个对象发送一条消息”



**比较遗憾的是，Java并不支持默认参数**

### 方法重载

通常来说，同一个词可以表达几种不同的含义，这就是**重载（overloaded）**。重载很有用，尤其是在涉及细微差别时，它允许不同参数类型的方法有着相同的名字。



通过**参数类型以及参数顺序**来区分不同的重载版本：

~~~java
void f(int val, String str);
void f(String str, int val);
~~~



类型提升与方法重载的匹配规则：

- 精准匹配优于类型提升
- 没有精准匹配，但有多个通过类型提升而匹配的重载版本，抛出编译期异常

~~~java
void g(int i, double d) {}
void g(float l, float f) {}
//g((byte)1, 3.14f);
g(1, 3.14);
~~~



为什么不通过返回类型来区分方法重载呢？下面给出一个例子：

~~~java
void f() {}
int f() { return 0;}
int val = f();  //int f() 
f()			   //哪一个？每个函数都可能有副作用
~~~



记录一个可能是语言设计上的错误

~~~java
void f(byte b) {}
byte b = 1;		 //通过
f(1);			//编译报错
~~~



### 可变参数列表

实现方法一：Object[]

~~~java
void f(int i, Object[] args) {}
 f(1, new Object[] {"one", 3.14, 2});
~~~

实现方法二：语法糖 ...

~~~java
void f(int i, Object... args) {}		//编译器会为你自动生成一个数组
f(1, 2, 3, 4);
f(1, new int[] {1, 2, 3});
~~~



```java
static void f(float i, Character... args) { }
static void f(Character... args) { }
f(1, '2', '3');			//OK
f('a', '2', '3');		//匹配多个重载，抛出异常
```

## 控制流

Java控制流相关的关键字包括`if-else`、`while`、`do-while`、`for`、`return`、`break`以及选择语句`switch`



statement包括空语句`;`、单条语句、语句块`{}`。

### 条件分支

#### if-else

~~~java
if (boolean-expression)
    statement
else if (boolean-expression)
    statement
else
    statement
~~~



#### switch

~~~java
switch (integral-selector) {
    case integral-value1:
        statement;				//多条语句可以不使用语句块
        break;
    case integral-value2:
        statement;
        break;
    //...
    default:
        statement;
}
~~~

整数选择器（integral-selector）是一个能生成整数值的表达式，switch将这个表达式的结果与每个整数值（必须是编译期常量）相比较。如果发现相等，就执行对应的语句（单条语句或多条语句，不要求使用花括号包围）。若没有发现相等，就执行默认（default）语句。

break是可选的。如果省略，后面的case语句也会被执行，直到遇到一个break。



switch语句支持多重匹配，给出一个例子：

~~~java
 switch(c) {
     case 'a': case 'e': case 'i': case 'o': case 'u': 
         System.out.println("vowel"); break;
     case 'y': case 'w': 
         System.out.println("Sometimes vowel"); break;
     default:  
         System.out.println("consonant");
 }
~~~



Java 7的switch选择器不但可以使用整数值，还添加了使用字符串的能力。下面给出一个例子：

~~~java
 switch(color) {
     case "red": System.out.println("RED"); break;
     case "green": System.out.println("GREEN"); break;
     default: System.out.println("Unknown"); break;
 }
~~~



enum可以轻松地和switch结合使用。

### 迭代语句

#### while

~~~java
while (boolean-expression)
    statement
~~~

在每次迭代之前检查expression



#### do-while

~~~java
do
    statement
while (boolean-expression)
~~~

在每次迭代之后检查expression



#### for

~~~java
for (initialization; boolean-expression; step)
    statement
~~~

在执行for语句前，执行initialization。每次迭代前检查expression，每次迭代后执行step

编译器同等对待无限循环while(true)和for(;;)，所以具体选用哪个取决于你的编程习惯。



可以在for语句中的`initialization`、`step`部分使用逗号操作符

~~~java
for (int i = 10, j  = 1; i > j; i--, f(i, j), j++) {
    
}

void f(int i, int j) {
    System.out.println("i : " + i + " ,j : " + j);
}
/*
i : 9 j : 1
i : 8 j : 2
i : 7 j : 3
i : 6 j : 4
i : 5 j : 5
*/
~~~



#### for-in

Java 5引入了一种更加简洁的for语法，可以用于数组和容器以及`Iteratable`对象。大部分文档中直接将其称为foreach语法，这里我们称之为for-in。给出一个例子：

~~~java
for (char c : "Hello World".toCharArray()) {
    System.out.print(c + " ");
}
~~~





### 无条件分支

这些关键字包括`return`、`break`、`continue`。



#### return

`return`关键字有两个用途：

- 可以指定一个方法的返回值（如果不存在就返回void）
- 导致当前的方法退出，并且返回这个值



如果在一个返回了void的方法中没有return语句，那么该方法的结尾处会有一个隐含的return，所以方法里并不一定会有一个return语句。但是如果你的方法声明了它将返回一个非void的值，那就必须确保每一条代码路径都会返回一个值。



#### break continue

break会直接退出循环，不再执行循环里的剩余语句。continue则会停止执行当前的迭代，然后准备下一次迭代。



#### goto

尽管goto是Java中的一个保留字，但Java中并没有使用它——Java没有goto语句。然而Java提供带标签的break或continue来提供类似goto语句的功能，但具有一定的使用限制：在Java中，**放置标签的唯一地方是正好在迭代语句之前**。“正好”的意思就是，不要在标签和迭代之间插入任何语句！一定要记住，在Java里使用标签的唯一理由就是你用到了嵌套循环，而且你需要使用break或continue来跳出多层的嵌套。



标签是以冒号结尾的标识符：

~~~java
label1:
~~~

带标签的continue会跳到对应标签的位置，并重新进入这个标签后面的循环。

带标签的break会跳出标签所指的循环。

~~~java
lable1:
outer-iteration {
    inner-iteration {
        break;				//这里的break中断内部迭代，回到外部迭代。
        continue;			//这里的continue中断当前执行，回到内部迭代的开始位置。
        continue lable1:	//这里的continue label1会同时中断内部迭代以及外部迭代，直接跳到label1处，然后它实际上会重新进入外部迭代开始继续执行。
        break label1:		//这里的break label1也会中断所有迭代，跳回到label1处，不过它并不会重新进入外部迭代。它实际是完全跳出了两个迭代。

    }
}
~~~



下面给出一个例子：

~~~java
int i = 0;
outer: // Can't have statements here
for(; true ;) { // infinite loop
    inner: // Can't have statements here
    for(; i < 10; i++) {
        System.out.println("i = " + i);
        if(i == 2) {
            System.out.println("continue");
            continue;
        }
        if(i == 3) {
            System.out.println("break");
            i++; // Otherwise i never
            // gets incremented.
            break;
        }
        if(i == 7) {
            System.out.println("continue outer");
            i++; // Otherwise i never
            // gets incremented.
            continue outer;
        }
        if(i == 8) {
            System.out.println("break outer");
            break outer;
        }
        for(int k = 0; k < 5; k++) {
            if(k == 3) {
                System.out.println("continue inner");
                continue inner;
            }
        }
    }
}
// Can't break or continue to labels here
/* Output:
i = 0
continue inner
i = 1
continue inner
i = 2
continue
i = 3
break
i = 4
continue inner
i = 5
continue inner
i = 6
continue inner
i = 7
continue outer
i = 8
break outer
*/

~~~









