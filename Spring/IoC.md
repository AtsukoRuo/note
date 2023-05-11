# IoC

[TOC]

## IoC容器基本知识

**控制反转（Inversion of Control，IoC）**是一种决定容器如何装配组件的**模式**。这里所谓的**容器**，就是对**组件**（Bean，也就是业务对象）进行管理（例如，创建、销毁、维护依赖关系）的地方。

需要指出的一点是：组件之间的依赖关系原先是由组件自己定义的，并在其内部维护，而现在这些依赖被定义在容器中，由容器统一管理，并将其依赖的内容注入到组件中（这也被称为**依赖注入**，**Dependency Injection**）。

> Spring Framework、Google Guice等框架都提供了这样的容器。这里我们将Spring Framework的IoC容器称为Spring容器

![container magic](assets/container-magic.png)



容器初始化步骤如下：

1. 从XML、Java类或其他地方加载配置元数据
2. 通过`BeanFactoryPostProcessor`对配置元数据处理
3. 初始化`Bean`实例，并根据给定的依赖关系组装对象
4. 通过`BeanPostProcessor`对Bean进行处理



下面给出一个简单的例子来说明容器如何管理对象的：

~~~java
// Hello.java
public class Hello {
    public String hello() {
        return "Hello World!";
    }
}

// Application.java
public class Application {
    public static void main(String[] args) {
        Hello hello = new Hello();		//必须自己维护对象之间的关系
        System.out.println(hello.hello());
    }
}
~~~

可以将`Hello`类交给`Spring`容器托管。

首先需要配置元数据，我们通过XML文件定义配置元数据。注意要在`pom.xml`文件中引入`org.springframework:spring-beans`依赖

~~~xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hello" class="learning.spring.helloworld.Hello" />

</beans>
~~~

接下来我们将配置文件载入容器中，并从容器中获取组件：

~~~java
public class Application {
    public static void main(String[] args) {
        //定义并创建一个容器
        BeanFactory beanFactory = new DefaultListableBeanFactory();
        
         //将配置文件加载到容器中
        XmlBeanDefinitionReader reader = new XmlBeanDEfinitionRead(beanFactory);
        reader.loadBeanDefinitions("beans.xml");
        
        //Hello hello = (Hello)beanFactory.getBean("hello");
        
        //从容器中获取一个对象，会抛出BeansException。
        Hello hello = beanFactory.getBean("hello", Hello.class);	
        System.out.println(hello.hello());
    }
}
~~~

这样就**将业务逻辑的代码与管理组件生命周期的代码进行了解耦**。



注意这里的`BeanFactory`仅仅是一个接口，我们这里选用了`DefaultListableBeanFactory`实现类。不过在实际工作中，我们一般使用`ApplicationContext`接口。它定义在spring-context模块中，在pom.xml需要引入`org.springframework:spring-context`依赖。并继承了`BeanFactory`接口，提供了更加丰富的功能，例如事件传播、资源加载以及国际化支持等。

它的实现类有：

| 类名                               | 说明                     |
| ---------------------------------- | ------------------------ |
| ClassPathApplicationContext        | 从CLASSPATH中加载XML文件 |
| FileSystemApplicationContext       | 从文件系统中加载XML文件  |
| AnnotationConfigApplicationContext | 根据注解和Java类配置     |

~~~java
public class Application {
    public static void main(String[] args) {
        //定义并创建一个容器
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        
        Hello hello = applicationContext.getBean("hello", Hello.class);
    }
}
~~~



### 容器的继承关系

子容器可以继承芙蓉其中配置的组件。

~~~java
public class Hello {
    private String name;
    public String hello() {return "Hello World! by " + name;}
    public void setName(String name) {this.name = name;}
}
~~~

parent-beans.xml文件的部分代码

~~~xml
<bean id="parentHello" class="learning.spring.helloworld.Hello">
    <property name="name" value="PARENT" />
</bean>

<bean id="hello" class="learning.spring.helloworld.Hello">
    <property name="name" value="PARENT" />
