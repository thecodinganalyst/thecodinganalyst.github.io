---
title: "Resolve git SSL certificate problem on windows"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Git
  - Windows
---

While trying to push my project to Azure DevOps on my corporate windows machine, I got this error message when I am trying to push my code - `SSL certificate problem: unable to get local issuer certificate`. 

Turns out according to this [article](https://confluence.atlassian.com/bitbucketserverkb/ssl-certificate-problem-unable-to-get-local-issuer-certificate-816521128.html), git by default uses the "linux" crypto backend, which is openssl, and I don't have openssl in my local machine. To fix the issue, I just have to amend the git configuration `http.sslbackend` to use Windows built-in networking layer - [SChannel](https://www.techtarget.com/searchsecurity/definition/Microsoft-Schannel-Microsoft-Secure-Channel).

```
git config --global http.sslbackend schannel
```

Doing so will use the Windows certificate storage mechanism when required. Now when I try to push again, I am through!
