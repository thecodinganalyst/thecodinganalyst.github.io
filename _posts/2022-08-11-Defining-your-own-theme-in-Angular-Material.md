---
title: "Defining your own theme in Angular Material"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - Theming
  - Material-UI
  - Angular Material
---

Having your own theme in your web application is essential to make it look professional. Imagine using the default color themes provided from the framework you are using, which everyone is using, in your application. Especially when the theme colors are so prominent that anyone who has used the framework before can spot it with one look. The 4 themes provided by [Angular Material](https://material.angular.io/) aren't bad, but I will not think too highly of an application if it is using 1 of the 4 default themes. It kind of shows that not too much efforts went into the design, so it doesn't feel polished.

Creating your own theme colors in Angular Material is relatively easy, you just need to define 3 main colors for your application - #primary#, #accent#, and #warn#. #Primary# will be the color that will appear most frequently throughout your application. #Accent# is the color used to selectively highlight certain parts of your UI. #Warn# is the color to use for warnings and errors in your application, which is usually a shade of red. 

Then you need to decide whether the theme is a #dark# theme or a #light# theme. Just like it sounds, deciding on a dark theme will make the background of your application dark, and the white theme will give the same theme a default white background.

![dark theme](/assets/images/2022/08/dark-theme.png)

![light theme](/assets/images/2022/08/light-theme.png)

After you run `ng add @angular/material` in your angular app, your `styles.css` will be updated from a relatively blank file with a `$<app>-theme` defined with 1 of the 4 selected theme colors. For example, my app is named `bluesky`, so the following is created by default in my `styles.css`.

```
@use '@angular/material' as mat;
@include mat.core();

$bluesky-primary: mat.define-palette(mat.$indigo-palette);
$bluesky-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);

// The warn palette is optional (defaults to red).
$bluesky-warn: mat.define-palette(mat.$red-palette);

// Create the theme object. A theme consists of configurations for individual
// theming systems such as "color" or "typography".
$bluesky-theme: mat.define-light-theme((
  color: (
    primary: $bluesky-primary,
    accent: $bluesky-accent,
    warn: $bluesky-warn,
  )
));
```

The `$bluesky-theme` is defined as a light theme in the line `$bluesky-theme: mat.define-light-theme...`. You can change it to a dark theme by changing the `define-light-theme` to `define-dark-theme`. 

The `$bluesky-primary`, `$bluesky-accent`, and `$bluesky-warn` are defined using the default palettes - `$indigo-palette`, `$pink-palette`, `$red-palette` provided by Angular Material. Each palette is a SCSS map of a hue of colors from the lightest to the darkest. You can find the palette in the `node_modules\@angular\material\core\theming\_palette.scss` file, and below is an example of the `$pink-palette`. You can also see the colors for itself on the material design website - [https://material.io/archive/guidelines/style/color.html#color-color-palette](https://material.io/archive/guidelines/style/color.html#color-color-palette).

```
$pink-palette: (
  50: #fce4ec,
  100: #f8bbd0,
  200: #f48fb1,
  300: #f06292,
  400: #ec407a,
  500: #e91e63,
  600: #d81b60,
  700: #c2185b,
  800: #ad1457,
  900: #880e4f,
  A100: #ff80ab,
  A200: #ff4081,
  A400: #f50057,
  A700: #c51162,
  contrast: (
    50: $dark-primary-text,
    100: $dark-primary-text,
    200: $dark-primary-text,
    300: $dark-primary-text,
    400: $dark-primary-text,
    500: $light-primary-text,
    600: $light-primary-text,
    700: $light-primary-text,
    800: $light-primary-text,
    900: $light-primary-text,
    A100: $dark-primary-text,
    A200: $light-primary-text,
    A400: $light-primary-text,
    A700: $light-primary-text,
  )
);
```

Each of the numbers 50, 100, 200 - 900 are different shades of the color, and the A100, A200, A400, and A700 are just optional shades for easy reference of how light or dark you need for the color. The contrast colors will be the text color for the different shades. As you can see, shades 50 - 400 are light colors, so the contrast colors are `$dark-primary-text`. And the reverse is true for the darker shades 500 - 900. 

As a programmer, and not a design specialist, I'm not good at matching colors, but I still want something unique for my application, so instead of relying on the colors provided, I will create my own palette from a palette creator site. Go to [https://coolors.co/](https://coolors.co/e8aeb7-b8e1ff-a9fff7-94fbab-82aba1) and start the generator, removing 2 of the 5 colours provided as we only need 3 colors. 

![coolors palette generator](/assets/images/2022/08/coolors.png)

Hit the `space` button to let it randomly generate a palette for you. But when you found a color that you like, in my case I'm looking for a theme with blue color, mouse over the color and click on the `lock` icon so that the color will stay when you click the `space` button to try new themes.

![coolor lock](/assets/images/2022/08/coolor-lock.png)

Then I will generate the themes until I get a red color and lock it, because I'll need it for the #warn# in my theme. Then continue to hit `space` until the desired #accent# color is achieved. 

To generate the different shades of the colors, we can use the color palette tool from the material design website - [https://material.io/design/color/the-color-system.html#tools-for-picking-colors](https://material.io/design/color/the-color-system.html#tools-for-picking-colors). 

![tools for picking color](/assets/images/08/tools-for-picking-color.png)

Enter the color codes on the right, and the shades 50 - 900 will be generated for you. You can leave out the A100 - A700 as they are optional. Then create the palette SCSS map variable in your `styles.css` as follows.

```
$bright-navy-blue-palette: (
  50: #e3f2fd,
  100: #badffb,
  200: #8fcbfa,
  300: #61b7f7,
  400: #3ca7f6,
  500: #0c98f5,
  600: #078ae7,
  700: #0078d4,
  800: #0067c2,
  900: #0049a3,
  A100: #c2e4ff,
  A200: #addcff,
  A400: #1f9eff,
  A700: #00518f,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: rgba(black, 0.87),
    400: rgba(black, 0.87),
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
    A100: rgba(black, 0.87),
    A200: rgba(black, 0.87),
    A400: white,
    A700: white,
  )
);
```

And instead of using `$dark-primary-text` for the contrast, I'm just using `rgba(black, 0.87)` instead as defined in the `_palettes.scss`. The `$light-primary-text` is just `white`. 

Something neat from the `coolors.co` palette generator website is that it also gives you the name of the color, so you can use it as your variable name instead of going with light-blue, dark-blue, etc for all shades of blue. 

```
$bluesky-primary: mat.define-palette($bright-navy-blue-palette, 700, 300, 900);
$bluesky-accent: mat.define-palette($gainsboro-palette, 300, 100, 500);

// The warn palette is optional (defaults to red).
$bluesky-warn: mat.define-palette($crimson-palette, 700);
```

As above, we define the colors using the `mat.define-palette` function. The first and only required parameter requires a SCSS map palette of colors, which I can use my newly created `$bright-navy-blue-palette`. The following are optional keys of the palette which should point to the default hue, lighter hue, and darker hue respectively. By default, they will be 500, 100 and 700. So if your default hue is not 500 as generated from the material design tools for picking colors, be sure to specify the key for your default hue. 

My final result is as follows for your reference.

```
@use '@angular/material' as mat;

@include mat.core();

$bright-navy-blue-palette: (
  50: #e3f2fd,
  100: #badffb,
  200: #8fcbfa,
  300: #61b7f7,
  400: #3ca7f6,
  500: #0c98f5,
  600: #078ae7,
  700: #0078d4,
  800: #0067c2,
  900: #0049a3,
  A100: #c2e4ff,
  A200: #addcff,
  A400: #1f9eff,
  A700: #00518f,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: rgba(black, 0.87),
    400: rgba(black, 0.87),
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
    A100: rgba(black, 0.87),
    A200: rgba(black, 0.87),
    A400: white,
    A700: white,
  )
);

$gainsboro-palette: (
  50: #f9f9fa,
  100: #f3f3f4,
  200: #ebebec,
  300: #dcdcdd,
  400: #b9b9ba,
  500: #99999a,
  600: #707071,
  700: #5d5d5e,
  800: #3e3e3f,
  900: #1d1d1e,
  A100: #f5f5f5,
  A200: #eaeaeb,
  A400: #d6d6d7,
  A700: #838386,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: rgba(black, 0.87),
    400: rgba(black, 0.87),
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
    A100: rgba(black, 0.87),
    A200: rgba(black, 0.87),
    A400: rgba(black, 0.87),
    A700: white,
  )
);

$crimson-palette: (
  50: #ffebef,
  100: #ffcdc5,
  200: #f0999f,
  300: #e77179,
  400: #f24e58,
  500: #f83a40,
  600: #e9313e,
  700: #d72638,
  800: #ca1e30,
  900: #bb0d24,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: rgba(black, 0.87),
    400: white,
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
  )
);

$bluesky-primary: mat.define-palette($bright-navy-blue-palette, 700, 300, 900);
$bluesky-accent: mat.define-palette($gainsboro-palette, 300, 100, 500);
$bluesky-warn: mat.define-palette($crimson-palette, 700);

$bluesky-theme: mat.define-light-theme((
  color: (
    primary: $bluesky-primary,
    accent: $bluesky-accent,
    warn: $bluesky-warn,
  )
));

@include mat.all-component-themes($bluesky-theme);

html, body { height: 100%; }
body { margin: 0; font-family: Roboto, "Helvetica Neue", sans-serif; }
```
