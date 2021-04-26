---
title: "How to publish Angular component to NPM"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - NPM
---

There are some requirements that a module needs to fullfil before it can be published to npm. For example, you'll need a main field in your package.json (https://docs.npmjs.com/about-packages-and-modules). Plus, Angular is running typescript, which requires some extra steps to compile it to javascript. So it is easier to use (ng-packagr)[https://www.npmjs.com/package/ng-packagr] to help with the process. 

This is based on my published ItemsTextBox angular component in npm (https://www.npmjs.com/package/items-text-box), repository available on github - https://github.com/thecodinganalyst/NgItemsTextBox.


Documenting my steps over here:

1.  Install `ng-packagr` in the devDependencies

```
npm install ng-packagr --save-dev
```

2. Create the `ng-package.json` file in the top folder with the following content.

```json
{
  "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
  "lib": {
    "entryFile": "public_api.ts"
  }
}
```

<!--more-->

3. Create the `public_api.ts` file in the top folder with the following content. 

> Note that the path should link to the module.ts file, but you omit the `.ts` extension. In the below example, the module which I want to publish is not the main app.module.ts, but i created a module in the `modules` folder. 

```
export * from './src/app/modules/item-text-box/item-text-box.module';
```

4. Create the `packagr` command in the `package.json` (so that you can easily run the ng-packagr to build your npm compatible package), and set the `private` property to `false`

```
"scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "packagr": "ng-packagr -p ng-package.json"
  },
```

5. Run the `packagr` command, to package the module you want to publish into the `dist` folder. 

```
npm run packagr
```

6. Go into the `dist` folder, and `npm pack` to create the tar ball for distribution.

```
cd dist
npm pack
```

7. Create an account on https://www.npmjs.com/ if you haven't. Login to your npm account from the command line.

```
npm login
```

8. Publish your module, then head over to your npm page to see the updates.

```
npm publish
```