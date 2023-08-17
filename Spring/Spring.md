# Spring

[TOC]

> 一键式创建SpringBoot项目：https://start.spring.io/ 

框架与设计模式的出现就是为了**抵抗变化**。可以通过增加中间层，降低程序的耦合度，来尽可能地隔离变化所带来的影响。而Spring框架就是一个中间层，通过解耦，让开发者专注于业务逻辑的开发，而无需过多关心与业务无关的部分。



Spring框架分为多个Spring Project，包括但不限于Spring Famework、SpringBoot、Spring JPA，通过引入新的Spring Project来推动Spring框架的演化。每个Spring Poject又分为多个Spring Moudules。开发人员可以按需使用Moudules。

![image-20230704141946297](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704141946297.png)



## Spring Framework Core

`Spring IoC`通俗点讲，就是负责对象的创建、处理对象之间的依赖关系以及管理对象生命周期的构件，即在业务逻辑代码与管理对象生命周期的代码之间进行解耦。



**Spring Container**（又称**IoC Container**、**Spring Context**）接受POJO对象以及Config（配置类或配置文件）作为输入，输出一个就绪的运行时系统。

Spring Container有两种类型：

- **Bean Factory**：basic spring container
- **Application Context**（推荐使用）：advanced spring container with enterprise-specific features，例如国际化、与Spring AOP的整合





- **POJO（Plain Old Java Object）**：没有任何限制。任何Java对象都是一个POJO

- **JavaBean**（也称为Enterprises Java Bean，EJB），它具有以下限制：

  - 一个public无参构造器

  - 每个成员为私有的，而且必须提供getter以及setter

  - 实现Serializable接口，该接口中什么也没有


- **Spring Bean**：由Spring Container管理的Java对象