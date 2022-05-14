---
title: "Basic guide to Semantic Release"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring
  - Java
  - Semantic Release
  - Semantic Versioning
  - GitHub Actions
  - Continuous Integration
---

When it comes to releasing software projects to production, versioning can be a topic that is easily overlooked as something that is too trivial and done manually. How do we determine the next version number? Should it be a major version  (v1 to v2) or a minor version (v1.1 to v1.2)? [Semantic Release](https://github.com/semantic-release/semantic-release) is a software used by software developers to manage the versioning of our code for us automatically.

Semantic Release follows the standard [Semantic Versioning](https://semver.org/) format of v{Major}{Minor}{Patch}, e.g. v1.2.3. So when you run semantic release on your project, semantic release will help to ensure that all versions created will strictly adhere to semantic versioning, so that you cannot jump versions (v1 to v3) or move backwards (v2.1 to v2.0). Some basic guidelines of semantic versioning is as follows:

. No modifications once a version is released

. Any change must be a new version

. Version numbers can only go up

. {Patch} → backward compatible bug fixes

. {Minor} → new backward compatible features, deprecation of public api

. {Major} → new backward incompatible api

. {Major} = 0 → Initial development when anything can change

. {Major} >= 1 → 1st public stable release

. Pre-release version → e.g. 1.0.0-alpha, 1.0.0-beta before 1.0.0

## How Semantic Release works?

Semantic Release is developed in NodeJS, but it can be used on any code base, e.g. java, kotlin, c#, golang, etc. Why? Because it is not exactly part of our code (except for the configurations) but Semantic Release is used and installed in our Continuous Integration (CI) environment (e.g. GitHub, GitLab, etc). When there are any code changes (push, merge, etc) to our code repository, semantic release runs to determine if a new version is needed, and if so, it will create a [git tag](https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-tag) for it, and publish a release version in our git. 

How then does it know if a new version is required? It is determined by the commit messages when we commit our code. So, in order to use semantic release correctly, developers need to follow certain conventional commit message formats when committing. By default, semantic release follows the [Angular Commit Message Convention](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-format). Some basic guidelines are as follows:

. Commit header format: <Type>(Scope): <Short Summary>

    . Type and Short summary are mandatory

    . Type: **fix**, **feat**, perf, ci, build, docs, refactor

    . Scope: affected package

. Use imperative present tense. E.g “fix” not “fixes” or “fixed”

. Don’t capitalise first letter

. No “.” at end of sentence

. Commit footer format (if necessary)

    . **BREAKING CHANGE**: <Summary of breaking change> <blank line> <description if any>

    . DEPRECATED: <What is deprecated> <blank line> <description if any>

The most common way of determining the version number is with the `fix` and `feat` commit header type. 

| Commit Header                                                            | Change | Release |
| fix(core): fix bug A                                                     | Patch  | Fix     |
| feat(core): new feature B                                                | Minor  | Feature |
| perf(core): performance improvement on C <br/> BREAKING CHANGE: Change D | Major  | Breaking|

From the above table, if you have a `fix: xxx` or `fix(scope): xxx`  in your commit message, when semantic release runs, it will create a patch version. So if the current version is 2.1.0, the release will be 2.1.1. Similarly, if you have a `feat: xxx` message, the next release version will be 2.2.0.

Internally, when semantic release is executed, there are 9 steps to be executed one after another, as follows:

1. verifyConditions

2. analyzeCommits

3. verifyRelease

4. generateNotes

5. prepare

6. publish

7. addChannel

8. success

9. failure

These steps are executed by plugins, and plugins are other npm modules that can implement one or more of the above steps. By default, there are only 4 plugins that comes together with semantic release. These 4 plugins and the steps which they implement are as follows:

1. @semantic-release/commit-analyzer - analyzeCommits

2. @semantic-release/release-notes-generator - generateNotes

3. @semantic-release/npm - verifyConditions, prepare, publish

4. @semantic-release/github - verifyConditions, publish, success, failure

As you can see, not all of the 9 steps are implemented. If it is not implemented, then the step is skipped. Only the analyzeCommits step is compulsory and the rest of the steps are all optional. During each step, semantic release will run every plugin defined, as long as the plugin implements the step. We can define what plugins to run in the configuration file for semantic release. This configuration file is usually the `.releaserc` file written in either json or yaml format in the main project folder. The list of official and community plugins are available on [https://semantic-release.gitbook.io/semantic-release/extending/plugins-list](https://semantic-release.gitbook.io/semantic-release/extending/plugins-list). More information on the format and other configurations available can be found at [https://semantic-release.gitbook.io/semantic-release/usage/configuration](https://semantic-release.gitbook.io/semantic-release/usage/configuration). One thing to note that, in order to run the non-default plugins, you will need to `npm install` the plugins in your CI file, as well as specifying the plugins in this configuration file. The example in the next section will demonstrate that by installing the `@semantic-release/git` and `@semantic-release/changelog` plugin in the GitHub Action. 

## Demonstration of semantic release on a Java project with GitHub Actions

To start having semantic release on a java project hosted on GitHub, the first step is to enable GitHub Actions. Click on the `Actions` tab in the github project page, and you will be presented with a variety of templates to choose from. Semantic Release documentation also have a template available on [https://github.com/semantic-release/semantic-release/blob/master/docs/recipes/github-actions.md](https://github.com/semantic-release/semantic-release/blob/master/docs/recipes/github-actions.md), but it is more applicable for a nodejs project, as it is using the default npm plugin which will publish the project to npm. For a java project, a typical workflow will be something as follows:

1. Checkout source code - actions/checkout@v2

2. Setup Java - actions/setup-java@v2

3. Setup Node - actions/setup-node@v2

4. Build/Test the java code

5. Run npm install to install semantic release and whichever plugins you need

6. Run semantic release

We still need to install nodejs in our VM, because semantic release will be running from the npx command. A very basic github action workflow script is as such

```.github\workflows\release.yml
name: Release

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

env:
  GH_TOKEN: {% raw %}${{ secrets.GH_TOKEN }}{% endraw %}

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
        cache: maven
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: 'lts/*'
    - name: Build with Maven
      run: mvn -B package --file pom.xml
    - name: setup semantic-release
      run: npm install -g semantic-release @semantic-release/git @semantic-release/changelog -D
    - name: release
      run: npx semantic-release

```

For checking out the codes and installing java and node, we are just using the templates available on [github marketplace](https://github.com/marketplace?category=&query=&type=actions&verification=), then we are just using shell scripts to run `mvn package` to package our code. If this is successful, meaning all the tests within the project passed, then it will move on to the next step to install semantic release and the plugins with `npm install`. Note that other than semantic-release, we are also installing 2 other plugins here. Lastly, we run the semantic release with `npx semantic-release`.

Do note that we had configured this workflow to only run in the master branch, whenever there is a push or pull request. We have also set up the environment variable named `GH_TOKEN`, and the value is linked to a secret defined in the current github project. This `GH_TOKEN` is a variable required by the [github semantic-release plugin](https://github.com/semantic-release/github) that we will be using to publish the release in github. Basically, the plugin need to be authenticated in order to perform the required git actions. So in this case, we need to create a variable named `GH_TOKEN` in the `Settings -> Secrets` of our github project, so that it is secured, and will not be downloaded when anyone clone the project. 

GitHub Action will automatically run this workflow after you commit this file in the master branch, and it will fail. The error message will probably be `ENOPKG Missing package.json file`. This is a failure in the `verifyConditions` step in the default `@semantic-release/npm` plugin. As we are not running a nodejs project, we don't have a package.json, and we are not going to use the npm plugin to publish the package to npm too. So we need to create a `.releaserc` file to overwrite the default plugins. A sample is as below:

```
{
  "branches": "master",
  "repositoryUrl": "https://github.com/thecodinganalyst/semantic-java",
  "debug": "false",
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "CHANGELOG.md",
        "changelogTitle": "# Semantic Versioning Changelog"
      }
    ],
    [
      "@semantic-release/git",
      {
        "assets": ["CHANGELOG.md"]
      }
    ],
    [
      "@semantic-release/github",
      {
        "assets": [
          {
            "path": "release/**"
          }
        ]
      }
    ]
  ]
}
```

We removed the npm plugin, and introduced 2 other plugins - `changelog` and `git`. `changelog` will help to create and maintain a `CHANGELOG.md` in our project folder every time a release is published. `git` will then include the CHANGELOG.md in the release. After we commit this file, the workflow will run again, and this time it will succeed. 

A sample repo of a working java project with semantic release running on github action is available at https://github.com/thecodinganalyst/semantic-java. 

