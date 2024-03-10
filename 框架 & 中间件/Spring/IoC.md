# IoC

[TOC]

## 概述

Spring 家族中的主要成员：

- **Spring Framework**：提供了依赖注入、AOP、容器、资源管理等特性。
- **Spring Boot**：包含了健康检查、监控、度量指标、外化配置等生产所需的功能，降低了开发生产级 Spring 应用的门槛。此外，它还提供了**起步依赖**（starter dependency）很好地解决了 Spring 应用的依赖管理困境。而且提供了**自动配置**来减少了 Spring 应用的配置量
- **Spring Cloud**，它是一系列模块的集合，这些模块分别实现了服务发现、配置管理、服务路由、服务熔断、链路追踪等功能
- **Spring Data**，Spring Framework 为传统的关系型数据库操作提供了统一的抽象。



> SSH 的三个字母分别指代 Spring Framework、Struts 和 Hibernate。现在 Spring MVC 替代了 Struts



通过 Spring Initializr（[https://start.spring.io](https://start.spring.io/)）工具来创建工程。



一个标准的Maven工程结构包含

-  pom.xml：包含工程元数据、依赖和插件配置
-  application.properties：工程的配置文件
- ApplicationTests：测试类
- Application：入口程序

![image-20231129141532489](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20231129141532489.png)



## 容器 & 控制反转

**控制反转（Inversion of Control，IoC）**、**依赖注入（Dependency Injection）**与**面向切面编程（Aspect Oriented Programming，AOP）**是 Spring Framework 中最重要的概念。

首先我们来介绍下**「控制反转」（Inversion Of Control）**。这里的“控制”指的是对程序执行流程的控制，而“反转”指的是调用者与被调用者之间的控制权转移。例如，在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程可以通过框架来控制（模板方法）。流程的控制权从程序员“反转”到了框架。

**依赖注入（Dependency Injection）**：不通过new()的方式在类内部创建依赖类对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（注入）给类使用。也就是说，将对象的创建**控制反转**给上层来处理。

在实际的软件开发中，一些项目可能会涉及几十、上百、甚至几百个类，类对象的创建、依赖注入、组件之间依赖关系的处理会变得非常复杂。而这些工作跟具体的业务逻辑是无关的，此时我们完全可以抽象成框架来自动完成，从而减少手动装配出错的可能性。Spring Framework 的 IoC 容器正是基于依赖注入的思想，将组件内部的依赖管理、生命周期管理的逻辑抽离出来，从而让开发人员专注于业务逻辑的实现。

<img src="https://www.ituring.com.cn/figures/2023/LearnSpring/009.jpg" alt="{%}" style="zoom:10%;" />



为了使用 Spring 的容器，需要在 pom.xml 文件中引入 `org.springframework:spring-beans` 依赖：

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.3.15</version>
</dependency>
~~~

在`resources/`目录下创建`.xml`配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
~~~

编写Bean对象

~~~java
public class Person { }
~~~

向配置文件中注册Bean对象

~~~xml
<bean id="person" class="${全限定包名}"></bean>
~~~

获取Bean对象

~~~java
public static void main(String[] args) throws Exception {
    // 加载配置
    BeanFactory factory = new ClassPathXmlApplicationContext("basic_dl/quickstart-byname.xml");
    // 获取Bean对象，按ID获取
    Person person = (Person) factory.getBean("person");
    // 获取Bean对象，按类名获取
    Person person = factory.getBean(Person.class);
}
~~~



`BeanFactory` 是容器的基础接口。`ApplicationContext` 接口继承了 `BeanFactory`，在它的基础上增加了更多企业级应用所需要的特性（国际化、事件发布、生命周期管理等等）。常见的 `ApplicationContext` 实现如下：

| 类名                                 | 说明                                                    |
| ------------------------------------ | ------------------------------------------------------- |
| `ClassPathXmlApplicationContext`     | 从 CLASSPATH 中加载 XML 文件来配置 `ApplicationContext` |
| `FileSystemXmlApplicationContext`    | 从文件系统中加载 XML 文件来配置 `ApplicationContext`    |
| `AnnotationConfigApplicationContext` | 根据注解和 Java 类配置 `ApplicationContext`             |

如果要使用 `ApplicationContext`，那么需要在 pom.xml 文件中引入 `org.springframework: spring-context` 依赖。

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.15</version>
</dependency>
~~~

然后在代码中使用ApplicationContext对象

~~~java
public class Application {
    public static void main(String[] args) {
        // 直接读取配置文件到容器中，比DefaultListableBeanFactory 和 XmlBeanDefinitionReader 的组合要简洁
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        
        Hello hello = applicationContext.getBean("hello", Hello.class);
    }
}
~~~



`Spring`容器之间存在着继承关系——子容器可以继承父容器中配置的组件。假设现在有两个xml文件`child-beans.xml`与`parent-beans.xml`。如果我们要指定`parent-beans`为`child-beans`的父配置，那么需要在代码中：

~~~java
public class Application {
    private ClassPathXmlApplicationContext parentContext;
    private ClassPathXmlApplicationContext childContext;


    public Application() {
        parentContext = new ClassPathXmlApplicationContext("parent-beans.xml");
        
        // 从这里指定继承关系
        childContext = new ClassPathXmlApplicationContext(
                new String[] {"child-beans.xml"}, true, parentContext);
        
        // 设置容器的标识符
        parentContext.setId("ParentContext");
        childContext.setId("ChildContext");
    }
}
~~~

现在我们说明继承容器中 Bean 的可见性和覆盖情况

- 子容器可以看到父容器中定义的 `Bean`，反之则不行
- 子容器中可以定义与父容器同 ID 的 `Bean`，它们各自都能获取自己定义的 Bean（覆写）。

通过容器的 `setAllowBeanDefinitionOverriding()` 方法可以设置覆盖同ID Bean的特性。如果参数设为false，那么在Spring配置信息解析过程中，一旦遇到新的同名Bean定义，框架就会抛出异常。记得要刷新容器



## Beans

JavaBeans 是 Java 中一种特殊的类，它满足：

- 可序列化
- 提供public无参构造器
- 属性都是私有的，并且为每一个属性提供getter、setter方法



其中，名称中的Bean是指**可复用软件组件**。Spring 容器也遵循这一惯例，因此将容器中管理的可复用组件称为 **Spring Bean**（以下简称**Bean**）。在一个 Bean 的定义中，会包含如下部分：

- Bean 的名称，一般是 Bean 的 `id`，也可以为 Bean 指定别名（alias）；
- Bean 的具体类信息，这是一个全限定类名；（BO、VO、POJO等）
- Bean 的作用域，是单例（singleton）还是原型（prototype）；前者每次获取返回的是同一个 Bean，而后者每次获取返回的是一个新的对象。**通常来说Prototype对应有状态（Stateful）的Bean对象，而Singleton对象无状态（Stateless）的Bean对象**
- 依赖注入相关信息，构造方法参数、属性以及自动织入（autowire）方式；
- 创建销毁相关信息，懒加载模式、初始化回调方法与销毁回调方法。

### Bean的id、name属性

`id`属性唯一地标识某个Bean对象

`name`属性用于定义Bean实例的别名(aliases)，一个Bean实例可以有多个别名，在name属性中用`,`或`;`分隔各个别名即可：

~~~xml
<bean name="map,java_map;jdk_map" class="java.util.HashMap" />
~~~

如果一个bean标签未指定id属性，那么将class属性中的全限定类名作为bean的默认id。如果有多个bean未指定id，而且class属性值相同，那么会按照其出现的次序，分别给其的id设置为 「全限定类名#1」, 「全限定类名#2」

在同一个配置文件中，任意Bean对象之间的id、name属性值是禁止重复的

|                           非法配置                           |                   含义                   |
| :----------------------------------------------------------: | :--------------------------------------: |
| `<bean id="map" class="java.util.HashMap" />`<br>`<bean id="map" class="java.util.HashMap" />` |            两个bean的id值重复            |
| `<bean name="map" class="java.util.HashMap" />` <br>`<bean name="map" class="java.util.HashMap" />` |           两个bean的name值重复           |
| `<bean id="map" class="java.util.HashMap" />`<br/>`<bean name="map" class="java.util.HashMap" />` | 第一个bean的id值与第二个bean的name值重复 |

### Bean的作用域

| 作用域类型  | 概述                                         |
| ----------- | -------------------------------------------- |
| singleton   | 一个 IOC 容器中只有一个【默认值】            |
| prototype   | 每次获取创建一个                             |
| request     | 一次请求创建一个（仅Web应用可用）            |
| session     | 一个会话创建一个（仅Web应用可用）            |
| application | 一个 Web 应用创建一个（仅Web应用可用）       |
| websocket   | 一个 WebSocket 会话创建一个（仅Web应用可用） |

bean的创建时机：

- 对于`Singleton` Bean以及`FactoryBean` 本身，都是伴随容器初始化而创建
- 而FactoryBean创建的Bean以及 Prototype Bean，都是延迟创建的（按需创建），<del>而且不会触发后置处理器的`postProcessBeforeInitialization()`方法。</del>

### Bean 的三种配置方式

Spring Framework 提供了多种不同风格的配置方式：

- 基于XML文件的配置

  - 在XML中编写Bean标签
  
- 基于注解的配置，是配合Bean XML或者Java配置类来一起使用的

  - Bean XML：`<context:component-scan base-package="learning.spring"/>`
  - Java配置类：`@ComposeScan`

- Java配置类一般和SpringBoot配合使用，兼容XML以及注解

  - `@Bean`
  - `@ComposeScan`，兼容注解
  - `@ImportResource`，兼容XML


#### 基于 XML 文件的配置

有两种基本的注入方式：

- 基于构造方法的注入
- 基于Setter方法的注入

~~~java
public class Hello {
    private String name;
    public Hello(String name) {
        this.name = name;
    }
}
~~~

在对应的 XML 配置文件中，使用 `<constructor-arg/>` 来传入构造方法所需的内容

~~~xml
<bean id="hello" class="learning.spring.helloworld.Hello">
    <constructor-arg value="Spring"/>
</bean>
~~~

`<constructor-arg/>` 的可配置属性如下：

|  属性   |                    作用                     |
| :-----: | :-----------------------------------------: |
| `value` |                所要注入的值                 |
|  `ref`  |      所要注入的Bean对象，其值为Bean ID      |
| `type`  |        通过类型来指定所要传入的参数         |
| `index` | 通过位置来指定所要传入的参数，从 0 开始计算 |
| `name`  |        通过命名来指定所要传入的参数         |

每一个`constructor-arg`标签都对应构造函数中的一个参数。所以有N个`constructor-arg`标签就对应具有N个参数的构造函数。



通过`<list>`、`<set>`、`<map>`等子标签，可以注入集合类型：

~~~xml
<bean class="live.sunhao.vo.Student">
	<constructor-arg>
		<list>
			<value>12</value>
			<value>Tomcat</value>
			<ref bean="other_bean"/>
			<bean class="java.lang.String">
				<constructor-arg value="This is a list"></constructor-arg>
			</bean>
		</list>
	</constructor-arg>
    
    <constructor-arg>
		<set>
			<value>12</value>
			<value>Tomcat</value>
			<ref bean="other_bean"/>
			<bean class="java.lang.String">
				<constructor-arg value="This is a set"></constructor-arg>
			</bean>
		</set>
	</constructor-arg>
    
    <constructor-arg>
		<map>
			<entry key="name" value="myself"></entry>
			<entry key="age" value="22"></entry>
			<entry key="now" value-ref="da"></entry>
		</map>
	</constructor-arg>
</bean>
~~~

如果Map的key/value的类型为基本类型或者String，那么可以直接通过key/value属性来设置。如果是其他类类型，那么就需要通过key-ref/value-ref来设置



通过`<property/>`标签，我们可以基于Setter方法来依赖注入。注意，Setter只接受一个参数

~~~xml
<bean id="..." class="...">
    <!--name为要注入的方法的名字-->
    <property name="xxx">
        <!-- 直接定义一个内部的Bean -->
        <bean class="..."/>
    </property>

    <property name="yyy">
        <!-- 定义依赖的Bean -->
        <ref bean="..."/>
    </property>

    <property name="zzz">
        <!-- 定义一个列表 -->
        <list>
            <value>aaa</value>
            <value>bbb</value>
        </list>
    </property>
</bean>
~~~



在 `<bean/>` 中可以通过 `autowire` 属性来设置使用何种自动装配的方式

|     名称      |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|     `no`      |                        不进行自动织入                        |
|   `byName`    | 若某个Bean对象的ID属性与所要注入的属性的名字匹配，那么就将该Bean对象注入到该属性中 |
|   `byType`    | 若某个Bean对象的Class属性与所要注入的属性的类型匹配，那么就将该Bean对象注入到该属性中 |
| `constructor` |        同 `byType`，但是通过构造函数的参数类型来匹配         |

~~~XML
<!-- 使用引用的方式 -->
<bean id="cat" class="com.wei.pojo.Cat"></bean>
<bean id="user" class="com.wei.pojo.User" autowire="byType">
    <property name="cat" ref="cat" />
</bean>

<!-- 使用autowire的方式-->
<bean id="user" class="com.wei.pojo.User" autowire="byType"></bean>
~~~

在使用自动装配时，需要注意以下事项：

- 开启自动织入后，仍可以手动设置依赖，手动设置的依赖优先级高于自动织入；
- 自动装配无法注入基本类型和字符串；
- 如果有多个匹配的Bean对象，那么就会抛出异常
- 对于集合类型，使用`byType`装配模式，此时将匹配到多个的Bean对象都依赖注入到该集合中，并不会像第三点那样抛出异常。

为了避免第三点中说到的问题，可以将

-  `<bean/>` 的 `autowire-candidate` 属性用来指定该bean是否可以被自动装配到其他的bean中。设置为 `false`即可。
-  将某一个候选 Bean 中的`<bean/>` 中的 `primary`属性 设置为 `true`



如何指定 Bean 的初始化顺序？

- 一般情况下，Spring 容器会根据依赖关系来决定 Bean 的初始化顺序。不过，有时 Bean 之间的依赖关系是循环的，容器可能无法按照我们的预期进行初始化。此时，我们可以通过`<bean/>` 的 `depends-on` 属性来指定当前 Bean 还要依赖哪些 Bean，来确定Bean的依赖顺序（`@DependsOn` 注解）

#### 基于注解的配置

我们需要在xml文件中写入以下配置，来启用基于注解的配置：

~~~xml

<context:component-scan base-package="learning.spring"/>
~~~

上述配置会扫描 `learning.spring` 包内的类。对于添加以下四个注解的类，Spring 容器把它们注册为 `Bean`对象

| 注解          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| `@Component`  | 将类标识为普通的组件，即一个 Bean                            |
| `@Service`    | 将类标识为服务层的服务                                       |
| `@Repository` | 将类标识为数据层的数据仓库，一般是 DAO（Data Access Object） |
| `@Controller` | 将类标识为 Web 层的 Web 控制器（后来针对 REST 服务又增加了一个 `@RestController` 注解） |

`@Controller` 、`@Service` 、`@Repository`本质上都是`@Component`：

~~~java
@Component
public @interface Controller { ... }
~~~



~~~java
@Component("aaa")
public class Person { }
~~~

如果不指定 Bean 的名称，那么默认名称是 首字母小写的类名（例如 `DepartmentServiceImpl` 的默认名称是 `departmentServiceImpl` ）。



在Spring Boot中，如果一个@Component类有多个构造器，Spring会尝试选择一个最合适的构造器来实例化该类：

- 如果该类中只有一个构造器，Spring会使用该构造器来实例化该类。
- 如果该类中有多个构造器，Spring会首先尝试使用默认构造器（即无参构造器）来实例化该类。如果没有默认构造器，Spring会优先选择带有@Autowired注解的构造器（如果有的话），并将其用于实例化该类。
- 如果有多个构造器都带有@Autowired注解，Spring会选择参数数量最多的构造器来实例化该类
- 如果有多个构造器的参数数量相同，则Spring会引发异常，因为无法确定应该使用哪个构造器。



如果要注入依赖，可以使用如下的注解：

| 注解         | 说明                                 |
| :----------- | :----------------------------------- |
| `@Autowired` | **根据类型注入依赖**                 |
| `@Resource`  | JSR-250 的注解，**根据名称注入依赖** |
| `@Inject`    | JSR-330 的注解，同 `@Autowired`      |





@Autowired可以作用在构造函数、属性、setter上：

~~~java
@Service
public class ConstructorServiceImpl implements ConstructorService {
    // 1.属性注入
    @Autowired
    private UserService userService;
    
    // 2.构造函数注入（推荐）
    // 构造器参数列表过长的情况，可能是这个 Bean 承担的责任太多，应该考虑组件的责任拆解。
    @Autowired
    public ConstructorServiceImpl(UserService userService) {
        this.userService = userService;
    }
    
    // 2. 非显式的构造函数注入，无需Autowired
	public ConstructorServiceImpl(UserService userService) {
        this.userService = userService;
    }
    
    // 3. setter注入
    @Qualifier("userService2")
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
~~~

@Autowired的注入逻辑如下：

1. 按照类型，来匹配 Bean 类型与之相同的Bean对象（考虑向上兼容）
2. 如果有多个匹配的Bean，
   - 按被注入对象的属性名来继续匹配Bean ID。
3. 否则默认抛出异常，如果设置了`@Autowired(required = false)`，那么返回`null`

@Autowired的细节

- 只有使用构造函数注入才能注入final字段
- 执行顺序：构造函数注入/构造函数 -> 字段注入/setter注入

ObjectProvider可以和@Autowired搭配使用

~~~java
@Autowired
public Dog(ObjectProvider<Person> person) {
    // 如果没有Bean，则采用缺省策略创建
    this.person = person.getIfAvailable(Person::new);
}
~~~





Spring可能会匹配到多个Bean，此时这些Bean统称为**候选者（candidates）。**可以使用`@Primary`注解或者`@Qualifier` 注释来决定在冲突时使用哪一个Bean。优先级@Qualifier > @Primary 。**一般来说通过Primary来指定默认行为，而通过@Qualifier来指定特定行为。**

@Primary添加在候选者上，而@Qualifier添加在被注入的对象上（通常和@Autowired一起）。@Qualifier("foo")，将匹配Bean的ID为foo的Bean。

此外，集合类型可以把所有指定类型的 Bean 都注入。可以避免上述冲突问题

~~~java
@Autowired
private List<Person> persons;
~~~



`@Resource`的注入逻辑

- 如果同时指定了name和type，则注入同时匹配name和type的Bean
- 如果指定了name，则注入匹配ID的Bean
- 如果指定了type，则注入匹配类型的Bean
- 如果既没有指定name，又没有指定type，则首先按照byName方式进行装配；如果没有匹配，则按照byType方式进行装配





#### 基于 Java 类的配置

要使用基于Java类的配置，就要与Spring Boot 一起使用。`Spring`配置类的定义例子：

```java
@Configuration
@ComponentScan("learning.spring")
@ImportResource("classpath:annotation/beans.xml")
@Import({ CongfigA.class, ConfigB.class })
public class Config {
    @Bean
    @Lazy
    @Scope("prototype")
    public Hello helloBean() {
        return new Hello();
    }
}
```

- `@Configuration` 注解表明这是一个 Java 配置类。实际上，它也是一个`@Component` 

  ~~~java
  //...
  @Component
  public @interface Configuration { ... }
  ~~~

  向BeanFacory注册配置类：

  ~~~java
  AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
  ~~~

-  `@ComponentScan` 注解将指定包下的Bean对象（@Component、@Configuration等注解的类）添加到当前配置类中（兼容基于注解的配置）。 如果不写 `@ComponentScan` ，也是可以做到组件扫描的，可以在 `AnnotationConfigApplicationContext` 的构造方法中指定

  ~~~java
  ApplicationContext ctx = new AnnotationConfigApplicationContext("com.linkedbear.spring.annotation.c_scan.bean");
  ~~~

- `@ImportResource`将指定XML配置文件中的Bean对象添加到当前配置类中（兼容基于XML的配置）。 

- `@Import`导入其他配置类

- `@Bean` 注解表示该方法的返回对象会被当做容器中的一个 Bean，

- `@Lazy` 注解说明这个 Bean 是延时加载的，

- `@Scope` 注解则指定了 `Bean`是原型的。





`@Bean` 注解的属性如下：

| 属性                | 默认值                                | 说明                                             |
| :------------------ | :------------------------------------ | :----------------------------------------------- |
| `name`              | `{}`                                  | Bean 的名称（相当于xml中的id属性），默认同方法名 |
| `value`             | `{}`                                  | 同 `name`                                        |
| `autowire`          | `Autowire.NO`                         | 自动织入方式                                     |
| `autowireCandidate` | `true`                                | 是否自动装配到其他 Bean 中                       |
| `initMethod`        | `""`                                  | 初始化方法名                                     |
| `destroyMethod`     | `AbstractBeanDefinition.INFER_METHOD` | 销毁方法名                                       |

AbstractBeanDefinition.INFER_METHOD表示在Bean销毁时，自动调用修饰符为 `public`、没有参数且方法名是 `close` 或 `shutdown` 的方法（`close`优于`shutdown`）。



在 Java 配置类中指定 Bean 之间的依赖关系有两种方式：

- 通过方法的参数注入依赖
- 直接调用类中带有 `@Bean` 注解的方法

~~~java
@Configuration
public class Config {
    @Bean
      public Foo foo() {
        return new Foo();
    }

    @Bean
    public Bar bar(Foo foo) {			// 方式1
        return new Bar(foo);
    }

    @Bean
    public Baz baz() {
        return new Baz(foo());		 	// 方式2
    }
}
~~~

Spring Framework 针对 `@Configuration` 类中带有 `@Bean` 注解的方法通过 CGLIB（Code Generation Library）做了特殊处理。对于返回单例Bean的方法，只会执行一次，之后的多次调用都直接返回相同的Bean对象，并不会执行该方法。





### Bean的获取

getBean()再按类型查询时，要求只有一个符合类型的Bean对象，否则抛出`NoUniqueBeanDefinitionException`异常

~~~java
ctx.getBean(Color.class);
~~~

但是ApplicationContext提供了getBeansOfType，可以返回多个符合类型的Bean对象：

~~~java
ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-oftype.xml");
Map<String, DemoDao> beans = ctx.getBeansOfType(DemoDao.class);
beans.forEach((beanName, bean) -> {
    System.out.println(beanName + " : " + bean.toString());
});
~~~

此外，还可以获取标有特定注解的Bean对象

~~~java
ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-withanno.xml");
// 这里的Color是一个注解
Map<String, Object> beans = ctx.getBeansWithAnnotation(Color.class);
beans.forEach((beanName, bean) -> {
    System.out.println(beanName + " : " + bean.toString());
})
~~~

获取每一个Bean的名字：

~~~java
ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-withanno.xml");
String[] beanNames = ctx.getBeanDefinitionNames();
Stream.of(beanNames).forEach(System.out::println);
~~~



如果未查找到Bean对象，会抛出`NoSuchBeanDefinitionException`异常。我们可以通过捕获异常来启用缺省策略：

~~~java
try {
    dog = ctx.getBean(Dog.class);
} catch (NoSuchBeanDefinitionException e) {
    // 找不到Dog时手动创建
    dog = new Dog();
}
~~~

这样编写代码并不优雅，好在ApplicationContext为我们提供了`containsBean()`方法，用于判断是否存在这样一个Bean

~~~java
Dog dog = ctx.containsBean("dog") ? (Dog) ctx.getBean("dog") : new Dog();
~~~

此外，还有`ObjectProvider`，它可以延迟查找：

~~~java
 // 下面的代码会报Bean没有定义 NoSuchBeanDefinitionException
// Dog dog = ctx.getBean(Dog.class);

// 这一行代码不会报错
ObjectProvider<Dog> dogProvider = ctx.getBeanProvider(Dog.class);
~~~

只有调用`ObjectProvider`的`getObject()`方法时，才去获取Bean对象。若未查找到，则抛出异常。它还有其他方法

~~~java
// 当未查找到时，调用Suppplier回调函数
Dog dog = dogProvider.getIfAvailable(Dog::new);

dogProvider.ifAvailable(dog -> System.out.println(dog));
~~~



## 定制容器与 Bean 的行为

### Bean的生命周期

<img src="assets/010.jpg" alt="{%}" style="zoom: 25%;" />

可以通过以下三种方式来注册创建或销毁Bean时的回调：

- 实现 `InitializingBean` 和 `DisposableBean` 接口；

  - `InitializingBean` 接口有一个 `afterPropertiesSet()` 方法
  - `DisposableBean` 接口中的 `destroy()` 方法

- 使用 JSR-250 的 `@PostConstruct` 和 `@PreDestroy` 注解；

- 在 `<bean/>` 或 `@Bean` 里配置初始化和销毁方法。

  - 在 `<bean/>` 中指定 `destroy-method`，或者在 `@Bean` 中指定 `destroyMethod`。

  - 在 `<bean/>` 中指定 `init-method`，或者在 `@Bean` 中指定 `initMethod`。

    ~~~xml
    <context:annotation-config />
    <bean id="hello" class="learning.spring.helloworld.Hello" init-method="init" />
    ~~~
    
    ~~~java
    @Bean(initMethod="init")
    public Hello hello() {...}
    ~~~
    


无论是初始化还是销毁，Spring 都会按照如下顺序依次进行调用（销毁并不会逆序执行）：

1. 添加了 `@PostConstruct` 或 `@PreDestroy` 的方法；

2. 实现了 `InitializingBean` 的 `afterPropertiesSet()` 方法，或 `DisposableBean` 的 `destroy()` 方法；

3. 在 `<bean/>` 中配置的 `init-method` 或 `destroy-method`，`@Bean` 中配置的 `initMethod` 或 `destroyMethod`。



对于原型Bean来说，并不会像单例Bean那样，在容器初始化时必定执行回调，而是在延迟创建（按需创建）时，执行初始化回调。并且在销毁时，不会执行`destroy-method` 标注的方法。

何时执行销毁回调

~~~java
ctx.getBeanFactroy().destroyBean(pen);		// 从容器中删除Bean时
ctx.close();							 // 容器关闭时
~~~



对于初始化以及销毁方法的要求：

1. 方法无参数
2. 方法无返回值

### Aware接口

| 接口名                         | 用途                                 |
| ------------------------------ | ------------------------------------ |
| BeanFactoryAware               | 回调注入 BeanFactory                 |
| ApplicationContextAware        | 回调注入 ApplicationContext          |
| EnvironmentAware               | 回调注入 Environment                 |
| ApplicationEventPublisherAware | 回调注入事件发布器                   |
| ResourceLoaderAware            | 回调注入资源加载器                   |
| BeanClassLoaderAware           | 回调注入加载当前 Bean 的 ClassLoader |
| BeanNameAware                  | 回调注入当前 Bean 的名称             |



例子：

~~~java
@Component
public class MyBean implements ApplicationContextAware, BeanNameAware {
    private ApplicationContext applicationContext;
	private String beanName;
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    
	@Override
    public void setBeanName(String name) {
        this.beanName = name;
    }
    
    public void doSomthing() {
        // 通过ApplicationContext来获取Bean实例
        Stream.of(ctx.getBeanDefinitionNames())
            .forEach(System.out::println);
    }
}
~~~



### 事件机制

`ApplicationContext` 提供了一套事件机制，容器在特定条件时，会向实现 `ApplicationListener` 接口的类通知相应的ApplicationEvent事件。例如，`ApplicationContext` 在启动、停止、关闭和刷新时，分别会通知`ContextStartedEvent`、`ContextStoppedEvent`、`ContextClosedEvent` 和 `ContextRefreshedEvent` 事件。



SpringFramework 中内置的监听器接口是 `ApplicationListener`

~~~java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	void onApplicationEvent(E event);
}
~~~

我们要自定义监听器，只需要实现这个 `ApplicationListener` 接口即可。

~~~java
@Component
public class ContextClosedEventListener implements ApplicationListener<ContextClosedEvent> {
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        System.out.println("[ApplicationListener]ApplicationContext closed.");
    }
}
~~~

