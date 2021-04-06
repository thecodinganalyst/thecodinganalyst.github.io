---
title: "The fatal refusing to merge unrelated history"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - git
---

It occurs to me a few times when I have written my code, then I want to create a git repository to store it. Then I selected the option to create a README file in github when I am creating the repo. 

Then after I initialise the git locally and added the remote, I got the error `fatal: refusing to merge unrelated histories` when i try to pull.

## Solution

Add the *allow-unrelated-histories* tag in the git command.

`git pull origin main --allow-unrelated-histories`

This tag is also useful when the `.git` folder is accidentally deleted in the local. 