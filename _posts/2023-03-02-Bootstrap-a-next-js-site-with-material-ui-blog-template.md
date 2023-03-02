---
title: "Bootstrap a NextJS Site with Material UI Blog Template"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - NextJS
  - Material-UI
---

I was just trying out to bootstrap a website with [NextJS](https://nextjs.org/) and I wanted to apply the [Blog template](https://mui.com/material-ui/getting-started/templates/blog/) from [Material UI Getting Started Templates](https://mui.com/material-ui/getting-started/templates/) but it wasn't as fool-proof as I expected it to be. So I'm documenting the solution.

To start, we can either install NextJS and Material UI separately, or just download the Material UI NextJS example from [https://github.com/mui/material-ui/tree/master/examples/material-next-ts](https://github.com/mui/material-ui/tree/master/examples/material-next-ts). 

Run `npm install` to install the dependencies and `npm run dev` to start the application. Navigate to `http://localhost:3000` and it should be working fine.

We can `CTRL+C` to stop the application, while we add in the template files. Copy just the `.tsx` and `.md` files from [https://github.com/mui/material-ui/tree/v5.11.11/docs/data/material/getting-started/templates/blog](https://github.com/mui/material-ui/tree/v5.11.11/docs/data/material/getting-started/templates/blog) to the `src` folder. The template provides both javascript and typescript files, which are duplicate. Since we are using typescript in our project, so we can leave out the `.js` files.

As mention in the readme, we need to ensure the following dependencies are installed - @mui/material, @mui/icons-material, @emotion/styled, @emotion/react, markdown-to-jsx. Only `markdown-to-jsx` is not already in the example we are using, so we have to run `npm install markdown-to-jsx` to install it. 

Then we replace the `pages/index.tsx` with the content below to use the Blog template as the main page.

```
import * as React from 'react';
import Blog from "../src/Blog";

export default function Home() {
  return (
    <Blog />
  );
}
```

And we run `npm run dev` again to start the site.

![Material Template Error](/assets/images/2023/03/material-template-error.png)

It didn't quite work. This is because our react app is bundled with [webpack](https://webpack.js.org/guides/getting-started), and when webpack encounters an `import` statement for a non javascript file, it will try to parse and process the file according to its configured rules and loaders. And it will throw an error if it can't parse the file. Webpack does not have a built in way to parse the markdown syntax, and the error we're seeing is because webpack is trying to parse the markdown file as if it is a javascript or typescript code and it encounters unexpected characters. So, we need to install the [loader](https://webpack.js.org/concepts/loaders) for markdown file and configure webpack to use the loader to process markdown files.

So now we replace our `next.config.js` file with the following content

```
/** @type {import('next').NextConfig} */
module.exports = {
  reactStrictMode: true,
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.md$/,
      use: [
        {
          loader: "raw-loader",
        },
        {
          loader: "remark-loader",
          options: {
            // Add any remark plugins or options you want to use here
          },
        },
      ],
    });
    return config;
  },
};
```

Then install `raw-loader` and `remark-loader` with `npm install raw-loader remark-loader --save-dev`. Run `npm run dev` again.

![Material template working](/assets/2023/03/material-template-working.png)

A working sample of the above is available on [https://github.com/thecodinganalyst/material-next-blog-ts](https://github.com/thecodinganalyst/material-next-blog-ts).