</bean>
~~~

child-beans..xml文件的部分代码

在Application类中，我们从不同的容器中获取不同的Bean，以测试继承容器中Bean的可见性与覆盖情况。

~~~java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext parentContext = new ClassPathXmlApplicationContext("parent-beans.xml");
        ClassPathXmlApplicationContext childContext = new ClassPathXmlApplicationContext(new String[] {"child-beans.xml"}, true, parentContext);
        //标识容器
        parentContext.setId("ParentContext");
        childContext.setId("ChildContext");
        testVisibility(parentContext, "parentHello");
        testVisibility(childContext, "parentHello");
        testVisibility(parentContext, "childHello");
        testVisibility(childContext, "childHello");
        testOverridden(parentContext, "hello");
        testOverridden(childContext, "hello");
    }
    private void testVisibility(ApplicationContext context, String beanName) {
        System.out.println(context.getId() + " can see " + beanName + ": " + context.containsBean(beanName));
    }

    private void testOverridden(ApplicationContext context, String beanName) {
        System.out.println("sayHello from " + context.getId() +": " + context.getBean(beanName, Hello.class).hello());
    }
~~~

结论 

- 子容器可以访问到父容器中的bean
- 子容器可以覆盖父容器的同名bean



默认情况下，单个配置文件中存在id或者name相同的bean定义，spring解析时会报错；不同配置文件中存在id或者name相同的bean定义，后面加载的bean定义会覆盖前面的bean定义。可以通过容器对象的`setAllowBeanDefinitionOverriding(Boolean)`方法禁止覆盖

## Bean基础知识

Bean是Java中可重用软件组件。Spring容器要求Bean是一个POJO（Plain Old Java Object）对象，它满足：

- 提供无参构造器
- 每一个字段都是private，有相应的Getter、Setter方法



在一个Bean的定义中包含：

- 名称：Spring容器可以自动设置名称，命名方式为类名的首字母小写，以及驼峰（camel-cased）规则。比如类型为HelloService的Bean，自动生成的名称为helloService。同时还可以给bean指定一个别名（alias）。
- 具体类型：一个全限定类型
- 作用域：单例（singleton）还是原型（prototype）
- 依赖注入信息
- 创建销毁信息



~~~xml
<bean id="hello" class="learning.spring.helloworld.Hello" scope="singleton" lazy-init="ture" depends-on="" factory-method="create" >
</bean>
~~~

`factory-method`属性指定对象的`create`静态方法来创建`bean`。



## 依赖注入

有两种基本的注入方式：

- 基于构造方法的
- 基于Setter方法的

这两种方式可以混用，但是不推荐。

### 基于构造方法的

constructor-arg中可配置的属性：

| 属性  |                           |
| ----- | ------------------------- |
| value | 注入基本类型或String      |
| ref   | 注入一个对象，值为bean id |
| type  | 按类型匹配                |
| index | 按参数的位置匹配          |
| name  | 按参数名匹配              |

`ref`实现bean之间的依赖注入。这样，各个bean之间可以相互协作，而不需要显式地创建它们的依赖项，从而简化了应用程序的开发和维护。

~~~xml
<bean id="" class="">
    <constructor-arg value=""/>
    <constructor-arg value=""/>
</bean>
~~~





### 基于Setter方法

~~~xml
<bean id="" class="">
    <property name="" value="" />
    
    <property name="" ref="" />
    <!--ref是下面的简写形式-->
    <property name="">
        <ref bean="" />
    </property>
</bean>
~~~

这里`name`为字段名



~~~xml
<bean id="myBean" class="com.example.MyBean">
    <property name="innerBean">
        <bean class="com.example.InnerBean">
            <property name="innerProperty" value="innerValue"/>
        </bean>
    </property>
</bean>
~~~

对应的POJO

~~~java
public class MyBean {
    private InnerBean innerBean;

    public void setInnerBean(InnerBean innerBean) {
        this.innerBean = innerBean;
    }

    // getters and setters
}
~~~