除了实现`ApplicationListener` 接口，还可以通过`@EventListener`来为特定事件注册回调函数：

~~~java
@Component
public class ContextClosedEventAnnotationListener {
    @EventListener
    // 通过参数类型指定要监听的数据
    public void onEvent(ContextClosedEvent event) {
        System.out.println("[@EventListener]ApplicationContext closed.");
    }
}
~~~

`@EventListener` 还有一些其他的用法，比如，在监听到事件后希望再发出另一个事件，这时可以将方法返回值从 `void` 修改为对应事件的类型

`@EventListener` 也可以与 `@Async` 注解结合，实现在另一个线程中处理事件。



我们还可以自定义事件，只需要继承`ApplicationEvent`即可

~~~java
public class CustomEvent extends ApplicationEvent {
    public CustomEvent(Object source) {
        super(source);
    }
}
~~~

然后，我们通过`ApplicationEventPublisher`来发布自定义事件即可

~~~java
@Component
public class CustomEventPublisher implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher publisher;

    public void fire() {
        publisher.publishEvent(new CustomEvent("Hello"));
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }
}
~~~



### 容器的扩展点

Spring Framework 中有很多机制是通过容器自身的扩展点来实现的，比如 **Spring AOP** 等。我们可以通过这些扩展点，来自定义一些功能。

