---
title: "The SOLID Principle in internet lingo"
excerpt_separator: "<!--more-->"
categories:
  - Software Engineering
tags:
  - SOLID
  - Single Responsibility
  - Open-Closed
  - Liskov Substitution
  - Interface Segregation
  - Dependency Inversion
  - interview questions
  - Object-oriented Design
---

The SOLID principle is one of the most common acronym to describe the principles of object-oriented programming, which is subset of the many principles introduced by Rob C Matin in his 2000 paper - *Design Principles and Design Patterns*, in the article - [PrinciplesOfOod](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod).

In my own words, the 5 principles can be summed up as such in today's internet lingo. 

|-----------------------|----------------------------------------------------------------------------------------------------------------|
| Single Responsibility | A class should only have 1 job                                                                                 |
| Open-Closed           | Because a class should have only 1 job, if more functionalities is required, extend the class, don't modify it |
| Liskov Substitution   | Since the class is extended, I can replace the original class with the extended class without problems         |
| Interface Segregation | An interface should also have only 1 job too                                                                   |
| Dependency Inversion  | Don't be racist, define the variable by who can do the job (use interface), not by lineage (class hierarchy)   |

With the above principles, it means that we should break down our application to be run by classes. Each class should be small enough for 1 purpose only. And we define the variables, we define by what it can do, which is by interface, not which class it should belong to. So that if there is another class who can do the same job better, we can give the job to that class. 
