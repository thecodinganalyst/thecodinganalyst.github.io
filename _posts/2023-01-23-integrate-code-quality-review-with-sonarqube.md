---
title: "Integrate code quality review with SonarQube"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Code review
  - SonarQube
  - BA
---

In today's environment, when we are having tons of dependencies in our application project, as developers we need to take care not just of the business logic, but also the code quality. Code quality is a wide spectrum including security, reliability, maintainability, test coverage, performance, etc. It is hard to quantify where our code stands according to widely accepted standards, yet it is all the more important to know where we stand especially if it the project is running in the public domain, susceptible to external attacks as well as online reviews. 

A good tool to use is [SonarQube](https://www.sonarsource.com/products/sonarqube/), which one can install it in your local computer and run an automated analysis on your code every time a code is committed. After scanning the code, Sonarqube provides a report of the analysis of your overall code, as well as the new codes commited. The overview shows the number of bugs, vulnerabilities, security hotspots, technical debt, code smells, etc.

![sonarqube dashboard](/assets/images/2023/01/sonarqube-dashboard.png)

It pinpoints all the issues with your code, so that you can know where to change your code. 

![sonarqube issues report](/assets/images/2023/01/sonarqube-issues.png)

And it also explains why each issue is flagged out, and provides examples for non-compliant and compliant samples.

![sonarqube reason](/assets/images/2023/01/sonarqube-reason.png)

In terms of the different measures like reliability, security, maintainability, test coverage, it provides individual ratings for each measure to let you know where your code stands.

![sonarqube measures](/assets/images/2023/01/sonarqube-measures.png)

Integrating sonarqube into our application project is easy, and I shall demonstrate how to do that with the project in [Spring boot application getting started](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/).

First and foremost, we have to install Sonarqube. Rather than installing it in my local computer, I'd prefer to get it in a docker container. Following the steps in https://docs.sonarqube.org/9.6/try-out-sonarqube/, run the following command to install the official sonarqube docker image to port 9000.

```
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```

Then you can access the sonarqube application on `http://localhost:9000/` in your local computer. Follow the steps in the link to setup a new project. 

After setting up our [project](https://github.com/thecodinganalyst/forum) in sonarqube, we just need to update our `build.gradle` with the sonarqube and jacoco plugin. Although sonarqube supports the reporting of test coverage as part of the analysis, it does not generate the coverage report itself, hence a third-party tool is required to generate the coverage report, and Jacoco is supported. To know more about Jacoco and test coverage, do refer to [Code coverage for Java - Jacoco](https://thecodinganalyst.github.io/software%20quality/Code-coverage-for-java-Jacoco/). 

```
plugins {
    id "org.sonarqube" version "3.5.0.2730"
    id "jacoco"
}
``` 

> if the `plugins` closure already exists, just add the 2 plugin lines in the existing `plugins` closure. 

Then we add the following block in our `build.gradle` so that the `jacocoTestReport` task will run at the end of the `test` task. We also make the `test` task a dependency of the `jacocoReport`, and enable the xml report from jacoco so that the result can be fed to sonarqube. 

```
test{
    finalizedBy jacocoTestReport
}

jacocoTestReport{
    dependsOn test
    reports {
        xml.enabled true
    }
}
```

Lastly, we add our project information as set up in sonarqube, to the properties of our sonarqube task.  

```
sonarqube {
    properties {
        property "sonar.host.url", property('sonar.host.url')
        property "sonar.projectKey", property('sonar.project.key')
        property "sonar.login", property('sonar.login')
    }
}
```

We are using [gradle project properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:project_properties) to supply the sonarqube credentials, so that we don't have to check in the credentials in our git repository. A new `gradle.properties` file should be created in the project root folder with values like such.

```
sonar.project.key=Forum
sonar.host.url=http://localhost:9000
sonar.login=GET_YOUR_OWN_TOKEN_DURING_SONARQUBE_PROJECT_SETUP
```

Now all is set, you should be able to see the new `jacocoTestReport` and `sonarqube` tasks under the `verification` group in your IDE's gradle plugin. Go ahead and run it, then open your sonarqube to view the code review results and improve your code.

A full example of this setup is available on my github repository in the [initial-gradle tag of the forum project](https://github.com/thecodinganalyst/forum/tree/gradle-sonarqube).

This is part of a series illustrating how to build a backend Spring boot application.
- [Getting Started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/)
- [Deploying to Docker](https://thecodinganalyst.github.io/tutorial/Deploying-mult-container-application-to-docker/)
- [Spring Data Testing](https://thecodinganalyst.github.io/tutorial/how-to-test-spring-data-repository/)
- [Testing Services](https://thecodinganalyst.github.io/tutorial/how-to-test-services-in-a-spring-boot-application/)
- [Unit Testing of Controller](https://thecodinganalyst.github.io/tutorial/how-to-unit-test-rest-controller-in-a-spring-boot-application/)
- [Integration Testing](https://thecodinganalyst.github.io/knowledgebase/how-to-do-integration-testing-in-spring-boot-rest-application/)
- [Code quality review with Sonarqube](https://www.thecodinganalyst.com/tutorial/integrate-code-quality-review-with-sonarqube/)
- [Configure Spring Security CSRF for testing on Swagger](https://www.thecodinganalyst.com/tutorial/Configure-spring-security-csrf-for-testing-on-swagger/)
- [Configure Access Management in Spring Security](https://www.thecodinganalyst.com/tutorial/how-to-configure-access-management-in-spring-security/)
- [Validate inputs in Spring Boot RestController](https://www.thecodinganalyst.com/tutorial/how-to-validate-input-in-spring-boot-restcontroller/)