# Spring Framework Core

https://start.spring.io/ 创建SpringBoot项目

框架与设计模式的出现就是为了**抵抗变化**。可以通过增加中间层，降低程序的耦合度，来尽可能地隔离变化所带来的影响。而Spring Framework就是一个中间层，让开发者专注于业务逻辑的开发，而无需关心请求处理等与业务无关的部分。

> 注：可以通过衡量修改程序的某一项功能所花费的时间、资金来定义代码的耦合度。



`Spring IoC`通俗点讲，就是负责对象的创建、处理对象之间的依赖关系以及管理对象生命周期的构件，即将业务逻辑代码与管理对象生命周期代码进行解耦。



Spring Container、IoC Container、 Spring Context都是一个东西。它们接受Pojo对象以及Config（配置类或配置文件），输出一个就绪的运行时系统。Spring Container有两种类型：

- Bean Factory：basic spring container
- Application Context（推荐使用）：advanced spring container with enterprise-specific features，例如国际化、与Spring AOP的整合



POJO（Plain Old Java Object）：没有任何限制。任何Java对象都是一个POJO

JavaBean（也称为Enterprises Java Bean，EJB），它具有以下限制：

- 一个public无参构造器
- 每个成员为私有的，而且必须提供getter以及setter
- 实现Serializable接口，该接口中什么也没有

Spring Bean：由Spring Container管理的Java对象







Spring配置类：

~~~java
@Configuration
public class GamingConfiguration {
    @Bean
    public GamingConsole game() {
        var game = new PacMan();
        return game;
    }

    @Bean
    public GameRunner gameRunner(GamingConsole game) {
        var gameRunner = new GameRunner(game);
        return gameRunner;
    }
}
~~~

Spring Container：

~~~java
public class App03GamingSpringBeans {
    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(GamingConfiguration.class);) {
            context.getBean(GamingConsole.class);		//获取返回类型为GamingConsole（会考虑向上转型）的Bean对象
            context.getBean(GameRunner.class);
            context.getBean("person3Parameters");	//获取名为person3Parameters的Bean对象
        }
    }
}
~~~

Bean的名字默认与方法名同名，可以通过@Bean(name="")注解来修改Bean的名字。



Spring可能会匹配到多个Bean，此时这些Bean统称为**候选者（candidates）。**可以使用`@Primary`注解或者`@Qualifier` 注释来决定在冲突时使用哪一个Bean。优先级@Qualifier > @Primary 。

~~~java
@Bean
public Person person3Parameters(String name, int age, Address address3) { 
    ////调用address()，因为通过@Primary来指定的
    return new Person(name, age, address3);
}

@Bean
@Primary
public Person person4Parameters(String name, int age, @Qualifier("address3qualifier") Address address) {	//抵用address3()，因为通过@Qualifier来指定的
    return new Person(name, age, address);
}

@Bean(name="address2")
@Primary
public Address address() {
    return new Address("Baker Street", "London");
}

@Bean(name="address3")
@Qualifier("address3qualifier")
public Address address3() {
    return new Address("Motinagar", "London");
}


beanFactor.getBean(Address.class);		//调用address()，因为通过@Primary来指定的
~~~



注意：person3Parameters中的Address address参数使用了**自动装配（autowire）**。在创建Bean时，Spring会自动调用返回类型为Address（考虑向上转型）的Bean，并将返回值赋值到address变量中。







@Component注解为一个类自动生成对应的Bean，再在配置类中使用@ComponentScan添加这些类，此时可以通过context.getBean(Class class)来获取Bean对象。

@Component扫描与配置类同包的类

@Component("package")扫描指定类中的类

~~~java
package cn.atsukoruo.learnspringframework.game;
@Component
public class GameRunner  { }

@Component
public class PacMan implements GamingConsole{ }

