# Spring Framework 02

![image-20230704092739559](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704092739559.png)





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





Spring框架分为多个Spring project，包括Spring Famework、SpringBoot、Spring JPA等。每个Spring poject又分为多个Spring Moudules。应用程序可以按需使用这些Moudules，这有很大的灵活性。

![image-20230704141946297](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704141946297.png)



