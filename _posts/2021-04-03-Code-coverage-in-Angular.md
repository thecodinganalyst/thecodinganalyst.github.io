---
title: "Code coverage in Angular"
excerpt_separator: "<!--more-->"
categories:
  - Software Quality
tags:
  - Angular
  - Istanbul JS
  - Code Coverage
  - Javascript
---

Unit testing has been the quintessential way to ensure whatever code we add to our program doesn't break anything, but how do we tell if the unit tests are adequate? 

An easy way to determine is by running the test cases through your code, and keep track of which lines in the actual code are run, how many times, and which are not. From the results, we can determine a percentage of how much code is covered by the test cases. It might not be absolutely foolproof to ensure your code is working as it should be, but it's something, and a pretty decent way.

[Istanbul JS](https://istanbul.js.org/) is one such utility for javascript which does exactly that, and it comes together with [Angular](https://angular.io/).

<!--more-->

Simply run `ng test --no-watch --code-coverage` if you are using Angular, and the test cases will run automatically and the summary will be displayed in the terminal. Something like this

`
=============================== Coverage summary ===============================
Statements   : 97.12% ( 101/104 )
Branches     : 82.76% ( 24/29 )
Functions    : 96.77% ( 30/31 )
Lines        : 96.91% ( 94/97 )
================================================================================
`

And a folder named `coverage` will be created containing a html report which you can navigate to see which are the lines in which file that didnt meet it's mark. 

![IstanbulJS Code Coverage Report](/assets/images/2020/04/IstanbulJs_Code_Coverage_Report.png)

Navigating through the folders and looking through individual files will show you which line of code is not run, any branches of code (e.g. if-else scenarios) you missed out, which function you have forgotten to test, etc.

![IstanbulJS Report Details highlighting row of code that is not run](/assets/images/2020/04/IstanbulJs_Code_Coverage_Report.png)

Pretty neat to get you started with making sure you write your test cases, and do it enough. One might get obsessed with getting 100% for every segment, though probably a green rating is good enough. Yes, the results are color coded on how well you scored. 