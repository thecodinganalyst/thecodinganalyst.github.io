---
title: "Quick start tutorial on MicroServices with Spring Cloud Netflix Eureka"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Java
  - Spring Cloud
  - Netflix Eureka
  - Microservices
  - OpenFeign
---

In traditional application development, we have everything under a single project. But when the project gets bigger, involving multiple teams, it might make sense to split a monolithic application into multiple smaller applications, each governed by a different team, and each of the smaller applications can still work together like one application. That is the concept of microservices.  

Splitting an application into different smaller applications inevitably comes with additional complexities. The most eminent one is that we now need a way for the applications to talk to one another. Unlike a monolith application, we cannot just import the files and call the functions. Since our applications now reside in different servers, we have to call one another over the network, usually with api calls. And in order to call other applications reliably, we need another service to register all the microservices, and provide the address of each microservice, so that we don't need to fix the api calls to certain ip addresses. Certain microservices can be expecting more load, and will be available in more than 1 instance, so the server will also need to provide load balancing to distribute the load. These are additional tasks for the application as a whole, on top of providing what the application is meant to do. So unless there is real requirement to split the applications to be hosted at different places, even while still on the same cloud environment, microservices can meant additional costs in time, complexity, and money. 

Because it is so complex, having a framework to take care of these additional complexities can really help. [Spring Cloud](https://spring.io/projects/spring-cloud) is the goto framework to use if you are using java and spring boot. There are a few implementations for the microservice architecture mentioned above that are directly supported by the spring cloud project, namely [Netflix Eureka](https://github.com/Netflix/eureka), [Hashicorp Consul](https://www.consul.io/), and [Apache Zookeeper](https://github.com/Netflix/eureka), serving more or less the same functionalities of service discovery, coordination, and configuration management in microservices architecture. In this article, I will describe how to create a microservice architecture using Netflix Eureka with the [Spring Cloud Netflix Eureka](https://spring.io/projects/spring-cloud-netflix) project.  

Considering a ISP Service Centre, where the ISP provides broadband, mobile, and cable tv services, and the service centre provides customer service to walk-in customers who wanted to get help for the services they subscribed from this ISP. There is a central queue, where walk-in customers will register their mobile number to get a queue ticket, regardless of which service they need help for.

![ISP Service Centre Scenario](/assets/images/2023/07/isp-service-centre.png)

To kickstart the development, we shall create a Eureka Server application to handle the registration of the microservices, we shall name the project - `EurekaServer`. In our IDE, we will create a new spring project with 2 dependencies - [`Spring Boot Starter Web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) and [`Spring Cloud Starter Netflix Eureka Server`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-server). Our `build.gradle` file will look something like this

```
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.2'
    id 'io.spring.dependency-management' version '1.1.2'
}

group = 'com.hevlar.queue'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

ext {
    set('springCloudVersion', "2022.0.3")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

tasks.named('test') {
    useJUnitPlatform()
}
```

Then in our application class, we simply add the `@EnableEurekaServer` annotation.

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

Then we populate the `src\main\resources\application.properties` with the application name, server, as well as the following eureka client properties.

```
spring.application.name=eureka-server
server.port=8761

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

The 2 eureka properties are required for the eureka server instance, so that it will not keep trying to ping the eureka server. That is because the eureka server is also a eureka client, and eureka clients are supposed to ping the server every now and then so that the server can know that it is alive. We don't need the server to do that, so we set these 2 properties to `false`.

That's it! If we just `boot-run` the application and navigate to `http://localhost:8761`, we should see the following page.

![eureka server](/assets/images/2023/07/eureka-server.png)

Next, we shall create the application for the central queue system, and we create another spring boot project named - `QueueProvider`, with the following dependencies - [`Spring Boot Starter Web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) and [`Spring Cloud Starter Netflix Eureka Client`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client). 

Our `build.gradle` will look like this 

```
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.2'
    id 'io.spring.dependency-management' version '1.1.2'
}

group = 'com.hevlar.queue'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

ext {
    set('springCloudVersion', "2022.0.3")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

tasks.named('test') {
    useJUnitPlatform()
}
```

And again, we update our `application.properties` with the application name and server port. 

```
spring.application.name=queue-provider
server.port=8100

eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

The `eureka.client.service-url.defaultZone` is a map of the urls to communicate with the eureka server, for the client to poll the server to let it know it's alive.


This `QueueProvider` application will serve a queue number api from the default host - `http://localhost:8100/`. For simplicity sake, we just update our application class with the `@RestController` annotation, and create a `@GetMapping` function to serve an [AtomicInteger](https://www.baeldung.com/java-atomic-variables), so that the queue number can be thread-safe. 

```
@SpringBootApplication
@EnableDiscoverClient
@RestController
public class QueueProviderApplication {

    AtomicInteger centralQueue = new AtomicInteger(1);

    public static void main(String[] args) {
        SpringApplication.run(QueueProviderApplication.class, args);
    }

    @GetMapping
    public Integer getQueue(){
        return centralQueue.getAndIncrement();
    }

}
```

The `@EnableDiscoveryClient` is part of the spring cloud ecosystem, and is used to enable service discovery capabilities in spring cloud applications. It also works with other implementations like Hashicorp Consul and Apache Zookeeper. Service discovery is a critical aspect in microservices architecture, where multiple services need to communicate with each other. In a microservices architecture, the services are dynamic, and can be added and removed under different scenarios based on scaling requirements, service failures, etc. Service discovery allow services to find and communicate with one another without relying on hardcoded URLs or IP addresses, which are not practical in a dynamic environment. With the `@EnableDiscoveryClient` annotation, the application will register itself with the chosen service registry (in our case, the Eureka server) with its name, instance id, port, etc, so that the service is visible to other services. Then it will also send regular heartbeats to the service registry so that it the service registry will know that the service is up. Then other services in the registry can lookup available instances of a particular service based on the service name. 

We can now `bootRun` this application, and navigate to `http://localhost:8100` to get our queue number served. Do refresh it a few times to see the queue number increments itself. 

We navigate back to our eureka server, and we should see the new QueueProvider instance in the `Instances currently registered with Eureka` section.

![QueueProvider instance](/assets/images/2023/07/queueprovider-detected.png)

Lastly, we want to create the `BroadbandQueue` application to simulate the broadband branch of the ISP, getting the central queue number from the `QueueProvider` application. So, we create another eureka client application with the same [`Spring Boot Starter Web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) and [`Spring Cloud Starter Netflix Eureka Client`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client) dependencies. And in order to call the api from `QueueManager`, we will need a web service client to call the api, and for this application, we are going to use [`OpenFeign`](https://spring.io/projects/spring-cloud-openfeign), adding [`Spring Cloud Starter OpenFeign`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-openfeign) to the dependencies. 

```
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.2'
    id 'io.spring.dependency-management' version '1.1.2'
}

group = 'com.hevlar.queue'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

ext {
    set('springCloudVersion', "2022.0.3")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

tasks.named('test') {
    useJUnitPlatform()
}
```

Just like the `QueueProvider`, we need to name our application name, define the port, and set the eureka server in our `application.properties`.

```
spring.application.name=broadband-queue
server.port=8200

eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

In our application class, we need to add the `@EnableDiscoveryClient` and `@EnableFeignClients` annotations.

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class BroadbandQueueApplication {

    public static void main(String[] args) {
        SpringApplication.run(BroadbandQueueApplication.class, args);
    }

}
```

The `@EnableFeignClients` will enable the automatic scanning of Feign client interfaces within the package. Feign is a declarative web service client developed by Netflix and integrated into the spring cloud ecosystem, providing an easy and concise way to interact with RESTful services. To use feign, we need to declare interfaces annotated with the `@FeignClient` interface, and provide the apis as the interface methods. The application annotated with `@EnableFeignClients` will scan for interfaces annotated with the `@FeignClient` and create proxy implementation for each discovered interface. Then these feign clients can be injected into other components and used to make http requests to the corresponding services. 

So we shall declare our feign interface - `external/QueueProvider`.

```
@FeignClient("queue-provider")
public interface QueueProvider {
    @RequestMapping(value = "/", method = RequestMethod.GET)
    Integer getQueue();
}
```

The value `queue-provider` is the service name of the QueueProvider service we created earlier. By default, the service name will be the spring application name defined in the `spring.application.name` property. It's good that because we are using Feign, we don't need to specify a hardcoded ip address like `http://localhost:8100`, but just the service name, so that it can be more dynamic. The `getQueue` method, annotated with the `@RequestMapping`, defines the actual api our application will need to call. In our example, we are just calling the root - `http://localhost:8100/`, so the value in the `@RequestMapping` is just `/`, and we define the method to use as `GET`. As defined in the return type of the `getQueue` method, the api is supposed to return just an Integer. 

So now that we have the api to use defined, we can now create the api we are going to expose in our `BroadbandQueue` application. In our example, we are going to expose the api `http://localhost:8200/queue/{mobileNo}`, which requires a `mobileNo` as the parameter. This simulates the scenario in a service centre, that a walk-in customer will need to enter his/her mobile number to register for a queue number, so that the application can send an sms to the customer when it is his/her turn. So now, we define a `BroadbandQueueController` class to define the api we are going to expose. 

```
@RestController
public class BroadbandQueueController {

    private final QueueProvider queueProvider;

    public BroadbandQueueController(QueueProvider queueProvider){
        this.queueProvider = queueProvider;
    }

    @RequestMapping("/queue/{mobileNo}")
    public BroadbandQueue registerQueue(@PathVariable @NonNull String mobileNo){
        return new BroadbandQueue(mobileNo, queueProvider.getQueue());
    }
}
```

The function returns a `BroadbandQueue` defined as such - 

```
public record BroadbandQueue(String mobileNo, Integer queueNo){}
```

So now, if we `bootRun` this `BroadbandQueue` application and look at our Eureka server, we can see it in the list of instances.

![BroadbandQueue instance added to the service registry](/assets/images/2023/07/broadbandqueue.png)

And we can navigate to the `BroadbandQueue` application on `http://localhost:8200/queue/{arbitrary mobile no}`, we can see it working.

![Broadband Queue application](/assets/images/2023/07/broadbandqueue-app.png)

You can refresh the page a few times to see the queue number increasing. 

In our `BroadbandQueueController`, we inject the `QueueProvider` feign client interface we defined earlier, and call the `getQueue()` method to get the queue number from the `QueueProvider` service, and pass the value in its own api. So in this way, we demonstrated how to call another service from a service. 

The source code for the example above is available on [https://github.com/thecodinganalyst/QueueSystem](https://github.com/thecodinganalyst/QueueSystem). 

