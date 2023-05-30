# Spring Framework



框架与设计模式的出现就是为了**抵抗变化**。可以通过增加中间层（本质上降低程序的耦合度），来尽可能地隔离变化所带来的影响。而Spring Framework就是一个中间层，让开发者专注于业务逻辑的开发，而无需关心请求处理等与业务无关的部分。

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



Spring可能会匹配到多个Bean，此时这些Bean统称为**候选者（candidates）。**可以通过@Primary注解来选择使用一个Bean

~~~java
@Bean
@Primary
public Person person4Parameters(String name, int age, Address address{
    return new Person(name, age, address);
}
 
@Bean
public Person person2MethodCall() {
   return new Person(name(), age(), address());
}
 
@Bean
public Person person3Parameters(String name, int age, Address address3) {
   return new Person(name, age, address3);
}
~~~



