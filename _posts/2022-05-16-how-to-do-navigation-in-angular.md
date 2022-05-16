---
title: "How to do navigation in Angular"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - Router
---

When you create a new angular app using `ng new <application name>`, the first question you get asked is `? Would you like to add Angular routing? (y/N)`. If you answered `y`, then all the imports are set up for you, and you can skip the first 2 steps below. If not, it is not difficult to add them. 

Firstly, you create a new module file named `app-routing.module.ts` directly under your `/app` folder with the following content.

```
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

The `@NgModule` decorator identifies the class as a module, so that you can import it in your main `app.module.ts`. This module will import `RouterModule` and `Route` from `@angular/router`, specify all the routes available with the `routes` constant, and export the `RouterModule`.

Next, in your `app.module.ts`, add the newly created `AppRoutingModule` into your `imports`. You'll also need to manually add the import statement - `import { AppRoutingModule } from './app-routing.module';` if you are not using an IDE which can do that for you.

Then, in your main container component, usually where your navigation is, you add the `<router-outlet></router-outlet>` tag. This is like an `iframe` tag where all the changes to the components will take place. 

To create the links, simply go back to your `AppRoutingModule`, and add it to the `routes` like below.

```
const routes: Routes = [
  { path: "this", component: ThisComponent },
  { path: "that", component: ThatComponent }
];
```

You must have your components already created, and you decide what is the path that should link to the component.

Then, in your navigation page, add the `routerLink` attribute to your navigation component/tag with the desired `path` as the value.

```
<a routerLink="this">This</a>
```

Then, when the `a` is clicked, it will show `ThisComponent` inside the `<router-outlet></router-outlet>`.

Sample code available on [https://github.com/thecodinganalyst/router-demo](https://github.com/thecodinganalyst/router-demo).