`BeanPostProcessor` 接口是用来定制 Bean 的，该接口中有两个方法：

- `postProcessBeforeInitialization()` 方法在 Bean 初始化前执行，
- `postProcessAfterInitialization()` 方法在 Bean 初始化之后执行

如果有多个 `BeanPostProcessor`，可以通过 `Ordered` 接口或者 `@Order` 注解来指定运行的顺序

~~~java
@Configuration
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(Application.class);

        applicationContext.close();
    }

    @Bean
    public Hello hello() {
        return new Hello();
    }
	
    // 向 Spring 返回一个 Bean 处理器
    @Bean
    public HelloBeanPostProcessor helloBeanPostProcessor() {
        return new HelloBeanPostProcessor();
    }
}

class Hello {
    @PostConstruct
    public void init() {
        System.out.println("Hello PostConstruct");
    }

    @PreDestroy
    public void dispose() {
        System.out.println("Hello PreDestroy");
    }
    
    public String hello() {
        return "Hello World!";
    }
}


class HelloBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if ("hello".equals(beanName)) {
            System.out.println("Hello postProcessBeforeInitialization");
        }
        return bean;
    }


    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if ("hello".equals(beanName)) {
            System.out.println("Hello postProcessAfterInitialization");
        }
        return bean;
    }
}

