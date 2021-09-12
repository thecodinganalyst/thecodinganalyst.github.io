---
title: "Mutation testing with Pitest"
excerpt_separator: "<!--more-->"
categories:
  - Software Quality
tags:
  - Mutation Testing
  - Java
  - Pitest
  - Gradle
---

Beyond just ensuring every line of code is executed with Code Coverage test, there is another way to ensure the test cases written are adequate, and that is mutation testing. In simple terms, mutation testing is done by having a software to mutate small operations in the bytecode, like mutating an increment `i++` to a decrement `i--`, and expect the test case to fail. The test case should fail, because if something changes in the actual code and the test case which is supposed to ensure the code is doing what it needs to do didn't fail, that means the test case isn't adequate. The mutation is defined as `killed` when the test fails, and likewise the mutation will be regarded as `survived` if the test didn't fail. 

Doing these isn't a small feat, especially as it is in the bytecode level. There could be a lot of mutations available, and the whole test suite is ran for each mutation. [Pitest](https://pitest.org/) is such a software that enables this kind of testing. 

Setting it up in gradle is simple. First, add the [plugin](https://gradle-pitest-plugin.solidsoft.info/) to the build.gradle

```
plugins {
    id 'info.solidsoft.pitest' version '1.5.1'
}
```

Then add the configuration in the lower section of the same file. Example below, not that `com.hevlar.accounting` is the package of my java project, and it should be changed according to what is your package.

```
pitest {
    targetClasses = ['com.hevlar.accounting.*']  //by default "${project.group}.*"
    targetTests = ['com.hevlar.accounting.*']
    threads = 4
    outputFormats = ['XML', 'HTML']
    timestampedReports = false
    junit5PluginVersion = '0.12'
    excludedTestClasses = ['*.*IT']
}
```

After refreshing gradle in the IDE, the `pitest` task should appear under the `verification` section of gradle. Double click to run the pitest, and a html report will be available in the `build/report/pitest` folder. 

As mutation testing is running on the bytecode level, it is not suitable for running integration tests where database is accessed. Such tests do not run isolated in memory, but access their environment (file system, database, etc), and introducing arbitrary errors might have unexpected results [Source](https://github.com/hcoles/pitest/issues/170#issuecomment-231168929). It happened to me that all my unit tests and integration tests passed when I ran them, but pitest complains that some of my integration tests failed without mutations, which shouldn't be the case. So it seems like it isn't suitable for such environment. As in my configuration above, i added the `excludedTestClasses` to exclude my integration tests, ending with `IT` from pitest. 

