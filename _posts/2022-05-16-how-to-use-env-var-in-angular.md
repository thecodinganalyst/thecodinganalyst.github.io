---
title: "How to do use environment variable in Angular"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - Environment Variable
---

Angular automatically creates 2 files - `environment.ts` and `environment.prod.ts`, in the `environment` folder when the project is first created. Both of them has the same `environment` constant declared. To add new environment variables, simply extend the constant in both files, like below.

```
export const environment = {
  production: false,
  text: 'This is development environment'
};
```

```
export const environment = {
  production: true,
  text: 'This is production environment'
};
```

In whichever component you need to use it, simply import the `environment` constant with `import {environment} from "../environments/environment";`, and use it as needed.

```
import { Component } from '@angular/core';
import {environment} from "../environments/environment";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'Environment Variable Demo';

  displayText = environment.text;
}
```

In normal situations, when running `ng serve`, it will use the value in the `environment.ts`. But if you deploy it or run it with `ng serve --configuration=production`, it will take the value from the `environment.prod.ts`.

This is due to the following **fileReplacement** lines in the `angular.json`. When the configuration is `production`, it will replace the `environment.ts` with `environment.prod.ts`.

```
"configurations": {
  "production": {
    ...
    "fileReplacements": [
      {
        "replace": "src/environments/environment.ts",
        "with": "src/environments/environment.prod.ts"
      }
    ],
    "outputHashing": "all"
  },
```

One application is if you are using the [InMemoryWebApi](https://github.com/angular/in-memory-web-api) to simulate a http server in your development, and you want it to not be imported in production. 

```
imports: [
  HttpClientModule,
  environment.production ? [] : HttpClientInMemoryWebApiModule.forRoot(InMemHeroService)
  ...
]
```

Sample for the above application on [https://github.com/thecodinganalyst/env-demo](https://github.com/thecodinganalyst/env-demo).

