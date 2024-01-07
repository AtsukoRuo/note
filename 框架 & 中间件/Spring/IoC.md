# IoC

[TOC]

## 初步

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



## 容器 & 控制反转

**控制反转（Inversion of Control，IoC）**、**依赖注入（Dependency Injection）**与**面向切面编程（Aspect Oriented Programming，AOP）**是 Spring Framework 中最重要的概念。

首先我们来介绍下**「控制反转」（Inversion Of Control）**。这里的“控制”指的是对程序执行流程的控制，而“反转”指的是调用者与被调用者之间的控制权转移。例如，在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程可以通过框架来控制（模板方法）。流程的控制权从程序员“反转”到了框架。

**依赖注入（Dependency Injection）**：不通过new()的方式在类内部创建依赖类对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（注入）给类使用。也就是说，将对象的创建**控制反转**给上层来处理。读者可能认为这样做破坏了封装性，实则不然，因为仅仅是将私有状态的初始化操作交给调用者来负责而已，并未暴露该状态的修改方法。

在实际的软件开发中，一些项目可能会涉及几十、上百、甚至几百个类，类对象的创建、依赖注入、组件之间依赖关系的处理会变得非常复杂。而这些工作跟具体的业务逻辑是无关的，此时我们完全可以抽象成框架来自动完成，从而减少手动装配出错的可能性。Spring Framework 的 IoC 容器正是基于依赖注入的思想，将组件内部的依赖管理、生命周期管理的逻辑抽离出来，从而让开发人员专注于业务逻辑的实现。

<img src="https://www.ituring.com.cn/figures/2023/LearnSpring/009.jpg" alt="{%}" style="zoom:10%;" />

容器初始化的大致步骤如下：

1. 从 XML 文件、Java 类或其他地方加载配置元数据。
2. 通过 `BeanFactoryPostProcessor` 对配置元数据进行一轮处理。
3. 初始化 Bean 实例，并根据给定的依赖关系组装对象。
4. 通过 `BeanPostProcessor` 对 Bean 进行处理，期间还会触发 Bean 的回调方法。



为了使用 Spring 的容器，需要在 pom.xml 文件中引入 `org.springframework:spring-beans` 依赖：

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.3.15</version>
</dependency>
~~~

然后，在resources文件夹下创建 beans.xml 配置文件：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hello" class="learning.spring.helloworld.Hello" />

</beans>
~~~

最后我们在代码中，将配置文件载入到容器中，随后就可以通过容器获取到Bean对象了

~~~java
public class Application {
	private BeanFactory beanFactory;			// Bean容器，用于获取Bean对象

	public static void main(String[] args) {
		Application application = new Application();
		application.sayHello();
	}

	public Application() {
		beanFactory = new DefaultListableBeanFactory();			// 创建一个Bean容器
        // 下面两行代码就是通过XmlBeanDefinitionReader对象将配置文件加载到beanFactory中
		XmlBeanDefinitionReader reader =
			new XmlBeanDefinitionReader((DefaultListableBeanFactory) beanFactory);
		reader.loadBeanDefinitions("beans.xml");
	}

	public void sayHello() {
		// Hello hello = (Hello) beanFactory.getBean("hello");
        
        // 获取Bean对象
		Hello hello = beanFactory.getBean("hello", Hello.class);
		System.out.println(hello.hello());
	}
}
~~~

这里我们看到获取Bean对象的两种方法

- 获取 `Object` 类型的 Bean，然后自己做类型转换
- 在参数里指明返回 Bean 的类型