/*****************************************************/
package cn.atsukoruo.learnspringframework;
@Configuration
@ComponentScan("cn.atsukoruo.learnspringframework.game")
public class GamingAppLauncherApplication {
    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(
                GamingAppLauncherApplication.class
        )) {
            context.getBean(GamingConsole.class).up();
            context.getBean(GameRunner.class).run();
        }
    }
}

//等价于
/**************************************************/
@Configuration
public class GamingAppLauncherApplication {
    @Bean 
    GamingConsole getGameingConsole() {
        return GamingConsole();
    }
    
    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(
                GamingAppLauncherApplication.class
        )) {
            context.getBean(GamingConsole.class).up();
            context.getBean(GameRunner.class).run();
        }
    }
}
~~~

再次提醒，这里getBean(GamingConsole.class)会考虑GamingConsole的子类型



此时@Component类中的成员可以通过@Autowire来自动装配

~~~java
@Component 
Class A{
    @Autowire
    private B b;		//注意：B类也用@Component来注解，而且在配置类中@ComponentScan中包括进来
}
~~~





@Component也会遇到冲突问题。例如

~~~java
@Component
public class PacMan implements GamingConsole{ }

@Component
public class LOL implements GamingConsole { }

context.getBean(GamingConsole.class);		//错误，多个匹配
~~~

为此可以通过@Primary或者@Qualifier来解决这个问题

![image-20230610160617125](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230610160617125.png)

@Autowired ：使用@Primary注解的类，即使用默认的类

@Autowired + Qualifier：使用指定的类



前面我们或多或少地使用了依赖注入（autowire）

依赖注入的三种类型：

- Constructor-based（十分推荐！）
- Setter-based
- Field：No setter or constructor. Dependency is injected using reflection

~~~java
@Component
class YourBusinessClass {
    //Field Injection
    //@Autowired
    Dependency1 dependency1;
    //@Autowired
    Dependency2 dependency2;
    //@Autowired  or not use
    public YourBusinessClass(Dependency1 dependency1, Dependency2 dependency2) {
        super();
        System.out.println("Constructor Injection - YourBusinessClass");
        this.dependency2 = dependency2;
        this.dependency1 = dependency1;
    }
   //@Autowired
    public void setDependency1(Dependency1 dependency1) {
        System.out.println("Setter Injection - setDependency1");
        this.dependency1 = dependency1;
    }
    //@Autowired
    public void setDependency2(Dependency2 dependency2) {
        System.out.println("Setter Injection - setDependency2");
        this.dependency2 = dependency2;
    }
	//@Autowired
    public String toString() {
        return "Using " + dependency1 + " and " + dependency2;
    }
}

@Component
class Dependency1 {}

@Component
class Dependency2 {}
~~~



![image-20230704090555454](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704090555454.png)



在Spring Boot中，如果一个@Component类有多个构造器，Spring会尝试选择一个最合适的构造器来实例化该类：

- 如果该类中只有一个构造器，Spring会使用该构造器来实例化该类。

- 如果该类中有多个构造器，Spring会首先尝试使用默认构造器（即无参构造器）来实例化该类。如果没有默认构造器，或者默认构造器不适合当前的情况，Spring会优先选择带有@Autowired注解的构造器（如果有的话），并将其用于实例化该类。

- 如果有多个构造器都带有@Autowired注解，Spring会选择参数数量最多的构造器来实例化该类
- 如果有多个构造器的参数数量相同，则Spring会引发异常，因为无法确定应该使用哪个构造器。

可以使用@Configuration注解的类中的@Bean方法，指定一个构造器来创建该类的实例。



为什么我们需要依赖？

![image-20230704090837039](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704090837039.png)



一个例子：

![image-20230704090919447](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704090919447.png)







SpringBean的初始化方式默认是Eager，即在启动阶段执行Bean对象的初始化。但是添加@Lazy后，Spring框架在启动阶段并不会初始化Bean对象，只有在第一次使用时（例如getBean，Autowired）才会执行初始化。

~~~java
@Component
@Lazy
class ClassB {
    public ClassB(ClassA classA) { System.out.println("Some Initialization logic"); }
}

