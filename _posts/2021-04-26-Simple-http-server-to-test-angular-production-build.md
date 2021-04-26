---
title: "Simple http server to test Angular production build"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - Http Server
---

After creating the production build for an Angular project with `ng build --prod`, the output is a folder of html and javascript, which will only work if you serve it from an actual http server. This can be troublesome sometimes if you are not running a http server. 

An easy way is to install this nodejs (http-server)[https://www.npmjs.com/package/http-server]. Then we can just run a simple command to serve the `dist` folder.

2 ways to do so, either install it as a homebrew package or npm globally.

## Homebrew

### Installation

```
brew install http-server
```

### Serving

```
http-server [path] [options]
```

## NPM

### Installation

```
npm install -g http-server
```

### Serving

```
npx http-server [path] [options]
```

Lookup the npm site mentioned above for the options if you need, otherwise, it defaults to port 8080 on your localhost. 
