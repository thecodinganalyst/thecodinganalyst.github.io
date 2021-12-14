---
title: "No tests found when running Cucumber JUnit"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Java
  - Kotlin
  - Cucumber
  - JUnit
---

Sometimes it can be pretty frustrating when the error message doesn't show the root cause of the problem, usually that's because it is not straightforward when we are using frameworks. I've just managed to get out of one while trying to setup Cucumber JUnit in my project.

```kotlin
@RunWith(Cucumber::class)
@CucumberOptions(
    plugin = ["pretty", "html:build/reports/tests/cucumber/cucumber-report.html"],
    features = ["src/test/resources/AccountController.feature"]
)
class RunCucumberTests {
}
```

I'm running the above code in my IntelliJ, build with gradle, configured with JUnit5. So it will run the gradle command `:test --tests "com.hevlar.eule.cucumber.RunCucumberTests"`, which will in turn run the [Cucumber](https://github.com/cucumber/cucumber-jvm/blob/main/junit/src/main/java/io/cucumber/junit/Cucumber.java) class. However, when I tried to run it, it will always give me `No tests found for given includes: [com.hevlar.eule.cucumber.RunCucumberTests](filter.includeTestsMatching)`. Took me a while to realise that it is due to the lack of the junit-vintage-engine dependency - `testRuntimeOnly("org.junit.vintage:junit-vintage-engine")`.

*Explanation*: 
Cucumber JUnit is currently only built with JUnit4, as evident in the use of the `@RunWith` annotation. It woould be using `@ExtendWith` if it is JUnit5. So for backward compatibility, the junit-vintage-engine is required. Without it, the class would not be recognized by the default JUnit5 engine, and so it can't detect the CucumberOptions feature property to run the features as tests.