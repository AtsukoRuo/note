# Microservices with SpringCloud

> The more distributed a system is, the more places it can fail.



实现所有微服务模式将需要大量的工作。幸运的是，Spring团队将大量经过实战测试的开源项目整合到一个称为Spring Cloud的Spring子项目中(https://projects.spring.io/spring-cloud/).。Spring Cloud提供一组特性（如服务注册和发现、断路器、监控等），使我们能够快速构建需要最少配置的微服务架构。



![image-20240106114600693](assets/image-20240106114600693.png)



- **Spring Cloud Config**：Spring Cloud Config通过中心化的服务管理应用配置数据。你的应用配置数据（尤其是特定环境配置数据）被整洁地从部署的微服务中分离出来。这确保无论你启动多少微服务实例，他们的配置总是一致的。Spring Cloud Config有自己的属性管理库，也整合了如下的开源项目：

  - Git (https://git-scm.com/)：Spring Cloud Config integrates with a Git backend repository and reads the application’s configuration data from the repository
  - Consul (https://www.consul.io/)：—An open source service discovery that allows service instances to register themselves with a service. Service clients can then query Consul to find the location of their service instances. Consul also includes a key-value store database that Spring Cloud Config uses to store application configuration data.
  - Eureka (https://github.com/Netflix/eureka)：An open source Netflix project that, like Consul, offers similar service discovery capabilities. Eureka also has a key-value database that can be used with Spring Cloud Config.

- **Spring Cloud Service Discovery**：With Spring Cloud Service Discovery, you can abstract away the physical location (IP and/or server name) of where your servers are deployed from the clients consuming the service. Spring Cloud Service Discovery also handles the registration and deregistration of service instances as these are started and shut down. Spring Cloud Service Discovery can be implemented using the following services:

  - Consul (https:// www.consul.io/)
  - Zookeeper (https://spring.io/projects/spring-cloud-zookeeper)
  - Eureka (https://github.com/Netflix/eureka) 

- **Spring Cloud LoadBalancer and Resilience4j**：

  - **Resilience4j**（ https://github.com/resilience4j/resilience4j） —you can quickly implement service client resiliency patterns such as circuit breaker, retry, bulkhead, and more. 
  - **Spring Cloud LoadBalance** —While the Spring Cloud LoadBalancer project simplifies integrating with service discovery agents such as Eureka, it also provides client-side load balancing of calls from a service consumer

- **Spring Cloud API Gateway**：The API Gateway provides service-routing capabilities for your microservice application. Like the name says, it is a service gateway that proxies service requests and makes sure that all calls to your microservices go through a single “front door” before the targeted service is invoked. With this centralization of service calls, you can enforce standard service policies such as security authorization, authentication, content filtering, and routing rules. You can implement the API Gateway using Spring Cloud Gateway (https://spring.io/projects/spring-cloud-gateway).

- **Spring Cloud Stream**—Using Spring Cloud Stream, you can build intelligent microservices that use asynchronous events. You can also quickly integrate your microservices with message brokers such as RabbitMQ (https://www.rabbitmq .com) and Kafka (http://kafka.apache.org).

- **Spring Cloud Sleuth**—Spring Cloud Sleuth (https://cloud.spring.io/spring-cloud-sleuth/) lets you integrate **unique tracking identifiers**（correlation ID or  trace ID） into the HTTP calls and message channels (RabbitMQ, Apache Kafka) used within your application.  The real beauty of Spring Cloud Sleuth is seen when it’s combined with loggingaggregation technology tools like the ELK Stack (https://www.elastic.co/what-is/ elk-stack) and tracking tools like Zipkin (http://zipkin.io).

  The ELK Stack is the acronym for three open source projects: 

  - **Elasticsearch** (https://www.elastic.co) is a search and analytics engine. 
  - **Logstash** (https://www.elastic.co/products/logstash) is a server-side, data-processing pipeline that consumes data and then transforms it in order to send it to a “stash.”
  - **Kibana** (https://www.elastic.co/products/kibana) is a client UI that allows the user to query and visualize the data of the whole stack. 

- **Spring Cloud Security**：Spring Cloud Security (https://cloud.spring.io/spring-cloud-security/) is an authentication（认证） and authorization（授权） framework that controls who can access your services and what they can do with them. Because Spring Cloud Security is token-based, it allows services to communicate with one another through a token issued by an authentication server. Each service receiving an HTTP call can check the provided token to validate the user’s identity and their access rights. 

  Spring Cloud Security also supports JSON Web Tokens (JWT). JWT (https://jwt.io) standardizes the format for creating an OAuth2 token and normalizes digital signatures for a generated token.





## Case

~~~java
@SpringBootApplication
@RestController
@RequestMapping(value="hello")
@EnableEurekaClient 
//  This annotation tells your microservice to register itself with a Eureka service discovery agent because you’re going to use service discovery to look up remote REST service endpoints.

// Note that the configuration happens in a property file, giving the simple service the location and port number of a Eureka server to contact. 

public class Application {
    public static void main(String[] args) {
        SpringApplication.run(ContactServerAppApplication.class, args);
    }
    
    public String helloRemoteServiceCall(String firstName, String lastName) {
        
        // The RestTemplate class lets you pass in a logical service ID for the service you’re trying to invoke
        // Under the covers, the RestTemplate class contacts the Eureka service and looks up the physical location of one or more of the named service instances
        // The RestTemplate class also uses the Spring Cloud LoadBalancer library. This library retrieves a list of all the physical endpoints associated with a service. Every time the service is called by the client, it “round-robins” the call to different service instances without having to go through a centralized load balancer. By eliminating a centralized load balancer and moving it to the client, you eliminate another failure point (the load balancer going down) in your application infrastructure. 
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> restExchange = restTemplate.exchange(
            "http://logical-service-id/name/" + "{fisrName}/{lastNmae}", 
            HttpMethod.GET, 
            null, 
            String.class, 
            firstName, 
            lastName);
        
        return restExchange.getBody();
    }
    
	@RequestMapping(value="/{firstName}/{lastName}",
         method = RequestMethod.GET)
         public String hello(
             @PathVariable("firstName") String firstName,
             @PathVariable("lastName") String lastName) {
         return helloRemoteServiceCall(firstName, lastName);
     }
    
}
~~~



Spring Cloud allow us to quickly build microservice architectures with minimal configurations.  That’s the real beauty behind Spring Cloud.



## 云原生微服务



**Cloud** is a technology resource management system that lets you replace local machines and private data centers by using a **virtual infrastructure**. There are several levels or types of cloud applications：

- A **cloud-ready** application：A cloud-ready application is an application that was once used on a computer or on an onsite server. With the arrival of the cloud, these types of applications have moved from static to dynamic environments with the aim of running in the cloud. we need to externalize the application’s configuration so that it quickly adapts to different environments. By doing this, we can ensure the application will run on multiple environments without changing any source code during the builds. （是将传统应用无缝移植到云环境的一种技术）

- A **cloud-native** application is designed specifically for a cloud computing architecture to take advantage of all of its benefits and services.  When creating this type of application, developers divide the functions into microservices with scalable components like containers, enabling these to run on several servers. These services are then managed by virtual infrastructures through the DevOps processes with continuous delivery workflows.

  ![image-20240106144655742](assets/image-20240106144655742.png)

- ......





 The four principles of native cloud development are：

- **DevOps is the acronym for development (Dev) and operations (Ops)**：It refers to a software development methodology that focuses on communication, collaboration, and integration among software developers and IT operations. The main goal is to automate the software delivery processes and infrastructure changes at lower costs.
- **Microservices are small, loosely coupled, distributed services**：These allow you to take a large application and decompose it into easy-to-manage components with narrowly defined responsibilities.
- **Continuous delivery is a software development practice.**：With this practice, the process of delivering software is automated to allow short-term deliveries to a production environment.
-  **Containers are a natural extension of deploying your microservices on a virtual machine (VM) image**：Rather than deploying a service to a full VM, many developers deploy their services as Docker containers (or similar container technology) to the cloud.





## Twelve-Factor App

In order to build high-quality microservices, we will use Heroku’s best practice guide—called **the twelve-factor app**(https://12factor.net/)



![image-20240107100135534](assets/image-20240107100135534.png)



- **Codebase**：
- **Dependencies**
- **Config**
- **Backing Service**
- **Build, Release, Run**
- **Processes**
- **Port Binding**
- **Concurrency**
- **Disposability**
- **Dev/Prod Parity**
- **Logs**
- **Admin Processes**



























