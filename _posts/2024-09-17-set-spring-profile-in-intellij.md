---
title: "Set spring profile in IntelliJ"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - IntelliJ
  - Spring
---

To set the spring profile in IntelliJ, just edit the configuration, usually `spring-boot:run` if you are using maven, or `bootRun` if you are using gradle, and add the environment variable `SPRING_PROFILES_ACTIVE`.
In the below example `SPRING_PROFILES_ACTIVE=dev`, the spring profile is set to `dev`. 

![IntelliJ edit configuration](/assets/images/2024/09/intellij_spring_profile.png)

And to confirm that the profile is really activated, check that the log `The following 1 profile is active: ` followed by the activated profile name.
Example - `[demo] [           main] com.example.demo.DemoApplication         : The following 1 profile is active: "dev"`.

