---
title: "Harmonizing line breaks while working with different OS"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Git
---

When working in a team, someone will be using windows, while some will be on mac. Usually, it won't be an issue, but windows and mac/linux has a different way to interpret line breaks. It's not displayed, but a line break consists of 2 characters in windows - `CRLF` (Carriage return & Line break), but it is only 1 character on mac/linux - `LF` (Line feed). 

Usually the IDE is already defaulted to update the line breaks to the system default, even when the commited code is using another system. Then test failures might occur when your code is comparing texts with line breaks. It occurred to me recently when I was working on a new project. The codes are using `LF`, and I happen to add a new unit test which compares some huge chunks of text, and I'm on windows with `CRLF`. 

The solution is to set the `autocrlf` config in my git to `true`. In this way, git will automatically convert the `CRLF` to `LF` when a file is added to the index, and `LF` to `CRLF` when code is checked out. 

For windows users, `git config --global core.autocrlf true`.

For linux users, who has someone on the team using windows, `git config --global core.autocrlf input`. This will convert `CRLF` to `LF` when code is committed, but will not convert `LF` to `CRLF` when you check out codes.


So in this way, the codes on the repository will keep to just using `LF`, but allowing windows users to have `CRLF` when the code is local. 

reference: [https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)