---
title: "How to rename a remote branch in Git"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Git
---

While working in a team, there are usually some naming conventions for the branches, like prefixing the branch with "feat/" for feature branch, or "fix/" for bug fixes. Then forgetful me will tend to forgot the prefix some of the times. So, here are the steps to rename a remote branch.

1. Rename the local branch

```
git branch -m <new name>
```

2. Push the branch and set a new upstream branch

```
git push origin -u <new name>
```

3. Delete the old remote branch

```
git push origin --delete <old name>
```

Explanation: After you renamed the branch locally, the upstream branch is not updated automatically, so a normal git push will still push the old name to the remote. So it is necessary to add the `-u` option to set a new upstream branch. After that, you basically have both the new and old branches on the remote. Thus, you will need to do a push with the delete option to delete the old branch on the remote. 