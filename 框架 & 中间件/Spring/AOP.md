# AOP

[TOC]

## 概述

**「Aspect Oriented Programming（面向切面编程）」**是一种编程范式，在于通过**「分离关注点（cross-cutting concerns）」**来提升代码的模块化程度。这里的关注点就是一个**特定的功能**。

通过下面例子，我们来理解何为「分离关注点」。一个后台客服系统的每个模块都需要记录客服的操作日志，这就是一个能从业务逻辑中分离出来的关注点，完全不用耦合在每个模块的代码中，可以作为一个单独的模块存在。

**AOP 中的几个重要概念**：

|         概念         |                 说明                 |
| :------------------: | :----------------------------------: |
|    切面（aspect）    |            通知 + 切入点             |
| 连接点（join point） | 在应用执行过程中能够插入切面的一个点 |
|    通知（advice）    |   定义了切面要做什么以及何时使用。   |
|  切入点（pointcut）  | 定义了切面在何处（哪一个连接点）使用 |

AOP 的实现原理就是使用了**动态代理技术**。 这种技术可以用JDK 动态代理（`InvocationHandler`接口 + `Proxy.newProxyInstance()`）或者是 CGLIB 代理来实现。其中，CGLIB是一个生成或转换 Java 字节码的 API，广泛应用于各种框架中。

|              | 必须要实现接口 | 支持拦截 `public` 方法 | 支持拦截 `protected` 方法 | 拦截默认作用域方法 |
| :----------- | :------------: | :--------------------: | :-----------------------: | :----------------: |
| JDK 动态代理 |       ✔️        |           ✔️            |             ❌             |         ❌          |
| CGLIB 代理   |       ❌        |           ✔️            |             ✔️             |         ✔️          |

最流行的AOP框架分别是`Spring AOP`和`AspectJ`。Spring Framework 同时支持 `@AspectJ` 注解和 XML Schema 这两种方式来使用 AOP，下面分别介绍它们。



EJB、Spring，Hibernate，JBOSS以及AOP都在试图摆脱Java静态类型的束缚，让某些应用模型具有更强的适应性

## @Aspect

要使用Aspect 相关的注解和功能，那么必须在配置文件中引入下述依赖：

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.1.1</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.8.M1</version>
</dependency>
~~~

然后在Java 配置类上增加 `@EnableAspectJAutoProxy` 注解

~~~java
@Configuration
@ComponentScan("com.linkedbear.spring.aop.b_aspectj")
@EnableAspectJAutoProxy
public class Config { }
~~~

`@EnableAspectJAutoProxy` 有两个属性

- `proxyTargetClass` 用于选择是否开启基于类的代理（是否使用 CGLIB 来做代理）默认值是 `false`。若`ProxyFactoryBean` 使用了 JDK 动态代理，那么`AopProxyFactory` 就会创建 `JdkDynamicAopProxy`；若使用是采用 CGLIB 代理，则会创建 `ObjenesisCglibAopProxy`。

- `exposeProxy` 用于选择是否将代理对象暴露到 `AopContext` 中，默认值是 `false`。



如果是通过XML来配置Bean对象的话（`<context:component-scan base-package="learning.spring"/>`），那么需要在XML中使用`<aop:aspectj-autoproxy/>`标签来开启对 `@AspectJ` 支持



在完成配置后，我们就可以使用 `@Aspect` 注解来声明切面了：

~~~java
@Aspect
@Component
public class MyAspect {
	@After("execution(* com.linkedbear.spring.aop.b_aspectj.service.*.*(String)))")
public void afterPrint() {
    	System.out.println("Logger afterPrint run ......");
	}
}
~~~

-  添加 `@Aspect` 注解只是告诉 Spring这个类是切面，而不是定义一个Bean
- Spring Framework 会对带有 `@Aspect` 注解的类做特殊对待，因为其本身就是一个切面，所以不会被别的切面自动拦截。

这里，通过包扫描机制，向IoC容器注册这个切面类。

### @Pointcut

我们可以通过`@Pointcut`把切入点提取出来，从而实现复用。

我们先来看一下切入点的定义，它包括两个部分

- **切入点表达式**：用来描述要匹配的连接点。它支持与（`&&`）、或（`||`）、非（`!`）运算符
- **切入点方法签名**：可以用来引用其他的切入点

~~~java
@Aspect
public class HelloPointcut {
    @Pointcut("target(learning.spring.helloworld.Hello)")	//切入点表达式
    public void helloType() {} 

    @Pointcut("execution(public * say())")			//切入点表达式
    public void sayOperation() {} 

    @Pointcut("helloType() && sayOperation()") // 切入点方法签名
    public void sayHello() {} 
    
