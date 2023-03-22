---
title: "How to fix a git rebase on a public master branch"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Git
  - Rebase
---

## What is Git Rebase

The `git rebase` is a command in Git that allows you to modify the commit history of a branch by moving or combining a sequence of commits to a new base commit, so that all the new commits in the feature branch will be together after the updated master branch where the feature branch branched out from.

![git rebase](/assets/images/2023/03/git-rebase.png)

This is usually the case when you are working on a feature branch, then there are some hotfixes in the master branch, which the feature branch should be built on top of. 

To rebase onto the master branch, simply run `git rebase master` while you are on the feature branch. Or if you are using an IDE like IntelliJ, you can simply right click the branch you want to rebase onto, then select `rebase 'feature' onto 'master'`.

![git rebase in intellij](/assets/images/2023/03/git-rebase-in-intellij.png)

## Pitfall

However, there's a certain pitfall to that. The [Golden Rule of Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) is to never use it on public branches. As in the marble diagram above, there are no issues if `A` and `C` have not been pushed to the remote. The final outcome will still follow the sequence `B`, `A`, `C` after pushing and merging to master. 

However, if `A` and `C` have already been pushed to remote, i.e. `A` and `C` are commits that you can see in your online repository (e.g. github, gitlab, bitbucket), the git rebase will make `A` and `C` in your local to become `A2` and `C2`. Meaning the changes are the same, but they will have different commit hashes. Do remember that this only happens in your local, so it is still just `A` and `C` in your remote. 

When you do a `git status`, git will identify that you doesn't have `A` and `C`, but you have new commits `B`, `A2`, and `C2`. So if you try to do a `git push`, it will not allow you to do so, but ask you to do a `git pull` first. And the result of doing that will be like in the diagram below, now you have all `A`, `C`, `A2`, and `C2`, which are duplicates. The `M` and `M2` are just the merge commits to show that merging was performed.

![git rebase scenario](/assets/images/2023/03/marble-git-rebase.png)

This is not what we want, as the history will not be clean.

## Solution

Then what can I do if this already happened? Unfortunately, the only way is to rewrite history if you want clean commits. It does not matter much if you are the only one working on the feature branch. It's your history, you can rewrite it the way you need. Furthermore, the feature branch is going to be deleted after merging to the master. 

So we use `git reset` to reset our local history to the commit just before we do the rebase. And to find the commit, we use `git reflog`, which gives us a list of all the actions we have performed.

![git reflog and git reset](/assets/images/2023/03/git-reflog.png)

As in the screenshot above, we find the commit just before the merge `f9b337f`, which is `d99d47e`, and we do `git reset --merge d99d47e`. Thus we are rewriting our local history to just before we did the `git pull`, after the `git-rebase`.

Now, instead of `git pull`, you can just force the history change in the remote with `git push --force`. If unfortunately someone is also working on the same feature branch, just get them to [stash their uncommitted changes](https://www.thecodinganalyst.com/knowledgebase/Stashing-local-configurations/) and pull the changes. 

## Bonus

If you already did more commits in the feature branch after the git pull (which is after M in the diagram titled `After Pulling`), you still can execute the same method above. After the reset, you can still use `git reflog` to get the hashes of all previous commits you have done. So you can [cherry-pick](https://git-scm.com/docs/git-cherry-pick) the commits into your now altered history before you do a force push. 