/** output: 
 Hello postProcessBeforeInitialization
 Hello PostConstruct
 Hello postProcessAfterInitialization
 Hello PreDestroy
 */
~~~

此外，还有BeanFactoryPostProcessor接口，它是`BeanFactory` 的后置处理器，我们可以通过它来定制 Bean 的配置元数据。其中的 `postProcessBeanFactory()` 方法会在 `BeanFactory` 加载所有 Bean 定义但尚未对其进行初始化时（在`postProcessBeforeInitialization()`之前）介入

~~~java
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(Application.class);

        applicationContext.close();
    }

    @Bean
    static public HelloBeanPostProcessorFactory helloBeanPostProcessorFactory() {
        return new HelloBeanPostProcessorFactory();
    }

    @Bean
    public Hello hello() {
        return new Hello();
    }
}

class HelloBeanPostProcessorFactory implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition helloDefinition = beanFactory.getBeanDefinition("hello");
        System.out.println(helloDefinition);
    }
}
/**
Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=application; factoryMethodName=hello; initMethodName=null; destroyMethodName=(inferred); defined in learning.spring.helloworld.Application
*/
~~~

需要注意的是：

- 如果用 Java 配置类来注册，那么返回Bean的方法需要声明为 `static`
- 由于 Spring AOP 也是通过 `BeanPostProcessor` 实现的，因此实现该接口的类，以及其中依赖注入的 Bean 都会被特殊对待，**不会**被 AOP 增强。而且BeanPostProcessor所依赖注入的Bean而不会走`postProcessBeanFactory()`方法
- 此外，`BeanPostProcessor` 和 `BeanFactoryPostProcessor` 都仅对当前容器上下文的 Bean 有效，不会去处理其他上下文。

