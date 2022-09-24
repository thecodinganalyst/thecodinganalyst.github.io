---
title: "Testing MatOptionHarness in MatSelectHarness"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Angular
  - MatSelect
  - MatOption
  - Material Angular
---

Reminder to self: When trying to get the options in the [MatSelectHarness](https://material.angular.io/components/select/api#MatSelectHarness) in Angular Material, you need to simulate a click on the MatSelectHarness - `await select.open();`, else the MatOptions will not be loaded. 

```
it('should display the options for the select', async () => {
    let field = await loader.getHarness(MatFormFieldHarness.with({floatingLabelText: 'Country *'}))
    let select = await field.getControl() as MatSelectHarness;
    expect(select).toBeTruthy();
    await select.open();
    let options = await select.getOptions();
    expect(options.length).toBe(countries.length);
});
```