`BeanFactory` 是容器的基础接口。`ApplicationContext` 接口继承了 `BeanFactory`，在它的基础上增加了更多企业级应用所需要的特性。常见的 `ApplicationContext` 实现如下：

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
    private ApplicationContext applicationContext;

    public static void main(String[] args) {
        Application application = new Application();
        application.sayHello();
    }

    public Application() {
        // 直接读取配置文件到容器中，比DefaultListableBeanFactory 和 XmlBeanDefinitionReader 的组合要简洁
        applicationContext = new ClassPathXmlApplicationContext("beans.xml");
    }

    public void sayHello() {
        Hello hello = applicationContext.getBean("hello", Hello.class);
        System.out.println(hello.hello());
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

通过容器的 `setAllowBeanDefinitionOverriding()` 方法可以关闭覆盖同ID Bean的特性



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

`id`属性唯一地标识某个Bean对象。`name`属性用于定义Bean实例的别名(aliases)，一个Bean实例可以有多个别名，在name属性中用`,`或`;`分隔各个别名即可：

~~~xml
<bean name="map,java_map;jdk_map" class="java.util.HashMap" />
~~~

如果一个bean标签未指定id属性，那么将class属性中的「全限定类名」作为bean的默认id。如果有多个bean未指定id，而且class属性值相同，那么会按照其出现的次序，分别给其的id设置为 “全限定类名#1”, “全限定类名#2”

在同一个配置文件中，任意Bean对象之间的id、name属性值是禁止重复的

|                           非法配置                           |                   含义                   |
| :----------------------------------------------------------: | :--------------------------------------: |
| `<bean id="map" class="java.util.HashMap" />`<br>`<bean id="map" class="java.util.HashMap" />` |            两个bean的id值重复            |
| `<bean name="map" class="java.util.HashMap" />` <br>`<bean name="map" class="java.util.HashMap" />` |           两个bean的name值重复           |
| `<bean id="map" class="java.util.HashMap" />`<br/>`<bean name="map" class="java.util.HashMap" />` | 第一个bean的id值与第二个bean的name值重复 |



### XML Bean的依赖注入

Spring 容器在确定Bean之间的依赖关系后，就开始依赖注入。有两种基本的注入方式：

- 基于构造方法的注入
- 基于Setter方法的注入

#### 基于构造方法的注入

~~~java
public class Hello {
    private String name;
    public Hello(String name) {
        this.name = name;
    }
    public String hello() {
        return "Hello World! by " + name;
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

#### 基于Setter方法的注入

通过`<property/>`标签，我们可以基于Setter方法来依赖注入。

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



#### 自动装配

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

-  `<bean/>` 的 `autowire-candidate` 属性设置为 `false`
- 将某一个候选 Bean 中的`<bean/>` 中的 `primary`属性 设置为 `true`（@Primary注解）



如何指定 Bean 的初始化顺序？

- 一般情况下，Spring 容器会根据依赖关系来决定 Bean 的初始化顺序。不过，有时 Bean 之间的依赖关系是循环的，容器可能无法按照我们的预期进行初始化。此时，我们可以通过`<bean/>` 的 `depends-on` 属性来指定当前 Bean 还要依赖哪些 Bean，来确定Bean的依赖顺序（`@DependsOn` 注解）



### Bean 的三种配置方式

Spring Framework 提供了多种不同风格的配置方式：

- 基于XML文件的配置

  1. 在XML中编写Bean标签
  2. 在代码中创建BeanFactory

- 基于注解的配置，是配合Bean XML或者Java配置类来一起使用的

  - Bean XML：`<context:component-scan base-package="learning.spring"/>`
  - Java配置类：`@ComposeScan`

- Java配置类一般和SpringBoot配合使用，兼容XML以及注解

  - `@Bean`
  - `@ComposeScan`，兼容注解
  - `@ImportResource`，兼容XML

  

#### 基于 XML 文件的配置

~~~xml
<bean id="..." class="..." scope="singleton" lazy-init="true" depends-on="xxx" factory-method="create"/>
~~~



#### 基于注解的配置

我们需要在xml文件中写入以下配置，来启用基于注解的配置：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="learning.spring"/>
</beans>
~~~

> `<context:component-scan/>` 会隐式地配置 `<context:annotation-config/>`，后者其实替我们注册了一堆 `BeanPostProcessor`。

上述配置会扫描 `learning.spring` 包内的类。对于添加以下四个注解的类，Spring 容器把它们注册为 `Bean`对象

| 注解          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| `@Component`  | 将类标识为普通的组件，即一个 Bean                            |
| `@Service`    | 将类标识为服务层的服务                                       |
| `@Repository` | 将类标识为数据层的数据仓库，一般是 DAO（Data Access Object） |
| `@Controller` | 将类标识为 Web 层的 Web 控制器（后来针对 REST 服务又增加了一个 `@RestController` 注解） |

在Spring Boot中，如果一个@Component类有多个构造器，Spring会尝试选择一个最合适的构造器来实例化该类：

- 如果该类中只有一个构造器，Spring会使用该构造器来实例化该类。
- 如果该类中有多个构造器，Spring会首先尝试使用默认构造器（即无参构造器）来实例化该类。如果没有默认构造器，或者默认构造器不适合当前的情况，Spring会优先选择带有@Autowired注解的构造器（如果有的话），并将其用于实例化该类。
- 如果有多个构造器都带有@Autowired注解，Spring会选择参数数量最多的构造器来实例化该类
- 如果有多个构造器的参数数量相同，则Spring会引发异常，因为无法确定应该使用哪个构造器。



如果要注入依赖，可以使用如下的注解：

| 注解         | 说明                                                      |
| :----------- | :-------------------------------------------------------- |
| `@Autowired` | 根据类型注入依赖，可用于构造方法、`Setter` 方法和成员变量 |
| `@Resource`  | JSR-250 的注解，根据名称注入依赖                          |
| `@Inject`    | JSR-330 的注解，同 `@Autowired`                           |

@Autowired可以作用在构造函数、属性、setter上：

~~~java
@Service
public class ConstructorServiceImpl implements ConstructorService {
    // 1.属性注入
    @Autowired
    private UserService userService;
    
    // 2.构造函数注入（推荐）
    @Autowired
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
   - 如果某个Bean使用了@Primer注解，那么就匹配该Bean。
   - 如果被注入对象使用了@Qualifier注解，那么优先根据Bean对象上配置的标识符（Qualifier）来匹配，而 Bean 的 ID 正好是标识符的默认值。
3. 否则报错



我们需要注意：

1. 注入的顺序：1.构造器  2. Setter 3. 属性

2. 只有@Autowired构造器才能对final变量进行注入。

3. @Autowired默认是**byType**的。@Resource默认是**ByName**的，若ByName失败，则通过ByType



#### 基于 Java 类的配置

要使用基于Java类的配置，就要与Spring Boot 一起使用。`Spring`配置类的定义例子：

```java
@Configuration
@ComponentScan("learning.spring")
public class Config {
    @Bean
    @Lazy
    @Scope("prototype")
    public Hello helloBean() {
        return new Hello();
    }
}
```

`@Configuration` 注解表明这是一个 Java 配置类。`@ComponentScan` 注解将指定包下的Bean对象（@Component等注解的类）添加到当前配置类中（兼容基于注解的配置）。 

`@Bean` 注解表示该方法的返回对象会被当做容器中的一个 Bean，`@Lazy` 注解说明这个 Bean 是延时加载的，这里的`@Scope` 注解则指定了 `Bean`是原型的。`@Bean` 注解的属性如下：

| 属性                | 默认值                                | 说明                      |
| :------------------ | :------------------------------------ | :------------------------ |
| `name`              | `{}`                                  | Bean 的名称，默认同方法名 |
| `value`             | `{}`                                  | 同 `name`                 |
| `autowire`          | `Autowire.NO`                         | 自动织入方式              |
| `autowireCandidate` | `true`                                | 是否是自动装配的候选 Bean |
| `initMethod`        | `""`                                  | 初始化方法名              |
| `destroyMethod`     | `AbstractBeanDefinition.INFER_METHOD` | 销毁方法名                |

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
        return new Baz(foo());		 // 方式2
    }
}
~~~

Spring Framework 针对 `@Configuration` 类中带有 `@Bean` 注解的方法通过 CGLIB（Code Generation Library）做了特殊处理。对于返回单例Bean的方法，只会执行一次，之后的多次调用都直接返回相同的Bean对象，并不会执行该方法。



在配置类中也可以导入其他配置，例如，用 @Import 导入其他配置类，用 @ImportResource 导入配置文件

~~~xml
@Configuration
@Import({ CongfigA.class, ConfigB.class })
@ImportResource("classpath:/spring/*-applicationContext.xml")
public class Config {

}
~~~



Spring可能会匹配到多个Bean，此时这些Bean统称为**候选者（candidates）。**可以使用`@Primary`注解或者`@Qualifier` 注释来决定在冲突时使用哪一个Bean。优先级@Qualifier > @Primary 。**一般来说通过Primary来指定默认行为，而通过@Qualifier来指定特定行为。**

这里再次强调`@Qualifier`实际上匹配的是`Bean`对象的`Qualifier`，而`Bean ID`正好是`Qualifier`的默认值。

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
    

  注意，要在对应的 XML 文件中配置 `destroy-method`，需要用 `<context:annotation-config />` 开启注解支持。
  
  

无论是初始化还是销毁，Spring 都会按照如下顺序依次进行调用：

1. 添加了 `@PostConstruct` 或 `@PreDestroy` 的方法；

2. 实现了 `InitializingBean` 的 `afterPropertiesSet()` 方法，或 `DisposableBean` 的 `destroy()` 方法；

3. 在 `<bean/>` 中配置的 `init-method` 或 `destroy-method`，`@Bean` 中配置的 `initMethod` 或 `destroyMethod`。

### Aware接口

可以通过实现 `BeanFactoryAware` 或 `ApplicationContextAware` 接口，来让Bean获取容器中的信息：

~~~java
@Component
public class MyBean implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
	
    public void doSomthing() {
        // 通过ApplicationContext来获取其他的Bean实例或其他的组件
    }
}
~~~

MessageSourceAware接口可以让Bean获取到MessageSource对象

ResourceLoaderAware接口可以让Bean获取到ResourceLoader对象，通过这个对象，我们可以方便地进行资源加载操作。常见的应用场景包括：

- 加载配置文件
- 加载图片等静态资源

还有其他的Aware接口，再次就不再列出

### 事件机制

`ApplicationContext` 提供了一套事件机制，容器在特定条件时，会向实现 `ApplicationListener` 接口的类通知相应的ApplicationEvent事件。例如，`ApplicationContext` 在启动、停止、关闭和刷新时，分别会通知`ContextStartedEvent`、`ContextStoppedEvent`、`ContextClosedEvent` 和 `ContextRefreshedEvent` 事件：

~~~java
@Component
@Order(1)
public class ContextClosedEventListener implements ApplicationListener<ContextClosedEvent> {
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        System.out.println("[ApplicationListener]ApplicationContext closed.");
    }
}
~~~

