---
title: "How to use router-store in NgRx"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - NgRx
  - Router-Store
---

![NgRx router store](/assets/images/2022/08/router-store.png)

## What is the purpose of the router-store in NgRx?

As explained in my [previous article on NgRx](https://thecodinganalyst.github.io/knowledgebase/ngrx-explained/), the purpose of NgRx is to separate the data management part of our frontend 
application so that the code is easier to manage. We don't have to use emitters and inputs to pass data between components, but we reference the NgRx store directly for any data needs using actions and selectors. And sometimes the data we need is from the router, for example, the title of the route. We definitely can create `states` and `selectors` to retrieve the data using the normal NgRx way, however, NgRx provides the [Router Store](https://ngrx.io/guide/router-store) to make it easier to retrieve the navigation data. 

However, do note that all the data from the router store can actually be found in the [ActivateRoute](https://angular.io/api/router/ActivatedRoute). But if you are already using NgRx, getting the route data from ActivatedRoute defeats the purpose of separating the data from our user interface.

## Installation

```
ng add @ngrx/router-store@latest
```

## Usage

> Do note that `@ngrx/store` is required, `@ngrx/router-store` is an add-on and not meant to be standalone.

Add the `StoreRouterConnectingModule.forRoot()` to the `imports` array of your module. This should be done for you when you execute the above installation command.

Then add a new file `router.selectors.ts` to the same folder with all your other NgRx files.

```
import { getSelectors, RouterReducerState } from '@ngrx/router-store';

// `router` is used as the default feature name. You can use the feature name
// of your choice by creating a feature selector and pass it to the `getSelectors` function
// export const selectRouter = createFeatureSelector<RouterReducerState>('yourFeatureName');

export const {
  selectCurrentRoute, // select the current route
  selectFragment, // select the current route fragment
  selectQueryParams, // select the current route query params
  selectQueryParam, // factory function to select a query param
  selectRouteParams, // select the current route params
  selectRouteParam, // factory function to select a route param
  selectRouteData, // select the current route data
  selectUrl, // select the current url
  selectTitle, // Select the title if available
} = getSelectors();
```

The above code can also be found on the official NgRx router store page - [https://ngrx.io/guide/router-store/selectors](https://ngrx.io/guide/router-store/selectors). This provides all the selectors available in the router store to get whichever navigation data you need. 

Then you can just subscribe to the selector you need in your component and display it. For example, if you just want to get the title from the router-store, you can subscribe to the `selectTitle` selector from the `store`, and assign it to a local variable (also named `title` in our case) upon data received.

```
export class ChildComponent implements OnInit {

  title?: string;
  
  constructor(private store: Store) { }

  ngOnInit(): void {
  	this.store.select(selectTitle).subscribe(title => this.title = title);
  }

}
```  

Then you show it in your html template as such.

```
<b>Title: </b><div *ngIf="title">{{title}}</div>
```

A working demo with the other selectors are available on my [NgRxRouterDemo](https://github.com/thecodinganalyst/NgRxRouterDemo).