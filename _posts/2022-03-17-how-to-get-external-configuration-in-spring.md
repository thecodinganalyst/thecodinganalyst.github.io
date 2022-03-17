---
title: "How to get external configuration in Spring"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring
  - External Configuration
  - Kotlin
---

Sometimes we have some configuration data that we need in our code, that can be changed periodically, like mail server information, proxy server, etc. We can store this data in an `application.properties` file under the `src/main/resources` or `src/test/resources` folder in our application folder. Such data should be in the `{key}={value}` format, e.g. `application.config.language=English`.

To enable that in Spring, first we need to add the `spring-boot-configuration-processor` dependency as an `annotationProcessor` in the `build.gradle` file. 

```Gradle
dependencies {
    ...

    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")
}
```

We'll need a class to store these configuration values, and these values will be properties in the class. Make sure the values should not be read-only, so the setters for these properties should be available. In Kotlin, we can skip the setter creation by using a data class, which will create the getters and setters automatically. Add the `@ConfigurationProperties` annotation to the class. An example is as below:

```Kotlin
@ConstructorBinding
@ConfigurationProperties(prefix = "application.config")
data class ExternalConfig(
    var language: String,
    var currency: String?,
    var countries: List<String>,
    var data: Map<String, String>,
    var address: Address
)
```

The `prefix` allows you to differentiate different groups of configuration in case you have multiple configuration classes for different usages. In the above example, the corresponding properties will be in the below format. 

```
application.config.language=English
application.config.countries[0]=Singapore
application.config.countries[1]=Australia
application.config.data.foo=foo
application.config.data.bar=bar
application.config.address.building=ABC Building
application.config.address.unit=01-01
application.config.address.street=Sterling Street
application.config.address.city=Singapore
application.config.address.country=Singapore
application.config.address.postalCode=123456
```

Notice we can have lists, maps and even objects in our configuration. The `@ConstructorBinding` annotation enables the configuration values to be injected in the constructor directly. 

Next, we need to add the `@EnableConfigurationProperties` annotation with the classes to exposed as the values, to the `Configuration` class of the spring boot project, or simple add it to the `@SpringBootApplication` class.

```Kotlin
@SpringBootApplication
@EnableConfigurationProperties(ExternalConfig::class)
class SpringExternalConfigApplication
```  

When we need to use the values, we can just inject the Configuration class to wherever we need it. For example, to use the `ExternalConfig` in our RestController, we can just inject it in the constructor as below:

```Kotlin
@RestController
class GreetingController(val externalConfig: ExternalConfig) {
...
}
```

A full working example is available in the repository - [https://github.com/thecodinganalyst/SpringExternalConfig](https://github.com/thecodinganalyst/SpringExternalConfig).

