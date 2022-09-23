---
title: "How to pass data from angular to css"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Angular
  - CSS
  - Frontend
---

![How to pass data from angular to css](/assets/images/2022/09/how-to-pass-data-from-ng-to-css.png)

I am creating an angular template whereby I can create forms from a data file automatically, such as a json or yaml. For displaying on the monitor screen, the fields should have different sizes depending on what is the subject of the field. 

![angular-form](/assets/images/2022/09/angular-form.png)

Like in the example above, the building and the unit field will take up less screen space compared to other fields like street field. As the form is generated on demand, the fields are not known in advanced and my css isn't so clever to know which fields need less space than others. So I'd want to have the field length in my data file. Not in pixels, but just in percentage of the screen width. To do that, I need to pass data from my Angular to my css, which is a bit tricky, but I've found a way.

[CSS Variables](https://www.w3schools.com/css/css3_variables.asp) are just like normal variables that you can define in your css file or the style property of your html tags. The syntax to define the variable is `--[NAME]:[VALUE]`. So, to define a css variable named `size` with a value of 50% in my style property, I can do something like this:

```
<div style="--size:50%">hello</div>
```

Since this is in the html, I can use angular data binding to set the value - `{% raw %}--size:{{rect}}{% endraw %}`. 

Below is an example where I have an array of percentage width defined in the property named `rects`.

```
export class AppComponent {
  name = 'Angular ' + VERSION.major;
  rects = ['100%', '50%', '30%'];
}
```

Then in my html template, I define the css variable named `size`, with the values from the iteration of my `rects` property.

```
<div *ngFor="let rect of rects" style="--size:{% raw %}{{ rect }}{% endraw %}">
  {% raw %}{{ rect }}{% endraw %}
</div>
```

And lastly, I can get the value of my css variable with the syntax `--var([NAME])`. 

In my example, I set the value of the width css attribute of my `div` with the variable, to produce divs of different lengths based on the value in my angular data.

```
div {
  background-color: cornflowerblue;
  display: block;
  width: var(--size);
  margin: 10px;
}
```

A working example is available on my [Stackblitz](https://stackblitz.com/edit/thecodinganalyst-pass-data-from-angular-to-css).