@Configuration
@ComponentScan
public class LazyInitializationApplication {
    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext((LazyInitializationApplication.class))
        ) {
            
        }
    }
}
~~~

推荐使用默认初始化方式，这样错误在启动阶段就可以报告，而@Lazy只会在运行时报告出来。@Lazy唯一的好处就是减少内存的使用量直到Bean对象被初始化。





在SpringBoot中，所有Bean的获取默认都是单例模式，即任何时刻都只有一个Bean对象。可以通过@Scope注解改变这一行为。通过@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)，将对Bean的获取改为原型模式，即每次获取都是不同的Bean对象。通常来说Prototype对应有状态（Stateful）的Bean对象，而Singleton对象无状态（Stateless）的Bean对象

~~~java
@Scope
@Component
class NormalClass {

}

@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Component
class PrototypeClass {

}
@Configuration
@ComponentScan
public class BeanScopesLauncherApplication {
    public static void main(String[] args) {
        try (
                var context = new AnnotationConfigApplicationContext(BeanScopesLauncherApplication.class)
                ) {
            System.out.println(context.getBean(PrototypeClass.class));
            System.out.println(context.getBean(PrototypeClass.class));
            System.out.println(context.getBean(PrototypeClass.class));
            System.out.println(context.getBean(NormalClass.class));
            System.out.println(context.getBean(NormalClass.class));
            System.out.println(context.getBean(NormalClass.class));
        }
    }
}

/** output:
	PrototypeClass@1a4013
	PrototypeClass@1b6e1eff
	PrototypeClass@306f16f3
	NormalClass@702b8b12
	NormalClass@702b8b12
	NormalClass@702b8b12
*/
~~~

![image-20230704100922606](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704100922606.png)







~~~java
@Component
class SomeClass {
    public SomeClass(SomeDependency someDependency) {System.out.println("All dependencies are ready!");}
    
    @PostConstruct
    public void initialize() {System.out.println("initialize");}
    
    @PreDestroy
    public void cleanUp() {System.out.println("Cleanup");}
}
~~~

@PostConstruct注释的方法，会在Bean对象创建之后立即执行（@Lazy、@Scope的规则与之兼容）。而@PreDestroy注释的方法，会在容器将Bean销毁之前调用。但是如果Bean对象是原型创建的，那@PreDestroy注释的方法并不会执行





![image-20230704105348677](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704105348677.png)

你只需意识到JavaEE、J2EE、JakartaEE实际上都是一个东西。Spring框架与JakartaEE是竞争关系，但是Spring框架实现了Jakarta中的CDI标准。其中在CDI中，@Named代替@Component，而@Inject代替@Autowired。







一个XML的例子：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"> <!-- bean definitions here -->
    <bean id="age" class="java.lang.Integer">
        <constructor-arg value="1" />
    </bean>
	<context:component-scan base-package="cn.atsukoruo.learnspringframework.game" />	<!--将指定包下的@Component类添加进来-->
    
    <bean id="game" class="cn.atsukoruo.learnspringframework.game.PacMan" />
    <bean id="gameRunner2" class="cn.atsukoruo.learnspringframework.game.GameRunner">
        <constructor-arg ref="game" />	<!--value适用于指定基本类型值，而ref适用于指定对象引用-->
    </bean>
</beans>
~~~

~~~java
var context = new AnnotationConfigApplicationContext((LazyInitializationApplication.class))
~~~



XML与代码耦合度低，但是维护成本极高。推荐使用注解



![image-20230704114442000](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704114442000.png)

这些特殊的@Component为Spring框架提供更多的信息，可以使用更多的特性。





Spring框架分为多个Spring Project，包括但不限于Spring Famework、SpringBoot、Spring JPA。每个Spring poject又分为多个Spring Moudules。开发人员可以按需使用Moudules。

通过引入新的Spring Project来推动Spring框架的演化

![image-20230704141946297](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704141946297.png)