### 关闭容器

Java 进程在退出（正常退出、抛出异常、System.exit）时，我们可以通过 `Runtime.getRuntime().addShutdownHook()` 方法添加一些钩子，在关闭进程时执行特定的操作。

`ConfigurableApplicationContext` 接口扩展自 `ApplicationContext`，其中提供了一个 `registerShutdownHook()`。`AbstractApplicationContext` 类实现了该方法，正是调用了前面说到的 `Runtime.getRuntime().addShutdownHook()`，并且在其内部调用了 `doClose()` 方法。下面给出ApplicationContext的实现

~~~java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {
    @Override
    public void registerShutdownHook() {
        if (this.shutdownHook == null) {
            // No shutdown hook registered yet.
            this.shutdownHook = new Thread() {
                @Override
                public void run() {
                    synchronized (startupShutdownMonitor) {
                        doClose();
                    }
                }
            };
            // 也是通过这种方式来添加
            Runtime.getRuntime().addShutdownHook(this.shutdownHook);
        }
    }

    protected void doClose() {
        //...
    }
}
~~~

我们只要在创建AbstractApplicationContext时，覆写该方法即可：
~~~java
ConfigurableApplicationContext context = new AbstractApplicationContext() {
    @Override
    protected void doClose() { super.doClose(); }

    @Override
    protected void refreshBeanFactory() throws BeansException, IllegalStateException {}

    @Override
    protected void closeBeanFactory() {}

    @Override
    public ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException { return null; }
};
~~~



