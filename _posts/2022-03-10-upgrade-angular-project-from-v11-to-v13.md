---
title: "Upgrade Angular project from v11 to v13"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - Cypress
  - Protractor
  - Tslint
  - Eslint
---

I'm slow.... It's been a while since I last touched my [Angular repository](https://github.com/thecodinganalyst/elixir), that was created in early 2020, running on Angular 11. When I just went back to it yesterday, I realised that so much has changed. 

Tslint is now [deprecated](https://www.npmjs.com/package/tslint), and so was [Protractor](https://github.com/angular/protractor/issues/5502). But thanks to the documentations - https://www.bitovi.com/blog/angular-upgrades-painless-migration-from-tslint-to-eslint, and https://github.com/angular/protractor/issues/5502, migration of both of them from Tslint to Eslint, and protractor to [cypress](https://www.cypress.io/) is a breeze. 

Migrate tsLint to esLint:
```
ng add @angular-eslint/schematics
```

Migrate protractor to cypress:
```
ng add @cypress/schematic
```

Both will conveniently update the angular.js so that the commands `ng lint` and `ng e2e` are updated to use the new packages automatically. Though the old `e2e` folder and it's test scripts will no longer be relevant, and you'll need to rewrite the e2e test scripts in the cypress folder. 