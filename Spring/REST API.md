# REST API



一个简单的REST API示例：

~~~java
@RestController
public class HelloWorldController {
    /*
    value是字符串,path是字符串数组,所以path可以映射多个URL,value只能映射一个URL。
	使用path时,如果URL中包含扩展名,则会对扩展名做精确匹配。而value就算URL中包含扩展名,也会忽略扩展名做匹配。
	path支持使用通配符?,*,**。而value不支持。
	path支持使用{varName}形式的路径变量。value不支持
	*/
    //@RequestMapping(method = RequestMethod.GET, path = "/hello-world")
    @GetMapping(path = "/hello-world")
    public String helloWorld() {
        return "Hello World";
    }

    @GetMapping(path = "/hello-world-bean")
    public HelloWorldBean helloWorldBean() {
        return new HelloWorldBean("Hello World");
        //Bean -> JSON
    }
}
~~~

Spring Web build web, including RESTful, applications using Spring MVC. Uses Apache Tomcat as the default embedded container.

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~



所有的请求都先发送到DispatcherServlet，由它将请求转发到对应的Controller对象中。它是由DispathcerServletAutoConfiguration类来自动配置的。注意@RestController包括@ResponseBody注解，该注解 + JacksonHttpMessageConverters可以将返回的对象转换成JSON对象。其中Converters可以由JacksonHttpMessageConvertersConfiguration来自动配置。而对于非法的请求由ErrorMvc来处理，而它可以由ErrorMvcAutoConfiguration来自动配置。



RESTful API对资源的{ADD、UPDATE、DELETE、FIND}操作映射到HTTP的请求方法中{GET、POST、DELETE}。而相应的参数直接放在URL中，这也就是**路径参数（path Parameters）**的概念。这提供一个统一的接口形式，方便调用。而对于其他服务（登录、播放音乐）或者非功能需求（验证、性能）等，RESTful API就**不适用**了

资源的操作以及HTTP请求方法的映射关系如下：

GET：retrieve resource

POST：create a new resource

PUT：update whole resource

PATCH：update part of resource

DELETE：delete the resource



在spring mvc中按以下方式获取路径参数：

~~~java
@GetMapping(path = "/hello-world/path-variable/{name}")
public HelloWorldBean helloWorldPathVariable(
        @PathVariable String name
) {
    return new HelloWorldBean("Hello World, " + name);
}
~~~





在Google插件上安装Talend API Tester - Free Edition，用于发送POST请求



使用@RequestBody注解,Spring MVC会将请求体中的JSON数据绑定到Foo对象中。

~~~java
@PostMapping("/someApi")
public void someApi(@RequestBody Foo foo) {
    // ...
}
~~~

通过ResponseEntity对象，返回自定义的HTTP响应

~~~java
@PostMapping("/users")
public ResponseEntity<Object> createUser(@RequestBody User user) {
    service.save(user);
    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(user.getId())
            .toUri();
    return ResponseEntity.created(location).build();        //location:重定向的URI
}
~~~





`@ResponseStatus`注解通常用于在控制器方法上（或者异常类型上），用于当控制器方法执行完成后或者异常被抛出后，向客户端返回指定的HTTP状态码和响应信息

~~~java
@GetMapping("/user/{id}")
@ResponseStatus(HttpStatus.OK)
public User getUserById(@PathVariable Long id) {
    return userService.getUserById(id);
}


@ResponseStatus(code = HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException{
    public UserNotFoundException(String message) {
        super(message);
    }
}
~~~





ResponseEntityExceptionHandler是一个抽象类，用于处理所有由Spring MVC抛出的异常，并向前端返回这些异常信息。我们自己可以编写的异常处理类，向前端返回自定义的异常信息（默认JSON格式）。下面是一个示例

~~~java
public class ErrorDetails {
    private LocalDate timestamp;
    private String message;
    private String details;
	//Other
}


@ControllerAdvice
public class CustomizedResponseEntityExceptionHandler
    extends ResponseEntityExceptionHandler {
	
    //表明要处理的异常类型
    @ExceptionHandler(Exception.class)
    public final ResponseEntity<Object> handleAllException(Exception ex, WebRequest request) throws Exception {
        ErrorDetails errorDetails = new ErrorDetails(LocalDate.now(),
                ex.getMessage(), request.getDescription(false));

        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);
        //默认的异常处理器不会忽略异常对象上的@ResponseStatus，但是这里的编写的却会忽略
    }
}
~~~





@RequestMapping返回类型总结

- ModelAndView
- Void
- String：视图名或者字符串
- ResponseEntity：用于构造 Http 响应,包含状态码、头信息和 body 内容
- 其他类型：DispatcherServlet 会将对象通过适当的 HttpMessageConverter 转换为 JSON 或 XML 并写入 response body。