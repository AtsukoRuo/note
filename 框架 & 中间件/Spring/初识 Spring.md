# 初识 Spring

Spring 家族中的主要成员：

- **Spring Framework**：提供了依赖注入、AOP、资源管理等特性。
- **Spring Boot**：包含了健康检查、监控、度量指标、外化配置等生产所需的功能，降低了开发生产级 `Spring` 应用的门槛。此外，它还提供了**起步依赖**（starter dependency）很好地解决了 Spring 应用的依赖管理困境，而且提供了**自动配置**来减少了 Spring 应用的配置量
- **Spring Cloud**，它是一系列模块的集合，这些模块分别实现了服务发现、配置管理、服务路由、服务熔断、链路追踪等功能
- **Spring Data**，Spring Framework 为传统的关系型数据库操作提供了统一的抽象。但是随着数据库技术的不断发展，涌现了大量的新技术和新产品，如果把对它们的支持都放入 Spring Framework 中，会导致框架十分臃肿，于是就有了 Spring Data。



> SSH 的三个字母分别指代 Spring Framework、Struts 和 Hibernate。现在 Spring MVC 替代了 Struts



通过 Spring Initializr（[https://start.spring.io](https://start.spring.io/)）工具来初始化工程



一个标准的Maven工程结构包含

-  pom.xml：包含工程元数据、依赖和插件配置
-  application.properties：工程的配置文件
- ApplicationTests：测试类
- Application：入口程序

![image-20231129141532489](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20231129141532489.png)



下面我们在Application类编写以下代码：

~~~java
@SpringBootApplication
@RestController
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @RequestMapping("/helloworld")
    public String helloworld() {
        return "Hello World! Bravo Spring!";
    }
}
~~~

启动SpringBoot项目之后，在浏览器中访问 http://localhost:8080/helloworld 就能看到程序的输出





**控制反转（Inversion of Control，IoC）**、**依赖注入（Dependency Injection）**与**面向切面编程（Aspect Oriented Programming，AOP）**是 Spring Framework 中最重要的概念。



首先我们来介绍下**「控制反转」（Inversion Of Control）**。这里的“控制”指的是对程序执行流程的控制，而“反转”指的是调用者与被调用者之间的控制权转移。例如，在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程可以通过框架来控制。流程的控制权从程序员“反转”到了框架。



**依赖注入（Dependency Injection）**：不通过new()的方式在类内部创建依赖类对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（注入）给类使用。也就是说，将对象的创建反转给上层来处理。读者可能认为这样做破坏了封装性，实则不然，因为仅仅是将私有状态的初始化操作交给调用者来负责而已，并未暴露该状态的修改方法。

在实际的软件开发中，一些项目可能会涉及几十、上百、甚至几百个类，类对象的创建、依赖注入、组件之间依赖关系的处理会变得非常复杂。而这些工作跟具体的业务逻辑是无关的，此时我们完全可以抽象成框架来自动完成，从而减少手动装配出错的可能性。Spring Framework 的 IoC 容器正是将组件内部的依赖管理、生命周期管理的逻辑抽离出来，从而让开发人员专注于业务逻辑的实现。