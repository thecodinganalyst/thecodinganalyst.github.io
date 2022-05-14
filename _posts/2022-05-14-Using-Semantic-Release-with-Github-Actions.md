---
title: "Using Semantic Release with Github Actions"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Github Actions
  - Semantic Release
---

The purpose of having [Semantic-Release](https://github.com/semantic-release/semantic-release) is so that we can release workable versions to our users frequently. So we shouldn't be releasing non-workable versions, so before we began to use semantic release, we should have our working branches ready, and releases should only happen in the master branch. 

For today's example, we are setting up semantic release on a nodejs angular application. We start by installing the [Semantic Release npm package](https://www.npmjs.com/package/semantic-release) into our dev dependencies.

```
npm install -D semantic-release
```

Then we create a configuration file for our semantic release, named `.releaserc`. 

```
branches:
  - master
  - name: alpha
    prerelease: true
debug: true
ci: true
dryRun: false
plugins:
  - "@semantic-release/commit-analyzer"
  - "@semantic-release/release-notes-generator"
  - "@semantic-release/github"

```

In our example above, I am enabling semantic-release to only run on the `master` branch or `alpha` branch only. And we indicate that `alpha` is a prerelease branch. We also overwrite the default plugins, to remove the `@semantic-release/npm` plugin as we have no intention to publish this to the npm registry. 

Before we start to create our workflow, we need to create a repository secret, named `GH_TOKEN` in our github repository. This is a hard requirement from the default `@semantic-release/github` plugin, so that it can verify that the github repository is accessible, and it can create tags in our repository. So follow this [guide](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to create your personal access token on github. Then click on the `Settings` tab in your github repository, select `Secrets` > `Actions` from the menu, and click on the `New repository secret` to create a secret named `GH_TOKEN` with the personal access token you just created. 

The above token is just for verifying the access, when the github action runs, it uses an [automatically generated token]((https://docs.github.com/en/actions/security-guides/automatic-token-authentication)) named `GITHUB_TOKEN` to run the actions required. In order for it to do so, we need to allow the automatically generated token to have write permission. So, in the `Settings`, go to `Actions` > `General`, and scroll down to the `Workflow permissions` section, and select the `Read and write permissions` radio button. Don't forget to click the save button.

![github action token permission](/assets/images/2022/05/github-action-token-permission.png)

Then, we can head over to our github to create the github action workflow so that semantic-release can be triggered. Click on the `Actions` tab in your github repository, and select the Node.js workflow. Adjust the script to be as below and commit directly to the master branch. It will be saved to the folder `.github/workflows`.

```
name: Node.js CI

on:
  push:
    branches: [ master, alpha ]
  pull_request:
    branches: [ master, alpha ]

env:
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test -- --watch=false --browsers=ChromeHeadless
    - run: npx semantic-release
```

In the above script, we create an environment variable in our script called `GH_TOKEN` as it is [required](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md#ci-configuration) by the @semantic-release/github plugin, and we assign the value of the automatically generated token by using `${{ secrets.GITHUB_TOKEN }}`.  

We also add the `-- --watch-false --browsers=ChromeHeadless`, because we are using Karma for testing, if by default `watch=true`, it will still hang there after finish running the tests. And there is no chrome browser in the github runner, so we need to specify `ChromeHeadless` for our browser when running the `npm test`.

In the last part, we added the `run: npx semantic-release` to run it.

So, every time we push or merge a pull request to either the master or alpha branch, github will automatically run the above script to provision a ubuntu, checkout the code from this repository, set up nodejs in the ubuntu runner, and run `npm ci` to install all the necessary dependencies specified in the package.json of this project, run the build and test, and finally run semantic-release. It will then go through the commits to determine if a release is required. If it is needed, a new release will be created.

To simulate creating a pre-release branch, since we haven't quite started working on the app itself, we probably shouldn't have a release yet. So we'll create a branch named `alpha`, create a text with some mock features, and commit with the message `feat(feat1): mock feature 1`, and push it to our repo. As expected, semantic release will run and although it is only a new `feat` message that should only warrant a minor release, but since we are on version 0, and the recommendation of semantic release is to start from v1.0.0, the new release will be `v1.0.0-alpha.1`. Subsequent release in this alpha branch will be `v1.0.0-alpha.2` and so on. 

Then when we are ready to create our production version, we create a pull request to merge the alpha branch to our master branch. Semantic release will then run and create `v1.0.0`. 

The sample for the above example is available on [https://github.com/thecodinganalyst/semantic-node](https://github.com/thecodinganalyst/semantic-node).