除了实现`ApplicationListener` 接口，还可以通过@EventListener来为特定事件注册回调函数：

~~~java
@Component
public class ContextClosedEventAnnotationListener {
    @EventListener
    @Order(2)
    public void onEvent(ContextClosedEvent event) {
        System.out.println("[@EventListener]ApplicationContext closed.");
    }
}
~~~

我们还可以自定义事件，只需要继承ApplicationEvent即可

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

`@EventListener` 还有一些其他的用法，比如，在监听到事件后希望再发出另一个事件，这时可以将方法返回值从 `void` 修改为对应事件的类型

`@EventListener` 也可以与 `@Async` 注解结合，实现在另一个线程中处理事件。

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

虽然有 JVM 这层隔离，但我们的程序还是需要应对不同的运行环境细节：比如使用了 WebLogic 的某些特性会导致程序很难迁移到 Tomcat 上；此外，程序还要面对开发、测试、预发布、生产等环境的配置差异；在云上，不同可用区（availability zone）可能也有细微的差异。Spring Framework 的环境抽象可以简化大家在处理这些问题时的复杂度，代表程序运行环境的 `Environment` 接口包含两个关键信息——`Profile` 和 `Properties`

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
~~~

通过如下两种方式可以指定要激活的 `Profile`（多个 Profile 用逗号分隔）：

