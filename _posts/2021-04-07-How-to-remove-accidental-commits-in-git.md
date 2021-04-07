---
title: "How to remove accidental commits in git"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - git
---

This happens when I forgot to create the `.gitignore` file, resulting in the `target` folder containing the class files got uploaded to git.

## Solution

`git rm -r --cached --ignore-unmatch target/*` will send the command to delete the files. 

<!--more-->

- The `-r` is for recursion, so that any files and folders within the specified folder will be cleared. 
- The `--cached` parameter is so that it will keep the files on your local intact, since you most likely still need the files on your local, but not in your remote
- The `--ignore-unmatch` makes sure the command will still succeed if there are some files not matched. This happens when you have some files not added yet. 


Just like your `git add`, executing `git rm` does not make it happen until you commit. So a `git commit -m "delete file message"` is still required to confirm the deletion.

Then a `git push origin main` is required to push this change to your remote.

## Note

This only deletes the file, but the history of the files will still be there. So if you have uploaded sensitive information, like passwords or credit card information, these data can still be found in the history. For that case, a git filter-branch is required to clear the history.
