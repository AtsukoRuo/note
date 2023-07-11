

# Spring MVC

@SpringBootApplication注解包括@ComponentScan注解的功能



一个简单的示例：

~~~java
@SpringBootApplication
public class AtsukoruoApplication {
	public static void main(String[] args) {
		SpringApplication.run(AtsukoruoApplication.class, args);
	}
}

@Controller
public class SayHelloController {
    //"say-hello" => "Hello! What are you learning today?"
    //@RequestMapping("say-hello") is equal to the next line
    @RequestMapping("/say-hello")
    @ResponseBody
    public String sayHello() {
        return "Hello! What are you learning today?";
    }
}
~~~



其中@RequestMapping将URL请求转发给相应的方法，而@ResponseBody将方法的返回值作为响应体，而响应头需要HttpHeaders以及返回ResponseEntity，下面再给出一个例子：

~~~java
@GetMapping("/video")
@ResponseBody
public ResponseEntity<byte[]> getVideo() throws IOException {
    // 读取视频文件
    File videoFile = new File("video.mp4");
    byte[] videoBytes = Files.readAllBytes(videoFile.toPath());

    // 设置响应头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
    headers.setContentDisposition(ContentDisposition.builder("attachment").filename("video.mp4").build());

    // 返回视频文件的字节流
    return new ResponseEntity<>(videoBytes, headers, HttpStatus.OK);
}
~~~







spring.mvc.view.prefix是Spring MVC框架中的一个配置属性，用于指定视图（View）文件的前缀。在Spring MVC中，控制器（Controller）通常会返回一个逻辑视图名（Logical View Name），这个逻辑视图名会被解析成一个具体的视图文件，然后由视图解析器（View Resolver）将其解析成最终的响应体（Response Body）。

例如：

~~~properties
#这个目录在resources/META-INF/resources下
spring.mvc.view.prefix=WEB-INF/jsp/   
spring.mvc.view.suffix=.jsp
~~~

~~~java
@RequestMapping("say-hello-jsp")
public String sayHelloJsp() {
    return "sayHello";
}
~~~

如果未指定spring.mvc.view.prefix属性，则默认值为classpath:/templates/



SpringMVC 通过在控制器方法的参数中声明 ModelMap 类型的参数来识别它。当控制器方法被调用时，SpringMVC 会自动创建一个 ModelMap 对象，并将其作为参数传递给控制器方法。控制器方法可以使用 ModelMap 对象来设置键值对。在控制器方法执行完毕后，SpringMVC 将 ModelMap 对象中的键值对传递给视图，以便视图可以使用这些值来渲染页面。

~~~java
@Controller
public class MyController {
    @RequestMapping("/hello")
    public String hello(ModelMap model) {
        // 将数据存储到 ModelMap 中
        model.put("message", "Hello World!");
        // 返回视图名
        return "helloView";
    }
}


/**
	<html>
		<body>
			this message is ${message}
		<body>
	</html
*/
~~~

JSP启用了扩展语法：其中${}就是占位符，可以在Controller中通过ModelMap对象来设置。



如果要获取URL中的查询参数（例如：http:localhost:8080/login?name=123），可以通过@RequestParam注解来实现，下面给出一个例子：

~~~java
@Controller
public class LoginController {
    @RequestMapping("login")
    public String gotoLoginPage(
            @RequestParam(value = "account") String name,
            @RequestParam String pwd,		//如果未指定value，那么变量名作为对应的参数名
            ModelMap model) {
        model.put("name", name);
        System.out.println(name + ", " + pwd);
        return "login";					//这里是视图，也就是jsp页面的名字
    }
}
~~~



~~~properties
#spring框架在终端要打印信息的最低等级
logging.level.org.springframework=info
#cn.atsukoruo包下所有类的打印等级
logging.level.cn.atsukoruo=debug
~~~

使用Logger：

~~~java
public class LoginController {
    private Logger logger = LoggerFactory.getLogger(getClass());
    //Model map the java variable to the jsp
    @RequestMapping("login")
    public String gotoLoginPage(@RequestParam(value = "account") String name){
        logger.debug("Request param is {}", name);		//这里{}是一个占位符
        logger.info("I want this printed at info level");
        logger.warn("I want this printed at warn level");
        return null;
    }
}
~~~





这里简单介绍以下MVC结构：

- Model（模型）：指代应用程序中的数据和对应的业务逻辑，这里的业务逻辑是指与数据库交互以及验证数据正确性的逻辑
- View（视图）：指代应用程序中根据模型渲染的用户界面，同时处理用户的交互。
- Controller（控制器）：协调应用程序中的模型和视图。它负责接收用户输入并将其传递给模型进行处理。控制器还负责根据模型的状态更新视图。

此外，控制器还有一些相似的业务逻辑，例如鉴权、安全性、请求处理。抽象出一些共同的业务逻辑，形成Front Controller层。下面就是SpringMVC的框架图： 

