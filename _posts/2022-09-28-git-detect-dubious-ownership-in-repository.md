---
title: "Git detect dubious ownership in repository"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Git
  - Windows
  - SID
  - Security
---

When working in the corporate environment, we usually have a designated user space for each authenticated user on our computer. For example, in Windows, if our username is `abc`, we have a folder in `C:\Users`, named `C:\Users\abc` for all of user `abc`'s files. And if we put our git repository in this folder, you might get the following error message when you perform any git operations, like `git status`.

```
fatal: detected dubious ownership in repository at 'C:/Users/abc/Projects/my-awesome-project'
'C:/Users/abc/Projects/my-awesome-project' is owned by:
        'S-1-5-32-544'
but the current user is:
        'S-1-12-1-1347659835-1128888854-2982737882-1111120199'
To add an exception for this directory, call:

        git config --global --add safe.directory C:/Users/abc/Projects/my-awesome-project
```

It's nice to have the solution built in with the error message, but what does it mean, and why does it happen? And what are all the numbers in the user name?

Well, firstly the reason for the error message is because git will check if the current user is the owner of the git repository folder, and will produce the above error if the current user is not the owner. This is because when git will navigate to the `.git` folder directly under the git repository folder and execute some files there, in our case `C:\Users\abc\Projects\my-awesome-project\.git`. This can pose a security risk if the folder is not owned by the current user, and some malicious files can be placed there by others, which can compromise the computer when executed. This is explained in [https://github.com/git-for-windows/git/security/advisories/GHSA-gf48-x3vr-j5c3](https://github.com/git-for-windows/git/security/advisories/GHSA-gf48-x3vr-j5c3), and the commit that added the check is in [https://github.com/git/git/commit/8959555cee7ec045958f9b6dd62e541affb7e7d9](https://github.com/git/git/commit/8959555cee7ec045958f9b6dd62e541affb7e7d9), and it's available from git for windows v2.35.2 or newer.

So in our case, the `C:\Users\abc` folder was created by the system, and it is owned by `S-1-5-32-544`, not the current user `S-1-12-1-1347659835-1128888854-2982737882-1111120199`. So the error message appears.

The workaround provided by git is to add the current folder to the `safe.directory` global variable, so that git will regard the folder as safe. Hence it provides the solution `git config --global --add safe.directory C:/Users/abc/Projects/my-awesome-project`. Do note that even though we are in the windows environment, we need to use the forward slash `/` instead of the backslash `\`. Then you should be able to see the variable when you do `git config --global --list`. The safe directory is a multi-value variable, so we can add all the folder repos we need. 

> A full list of all git config variables is available at [https://git-scm.com/docs/git-config](https://git-scm.com/docs/git-config)

Let's address the numbers in the username. These are actually `SID`, which are like the `id`s of our user accounts. More information about SID at [https://www.techtarget.com/searchsecurity/definition/security-identifier](https://www.techtarget.com/searchsecurity/definition/security-identifier). You can get the SID of the current user by running the command `whoami /user`. The output will be something like this.

```
USER INFORMATION
----------------

User Name SID
========= ====================================================
ads\abc  S-1-12-1-1347659835-1128888854-2982737882-1111120199
```

Then who is `S-1-5-32-544`. Well, it is a well-known SID, referencing the `built-in Administrators group`, according to the [source](http://woshub.com/convert-sid-to-username-and-vice-versa/), and it is also published on the [Microsoft site](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids). 

> You can get the owner of the account by navigating to your repository folder and run `Get-Acl .` in powershell. 

Hope that explains, stay curious! 