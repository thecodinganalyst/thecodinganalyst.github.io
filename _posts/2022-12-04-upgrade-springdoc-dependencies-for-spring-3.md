---
title: "Upgrade SpringDoc dependencies for Spring 3.0"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - SpringDoc
  - Spring 3.0
  - Spring
---

In my previous article on how to [add Swagger on Spring with Kotlin](https://thecodinganalyst.github.io/knowledgebase/Adding-swagger-to-Kotlin-Spring/), the examples were built on Spring 2. With the introduction of [Spring 3](https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga), there are some updates to the SpringDoc dependencies to use.

The previous dependencies were as such

```
dependencies {
  ...
  implementation("org.springdoc:springdoc-openapi-data-rest")
  implementation("org.springdoc:springdoc-openapi-ui")
  implementation("org.springdoc:springdoc-openapi-kotlin")
}
```

The main dependency used is the `springdoc-openapi-ui`, which is to be replaced by [`springdoc-openapi-starter-webmvc-ui`](https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-starter-webmvc-ui). Both the `springdoc-openapi-data-rest` and `springdoc-openapi-kotlin` can be replaced with just 1 other [`springdoc-openapi-starter-common`](https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-starter-common), as described on [https://springdoc.org/v2/#spring-data-rest-support](https://springdoc.org/v2/#spring-data-rest-support) and [https://springdoc.org/v2/#kotlin-support](https://springdoc.org/v2/#kotlin-support).

So the new dependencies if you are using Spring 3 will be as such 

```
implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.0")
implementation("org.springdoc:springdoc-openapi-starter-common:2.0.0")
```
