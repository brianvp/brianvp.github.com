---
title: 'Angular2 Pipes'
date: 2016-09-23T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/23/Angular2-Pipes/
categories:
  - Research
tags:
  - Angular
  - Angular2
  - Pipes
---

Going over the [documentation](https://angular.io/docs/ts/latest/guide/pipes.html) for Angular2 pipes today.  

Pipes have changed since angular 1.  Most noticeable is the lack of filter/sorting pipes. The reason given for their removal was speed. While they worked well enough for demo applications, real-world usage often caused speed issues.  At my company we noticed this with lists only a few hundred items long.  

Angular comes with a few standard pipes

- Date
- UpperCase
- LowerCase
- Currency
- Percent

{% raw %}
```
{{ myValue | date}}

```
{% endraw %}


You pass parameters to the pipe with a `:`

{% raw %}
```
{{myValue | date:"shortDate"}}

```
{% endraw %}

The currency pipe requires multiple parameters, seperated by `:`
- symbolDisplay - USD, EUR, etc
- symbolDisplay - True to show $, False to show USD (false is the default)

{% raw %}
```
{{myValue | currency:"USD":true}}
```
{% endraw %}

It's simple to chain pipes:

{% raw %}
```
{{ myValue | uppercase | lowercase}}
```
{% endraw %}

If the built-in pipes do not meet your needs, you will need to process the data in the component, or create a custom pipe.  The angular team recommends handling expensive operations i.e. filtering in the component, instead of a pipe.   

To create the custom pipe you'll need to:

1. Add a new class that will implement the pipe as `<pipename>.pipe.ts`
2. Import the new pipe class in `app.module.ts`, specifying both as an import and a declaration.  
3. You do not need to declare anything in your component class that makes use of the pipe.

Here is a basic pipe implementation:

{% raw %}
```javascript
import { Pipe, PipeTransform } from '@angular/core';

// Take Name e.g. Bill Gates and return Gates, Bill
@Pipe({name: 'LastNameFirst',
       pure: true 
})
export class LastNameFirstPipe implements PipeTransform {
  transform(value: string): string {
    return value.substring(value.indexOf(' ') + 1) + ", " + value.substring(0, value.indexOf(' '));
  }
}
```
{% endraw %}

Notice the `pure` attribute.  Pipes in angular can be marked as `pure` or `unpure`

Pure pipes:

- only runs when a **pure change** is made to the input value.  
- a pure change is when a primitive type is updated, or a reference type is updated
- a pure change does not include changes within an object e.g. `{{customer.name | uppercase}}` a change to name would not cause the filter to be re-evaluated, since this is inside the customer object. This is for performance reasons.

Impure Pipes

- angular runs impure pipes on each component change detection cycle, meaning they run a lot.  
- the above customer.name example would be detected if we created a custom uppercase filter with `pure` set to `false`.  
- pipes set as impure should be as performant as possible.  


plunker demo:

### [plnkr.co/edit/G6MNM1Ys99al9svYFM3W](https://plnkr.co/edit/G6MNM1Ys99al9svYFM3W)