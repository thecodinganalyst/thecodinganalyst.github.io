---
title: "Stashing local configurations"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Git
---

In the project I'm working on, there are some local configurations (typically database connection strings) that I do not push to the git remote. This causes some inconveniences whenever I am creating or switching branches, as I will need to create and edit the files every time I switch branches. `git stash` is the solution to deal with this hassle. 

Git Stash is a temporary storage on your local computer to store changes in your code which you are not ready to commit yet. Simply enter `git stash` will shelve all your uncommited tracked files. But in my case, I also created new config files which I don't `git add` to track it, so `git stash -u` will settle my case better. But in order to reuse the same stashed copy every time, it's easier if I give it a name, so `git stash save "local dev" -u` will do the trick better. Because you can stash multiple times, it's easy to forget which is the stash to apply, so in my case, I saved it with the name "local dev", so that it's easy to reference.

Then when I switch to a new branch, I will do a `git stash list` to show all the stashes available, example output below, so that I can identify my saved "local dev" stash.

```
stash@{0}: WIP on feature/1: xxxx
stash@{1}: WIP on feature/2: yyyy
stash@{2}: local dev

```

Then I do a `git stash apply stash@{2}` to apply the changes in my "local dev" stash, without deleting it, and there I go. 

FYI:

`git stash pop` will apply the latest stash, aka stash@{0} and delete the stash. 

`git stash drop` will just delete the latest stash, stash@{0}

`git stash drop stash@{1}` will delete the particular stash

`git stash clear` will clear all the stash entries.


