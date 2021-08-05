---
title: "Using Lombok in Intelli J"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Gradle
  - Java
  - IntelliJ
  - Lombok
---

Lombok is a pretty neat tool when coding in Java, allowing me to save time writing repetitive codes. But when you are installing it in a new project or in a new computer after a long while, you might forgot to do the necessary steps to get it work in your IDE.

Supposing you have the following class using lombok. 

```java
@Data
@AllArgsConstructor
public class Greeting {
    private long id;
    private String content;
}
```

It's so convenient without the need to write all the getter, setters and constructors. But you might see this following error when you try to compile.

```
error: constructor Greeting in class Greeting cannot be applied to given types;
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
               ^
  required: no arguments
  found: long,String
  reason: actual and formal argument lists differ in length

```

That is because lombok only generates the getter, setters, constructors during compilation, and these are not compiled. Something needs to be done in the project settings in the IDE to make sure this is done. 

<!--more-->

In IntelliJ, you'll need to ensure annotation processing is enabled. From the Preferences menu in IntelliJ, navigate to `Build, Execution, Deployment > Compiler > Annotation Processors`, then make sure the `Enable annotation processing` is checked. Keep the `Obtain processors from project classpath` checked too, so that the IDE can get what it needs from the project. 

![Enable annotation processing in IntelliJ Preferences](/assets/images/2020/08/intellij-lombok.png)

If you are using Maven, you are fine. But if you are using Gradle, just install the gradle lombok plugin in your `build.gradle`.

Referring to the [lombok setup guide for grade](https://projectlombok.org/setup/gradle), linking to the [gradle lombok plugin](https://plugins.gradle.org/plugin/io.freefair.lombok), you just need to add the line - `id "io.freefair.lombok" version "6.1.0-m3` in the `plugins` block in your `build.gradle` file. 

So it should look something like this.

```
plugins {
    id 'org.springframework.boot' version '2.5.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id "io.freefair.lombok" version "6.1.0-m3"
    id 'java'
}
```

This time, it should compile. 