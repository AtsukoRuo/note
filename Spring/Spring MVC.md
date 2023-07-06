

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



Model的作用域仅仅在本次请求中。如果想要在多个Controller中的Model共享某个值，也就是Model的作用域延长到会话（session），也就是多个请求中，那么就使用@SessionAttributes("name")注解。下面给出一个例子：

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