- `ConfigurableEnvironment.setActiveProfiles()` 方法指定要激活的 `Profile`
- `spring.profiles.active` 属性指定要激活的 `Profile`

例如，启动程序时，在命令行中增加 `spring.profiles.active`：

~~~xml
▸ java -Dspring.profiles.active="dev" -jar xxx.jar
~~~



**PropertySource**对不同来源的属性值（系统属性、JVM属性、命令行参数）做了抽象，我们可以通过以下两种方式获取属性值：

- `Environment`对象

  ~~~java
  public class Hello {
      @Autowired
      private Environment environment;
  
      public void hello() {
          System.out.println("foo.bar: " + environment.getProperty("foo.bar"));
      }
  }
  ~~~

-  `@Value` 注解

  - @Value(“常量”) 常量,包括字符串,网址,文件路径等

  - @Value(“${}”) 读取配置文件

    ~~~java
    @Configuration
    @PropertySource("classpath:/META-INF/resources/app.properties")
    public class Config {
        @Value("${foo.bar:NONE})
        private String value
    }
    ~~~

    如果不指定`@PropertySource`，那么就从默认配置文件`application.yml`中读取

  - `@Value(“#{}”)` 读取某个`bean`的属性

  

  > Spring Framework通过`PropertySourcesPlaceholderConfigurer`（继承自`BeanFactoryPostProcessor`）来实现占位符解析的。如果使用 `<context:property-placeholder/>`，Spring Framework 会自动注册一个 `PropertySourcesPlaceholderConfigurer`。在它的 `postProcessBeanFactory()` 方法中，会用查找到的属性值来替换上下文中的对应占位符

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

