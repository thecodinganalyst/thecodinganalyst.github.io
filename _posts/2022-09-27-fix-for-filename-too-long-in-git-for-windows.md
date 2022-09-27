---
title: "Fix for filename too long in Git for Windows"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Git
  - Windows
---

If you are running git on windows, with a deeply nested folder structure in your code, you might get the error message - "Filename too long". That is because in windows, git uses an older version of windows api where there's a 260 character limit for filename. It's not a limitation of git but msys, - details on [https://github.com/msysgit/git/pull/110](https://github.com/msysgit/git/pull/110).

The solution is to set the `core.longpaths` config to `true`

```
git config --system core.longpaths true
```
