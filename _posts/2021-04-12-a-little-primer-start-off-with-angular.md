---
title: "A little primer to start off with Angular"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Angular
---

[Angular](https://angular.io/) is a javascript framework for creating websites. Instead of using regular html tags (like div, span, form, input), Angular lets you define new html tags with names so that it is more relatable to your context. These html tags that you define are called Components. 

However, our browsers only understand html tags that defined by [w3c](https://www.w3.org/TR/2012/WD-html-markup-20121025/elements.html), we can't just create a random html tag like `<word>something</word>`, and expect our browser to interprete it. So actually, the components which we create will all go through the compiler and the result is actually a compilation of html, css and javascripts. So what Angular does, is it sets up the environment for us to create our site with our self-defined tags, define how it looks and behave with css and javascript, and provide ways for us to test and deploy to regular html/css/js, so that it can be interpreted to be displayed correctly and behave accordingly in our regular browsers.

<!--more-->

The above is true, except for a little exception. In Angular, we write with typescript instead of javascript. Typescript is just like javascript, except that it has types (like string, number, boolean, etc) and many other features that a normal programming language will have. So it makes writing javascripts more structured. Similar to Angular, our browsers can't interprete typescripts, so a compiler will need to compile the typescript into javascript, before it can be deployed. 

## Set up

Other than the above, Angular also sets up environment for unit test, end-to-end testing, code coverage. So there's actually a lot of configurations and set-ups involved, that will be a pain if one does it himself. So Angular has some command lines to help us do all of these easily. We can start by installing the Angular Command Line (cli) by opening a terminal, and run the following command. Of course, (node and npm)[https://nodejs.org/en/download/] is a pre-requisite.

`npm install -g @angular/cli`

After we have angular cli installed, we can create a new angular project easily by runnning 

`ng new <project-name>`

It will begin by asking a few questions:

1. Do you want to enforce stricter type checking and stricter bundle budgets in t
he workspace? This setting helps improve maintainability and catch bugs ahead of time. For more information, see https://angular.io/strict
> Strict mode ensures that the code adheres to strict syntax checking, by having errors when part of your code is in the grey area, making it easier to maintain but those might not really become error in real life. But when it does, and strict mode is not enabled, the error might not be obvious to find.

2. Would you like to add Angular routing?
> Turn this on if you have multiple pages or screens for users to navigate within this project

3. Which stylesheet format would you like to use? (Use arrow keys)
```
  CSS
  SCSS   [ https://sass-lang.com/documentation/syntax#scss                ]
  Sass   [ https://sass-lang.com/documentation/syntax#the-indented-syntax ]
  Less   [ http://lesscss.org                                             ]
  Stylus [ https://stylus-lang.com                                        ]
```

> Let you choose which type of CSS to use for your component. Again, only CSS is accepted in browsers, the rest will be needed to be compiled to CSS, but Angular handles that for you.

Once it's done, a folder named by the project name you specified will be created for you with all the necessary files to configure typescript, lint, code compilation, unit tests, e2e tests, etc are all generated nicely for you. 

## Module

To keep this post simple, we'll just focus on the `src > app` folder, where your code should reside. Every angular project will be presided by a `Module`. The module is the entry point where you import all the dependencies you need. And when the project is to be used or imported into other projects, what we are importing is the Angular module. So the module is also what we are exporting. The `app.module.ts` is the default module definition which is created automatically when the `ng new <project-name>` is ran.

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Component

A component is also created by default, but it consists of 4 different files. Namely - `app.component.ts`, `app.component.html`, `app.component.css`, `app.component.spec.ts`.

The main file is `app.component.ts` where a typescript class is defined and exported as described by the `export class AppComponent {...}`. 

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'tasklist';
}
```

The `@Component` is the class decorator, and it helps to define the html tag to use for this component. In the example above, it is `<app-root></app-root>`. In order to describe how the `app-root` component will appear in the browser, we need to have some actual html tags to define what to display. This is known as the **Component Template**, and we have a separate html file for each component, and we define the location of the file with the `templateUrl` parameter in the `@Component` decorator. And we also have a separate css file just for this component too, and it's defined by the `styleUrls` parameter in the `@Component` decorator. Lastly, the `app.component.spec.ts` is the file for the unit test specially for this component. 

As mentioned in the beginning, what we are trying to achieve is just the creation of our self-defined html tag. And so, the 4 component files above are just so that the `<app-root></app-root>` tag can be legit. And to use the new html tag created, a html file is needed to contain it, and the html file is really what a browser is opening. As such, Angular has created an `index.html` file directly under the `src` folder when the project is created. 

## Testing how it works

To see how the application works, run the following command in the root folder of the project.

`ng serve`

Then open a browser and navigate to `http://localhost:4200`. Angular will compile everything that is needed, and start a local http server to serve the application. It's useful for testing, as everytime some parts of the code is updated, the application will recompile automatically, so that you can see the changes immediately.

## Production

The `ng serve` is only good for production as in real life, we probably need a more robust server application for handling the traffic and security, etc, that is not available in the little local server Angular used for testing. The following command will compile the index.html together with all the typescripts in a `dist` folder, so that you can copy the contents to an actual server. 

`ng build --prod`

> If you look into the folder, it's all just html, css and javascript, no typescripts, sass, etc.


