# Spring Cloud 初步

[TOC]

## 微服务是什么？

![image-20240104182836831](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20240104182836831.png)

Microservice is  a distributed, loosely coupled software service that carries out a small number of well-defined tasks. 

Let’s start by considering the differences between microservices and some other common architectures. 

- **N-tier architecture**：With this design, an applications is divided into multiple layers, each with their own responsibilities and functions, such as UI, services, data, testing, and so forth. As you create your application, you make a specific project or solution for the UI, then another one for the services, another for the data layer, and so on. In the end, you will have several projects that, combined, create an entire application. n-tier applications have many advantages, including these:

  - N-tier applications offer good separation of concerns. making it possible to consider areas like UI (user interface), data, and business logic separately.

  N-tier applications also have drawbacks:

  - You must stop and restart the entire application when you want to make a change.
  - Messages tend to pass up and down through the layers, which can be inefficient.

- **monolithic architectural**：All of the UI, business, and database access logic are packaged together into a unique application and deployed to an application server.   Monolithic architectures have all processes tightly coupled, and these run as a single servic

  Each development team is responsible for their own discrete piece of the application that usually targets specific customers. 

  ![image-20240105102106596](assets/image-20240105102106596.png)

  Monoliths are easier to build and deploy than more complex architectures like n-tier or microservices.  When an application begins to increase in size and complexity, however, monoliths can become difficult to manage. Each change to a monolith can have a cascading effect on other parts of the application, which may make it time consuming and expensive, especially in a production system. 

- **microservice**：The key concepts you need to embrace as you think about microservices are **decomposing** and **unbundling**. 

  ![image-20240105103751066](assets/image-20240105103751066.png)

  a microservice architecture has the following characteristics:

  - 应用被拆分为细粒度的组件，该组件具有良好的定义以及清晰的责任边界。 Each component has a small domain of responsibility and is deployed independently of the others. **A single microservice is responsible for one part of a business domain.** 
  - Microservices employ lightweight communication protocols such as HTTP and JSON (JavaScript Object Notation) for exchanging data between the service consumer and service provider. Because microservice applications always communicate with a technology-neutral format (JSON is the most common), **the underlying technical implementation of the service is irrelevant.** This means that an application built using a microservice approach can be constructed with multiple languages and technologies.

Figure 1.3 compares a monolithic design with a microservices approach for a typical small e-commerce application. 

![image-20240105104328055](assets/image-20240105104328055.png)

***Small, Simple, and Decoupled Services = Scalable, Resilient, and Flexible Applications***

- **Flexible**（灵活性）—Decoupled services can be composed and rearranged to quickly deliver new functionality. 
- **Resilient**（健壮性）— Failures can be localized to a small part of the application. This also enables the application to degrade gracefully in case of an unrecoverable error.
- **Scalable**（可扩展）—Decoupled services can easily be distributed horizontally across multiple servers, making it possible to scale the features/services appropriately. 

 Conway’s law states that

> “Organizations which design systems . . . are constrained to produce designs which are copies of the communication structures of these organizations.” 

Basically, what that indicates is that the way teams communicate within the team and with other teams is directly reflected in the code they produce. we can apply **Conway’s law** in reverse by using microservice. 





Figure 1.4 shows a high-level overview of some of the services and technology integrations that we will use throughout the book

![image-20240105111839400](assets/image-20240105111839400.png)



## 例子：基于SpringBoot构建的REST Service

Spring Boot simplifies the building of REST-based/JSON microservices

![image-20240105115141037](assets/image-20240105115141037.png)

~~~java
@SpringBootApplication
@RestController
@RequestMapping(value="hello")
public class SimpleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SimpleApplication.class, args);
    }

    //  you’re basically exposing one endpoint with a GET HTTP verb that takes two parameters (firstName and lastName) on the URL: one from the path variable (@PathVariable) and another one as the request parameter (@RequestParam)
    // http://localhost:8080/hello/Ruofan?lastName=Gao
    @GetMapping(value="/{firstName}")
    public String helloGET(
        @PathVariable("firstName") String firstName,
        @RequestParam("lastName") String lastName) {
        return String.format("{\"message\":\"Hello %s %s\"}",firstName, lastName);
    }

    @PostMapping
    public String helloPOST( @RequestBody HelloRequest request) {
        return String.format("{\"message\":\"Hello %s %s\"}",request.getFirstName(), request.getLastName());
    }
}
~~~



