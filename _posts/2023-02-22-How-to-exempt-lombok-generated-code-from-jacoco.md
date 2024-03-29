---
title: "How to exempt lombok generated code from Jacoco"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Lombok
  - Jacoco
  - Code Coverage
---

[Lombok](https://projectlombok.org/) makes life easier for java developers by generating the mundane code (e.g. getters and setters) automatically, but if you are running code coverage, your score will suffer because there aren't any unit tests for the ungenerated code. 

![Jacoco report without unit tests for ungenerated code](/assets/images/2023/02/jacoco-normal.png)

To resolve this, simply add a new file named `lombok.config` to your root directory with the following

```
config.stopBubbling = true
lombok.addLombokGeneratedAnnotation = true
```

The first `config.stopBubbling = true` tells Lombok not to look in the parent directory for configuration files. This is so because you can have the lombok config files in your hierarchy of folders, and the config files in the deeper folders can inherit the configurations from its ancestor folders and overwrite them. This configuration is `false` by default and it's a good practice to set it to `true` in the configuration of the root folder.

The second configuration `lombok.addLombokGeneratedAnnotation = true` will add the `@Lombok.Generated` annotation to the code it generated, and code coverage tools like Jacoco will skip those autogenerated code, so your code coverage score will not suffer, as there aren't unit tests for these generated codes and you shouldn't be wasting your time to write them either. 

![code with the @generated tags](/assets/images/2023/02/lombok-generated-tag.png)

And after cleaning, building and running code coverage again, your report should have a better score.

![Jacoco report skipping generated code](/assets/images/2023/02/jacoco-without-generated.png)
