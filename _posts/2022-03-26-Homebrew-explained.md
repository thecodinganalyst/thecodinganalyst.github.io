---
title: "Homebrew explained"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Homebrew
  - MacOSX
  - Cakebrew
---

## What is Homebrew

[Homebrew](https://brew.sh/) is the missing package manager for macOS. What does that mean? In layman terms, it is like the Appstore, but for opensource softwares, which are usually used for development, like node, java, mysql, etc. But you can also install normal software like chrome, firefox, acrobat, etc. 

But Homebrew is not meant for regular computer users, as it is a text-based terminal application. Meaning to use it, you need to open a terminal on your Mac, and type commands to run and use it. So it is usually meant for more tech-savvy users like developers. Though there is a user-interface software - [Cakebrew](https://www.cakebrew.com/) to run homebrew, you'll need to have homebrew installed first, and it still looks quite technical and not ready for non-technical users to understand. 

## Installation

To install homebrew, simply open a terminal in Mac and run `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

## Finding Software to Install

To list the softwares available to install, run `brew formulae`, or go to `https://formulae.brew.sh/`. Formulae are the more technical or text-based softwares like java, node, etc. Casks are the softwares with user interfaces like chrome, firefox. Use `brew casks` to list all the software with user interfaces. For simplicity sake, we'll address both of them as `software` in this article. 

If you have a particular software in mind, you can run `brew search <software>`. If it is available, it will be listed, whether it is a formulae or cask. For example, below is the result if you run `brew search java`.

```
==> Formulae
app-engine-java            java11                     libreadline-java
google-java-format         javacc                     pdftk-java
java ✔                     javarepl
java-service-wrapper       jslint4java

==> Casks
charles-applejava          java6                      eclipse-javascript
java-beta                  eclipse-java               oracle-jdk-javadoc
```

If you want to get a one-liner description of the software, run `brew desc <software>`. E.g. `brew desc java11`. 

```
openjdk@11: Development kit for the Java programming language
```

To get more information about it, run `brew info <software>`, e.g. `brew info node`, and it will show you the official website, the repository where this is installed from, the dependencies, whether you have already installed it, where is it installed, how many people has installed it for how many days, etc. 

```
node: stable 17.8.0 (bottled), HEAD
Platform built on V8 to build network applications
https://nodejs.org/
/usr/local/Cellar/node/17.6.0_1 (2,016 files, 46.0MB) *
  Poured from bottle on 2022-03-07 at 11:22:54
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/node.rb
License: MIT
==> Dependencies
Build: pkg-config ✔, python@3.9 ✘
Required: brotli ✔, c-ares ✔, icu4c ✔, libnghttp2 ✔, libuv ✘, openssl@1.1 ✘
==> Options
--HEAD
  Install HEAD version
==> Analytics
install: 531,246 (30 days), 1,290,818 (90 days), 4,970,569 (365 days)
install-on-request: 462,867 (30 days), 1,122,737 (90 days), 4,052,433 (365 days)
build-error: 535 (30 days)
```

## Installing Software with Homebrew

To install it, run `brew install <software>`, e.g. `brew install java`.
If it is a cask, you will need to run `brew install --cask <software>`, e.g. `brew install --cask cakebrew`.

To uninstall it, run `brew uninstall <software>`, e.g `brew uninstall java`.

By default, the formulae will be installed in the directory `/usr/local/Cellar/<software>/<version>`, whereas casks will be installed in the directory `/usr/local/Caskroom`. There will be a symlink created in `/usr/local/bin/<software>` so that the software is in your `$PATH` and can be accessible from any directory. In homebrew terms, `/usr/local/Cellar` is known as the `Cellar`. `/usr/local/Cellar/<software>` is known as the `rack`, and `/usr/local/Cellar/<software>/<version>` is known as the `keg`. So Cellar is the place where you keep everything, and each software is a rack in the cellar, and each version of the software is a keg. 

## Listing Software Installed with Homebrew

To list all the installed software, run `brew list`.

## Upgrade Software Installed with Homebrew

To upgrade any of the software installed, run `brew upgrade <software>`, and it will upgrade the software to its latest version. Run `brew upgrade` to upgrade all available softwares installed by homebrew. However, if you have a specific software which you don't want it to be upgraded, run `brew pin <software>`, and it will not be upgraded. To allow it to be upgraded again, run `brew unpin <software>`.

## Updating Homebrew

To update Homebrew itself, run `brew update`.

## Periodic Cleanup

Periodically, run `brew cleanup` to remove outdated files. Sometimes, when you install a software, it will install other software as dependencies. And if you uninstall the main software, the dependent software might not get uninstalled, so run `brew autoremove` to do that. You can run `brew autoremove --dry-run` to list what will be autoremoved without actually removing the unused software.

## Installing Multiple Versions of the Same Software

Sometimes you want to install multiple versions of the same software, take for example [NodeJs](https://nodejs.org/en/). To get the latest version of node, simply run `brew install node`. After installation, a symlink `/usr/local/bin/node` will be created, pointing to the latest node version (e.g. version 17) installed by homebrew, for example `node -> ../Cellar/node/17.6.0_1/bin/node`. So that when you run `node` on your terminal, it will be running the latest version of node. It will be updated to the latest node version when you run `brew upgrade node` or `brew upgrade`. But there might be times when you need to work on a legacy application that is running on node version 12. You can install the version 12 by running `brew install node@12`, and it will be installed into `/usr/local/bin/node@12`. But node@12 is keg-only, meaning a symlink will not be created at `/usr/local/bin/node` to point at it, as this symlink is reserved solely for the latest version of node. So when you run `node --version`, it will still point to the latest version, example version 17. 

To use node12, you have to unlink the existing node by running `brew unlink node`, then run `brew link node@12`. After that when you run `node --version`, you can see that it will be version 12 instead. To bring it back to the latest version, run `brew unlink node@12`, then run `brew link node`. 

## Other Sources

By default, all the software available for installation are from the Homebrew repository, so it is maintained by Homebrew. There are also third-party lists you can tap from, example mongodb/brew, aws/tap. To add a third-party list, simply run `brew tap <third-party-tap>`, for example `brew tap mongodb/brew`. 

To list the taps you have added, run `brew tap`. To find out more about the tap, like which repository is it from, and where is it installed in your system, run `brew tap-info <tap-name>`. 

To remove a tap, run `brew untap <tap-name>`. 

Usually, you will encounter the tap when you have a software in mind which you want to install, and the information will be available in the website of the software. For example, when you want to install the cli for AWS SAM, you can find the information to install it via homebrew at https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-mac.html.