    // 我们只需在切入点表达式中，标注方法名即可
    @After("helloType()")
    public void afterPrint() {
        System.out.println("Logger afterPrint run ......");
    }
}
~~~

注意，**@Pointcut所注解的方法应该是空的**



`@Pointcut` 中的 PCD（pointcut designator，切入点标识符）来匹配特定的连接点

| PCD         |                 说明                 |
| :---------- | :----------------------------------: |
| `execution` |       拦截匹配特定表达式的方法       |
| `within`    |          拦截指定包中的方法          |
| `this`      | 拦截为指定类型的目标对象下的任何方法 |
| `target`    |  拦截指定类型的目标对象下的任何方法  |
| `args`      |        拦截指定参数形式的方法        |

~~~java
execution(public * *(..))		// 拦截任意公共方法
execution(* set*(..))			// 拦截以set开头的任意方法
execution(* com.xyz.service.AccountService.*(..))	// 拦截AccountService类中的所有方法
execution(* com.xyz.service.*.*(..))	// 拦截com.xyz.service包中所有类中任意方法，不包含子包中的类
    
    
within(com.xyz.service.*)		// 拦截service包中任意类的任意方法
within(com.xyz.service..*)		// 拦截service包及子包中任意类的任意方法
    
args(com.ms.aop.args.demo1.UserModel)	// 拦截只有一个参数，且参数类型为com.ms.aop.args.demo1.UserModel的方法
~~~

 `execution` 的语法是：`execution([修饰符] <返回类型> [类名.]<方法>(<参数>) [异常])`

- 可以使用 `*` 通配符
- 类名中可以使用
  - 全限定类名
  -  `.*` 表示包中的所有类
  - `..*` 表示当前包与子包中的所有类

- 参数主要分为以下几种情况：
  - `()` 表示方法无参数
  - `(..)` 表示有任意个参数
  - `(*)` 表示有一个任意类型的参数
  - `(String)` 表示有一个 `String` 类型的参数
  - `(String,String)` 代表有两个 `String` 类型的参数

除了用`*`来匹配任何数量字符，还可以通过`+`匹配指定类型及其子类型



我们还可以使用`@target`、`@args`、`@annotation`来匹配带有特定注解的目标对象：

| PCD           | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| `@target`     | 拦截带有指定注解的类                                         |
| `@args`       | 拦截参数所属类型上有指定注解的方法，而不是参数本身带有指定注解的方法 |
| `@annotation` | 拦截带有指定注解的方法                                       |

~~~java
@Aspect
public class MyAspect {
    @Before("@annotation(Deprecated)")
    public void before() {
        System.out.println("拦截到了被@Deprecated标注的方法");
    }
}
~~~

### 通知

Spring AOP 中有多种通知类型：

- `@Before` ：定义一个前置通知。它可以对被拦截方法的参数进行加工，这是通过特殊的args语法做到的

  ~~~java
  @Before("learning.spring.helloworld.HelloPointcut.sayHello() && args(count)")
  // 这里将count参数名作为args的值，它有以下两个作用：
  // 1. args拦截只有一个参数，且参数类型与count对应的类型（AtomicInteger）相同的方法
  // 2. 可以将拦截到到的参数值传入到count形参中
  public void before(AtomicInteger count) {
      // 操作count
  }
  ~~~

- `@AfterReturing` ：拦截正常返回的调用

  ~~~java
  @AfterReturning(pointcut = "execution(public * say(..))", returning = "words")
  public void printWords(String words) {
      System.out.println("Say something: " + words);
  }
  ~~~

  这里`returning`注解元素限制了，该通知只拦截返回类型与words形参类型相同的方法。同时将拦截方法的返回值传入到words形参中

- `@AfterThrowing`：如果想要拦截抛出异常的调用，可以使用 `@AfterThrowing` 注解

  ~~~java
  @AfterThrowing(pointcut = "execution(public * say(..))", throwing = "exception")
  public void printException(Exception exception) {
      
  }
  ~~~

- `@After` ：如果想要拦截正常返回或者异常返回的方法，可以使用`@After` 注解。@After注释的切面方法获取到不返回值以及异常对象，一般只能做资源清理的工作。

-  `@Around`：环绕通知，不仅可以在方法执行前后加入自己的逻辑，甚至可以完全替换方法本身的逻辑，或者替换调用参数。@Around 注解的切面方法的第一个参数必须是 ProceedingJoinPoint 类型，方法的返回类型是被拦截方法的返回类型，直接使用Object类型即可。

   ~~~java
   @Aspect
   public class TimerAspect {
       @Around("execution(public * say(..))")
       public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
           long start = System.currentTimeMillis();
           try {
               return pjp.proceed();
           } finally {
               long end = System.currentTimeMillis();
               System.out.println("Total time: " + (end - start) + "ms");
           }
       }
   }
   ~~~

   其中的 `pjp.proceed()` 就是执行所拦截的方法，`proceed()` 方法也接受 `Ojbect[]` 参数，可以替代原先的参数。