一个 Bean 通过 `ApplicationContextAware` 注入了 `ApplicationContext`，业务代码根据逻辑判断从 `ApplicationContext` 中取出对应名称的 Bean，再进行调用；问题出现在容器关闭时（`context.close()`），容器已经开始销毁 Bean 了，可是这段业务代码还在执行，仍在继续尝试从容器中获取 Bean，这该如何是好？

针对这种情况，我们可以实现 Spring Framework 提供的 `Lifecycle` 接口，让Bean感知容器的启动和停止。

~~~java
public class Hello implements Lifecycle {
    private boolean flag = false;

    public String hello() {
        return flag ? "Hello World!" : "Bye!";
    }

    @Override
    public void start() {
        System.out.println("Context Started.");
        flag = true;
    }

    @Override
    public void stop() {
        System.out.println("Context Stopped.");
        flag = false;
    }

    @Override
    public boolean isRunning() {
        return flag;
    }
}
~~~

除此之外，我们还可以借助 `Spring Framework` 的事件机制，让`Bean`实现`ApplicationListener`接口来监听容器关闭事件。

### 容器的抽象

Spring Framework 针对研发和运维过程中的很多常见场景做了抽象处理。

#### 环境抽象

虽然有 JVM 这层隔离，但我们的程序还是需要应对不同的运行环境细节：比如使用了 WebLogic 的某些特性，会导致程序很难迁移到 Tomcat 上；此外，程序还要面对开发、测试、预发布、生产等环境的配置差异；在云上，不同可用区（availability zone）可能也有细微的差异。Spring Framework 的环境抽象可以简化大家在处理这些问题时的复杂度，代表程序运行环境的 `Environment` 接口包含两个关键信息——`Profile` 和 `Properties`