## 云计算是什么？

人们谈及微服务的时候，总是把它和云计算关联起来。那么它们俩之间存在何种关系呢？首先我们来认识一下云计算。

**「Cloud computing 」**is the delivery of computing and virtualized IT services—databases, networking, software, servers, analytics, and more—through the internet to provide a flexible, secure, and easy-to-use environment

 The cloud computing models let the user choose the level of control over the information and services that these models provide. These models are known by their acronyms, and are generically referred to as **XaaS**—an acronym that means anything as a service.The following lists the most common cloud computing models：

- 「**Infrastructure as a Service (IaaS)**」—The vendor provides the infrastructure that lets you access computing resources such as servers, storage, and networks. In this model, the user is responsible for everything related to the maintenance of the infrastructure and the scalability of the application. 

  > IaaS platforms include AWS (EC2), Azure Virtual Machines, Google Compute Engine, and Kubernetes

- **「Container as a Service (CaaS)」**—Unlike an IaaS model, where a developer manages the virtual machine to which the service is deployed, with CaaS, you deploy your microservices in a lightweight, portable virtual container (such as Docker) to a cloud provider. The cloud provider runs the virtual server the container is running on, as well as the provider’s comprehensive tools for building, deploying, monitoring, and scaling containers

  > CaaS platforms include Google Container Engine (GKE) and Amazon’s Elastic Container Service (ECS）

- **「Platform as a Service (PaaS)」**— This model provides a platform and an environment that allow users to focus on the development, execution, and maintenance of the application. Service Provider gives you the ability to deploy your services without having to know about the underlying application container. These provide a web interface and command-line interface (CLI) to allow you to deploy your application as a WAR or JAR file. 

  > PaaS platforms include Google App Engine, Cloud Foundry, Heroku, and AWS Elastic Beanstalk.

- **「Function as a Service (FaaS)」**— Serverless architecture allows us to focus only on the development of services without having to worry about scaling, provisioning（配置）, and server administration. Instead, we can solely concentrate on uploading our functions without handling any administration infrastructure. 

  > NOTE：If you’re not careful, FaaS-based platforms can lock your code into a cloud vendor platform because your code is deployed to a vendor-specific runtime engine. With a FaaS-based model, you might be writing your service using a general programming language (Java, Python, JavaScript, and so on), but you’re still tying yourself to the underlying vendor’s APIs and runtime engine that your function will be deployed to.

  > FaaS platforms include AWS (Lambda), Google Cloud Function, and Azure functions

- **「Software as a Service (SaaS)」**：Also known as software on demand, this model allows users to use a specific application without having to deploy or to maintain it. In most cases, the access is through a web browser（RESTful request）. Everything is managed by the service provider: application, data, operating system, virtualization, servers, storage, and network

  > SaaS platforms include Salesforce, SAP, and Google Business.

![image-20240105144923739](assets/image-20240105144923739.png)





One of the core concepts of a microservice architecture is that each service is packaged and deployed as its own discrete and independent artifact. Service instances should be brought up（启动） quickly, and each should be indistinguishable（无差别的） from another. When writing a microservice, sooner or later you’re going to have to decide whether your service is going to be deployed to one of the following

- **Physical server**— 如果要在原有物理服务器基础上进行微服务的横向扩展，成本可能会变得非常高。横向扩展指的是增加更多的服务器以提高系统的处理能力，而不是增加单台服务器的性能（称为纵向扩展）。在物理服务器上进行横向扩展会涉及到很多成本，包括硬件购置、软件许可、能源消耗、网络设施、维护人员以及管理软件等开销。
- **Virtual machine images**—One of the key benefits of microservices is their ability to quickly start up and shut down instances in response to scalability and service failure events. Virtual machines (VMs) are the heart and soul of the major cloud providers. 
- **Virtual container**—Virtual containers are a natural extension of deploying your microservices on a VM image. Rather than deploying a service to a full VM, many developers deploy their services as Docker containers (or equivalent container technology) to the cloud. Virtual containers run inside a VM, and using a virtual container, you can segregate a single VM into a series of self-contained processes that share the same image



云基础微服务的优势主要围绕**「弹性（elasticity）」**这一概念来展开的。-

- Server elasticity means that your applications can be more resilient. 如果微服务由于Bug导致下线，那么我们可以迅速再次启动该微服务来保证服务的可用性，同时争取到足够长的时间来修复Bug

- 弹性可以让我们动态地使用服务器资源（横向扩展），避免资源的浪费。

  

 For this book, **all the microservices and corresponding service infrastructure will be deployed to a CaaS-based cloud provider using Docker containers**. The most common characteristics of CaaS cloud providers are as follows：

- **「Simplified infrastructure management」**：New services can be started and stopped with simple API calls.
- **「Massive horizontal scalability」**：CaaS cloud providers allow you to quickly and succinctly start one or more instances of a service.
- **「High redundancy through geographic distribution」**：—By necessity, CaaS providers have multiple data centers. By deploying your microservices using a CaaS cloud provider, you can gain a higher level of redundancy beyond using clusters in a data center.

The services built in this book are packaged as Docker containers; the main reason is that Docker is deployable to all major cloud providers. 

## 微服务模式

Microservices are more than writing the business code. Writing a robust service includes considering several topics：

![image-20240105154743359](assets/image-20240105154743359.png)



上面这几个问题都可以通过微服务模式来解决。微服务模式主要用来解决分布式系统设计和管理的问题。具体包括：

- **Core development pattern**
- **Client resiliency patterns**
- **Logging and tracing patterns**
- **Build and deployment pattern**
- **Routing patterns**
- **Security patterns**
- **Application metrics patterns**



### Core microservice development pattern



![image-20240105155817162](assets/image-20240105155817162.png)

The core microservice development pattern addresses the basics of building a microservice.

- **「Service granularity」**服务粒度：Making a service too coarse-grained with responsibilities that overlap into different business-problems domains makes the service difficult to maintain and change over time. Making the service too fine-grained increases the overall complexity of the application and turns the service into a “dumb” （笨拙的）data abstraction layer with no logic except for that needed to access the data store.
- **「Communication protocols」**通信协议：How will developers communicate with your service? The first step is to define whether you want a synchronous or asynchronous protocol. 
  - **For synchronous,** the most common communication is HTTP-based REST using **XML (Extensible Markup Language)**, **JSON (JavaScript Object Notation)**, or a binary protocol such as **Thrift** to send data back and forth to your microservices. 
  - **For asynchronous,** the most popular protocol is AMQP (Advanced Message Queuing Protocol) using a one-to-one (queue) or a one-to-many (topic) with message brokers such as **RabbitMQ**, **Apache Kafka**, and **Amazon Simple Queue Service (SQS).** 
- **「Interface design」**接口设计：What’s the best way to design the actual service interfaces that developers are going to use to call your service?
- **「Configuration management of service」**服务的配置管理：How do you manage the configuration of your microservice so that it moves between different environments in the cloud?
- **「Event processing between services」**服务之间的事件处理：How do you decouple your microservice using events so that you minimize hardcoded dependencies between your services and increase the resiliency of your application?

### Microservice routing patterns

The microservice routing patterns deal with how a client application that wants to consume a microservice discovers the location of the service and is routed over to it. 

In a cloud-based application, it is possible to have hundreds of microservice instances running. To enforce security and content policies, it is required to abstract the physical IP address of those services and have a single point of entry for the service calls. How? The following patterns are going to answer that question.

- **「Service Discovery」**：With service discovery and its key feature, service registry, you can make your microservice discoverable so client applications can find them without having the location of the service hardcoded into their application. Remember the service discovery is an internal service, not a client-facing service.

  **Note that in this book, we use Netflix Eureka Service Discovery**, but there are other service registries such as etcd, Consul, and Apache Zookeeper.

- **「Service routing」：**With an API Gateway, you can provide a single entry point for all of your services so that security policies and routing rules are applied uniformly to multiple services and service instances in your microservices application

服务发现主要关注如何找到服务，而服务路由关注的是如何达到这些服务。

![image-20240105163545503](assets/image-20240105163545503.png)



### Microservice client resiliency（容错与韧性）

To prevent a problem in a service instance from cascading up and out to the consumers of the service, use the client resiliency patterns

- **Client-side load balancing**（**负载均衡**）：How you cache the location of your service instances on the service so that calls to multiple instances of a microservice are load balanced to all the health instances of that microservice
- **Circuit breaker pattern**（**熔断器模式**）：How you prevent a client from continuing to call a service that’s failing or suffering performance problems. When a service is running slowly, it consumes resources on the client calling it. You want these microservice calls to fail fast so that the calling client can quickly respond and take appropriate action.
- **Fallback pattern**：When a service call fails, how you provide a “plug-in” mechanism that allows the service client to try to carry out its work through alternative means other than the microservice being called.
- **Bulkhead pattern**（隔舱模式）：Microservice applications use multiple distributed resources to carry out their work. This pattern refers to how you compartmentalize（隔离） these calls so that the misbehavior of one service call doesn’t negatively impact（负面影响） the rest of the application

![image-20240105171700906](assets/image-20240105171700906.png)

###  Microservice security patterns

it is important to apply the following security patterns to the architecture in order to ensure that only granted requests with proper credentials can invoke the services

![image-20240105171922450](assets/image-20240105171922450.png)

- **Authentication**—How you determine the service client calling the service is who they say they are
- **Authorization**—How you determine whether the service client calling a microservice is allowed to undertake the action they’re trying to take
- **Credential management and propagation**—How you prevent a service client from constantly having to present their credentials for service calls involved in a transaction.（避免反复授权验证） To achieve this, we’ll look at how you can use token-based security standards such as **OAuth2** and **JSON Web Tokens (JWT)** to obtain a token that can be passed from service call to service call to authenticate and authorize the user.

> OAuth2 is a token-based security framework that allows a user to authenticate themselves with a third-party authentication service. If the user successfully authenticates, they will be presented with a token that must be sent with every request.  The main goal behind OAuth2 is that when multiple services are called to fulfill a user’s request, the user can be authenticated by each service without having to present their credentials to each service processing their request.



### Microservice logging and tracing patterns

The downside of a microservice architecture is that it’s much more difficult to debug, trace, and monitor the issues because one simple action can trigger numerous microservice calls within your application. The following three core logging and tracing patterns to achieve distributed tracing:

- **Log correlation **日志相关性：How you tie together all the logs produced between services for a single user transaction
- **Log aggregation ** 日志聚合性：With this pattern, we’ll look at how to pull together all of the logs produced by your microservices into a single queryable database across all the services involved
- **Microservice tracing**：We’ll explore how to visualize the flow of a client transaction across all the services involved and understand the performance characteristics of the transaction’s services.

![image-20240105171041989](assets/image-20240105171041989.png)





we can o implement distributed tracing with Spring Cloud Sleuth, Zipkin, and the ELK Stack. t

###  Application metrics pattern

The application metrics pattern deals with how the application is going to monitor metrics and warn of possible causes of failure within our applications or performance issues. 

This pattern contains the following three main components:

- Metrics—How you create critical information about the health of your application and how to expose those metrics
- Metrics service—Where you can store and query the application metrics
- Metrics visualization suite—Where you can visualize business-related time data for the application and infrastructur



The metrics service can obtain the metrics using the pull or push style: 

- **With the push style**, the service instance invokes a service API exposed by the metrics service in order to send the application data.
- **With the pull style**, the metrics service asks or queries a function to fetch the application data.

![image-20240105165750616](assets/image-20240105165750616.png)



### Microservice build/deployment patterns

One of the core parts of a microservice architecture is that each instance of a microservice should be identical to all its other instances. You can’t allow **configuration drift** (something changes on a server after it’s been deployed) to occur because this can introduce instability in your applications.

这种模式的目标是将你的基础设施配置直接整合到你的构建/部署过程中。你需要将你的微服务和它运行的虚拟服务器镜像作为构建过程的一部分进行构建和编译。然后，当你的微服务被部署时，整个搭载了运行服务的机器镜像就会一起部署。

- **「Build and deployment pipelines」**：How you create a repeatable build and deployment process that emphasizes one-button（一键式） builds and deployment to any environment in your organization.
- **「Infrastructure as code」**：How you treat the provisioning of your services as code that can be executed and managed under source control（源代码管理）
- **「Immutable servers」**：Once a microservice image is created, how you ensure that it’s never changed after it has been deployed.
- **「Phoenix servers」**：How you ensure that servers that run individual containers get torn down on a regular basis（定期被关闭） and re-created from an immutable image. The longer a server is running, the more opportunity there is for configuration drift. A configuration drift can occur when ad hoc changes（临时的改变） to a system configuration are unrecorded.

Our goal with these patterns and topics is to ruthlessly expose and stamp out（无情地暴露并消除） configuration drift as quickly as possible before it can hit your upper environments (stage or production).

![image-20240105165222617](assets/image-20240105165222617.png)