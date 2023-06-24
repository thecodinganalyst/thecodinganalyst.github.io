---
title: "Why I choose gradle over maven"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Gradle
  - Maven
---

Ever since coming across [gradle](https://gradle.org/) as the build tool for java while working in Zuhlke, I have never looked back to using [Maven](https://maven.apache.org/). The immediate difference is obvious to me, as using XML in Maven is too verbose. There's too much text to glimpse through the configuration file. Whereas in gradle, the syntax of choice is [groovy](https://groovy-lang.org/), which might seem daunty at first because it is a new language (or dialect) to learn, but the first impression is that it is very similar to json. 

This is much akin like clean code, with lesser text in the syntax, it just makes everything so clear. And it's so much shorter!

![comparison between gradle and maven](/assets/images/2023/06/gradle-vs-maven.png)

While working on a technical assessment with a local bank recently, I was asked to create a project with maven. Dread it of course, I created the project in gradle and got chatgpt to convert my gradle to maven. Then I realise another resounding benefit of gradle over maven - Dependency Management.

The pom.xml provided by chatgpt is the exact conversion of the gradle build file, so by right it should work as is. However, when more than 1 dependency listed have the same sub-dependency, maven doesn't seem to be able to resolve the conflict. After a few trial and error, I figured out that it is probably due to the jackson dependency in both spring boot and modelmapper. So the solution is to exclude the jackson from one of the dependencies, which I will be removing from spring-boot. 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.0</version>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

A working copy of the project is available on [https://github.com/thecodinganalyst/FinancialInvestment/](https://github.com/thecodinganalyst/FinancialInvestment/). The respective gradle build file and pom.xml for maven are [https://github.com/thecodinganalyst/FinancialInvestment/blob/main/build.gradle](https://github.com/thecodinganalyst/FinancialInvestment/blob/main/build.gradle) and [https://github.com/thecodinganalyst/FinancialInvestment/blob/main/pom.xml](https://github.com/thecodinganalyst/FinancialInvestment/blob/main/pom.xml).