- `@DeclareParents`：我们可以为 Bean 添加新的接口，并为新增的方法提供默认实现，这种操作被称为**引入**（Introduction）。使用示例：

  ~~~java
  // 原始类
  public interface Person {
      void likePerson();
  }
  
  @Component("women")
  public class Women implements Person {
  
      @Override
      public void likePerson() {
          System.out.println("我是女生，我喜欢帅哥");
      }
  }
  
  // 新添加的类
  public interface Animal {
      void eat(); 
  }
  
  @Component
  public class FemaleAnimal implements Animal {
  
      @Override
      public void eat() {
          System.out.println("我是雌性，我比雄性更喜欢吃零食");
      }
  }
  ~~~
  
  ~~~java
  @Aspect
  @Component
  public class AspectConfig {
      //"+"表示person的所有子类；defaultImpl 表示默认需要添加的新的类
      @DeclareParents(value = "com.lzj.spring.annotation.Person+", defaultImpl = FemaleAnimal.class)
      public Animal animal;
  }
  ~~~
  
  测试代码：
  
  ~~~java
  public static void main(String[] args) {
      AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotationConfig.class);
      Person person = (Person) ctx.getBean("women");
      person.likePerson();
      // person代理类已经实现了Animal接口
      Animal animal = (Animal)person;
      animal.eat();
  }
  ~~~
  
  引入声明后，在 Spring 容器中取到的 Bean 就已经完成了增强，**甚至在前置通知中也是如此。**




在这些通知注解上就可以指明切入点，为什么我们还要将这些切入点单独放在一个类中呢？答：复用性！要是存在多个通知作用于同一处，可以让切面类实现 `Ordered` 接口，或者在上面添加 `@Order` 注解。

## XML Scheme

Spring AOP 相关的 XML 配置，都要放在 `<aop:config/>` 中，

 `<aop:aspect/>` 来声明切面

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:config>
        <!--id，指定该切面的名字-->
        <!--ref指定一个切面类，里面包括要增强的方法-->
        <!-- order 属性也可以配置切面的顺序。-->
        <aop:aspect id="helloAspect" ref="aspectBean">
            <!--method指定了要实现增强功能的方法-->
            <!--pointcut指定了被增强对象-->
            <aop:before method="beforePrint"
                pointcut="execution(public void org.example.FinanceService.addMoney(double))"/>
        </aop:aspect>
    </aop:config>
	
    
    <aop:config>
        <!-- <aop:pointcut/>来配置切入点，在`<aop:config>`或者`<aop:aspect>`中放置该标签。通过 `pointcut-ref` 属性来使用事先定义好的切入点-->
        <aop:pointcut id="foo" expression="execution(public void org.example.FinanceService.addMoney(double))"/>
        <aop:aspect id="loggerAspect" ref="logger">
            <aop:before method="beforePrint"
                pointcut-ref="foo"/>
        </aop:aspect>
    </aop:config>
    
    <bean id="aspectBean" class="..." />
</beans>
~~~





基于 XML Schema 的通知同样分为六类

- `<aop:before/>`

- `<aop:after-returning/>`：正常返回

  ~~~xml
  <aop:after-returning pointcut="execution(public * say(..))"
      returning="words"
      method="printWords" />
  ~~~

- `<aop:after-throwing/>`：抛出异常

  ~~~xml
  <aop:after-throwing pointcut="execution(public * say(..))"
  	throwing="exception"
  	method="printException" />
  ~~~

- `<aop:after/>`

  ~~~xml
  <aop:after pointcut="execution(public * say(..))" method="afterAdvice" />
  ~~~

-  `<aop:around/>`

  ~~~xml
  <aop:aspect id="beforeAspect" ref="beforeAspectBean">
  	<aop:around pointcut="execution(public * say(..))" method="recordTime" />
  </aop:aspect>
  ~~~

-  `<aop:declare-parents/>`来定义引入通知

  ~~~xml
  <aop:aspect id="myAspect" ref="myAspectBean">
      <aop:declare-parents types-matching="learning.spring.helloworld.Hello+"
          implement-interface="learning.spring.helloworld.GoodBye"
          default-impl="learning.spring.helloworld.DefaultGoodByeImpl"/>
  </aop:aspect>
  ~~~









