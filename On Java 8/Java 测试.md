# 代码校验

>你无法证明自己的代码是正确的！



静态类型检查是一种非常有限的测试类型，它仅仅意味着编译器“测试通过”了代码的语法和基本类型规则，并不意味着代码满足了程序的目标。



## 单元测试

**单元测试**是一个将测试集成到你所编写代码里的过程，并在每次构建系统时运行这些测试。这样，构建过程不仅可以检查语法错误，还可以**检查语义错误**。这里的“单元”表明测试的是一小段代码。

这里使用`JUnit`作为单元测试的工具，它的Maven Dependency为：

~~~xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
~~~

- `@Test`注解标识即将测试的方法

- `@BeforeAll`注解标注的方法会在**所有测试**执行**之前**运行一次。方法都必须是静态的。

- `@AfterAll`注解标注的方法会在**所有测试**执行**之后**运行一次。方法都必须是静态的。

- `@BeforeEach`注解标注的方法会在**每次测试**执行**之前**运行一次。

- `@AfterEach`注解标注的方法会在**每次测试**执行**之后**运行一次。
- `assert`



## 前置条件

**前置条件（precondition） **的概念来自**契约式设计（Design By Contract, DbC）**，并使用了基本的**断言（assertion）**机制来实现。

通过断言可以验证程序执行期间是否满足某些条件。Java断言语法如下

~~~java
assert boolean-expression;
assert boolean-expression: information-expression;
~~~

如果断言失败，即boolean-expression为false，那么抛出`AssertionError`异常。

断言在一般情况下并不会执行。可以在虚拟机中设置`-ea`标志来启用断言，它也可以拼写为`-enableassertions`。或者执行`ClassLoader.getSystemClassLoader().setDefaultAssertionStatus(true)`语句为之后所有加载的类启动断言。