![image-20230705151909004](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230705151909004.png)

注：这里的View可以是JSP

Service层通常作为一种中间层，位于Controller层和Model层之间，负责协调Controller层和Model层之间的数据传输和业务逻辑处理。这里存留一个疑问，数据验证工作应该放在Service层中进行还是Model层中进行？答：视情况而定。





@RequestMapping还可以指定要处理的请求方法，同时@RequestParam还可以处理表单数据（`<input type="password" name="password">`）

~~~java
@RequestMapping(value="login", method = RequestMethod.GET)
public String gotoLoginPage() {
    return "login";
}

@RequestMapping(value="login", method = RequestMethod.POST)
public String gotoWelcomePage(@RequestParam String name
    , @RequestParam String password) {
~~~



Model的作用域仅仅在本次请求中。如果想要在多个Controller中的Model共享某个值，也就是Model的作用域延长到会话（session），也就是多个请求中，那么就使用@SessionAttributes("name")注解。每个用户都有自己的 HttpSession 对象，而`@SessionAttributes` 注解可以用来指定哪些模型属性需要存储到 HttpSession 中。下面给出一个例子：

~~~java
@Controller
@SessionAttributes("name")
public class LoginController {
    @RequestMapping(value="login", method = RequestMethod.POST)
    public String gotoWelcomePage(ModelMap model) {
        model.put("name", name);
        return "login";
    }
}

@Controller
@SessionAttributes("name")
public class TodoController {
    @RequestMapping("list-todos")
    public String listAllToodos(ModelMap model) {
        return "listTodos";
    }
}


//in listTodos.jsp   <div>hello ${name}</div>
//in login.jsp		<div>Welcome ${name}</div>
~~~

> 在 Spring MVC 中，每个用户的 HttpSession 是通过浏览器发送的 JSESSIONID cookie 来确定的.当用户第一次访问 Web 应用程序时，服务器会为该用户创建一个新的 HttpSession，并将一个唯一的标识符（称为 JSESSIONID）存储到该 HttpSession 中。服务器还会将 JSESSIONID 作为一个名为 JSESSIONID 的 cookie 发送到客户端浏览器中。
>
> 当客户端浏览器发送后续请求时，它会自动将 JSESSIONID cookie 包含在请求头中，以便服务器可以将请求映射到正确的 HttpSession 上。服务器会检查请求中是否包含有效的 JSESSIONID，如果包含，则将该请求映射到相应的 HttpSession 上；如果不包含，则会创建一个新的 HttpSession。



JSP是一种基于服务器渲染的模板技术，在前后端分离的大趋势下，不推荐使用它。但是做个人项目，还是可以考虑一手的。

添加以下依赖使用JSTL，它可以扩展HTML标签，这些标签可以执行一些简单的逻辑

~~~xml
<dependency>
    <groupId>jakarta.servlet.jsp.jstl</groupId>
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    <version>3.0.0</version>
</dependency>

<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>glassfish-jstl</artifactId>
</dependency>
~~~

https://docs.oracle.com/javaee/5/jstl/1.1/docs/tlddocs/index.html可以查看这些标签的含义





bootstrap、jquery框架的依赖如下：

~~~xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>5.1.3</version>
</dependency>

<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.6.0</version>
</dependency>
~~~

依赖不仅可以下载jar，还可以下载这些文件.css/.js

WebJars中的静态资源文件，例如CSS、JavaScript和图像等，都会被放置在`META-INF/resources/webjars`目录中。相对路径是相对于Web应用的上下文路径。即META-INF/resources：

~~~html
<link href="webjars/bootstrap/5.1.3/css/bootstrap.css" rel="stylesheet">
<script src="webjars/bootstrap/5.1.3/js/bootstrap.min.js"></script>
~~~



~~~java
@RequestMapping(value = "add-todo", method=RequestMethod.POST)
public String addNewTodo() {
    return "redirect:list-todos";		//重定向，注意这里list-todos必须是URL，而不是jsp文件名
    //否则返回的jsp中没有任何绑定的数据。
}
~~~

这里有个小注意点，用户不能从浏览器中直接访问大部分静态资源，只能被访问到用@RequestMapping注册的URL，此时静态资源的相对路径可能与暴露出来的URL是不同的，甚至暴露出来的URL可以对应多个静态资源（通过转发机制）。

> 重定向返回浏览器发送302状态码，此时浏览器会再次请求一次重定向的地址。而转发却不会，它直接返回处理后的结果，连URL都不带变的。



Spring Validations

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
~~~



2. CommandBean

   它是一个表单后端对象（Form Backing Object），它要求前端的Form元素与后端的某个Bean对象进行双向绑定。这样前端传来的数据可以写入到Bean当中

   首先，我们必须引入特殊的JSP标签

   ~~~jsp
   <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
   ~~~

   ~~~jsp
   <form:form method="post" modelAttribute="todo">			
       Description : <form:input type="text" name="description" required="required" path="description"/>
       <form:input type="hidden"  path="done"/>
       <form:input type="hidden"  path="id"/>
       <input type="submit" class="btn btn-success" />
   </form:form>
   ~~~

   `modelAttribute`必须和Bean的变量名字一致，而不是和Bean的类型名一致

   后端代码：

   ~~~java
   @RequestMapping(value = "add-todo", method = RequestMethod.GET)
   public String showNewTodoPage(ModelMap  model)
   {
       //这是必要的，可以视为表单的默认值
       //这是一个单向绑定 bean -> form(get)
       Todo todo = new Todo(0, (String)model.get("name"), "123123", LocalDate.now(), false);
       model.put("todo", todo);
       return "todo";
   }
   
   @RequestMapping(value = "add-todo", method=RequestMethod.POST)
   public String addNewTodo(ModelMap model, Todo todo) {
       //这是另一个单向绑定 form(post) -> bean
       todoService.addTodo((String)model.get("name"), todo.getDescription(),  LocalDate.now().plusYears(1), false);
       return "redirect:list-todos";
   }
   ~~~

2. Add Validations to Bean

   ~~~xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-validation</artifactId>
   </dependency>
   ~~~

   在Bean对象中添加：
   ~~~java
   @Size(min=10, message="Enter at least 10 characters")
   private String description;
   ~~~

   在Controller层添加：

   ~~~java
   public String addNewTodo(ModelMap model, @Valid Todo todo, BindingResult result) {
       if (result.hasErrors()) {
            return "todo";
       }
   }
   ~~~

   如果这里不使用BindingResult对象，那么后端异常直接报告给前端，这是我们不想看到的

3. Display Validation Errors in the View

   如果验证不通过，那么Spring MVC在返回JSP时，会渲染以下元素。否则就不渲染

   ~~~xml
   <form:errors path="description" cssClass="text-warning"/>
   ~~~



通过jspf文件来消除在jsp中的冗余代码，然后再jsp中使用`<%@ include file="common/navigation.jspf"%>`标签即可引入冗余代码





Spring Security

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
~~~

修改用户名以及密码

~~~java
@Configuration
public class SpringSecurityConfiguration {
    //LDAP or Database
    @Bean
    public InMemoryUserDetailsManager createUserDetailsManager() {
        UserDetails userDetails = User.withDefaultPasswordEncoder()			
                .username("in28minutes")
                .password("dummy")
                .roles("USER", "ADMIN")
                .build();
        UserDetails userDetails1 = User.builder().passwordEncoder(
                input -> passwordEncoder().encode(input))
                        .username("in28minutes")
                        .password("dummy")				
                        .roles("USER", "ADMIN")
                        .build();
        return new InMemoryUserDetailsManager(userDetails, userDetails1);		//向SpringSecurity注册这些用户
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
        //具体来说，当用户注册或修改密码时，应该将用户输入的密码使用 PasswordEncoder 进行加密，并将加密后的密码存储在数据库中。当用户登录时，应该将用户输入的密码使用 PasswordEncoder 进行加密，并与数据库中存储的加密后的密码进行比较，以验证用户的身份。
    }
    
    private String getUserName() {
        Authentication authentication =
                SecurityContextHolder.getContext().getAuthentication();
        return authentication.getName();
    }
}
~~~







~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
~~~





~~~java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
    http.authorizeHttpRequests(
            auth -> auth.anyRequest().authenticated()
    );
    http.formLogin(Customizer.withDefaults());

    http.csrf().disable();
    http.headers().frameOptions().disable();
    return http.build();
}
~~~

