---
title: "How to squash commits"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Git
  - Squash
  - Rebase
---

## Why do we need to squash commits

Sometimes when we are working on a feature, we might forget to run all the tests before we commit, and we could get test failures from our CI/CD process in the repository. That's fine, we can still fix it. But the resulting commit history might not look good, as we should have meaningful commit messages so that it's easy for someone else to understand what is supposed to change in a particular commit. As projects get bigger and more complex, teams getting larger and people move around quite often, the people fixing the bugs in the future might not have the same knowledge as the one who wrote the piece of code to be examined. So having meaningful commit messages helps people to understand what is supposed to happen.

![bad commit message](/assets/images/2023/03/git-log.png)

In the above example, the `fix test failure in feature1` and `fix linting failure in feature1` can ideally be combined in the commit of `feature1`, so that someone who needs to add on to feature1 in the future can get a full picture of what are the changes implemented to deliver `feature1`. This process of combining the commits is known as `squashing`

## How do we do it

There isn't a `git squash` command, but we are using `git rebase -i <hash before main commit to squash unto>`. In the above example, the commit with the message `feature1` is the `main commit to squash unto`, so the hash we need to use is the commit before it, which is the `setup`. Executing `git rebase -i 4f770f6` will open the default editor of your terminal (which is usually the VI editor) to edit the git commands.

![vi editor showing git commands](/assets/images/2023/03/git-message.png)

The lines starting with `#` doesn't matter, as they are just comments and are actually instructions to guide us how to amend the file.

To amend the file in VI, you would need to first go into the `Insert` mode by pressing `i`. The default mode when the file is opened is the `Command` mode, where you can execute the `write` and `exit` commands. You'll notice there bottom of the terminal is showing `-- INSERT --` to indicate that you are in the `Insert` mode.

![vi editor in insert mode](/assets/images/2023/03/git-message-insert-mode.png)

Then amend the 2 commits you want to squash by changing the `pick` to `squash`, as in the screenshot above. The `squash` commits must follow a `pick`, so that git knows which commit should these squash commits be squashed onto. So if you try to change the first commit (2f08fef) to `squash` also, it will result in an error.

Click the `Esc` key on your keyboard to go back to `Command` mode, and type `:wq` (meaning `write` and `quit`) to save and exit the VI editor.

![git squash message](/assets/images/2023/03/git-squash-message.png)

Now you will be presented with another VI editor to edit the commits. You can then edit the message to whichever you want it to be. For this example, I'm just going to delete the 2 messages which I don't need, and keep the original `feature1`. 

![git squashed messages](/assets/images/2023/03/git-squashed-message.png)

After all is done, use `git log --oneline` to check the commits

![git log updated](/assets/images/2023/03/git-log-oneline-after.png)

As you can see in the above screenshot, we don't have the unwanted lines anymore.
