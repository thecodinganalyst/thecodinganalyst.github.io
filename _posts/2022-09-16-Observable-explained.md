---
title: "Observables Explained"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Observable
  - RxJs
  - Observer Pattern
---

We all know what is a function. It takes some or no inputs and produce just 1 output.

```
function doSomething(input1, input2){
  return input1 + input2;
}
```

What if I want the function to keep producing output? Something like this

```
function doSomething(){
  return 1;
  return 2;
  return 3;
}
```

Yes, it can be done with [Observables](https://rxjs.dev/guide/observable). You can now return multiple values with a function, except that you cannot have input parameters anymore.

```
observable$ = new Observable<string>((subscriber) => {
    try {
        subscriber.next('a');
        subscriber.next('b');
        subscriber.next('c');
        subscriber.complete();
    } catch (err) {
        subscriber.error(err);
    }
});
```

The [Observable](https://angular.io/guide/observables-in-angular) is a very common data structure used in Angular for getting asynchronous data. It implements the [Observer Pattern](https://refactoring.guru/design-patterns/observer), which is like a magazine subscription, where you have a publisher pushing magazines to its subscribers over a period of time until the subscriber unsubscribes. 

[RxJs](https://rxjs.dev/guide/overview) is the javascript library that allows you to create and use observables. You can install it with `npm install rxjs`. But if you are on Angular, it is already part of the package.

As in the above function sample, you can create an [observable](https://rxjs.dev/api/index/class/Observable) by simply instantiating it, and passing a function that accepts a [subscriber](https://rxjs.dev/api/index/class/Subscriber) as a parameter. The subscriber has 3 main functions - `next()`, `complete()` and `error()`. Use the `next()` function to send a value to the subscriber. You can keep doing that forever, until you call the `complete()` function, which basically tells the subscriber that you are done, and there will be no more values. The `error()` function lets the subscriber knows that there is an error, and it completes too. 

To get the values from the publisher, a subscriber needs to subscribe to it.

```
dataList: Array<string> = [];
subscription = this.observable$.subscribe({
    next: (v) => this.dataList.push(v),
    error: (e) => this.dataList.push(e),
    complete: () => this.dataList.push("completed")
});
```

Just like what the observable is emitting, the receiving end, also known as the [observer](https://rxjs.dev/guide/observer) has 3 properties - `next`, `error`, and `complete` to handle the data received from the observable. 

A sample of a working code is available on [Stackblitz](https://stackblitz.com/edit/angular-ivy-lsmx6f?file=src/app/app.component.ts).