---
title: "Enable accented characters on Mac OSX"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - MacOSX
---

I used to be able to type accented characters on Mac OSX by long pressing the character I want to accent. But this functionality was somehow gone after an OS update. After some googling, I found the following solution: 

```
defaults write -g ApplePressAndHoldEnabled -bool true
```

Run the above command in a terminal, and restart the computer for it to take effect. 

![Accented characters](/assets/images/2022/03/accented-characters-on-mac-osx.png)
