---
title: "Updating the remote to incorporate the new personal access token requirement on github"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Git
  - Github
---

Recently github has discontinued the using of username and password - https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/. So for existing repositories on your local computers, you'll need to update your remote url to incorporate the personal access token (aka PAT) in the url. 

First, head over to Github, click on the profile picture on the top right hand side, and navigate to Settings > Developer Settings > Personal access tokens to generate a token. Give it a name, so that you can identify when or why you generate it, so that you can know which one to delete next time if required. Copy and save the key somewhere so you can reuse it without having to generate a new one everything you create a new repo. 

Then in the project folder on your local computer, run the command `git remote set-url origin https://<github_username>:<PAT>@<github url>`. The <github-url> is usually in the format `github.com/<github_username>/<repo_name>.git`. 