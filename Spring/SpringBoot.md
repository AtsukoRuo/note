# springBoot



在Spring时代中，开发之前的配置工作有以下痛点：

- 依赖包管理（pom.xml）
- Web APP的配置（web.xml）
- 管理SpringBeans（context.xml）
- 非功能要求，例如异常处理、监控、管理员登录等

SpringBoot采用“约定大于配置”的思想，它可以尽快地构建生产就绪的APP




一个简单的示例：

~~~java
@RestController
public class CourseController {
    @RequestMapping("/courses")
    public List<Course> retrieveAllCourses() {
        return Arrays.asList(
                new Course(1, "Learn AWS", "atsukoruo"),
                new Course(1, "Learn DevOps", "atsukoruo")
        );
    }
}
~~~

注意：这里@RestController注解是属于Spring MVC中

当你在浏览器输入http://localhost:8080/courses，会返回

~~~java
[{"id":1,"name":"Learn AWS","author":"atsukoruo"},{"id":1,"name":"Learn DevOps","author":"atsukoruo"}]
~~~

可以使用Chrome 插件：JSON Format 来美化一下。



@SpringBootApplication annotation is a combination of 3 annotations: @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan





SpringBoot Starter Project为你预先定义了特定项目所需的必要依赖，在pom.xml直接引用即可。

![image-20230704151611210](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704151611210.png)





SpringBoot将功能所需的配置抽象出来，程序员可以使用默认配置，也可以自定义配置，这简化了程序员的配置工作。

Spring Auto Config：

- SpringBoot启动时，根据src/main/resources/META-INF/spring.factories中的类以及classpath加载所有的自动配置类，这些自动配置类都实现了`org.springframework.boot.autoconfigure.EnableAutoConfiguration`接口。

  > 如果您的 Spring Boot 项目没有 `spring.factories` 文件，那么 Spring Boot 会使用默认的自动配置类。这些自动配置类通常在 Spring Boot 的依赖中定义，并且实现了 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 接口。

- 一旦所有的自动配置类都被加载进来，Spring Boot就会根据这些类中的配置信息，自动创建所需的Bean、服务和其他组件。

- 在自动配置完成后，如果应用程序需要对某个组件进行特殊的配置，那么可以通过提供自定义的Bean来覆盖自动配置。Spring Boot会优先使用自定义的Bean，而不是自动配置生成的Bean。



可以在不重启服务器的情况下，热更新代码

~~~
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
~~~



![image-20230704162030392](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704162030392.png)

~~~properties
logging.level.org.springframework=trace

logging.level.org.springframework=debug
spring.profiles.active=dev

logging.level.org.springframework=debug
spring.profiles.active=prod
~~~









`@ConfigurationProperties` 是 Spring Boot 提供的一种注解，它可以将配置文件中的属性值自动注入到 Java 对象中，简化配置工作。下面是一个简单的示例，

```properties
example.name=John
example.age=30
```

您可以创建一个 Java 类 `ExampleConfig`，并使用 `@ConfigurationProperties` 注解来指定需要注入的属性：

```java
@Component
@ConfigurationProperties(prefix = "example")
public class ExampleConfig {
    private String name;
    private int age;
    // getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

在 `@ConfigurationProperties` 注解中，`prefix` 属性指定了配置文件中属性的前缀，这里是 `example`。因此，`name` 和 `age` 属性的实际属性名分别为 `example.name` 和 `example.age`。然后，在需要使用 `ExampleConfig` 类的地方，您可以将其注入到 Spring 容器中，并使用 `@Autowired` 注解自动装配：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class ExampleController {
    @Autowired
    private ExampleConfig exampleConfig;

    @GetMapping("/example")
    @ResponseBody
    public String example() {
        String name = exampleConfig.getName();
        int age = exampleConfig.getAge();
        return "Name: " + name + ", Age: " + age;
    }
}
```





以前部署Spring项目需要

- 打包WAR包
- 安装Web服务器（Tomcat）
- Java

而SpringBoot简化了这项工作：

- 打包JAR包（Embedded Server）
- Java





通过在pom,.xml引入以下依赖，来添加Monitor功能：

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
~~~

Monitor功能记录应用的一些状态，可以通过/http://localhost:8080/actuator 来获取这些信息

在.properties文件中可以选择暴露出哪些状态：

~~~properties
management.endpoints.web.exposure.include=*
~~~






