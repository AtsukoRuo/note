# Java 杂项

## 包管理

Java通过一种新颖的方法解决了命名冲突的问题。为了能够清晰地描述库的名称，Java 语言的设计者所使用的方法是将你的**互联网域名反转**使用。因为域名是唯一的，所以一定不会冲突。而反转的域名之后的几个“.”实际上描述了子目录的结构。例如`com.ituring.utility.foibles`表示`com.ituring.utility`包下的`foibles`类这种方法会对我们管理源代码带来一些挑战。比如我们想要使用`com.ituring.utility.foibles`这个命名空间，那么我就需要创建名为`com`和`ituring`的空文件夹，其目的只是为了与反转的URL相对应。



可以使用通配符`*`来导入包下的所有类，例如`import java.util.*;`

所有的Java文件都自动导入一个特定的库——`java.lang`



此外，Java 消除了所谓的**“前向引用”（forward referencing）**问题。假如某个类存在于当前被调用的源代码文件中。你只要使用这个类就可以了，哪怕这个类稍后才会在文件中定义。

当不同库中的类名冲突时，解决方案则是

- 利用import关键字告知 Java 编译器你想要使用哪个类。
- 或者在类名前添加包名。

~~~java
import java.sql.Date
    
Date date = new Date();							//date是java.sql.Date类型
java.util.Date date2 = new java.util.Date();	  //date2是java.util.Date类型
~~~



## 编程风格

- 命名采用驼峰式命名法（Camel-Case）。对于类名，首字母要大写；对于方法名或对象名，首字母小写