`ref`和内部`bean`都是用于表示`bean`之间的依赖关系的机制，但它们的用法和含义是不同的。`ref`用于引用一个已经存在的`bean`对象，而内部`bean`仅对包含它的`bean`可见，同时无需`id`属性。



### 注入到List类型中

~~~xml
<bean id="myBean" class="com.example.MyBean">
    <!--基本类型与String-->
    <property name="listPropertyString">
        <list>
            <value>value1</value>
            <value>value2</value>
            <value>value3</value>
        </list>
    </property>
    <!--对象类型-->
    <property name="listProperty">
        <list>
            <!--ref也是可以的-->
            <bean class="com.example.Object1">
                <property name="prop1" value="value1"/>
                <property name="prop2" value="value2"/>
            </bean>
            <bean class="com.example.Object2">
                <property name="prop1" value="value3"/>
                <property name="prop2" value="value4"/>
            </bean>
        </list>
    </property>
</bean>
~~~

相应的Java代码

~~~java
public class MyBean {
    private List<MyObject> listProperty;
	private List<String> listPropertyString;
    public void setListProperty(List<MyObject> listProperty) {
        this.listProperty = listProperty;
    }
    // getters and setters
}
~~~



### 注入到Map中：

~~~xml
<bean id="myBean" class="com.example.MyBean">
    <property name="myMap">
        <map>
            <entry key-ref="keyBean1" value-ref="dependencyBean1"/>
            <entry key="" value="" />
        </map>
    </property>
</bean>

<bean id="dependencyBean1" class="com.example.DependencyBean"/>
<bean id="keyBean1" class="com.example.keyBean"/>
~~~



### autowire

Spring容器可以代替我们自动进行依赖注入，这种机制称为**autowire**。

先给一个例子：

~~~xml
<bean id="cat" class="com.pojo.Cat"></bean>
<bean id="dog" class="com.pojo.Dog"></bean>
<bean id="people" class="com.pojo.People" >
    <property name="dog" ref="dog"></property>
    <property name="cat" ref="cat"></property>
    <property name="name" value="" ></property>
</bean>
~~~

使用自动装配后

~~~xml
<bean id="cat" class="com.pojo.Cat"></bean>
<bean id="dog" class="com.pojo.Dog"></bean>
<bean id="people" class="com.pojo.People" autowire="byName">
    <property name="name" value="" ></property>
</bean>
~~~



autowire的模式

|    名称     |               说明               |
| :---------: | :------------------------------: |
|     no      |          不进行自动织入          |
|   byName    |  匹配bean id与字段名相同的bean   |
|   byType    | 匹配bean class与字段名相同的bean |
| constructor |   同Type，但用于构造方法的注入   |

- 手动设置的优先级高于自动织入
- 自动织入无法注入基本类型与字符串
- 对于集合类型的属性，自动织入会把上下文全部匹配的Bean都放进去。对于非集合类型的，有多个候选Bean就会有问题。可以通过以下方法解决此问题：
	- `<bean>`的`autowire-candidate`设置为`false`
	- `<bean>`的`primary`属性设置为`true`



一般情况下，Spring容器会根据依赖情况自动调整Bean的初始化顺序。我么也可以通过`<bean>`的`depends-on`属性指定Bean的依赖顺序



## Bean的三种配置方式

### 基于XML文件的配置

不再赘述

### 基于注解的配置

Bean创建相关的注解：

|                 |                        |
| --------------- | ---------------------- |
| @Component      | 标识为一个Bean         |
| @Service        | 标识为服务层的对象     |
| @Repository     | 标识为DAO              |
| @Controller     | 标识为Web层的Web控制器 |
| @RestController | REST服务的控制器       |

~~~java
@Componet("helloBean")
public class Hello { }
~~~





|            |                                                        |
| ---------- | ------------------------------------------------------ |
| @Autowired | 根据类型注入依赖，可用于构造方法、Setter方法、成员变量 |
| @Resource  |                                                        |
| @Inject    |                                                        |



### 基于Java类的配置