##### Profile

假设我们的系统在测试环境中不需要加载监控相关的 Bean，而在生产环境中则需要加载；亦或者针对不同的客户要求，A 客户要求我们部署的系统直接配置数据库连接池，而 B 客户要求通过 JNDI 获取连接池。此时，就可以利用 Profile 帮我们解决这些问题。

如果使用 XML 进行配置，可以在 `<beans/>` 的 `profile` 属性中进行设置，如果使用 Java 类的配置方式，添加 `@Profile` 注解，并在其中指定该配置生效的具体 `Profile`

~~~java
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public Hello hello() {
        Hello hello = new Hello();
        hello.setName("dev");
        return hello;
    }
}

@Configuration
@Profile("test")
public class TestConfig {
    @Bean
    public Hello hello() {
        Hello hello = new Hello();
        hello.setName("test");
        return hello;
    }
}

@Configuration
public class DataSourceConfiguration {
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return null;
    }
    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return null;
    }
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        return null;
    }
}
~~~

通过如下两种方式可以指定要激活的 `Profile`（多个 Profile 用逗号分隔）：

- `ConfigurableEnvironment.setActiveProfiles()` 方法指定要激活的 `Profile`

  ~~~java
  AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(TavernConfiguration.class);
  // 给ApplicationContext的环境设置正在激活的profile
  ctx.getEnvironment().setActiveProfiles("city");
  ~~~

  这样写是错误的，因为`AnnotationConfigApplicationContext`只能刷新一次。推荐这样写

  ~~~java
  AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
  ctx.getEnvironment().setActiveProfiles("city");
  ctx.register(TavernConfiguration.class);
  ctx.refresh();
  ~~~

- `spring.profiles.active` 属性指定要激活的 `Profile`

- 启动程序时，在命令行中增加 `spring.profiles.active`：

  ~~~shell
  java -Dspring.profiles.active="dev" -jar xxx.jar
  ~~~

  

##### Properties

`PropertySource`对不同来源（系统属性、JVM属性、命令行参数、属性文件）的属性值做了抽象，以及**外部化配置**。我们可以通过以下两种方式获取属性值：

- `Environment`对象

  ~~~java
  @Component
  public class Hello {
      @Autowired
      private Environment environment;
  
      public void hello() {
          System.out.println("foo.bar: " + environment.getProperty("foo.bar"));
      }
  }
  ~~~

-  `@Value` 注解

  ~~~java
  @Value("${red.name}")
  private String name;
  
  @Value("${red.order}")
  private Integer order;
  
  // 这里使用了一个常量，并不是SpEL表达式
  @Value("1")				
  private Integer age;
  ~~~
  
  这里的@Value中用到了 SpEL 表达式。
  
  SpEL 的语法统一用 **`#{}`** 表示
  
  - 算术运算符：加（+）、减（-）、乘（*）、除（/）、求余 （%）、幂（^）、求余（MOD）和除（DIV）等算术运算符
  - 关系运算符：等于（==）、不等于（!=）、大于（>）、大 于等于（>=）、小于（<）、小于等于（<=）、区间（between）运算 等。例如：#{2>3}的值为false。
  - 逻辑运算符：与（and）、或（or）、非（!或NOT）
  - 字符串运算符：连接（+）和截取（[ ]）。例如：#{'Hello ' + 'World!'}的结果为“Hello World!”；#{'Hello World!'[0]} 截取第一个字符“H”
  - 三目运算符
  - 正则表达式匹配符： matches。例如：#{'123' matches '\\d{3}' }返回true
  - 类型访问运算符： T(Type)。其中，“Type”表示某个Java类型，实际上对应于Java类的 java.lang.Class实例。Type必须是类的全限定名（包括包名），但是 核心包“java.lang”中的类除外。例如：T(String)表示访问的是 java.lang.String类，#{T(String).valueOf(1)}表示将整数1转换成字符串。
  - 变量引用符：SpEL提供了一个上下文变量的引用符“#”
  
  
  
  
  
  我们可以在配置类上添加`@PropertySource("classpath:basic_di/value/red.properties")`，其中注解值为属性文件的路径。那么@Value可以从属性文件中获取属性值。

#### 任务抽象

Spring Framework 通过 `TaskExecutor` 和 `TaskScheduler` 这两个接口分别对任务的异步执行与定时执行进行了抽象

 `TaskExecutor`是在`java.util.concurrent.Executor`的基础上又做了一层封装。`TaskExecutor` 有很多实现，例如：

- `SyncTaskExecutor`
- `SimpleAsyncTaskExecutor`
- `ConcurrentTaskExecutor`
- `ThreadPoolTaskExecutor`

我们可以像下面这样直接配置一个 `ThreadPoolTaskExecutor`：

~~~xml
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
    <property name="corePoolSize" value="4"/>
    <property name="maxPoolSize" value="8"/>
    <property name="queueCapacity" value="32"/>
</bean>

<!--等价配置-->
<task:executor id="taskExecutor" pool-size="4-8" queue-capacity="32"/>
~~~

