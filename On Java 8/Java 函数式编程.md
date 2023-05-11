# Java 函数式编程

函数式编程实现了对行为的抽象，使我们能够操纵代码片段。就像面向对象编程实现了对数据的抽象。

函数式编程的优点：

- 从代码创建、维护和可靠性的角度来看。通过整合现有、可以理解的、经过良好测试的代码来产生新的功能，而不是从零开始编写所有内容，由此我们会得到更可靠的代码，而且实现起来更快。

- 纯函数式语言在安全方面做出了更多努力。它规定了额外的约束条件：

	- 所有的数据必须是不可变
	- 函数无副作用

	这一编程范式解决了并行编程中最基本和最棘手的问题之一——“可变的共享状态”问题。因此，纯函数式语言经常被当作并行编程问题的解决方案



在Java中，只能通过**函数式接口（Functional Interface）**，或者称为**单一抽象方法（Single Abstract Method，SAM）**，来操纵代码片段。 这类接口只定义了一个抽象方法。可以通过`@FunctionalInterface`注解来标识一个函数式接口

~~~java
@FunctionalInterface
interface Strategy {
    String approach(String msg);
}
~~~

有四种方式将代码片段传递给方法

- 类
- 匿名类
- Lambda表达式
- 方法引用

给出一个例子：

~~~java
interface Strategy {
    String approach(String msg);
}

class Soft implements Strategy {
    @Override public String approach(String msg) {}
}

class Unrelated {
    static String twice(String msg) { }
}

public class Strategize {
    public static void main(String[] args) {
        Strategy[] strategies = {
            new Soft(),
            new Strategy() {                    
                public String approach(String msg) {
                    return msg.toUpperCase() + "!";
                }
            },
            msg -> msg.substring(0, 5),      
            Unrelated::twice                    
        };
    }
}
~~~



## Lambda表达式

`java.util.function`旨在创建一套足够完备的目标接口，我们无需再创建了

|  接口名  | 参数 |  返回值  |           方法            |                示例                 |
| :------: | :--: | :------: | :-----------------------: | :---------------------------------: |
| Supplier |  无  | 任意类型 | `get()`<br/>`getAstype()` | `Supplier<T>`<br/>`BooleanSupplier` |
| Consumer | 一个 |    无    |        `accept()`         |  `Consumer<T>`<br />`IntConsumer`   |

 可以看到基本类型给Java增加了多少复杂性。复杂性在这里体现为专门给基本类型提供一些函数式接口。

在早期设计Java中，出于效率考虑，设计者增添了基本类型。而效率问题很快就得以缓解了。现在，在这门语言的生命周期内，我们只能忍受一个糟糕的语言设计选择所带来的后果。



## 方法引用

## 高阶函数

## 闭包

## 柯里化和部分求值

