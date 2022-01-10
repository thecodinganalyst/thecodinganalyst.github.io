---
title: "Dependency Injection"
excerpt_separator: "<!--more-->"
categories:
  - Software Engineering
tags:
  - Dependency Injection
---

Dependency Injection is like a job matching service for variables in your code. You specify the job description, in the form of interface, or in a less ideal world, a class. Then the container running the dependency injection will find the right class that fits the job description for you. 

To do this, there should be a service that is dedicated to this function, in the spirit of single responsibility. This service, let's call it `JobMatcher`, needs to know what are the candidates available and what they can do. This is just like a real world job agency, in order to provide candidates to employers, the agency needs to have a pool of candidates and know what are the skills of these candidates, in order to match them with a suitable employer.

There are 3 ways for the `JobMatcher` to get it's candidates.

1. Configuration file 
2. Decorator 
3. Manual registration in code


The first, and most flexible way to do this is to have a configuration file, indicating what are the candidates available. This is just like specifying the beans in a XML file in the spring framework. This is the most flexible because you can even change the configuration file at runtime, since it is just a text file. 

Sometimes, XML is cumbersome as it is too lengthy and you need to remember the attributes, so programmers find it easier to just brand the classes that are open to job opportunities with a decorator, like the `@Component` decorator in Spring. Simply add the `@Component` before the class decoration, then it will get noticed by the `JobMatcher` and it will be kept in a list. 

The least flexible way, is to manually add the class in the code. Like `JobMatcher.register(Component)`. 

---

At the other end of the bridge, the `JobMatcher` needs to match the classes to the `Employer`. But it can only do so if the `Employer` creates the job opening. There are 3 ways this can be done.

1. Constructor
2. Setter
3. Interface

The first way is to add the positions as parameters in the constructor. 

```
public class WebAgency implements Employer{

  public WebAgency(FrontendDeveloper frontend){
    this.frontend = frontend;
  }

}


public class FrontendDeveloper implements ICanDoHTML, ICanDoCSS, ICanDoJS{
  ...
}

```

The candidate can be something like this:

```
public class AngularFrontendDeveloper extends FrontendDeveloper{
  ...
}

```

With this constructor method, the `JobMatcher` will automatically provide `AngularFrontendDeveloper` to `WebAgency` when `WebAgency` is declared. However, you might noticed that this can't really work if I have to instantiate `WebAgency` manually, like

```
WebAgency bestWebAgency = new WebAgency(...)
```

It's `...` because if we declare a FrontendDeveloper and put it as the parameter of WebAgency's constructor, then it defeats the purpose. So usually in this case, the framework that provides the dependency injection, just need the developers to specify what are all the components and service endpoints available, then it will instantiate all of them automatically when required. This is another topic which I shall discuss in another article. 

The 2nd way is to ensure the `Employer` has a public setter method, so that the `JobMatcher` can have a means to supply the candidate. 

```
public class WebAgency implements Employer{
  ...

  public void setWebDesigner(WebDesigner designer){
    this.webdesigner = designer;
  }
  ...
}
```

The 3rd way is to specify it as a function which the `Employer` need to implement, and this will be the doorway which the `JobMatcher` can inject the candidate.

```
public interface Employer {
  public void setUXDesigner(UXDesigner designer);
}


public class WebAgency implements Employer{
  ...
  public void setUXDesigner(UXDesigner designer){
    this.uxDesigner = designer;
  }
  ...
}

```

No matter which way it is, the end game is that the `Employer` needs to provide an opening for the `JobMatcher` to inject the suitable candidate. That's why it is called `Dependency Injection`. 

