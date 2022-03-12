---
title: "Adding Swagger to Kotlin Spring"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Kotlin
  - Spring
  - Swagger
  - documentation
  - SpringDoc
---

Spring makes it extremely easy to create REST api, but as a consumer of APIs, we need documentations to know what is available and how to use it. That's where [Swagger](https://swagger.io/) comes in and solved the problem. 

In short, Swagger defines the structure of the API documentation in the form of a json schema, specifying what are the paths available, the parameters required by the various paths, and what type of results can be expected. As such, humans and machines both can have a standard way to read the api. Reading is easy for the human, the naming conventions are very well defined for most people to understand easily without the need to refer to the schema documentation. However, writing it can be tricky as you got to remember the exact wordings for each item. 

There's an easy way out for Spring users. By simply adding the [SpringDoc](https://springdoc.org/) dependencies to your classpath, the Swagger documentation can be generated automatically when you run the project. 

```
dependencies {
  ...
  implementation("org.springdoc:springdoc-openapi-data-rest")
  implementation("org.springdoc:springdoc-openapi-ui")
  implementation("org.springdoc:springdoc-openapi-kotlin")
}
```

After you `bootRun` the application, the swagger ui will be available on your application accessible by the path -  `/swagger-ui/index.html`, e.g. `http://localhost:8080/swagger-ui/index.html`.

![Swagger UI](/assets/images/2022/03/swagger.png)

And the link to the actual json schema file is on the page `/v3/api-docs`, e.g. `http://localhost:8080/v3/api-docs`.