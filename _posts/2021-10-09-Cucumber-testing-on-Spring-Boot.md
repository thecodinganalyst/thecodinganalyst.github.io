---
title: "Cucumber testing on Spring Boot"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring
  - Spring Boot
  - Cucumber
  - BDD
---

[Cucumber](https://cucumber.io/) is a great way for users to automate acceptance testing, as it allows using somewhat natural language to define the test cases. With a little learning and practise, business analysts, who aren't that technical to write test cases in code, can use it to define their test scenarios. Cucumber can sit somewhere between the business and the technical, having the business analysts to write the features, while the QA or developer can furnish the glue code so that the tests can be automated.

In this article, I shall cover on the technical side of how to implement cucumber into a spring boot project. The codebase of the example used here are available on [https://github.com/thecodinganalyst/SpringCucumber](https://github.com/thecodinganalyst/SpringCucumber). 

<!--more-->

I'm using [gradle build tool](https://gradle.org/) for this example. 

Firstly, within the ``build.gradle`, add the dependencies for cucumber and junit-vintage engine.

```
testImplementation 'io.cucumber:cucumber-java:6.10.4'
testImplementation 'io.cucumber:cucumber-junit:6.10.4'
implementation group: 'io.cucumber', name: 'cucumber-spring', version: '6.10.4'
testImplementation 'org.junit.vintage:junit-vintage-engine:5.7.2'
```

Within the same `build.gradle`, add the following configurations so that there is a `cucumber` task in the `verification` group, so that you can easily run the cucumber tests from your IDE, or call it from you CI.

```
// Cucumber Settings
configurations {
    cucumberRuntime {
        extendsFrom testImplementation
    }
}

task cucumber() {
    group "verification"
    dependsOn assemble, testClasses
    doLast {
        javaexec {
            main = "io.cucumber.core.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = ['--plugin', 'pretty', '--glue', 'com.hevlar.springcucumber', 'src/test/resources']
        }
    }
}
// Cucumber Settings
```

In the SpringBootTest, add the `@CucumberContextConfiguration` annotation. This will let cucumber aware of the test configurations in order to do all the dependency injections.
```
@CucumberContextConfiguration
@SpringBootTest
class SpringCucumberApplicationTests {

}

```

With the above setup, you can then proceed to add the `.feature` files containing the test cases in the [Gherkin](https://cucumber.io/docs/gherkin/reference/) syntax under the `src/test/resources` folder. Then add in your StepDefinition java code so that the gherkin can be translated to actual codes. With the above setup, you should be able to use `@Autowire` in your StepDefinition to autowire your classes to run the tests. I shall demonstrate how to write the gherkin test cases and StepDefinition in another article, as this article is purely for the setup. 

To include running the cucumber in the `test` task of gradle, create a blank `RunCucumberTests` class with the `@RunWith(Cucumber.class)` and `@CucumberOptions` annotation.

```
@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = {"pretty", "html:build/reports/tests/cucumber/cucumber-report.html"},
        features = {"src/test/resources"}
)
public class RunCucumberTests {
}
```

We are using `@RunWith` here instead of `@ExtendWith`, that's why we have the junit-vintage-engine in our dependencies.


Reference: [https://github.com/cucumber/cucumber-jvm/tree/main/spring](https://github.com/cucumber/cucumber-jvm/tree/main/spring)