在配置好了 `TaskExecutor` 后

- 在依赖注入后，可以直接调用它的 `execute()` 方法，并传入一个 `Runnable` 对象

- 在方法上使用 `@Async` 注解

  ~~~java
  @Async("taskExecutor")
  public void runAsynchronous() {...}
  ~~~

  为了让该注解生效，需要在配置类上增加 `@EnableAsync` 注解，或者在 XML 文件中增加 `<task:annotation-driven/>` 配置。



`TaskScheduler`对定时任务有着很好的支持。

- 在依赖注入后，可以直接调用它的schedule()方法

- 也可以使用@Scheduled注解

  ~~~java
  @Scheduled(fixedRate=1000) // 每隔1000ms执行
  public void task1() {...}
  
  @Scheduled(fixedDelay=1000) // 每次执行完后等待1000ms再执行下一次
  public void task2() {...}
  
  @Scheduled(initialDelay=5000, fixedRate=1000) // 先等待5000ms开始执行第一次，后续每隔1000ms执行一次
  public void task3() {...}
  
  @Scheduled(cron="0 15 15 * * 1-5") // 按Cron表达式执行
  public void task4() {...}
  ~~~

  为了让该注解生效，需要在Java配置类上增加 `@EnableScheduling` 注解，或者在 XML 文件中增加 `<task:annotation-driven/>` 配置。



## 模块装配

定义EnableXXXX注解

~~~java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface EnableTavern {

}
~~~

在EnableXXXX注解上，添加`@Import`注解：

~~~java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import
public @interface EnableTavern {

}
~~~

在配置类上添加该注解：

~~~java
@Configuration
@EnableTavern
public class TavernConfiguration {
    
}
~~~

这样，通过`@import`注册的类，可以添加到该配置类上



`@Import`源码：

~~~java
@Documented
public @interface Import {
	/**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
	Class<?>[] value();
}
~~~

注册的四种方式：

- 在@Import接口中，直接导入类，或者配置类

  ~~~java
  @Import({Boss.class, BartenderConfiguration.class})
  ~~~

- 编写一个实现`ImportSelector` 接口的类：

  ~~~java
  public class BarImportSelector implements ImportSelector {
      
      @Override
      public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          // 返回要导入类的全限定类名
          return new String[] {Bar.class.getName(), BarConfiguration.class.getName()};
      }
  }
  
  ~~~

  然后在`@Import`注解中，注册该`ImportSelector`实现类即可

  ~~~java
  @Import(BarImportSelector.class)
  ~~~

- 编写一个实现`ImportBeanDefinitionRegistrar` 接口的类：

  ~~~java
  public class WaiterRegistrar implements ImportBeanDefinitionRegistrar {
      
      @Override
      public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
          // 通过BeanDefinitionRegistry注册Bean
          registry.registerBeanDefinition("waiter", new RootBeanDefinition(Waiter.class));
      }
  }
  ~~~
  
  然后在`@Import`注解中，注册该`ImportBeanDefinitionRegistrar`实现类即可
  
  ~~~java
  @Import({WaiterRegistrar.class})
  ~~~

## 条件装配

条件装配的一个注解是`@Profile`，在环境抽象一节中介绍过，不再阐述。

另一个是`@Conditional`注解。被 `@Conditional` 注解标注的组件，只有所有指定条件都匹配时，才有资格注册。如果 `@Configuration` 配置类被 `@Conditional` 标记，则与该类关联的所有 `@Bean` 的工厂方法，`@Import` 注解和 `@ComponentScan` 注解也将受条件限制。

`@Conditional` 注解中需要传入`Condition` 实现类的数组

~~~java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
~~~

~~~java
@FunctionalInterface
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
~~~



@Conditional注解可以派生出来。扫描注解A时，A的派生注解也会被扫描进来。

~~~java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Conditional(OnBeanCondition.class)
public @interface ConditionalOnBean {
    String[] beanNames() default {};
}
~~~

在@Conditional中指定Condition实现类，Condition实现类可以获取派生注解中的额外信息

~~~java
public class OnBeanCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取派生注解，并读取派生注解中的beanNames元素值。
        String[] beanNames = (String[]) metadata
            .getAnnotationAttributes(ConditionalOnBean.class.getName())
            .get("beanNames");
        return true;
    }
}
~~~

~~~java
@Bean
@ConditionalOnBean(beanNames = "com.linkedbear.spring.configuration.c_conditional.component.Boss")
public Bar bbbar() {
    return new Bar();
}
~~~



## 依赖查找 & 依赖注入

- 依赖查找

  - `getBean` ：根据 **name** 获取 / 根据 **Class** 获取指定的 bean

  - `ofType` ：根据 **Class** 获取容器中所有指定类型的 bean

  - `withAnnotation` ：获取标注了指定注解的 bean

  - `getBeanDefinitionNames` ：获取容器中的所有 bean 的 name

  - `getBeanProvider` ：延迟查找，先获取 `ObjectProvider` 后获取实际的对象，如果不存在可使用缺省值代替

- 依赖注入


  - xml 配置文件
    - 借助 `<property>` 标签给带有 setter 方法的属性赋值 / 注入依赖对象
    - 借助 `<constructor-arg>` 标签使用 bean 的构造器注入依赖对象 / 属性赋值
    
    
    - 注解驱动方式
      - 使用 `@Value` 给普通属性赋值
      - 使用 `@Autowired` / `@Resource` / `@Inject` 注解给组件依赖注入
      - 借助 `ObjectProvider` 可以实现组件延迟注入


  - 借助 `Aware` 系列接口实现回调注入





- 依赖注入通常借助一个上下文**被动**的接收（标注 `@Autowired` 注解 / `<property>` 标签配置），推荐可以做到解耦
- 依赖查找通常**主动**使用上下文搜索（拿到 `BeanFactory` / `ApplicationContext` 之后主动调用 `getBean` 方法）
