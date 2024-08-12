---
title: "Use artifact from another repository as a dependency in Azure DevOps (Part 1)"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Azure DevOps
  - Artifact Feed
  - Maven
  - Pipeline
  - CI/CD
---

In enterprise projects, it is quite common to have references to other projects as dependency. However, the dependent project is not supposed to be on the public domain. So within the enterprise, it is quite common to have an internal maven repository, using either [Sonatype Nexus](https://www.sonatype.com/products/sonatype-nexus-repository) or [JFrog Artifactory](https://jfrog.com/artifactory/). However, this might not be the case for a small startup. One alternative is to use what is available in the pipeline your project is using. For Azure DevOps, we can use the Artifacts Feed. 

In a simple example in my Azure DevOps project, I have 2 repositories - Common and Main. Common is a dependency in Main. To begin, I will create a Artifacts Feed for my artifacts, assigned with the relevant visibility, and calling it `Common-Feed`. 

So in my pipeline for Common, I will call `mvn deploy` to deploy the jar file to this feed, so that it can be consumed by Main. Usually, this will mean updating the `settings.xml` in the `Maven Home`, so I'll need some additional tasks in my pipeline to either get the file from the `Secured Files` and add it to the maven options for my deployment task. However, Azure DevOps has a handy shortcut to achieve that without the dirty work.

Before we run the deploy task, we'll need to call the [`MavenAuthenticate@0`](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/maven-authenticate-v0?view=azure-pipelines) task to do the authentication automatically. 

```
- task: MavenAuthenticate@0
  inputs:
    artifactsFeeds: Common-Feed
```

This step will update the `settings.xml` for us. Make sure to set the `artifactsFeed` to the name of the feed created earlier. 

Then in our deploy task, we set the deployment location to the url of the Artifacts Feed we created too, and **make sure the property `mavenAuthenticateFeed` is set to `false`**.

```
- task: Maven@4
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'deploy'
    options: '-Dmaven.test.skip=true -DaltDeploymentRepository=Common-Feed::default::https://pkgs.dev.azure.com/ORGANIZATION/PROJECT/_packaging/Common-Feed/maven/v1'
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '1.11'
    mavenAuthenticateFeed: false
```
Also do note thate the id of the `altDeploymentRepository` is the name of the feed too, in our case, it is `Common-Feed`, as mentioned earlier. 

Then, we also need to add this new `Common-Feed` to our `pom.xml` as a [repository](https://maven.apache.org/pom.html#Repositories). 

```
<repositories>
    <repository>
        <id>Common-Feed</id>
        <url>https://pkgs.dev.azure.com/ORGANIZATION/PROJECT/_packaging/Common-Feed/maven/v1</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

The full azure-pipelines.xml is as below. 

```
trigger:
  - main

pool:
  vmImage: ubuntu-latest

steps:
  - checkout: self
  - task: JavaToolInstaller@0
    inputs:
      versionSpec: '11'
      jdkArchitectureOption: 'x64'
      jdkSourceOption: 'PreInstalled'
  - task: MavenAuthenticate@0
    inputs:
      artifactsFeeds: Common-Feed
  - task: Maven@4
    inputs:
      mavenPomFile: 'pom.xml'
      goals: 'clean install'
      publishJUnitResults: true
      testResultsFiles: '**/surefire-reports/TEST-*.xml'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.11'
      mavenVersionOption: 'Default'
      mavenAuthenticateFeed: false
      effectivePomSkip: false
      sonarQubeRunAnalysis: false
- task: Maven@4
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'deploy'
    options: '-Dmaven.test.skip=true -DaltDeploymentRepository=Common-Feed::default::https://pkgs.dev.azure.com/ORGANIZATION/PROJECT/_packaging/Common-Feed/maven/v1'
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '1.11'
    mavenAuthenticateFeed: false
```

So after the feed runs, it will deploy an artifact to the Artifacts Feed, and it will be visible in the Artifacts tab in Azure DevOps. 

Do note that after the first artifact is available in the Artifacts Feed, you will need to update the version number of the project in the pom file every time you update the code.