这段代码是一个 Spring Security 的配置类，用于配置安全过滤器链（`SecurityFilterChain`）和 HTTP 安全性。

具体来说，这段代码实现了以下功能：

1. `authorizeHttpRequests()` 方法用于配置 HTTP 请求的权限控制，这里配置任何请求都需要进行认证才能访问。`anyRequest()` 方法表示匹配任何请求，`authenticated()` 方法表示需要进行认证才能访问。
2. `formLogin()` 方法用于配置表单登录，这里使用默认的表单登录页面和处理器。
3. `csrf().disable()` 方法用于禁用 CSRF 防护。CSRF（Cross-Site Request Forgery，跨站请求伪造）是一种网络攻击，在用户已经登录的状态下，攻击者利用用户的身份在用户不知情的情况下发起恶意请求。禁用 CSRF 防护可以加速开发和测试，但会降低应用程序的安全性。在生产环境中，应该启用 CSRF 防护。
4. `headers().frameOptions().disable()` 方法用于禁用 X-Frame-Options 防护，这是一种防止网页被嵌入到 iframe 中的安全策略。禁用 X-Frame-Options 防护可以允许网页被嵌入到 iframe 中，但会增加网页的安全风险。在需要使用 iframe 的场景下，可以考虑启用 X-Frame-Options 防护。
5. 最后，返回一个安全过滤器链的实例，该实例包含了上述配置。





