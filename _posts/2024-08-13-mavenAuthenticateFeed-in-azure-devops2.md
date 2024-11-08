---
title: "Use artifact from another repository as a dependency in Azure DevOps (Part 2)"
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
Now that we have successfully deployed the Common artifact to our Azure Artifacts Feed, we can set up the Main project to consume this dependency. This will allow us to share the Common project’s compiled code across other projects, streamlining dependency management within our Azure DevOps setup.

Step 1: Configure the Artifacts Feed in the Main Project
To access the artifact from the Common project, we need to add the Common-Feed as a repository in the Main project’s pom.xml. This will ensure Maven can resolve dependencies from this feed.

In the Main project’s pom.xml, add the following repository configuration:

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

Replace ORGANIZATION and PROJECT with the relevant names in your Azure DevOps organization.

Step 2: Add the Common Dependency
Next, add a dependency in the Main project’s pom.xml to specify the artifact from the Common project. Ensure that the artifactId, groupId, and version match those used in the Common project’s pom.xml.

```
<dependencies>
    <dependency>
        <groupId>com.yourcompany.common</groupId>
        <artifactId>common-artifact</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

After each update to the Common project, increment the version number to pull the latest changes.

Step 3: Update the Pipeline for Main
In the Main project pipeline, we’ll use MavenAuthenticate@0 to handle authentication for the feed and ensure Maven can access our internal artifact.

Here’s the updated pipeline YAML for the Main project, which will download and use the Common dependency from the Artifacts Feed.

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

  # Authenticate with the Common-Feed
  - task: MavenAuthenticate@0
    inputs:
      artifactsFeeds: Common-Feed

  # Build and test the Main project
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
```

Step 4: Verify the Dependency Resolution
After setting up the pipeline and pom.xml, run the pipeline in Azure DevOps. Maven should authenticate with Common-Feed automatically and retrieve the Common artifact. You should see the dependency being resolved in the pipeline logs during the build step.

Managing Artifact Versions
As noted previously, remember to update the version in pom.xml each time there’s a change in the Common project to reflect the latest updates.

Summary
By setting up an Artifacts Feed in Azure DevOps and using MavenAuthenticate@0, we have created a simple way for projects within an organization to share and manage dependencies without relying on public repositories. This setup is scalable and secure for any size of enterprise or startup, leveraging Azure DevOps’ integrated artifact management capabilities.